# Inventory — Dungeon Quest

All item data — weapons, shields, armor, accessories, rings, consumables, spell scrolls, dice, body slots, equip rules. Source of truth for design; live numbers in `EQUIPMENT_DEFS` / `SHOP_CONSUMABLES` ([index.html:249](index.html:249), [279](index.html:279)).

> **Companion docs:** [CLAUDE.md](CLAUDE.md) (mechanics, combat resolver) · [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md) (spell math) · [BESTIARY.md](BESTIARY.md) (enemies) · [STORY.md](STORY.md) (narrative).

> **🚧 In-flight refactor.** Sections 1, 2, 3–6, 8, and 13 describe the **new** four-tier rarity system + +N enhancement + spell scrolls. Code in `index.html` is still on the legacy binary `rare:true` flag — the next pass implements the new data model. Every implementation hook is flagged inline as **`▸ Next pass`**. Search for that arrow to find every touch point.

---

## 1. Rarity tiers (NEW)

Four-tier system replacing the legacy binary `rare:true` flag. Every `EQUIPMENT_DEFS` / `SHOP_CONSUMABLES` entry carries a `rarity` field.

| Tier | Shop drop weight | UI color (suggest) |
|---|--:|---|
| `common` | 55% | warm wood |
| `uncommon` | 30% | green |
| `rare` | 12% | blue |
| `legendary` | 3% | gold |

Weights tunable; ratios feel right for an "ooh, legendary!" moment without flooding the inventory.

> **▸ Next pass:** Add `RARITY_WEIGHTS = {common:55, uncommon:30, rare:12, legendary:3}` and `RARITY_COLORS` near §279. Add `rarity` field to every `EQUIPMENT_DEFS` and `SHOP_CONSUMABLES` entry — default `common`. Update `TileBox`/`EquipDisplay` to color-frame by tier.

---

## 2. +N enhancement (NEW)

Base **common** weapons, shields, armor, and stat-boost rings can spawn with a `plus:N` field (`N ∈ {1, 2, 3}`), stamped at spawn time. Each +N bumps the item up one rarity tier (common → uncommon → rare → legendary).

The bonus per N = **the slot's primary modifier**, one mod's worth per N.

| Slot category | Primary stat | +N bonus |
|---|---|---|
| weapon | ATK + HIT (MIGHT mod) | +N ATK **and** +5N% HIT |
| shield, helmet, torso, leg | ARM | +N ARM |
| arms | ATK | +N ATK |
| mask, cloak, feet | DOD | +5N% DOD |
| glasses | HIT | +5N% HIT |
| hands | CRIT | +5N% CRIT |
| amulet | HIT + CRIT (hybrid) | +5N% HIT **and** +5N% CRIT |
| belt | MAX HP | +2N MAX HP |
| ring (stat-boost) | the ring's own stat | +N to that stat (percentage rings get +5N%) |

**Display.** Items render with a `+N` prefix (e.g., `+1 SHORTSWORD`, `+2 IRON HELMET`, `+3 BUCKLER`). The prefix follows standard identify gating (§12) — illiterate sees `???`; half-identified sees the generic category; fully identified shows the `+N` prefix and full stats.

**Cost scaling per +N** (start geometric, playtest):

| +N | Cost mult |
|---|--:|
| +0 (base) | × 1 |
| +1 | × 1.75 |
| +2 | × 3 |
| +3 | × 5 |

Floor 1. So SHORTSWORD 14g → +1 = 25g → +2 = 42g → +3 = 70g. Sell value = `floor(modifiedCost / 2)`, floor 1.

**Uniques** (named items: VAMPIRIC BLADE, RING OF GREED, etc.) do **not** receive +N variants. They occupy whatever tier their power suggests; no `plus` field, fixed name. The +N system only stacks on the 5 base weapons, 3 base shields, 12 base armor pieces, and 6 base stat-boost rings.

**Legendary uniques don't have to be "+3-equivalent."** A "+2-equivalent base + one strong proc," or "+1-equivalent base + one powerful proc" both fit legendary tier. The system supports flexible legendary design beyond linear +N.

> **▸ Next pass:**
> - `applyEquipment` ([370](index.html:370)) — add a per-item +N accumulator using the slot→stat map above.
> - `describeItem`/`labelItem` ([504](index.html:504)) — prepend `+N` when identified.
> - `getShopCost` ([498](index.html:498)) — multiply by the +N cost curve before applying CHARM discount.
> - `pickShopEquipment` ([518](index.html:518)) — roll tier first via `RARITY_WEIGHTS`, then within tier; stamp `plus:N` for +N variants.
> - `addToInventory`/`removeFromInventory` ([527](index.html:527)) — stack key includes `plus` so `+1 SHORTSWORD` and `+2 SHORTSWORD` are distinct stacks.
> - `EquipDisplay`/`GearDetail`/`ItemButtons` ([1080](index.html:1080)) — render `+N` prefix and tier color.

---

## 3. Weapons (`EQUIPMENT_DEFS` [279](index.html:279))

5 base commons (accept +N) + 7 named uniques retiered across U/R/L.

The `hand` field gates equip:
- **`light`** — 1H, dual-wieldable (with Dual Wield perk), can sit in offhand
- **`one`** — 1H, shield-compatible offhand only
- **`two`** — 2H, locks the offhand slot

`dieIdx` indexes `DICE_PROGRESSION` (§9).

### Common base weapons — accept +N → U/R/L variants

| ID | Name | Emoji | Hand | Die | Stats | Cost |
|---|---|---|---|---|---|--:|
| `dagger` | DAGGER | 🗡️ | light | D4 | +10% CRIT | 8 |
| `shortsword` | SHORTSWORD | ⚔️ | light | D6 | +5% CRIT | 14 |
| `longsword` | LONGSWORD | 🗡️ | one | D8 | — | 22 |
| `spear` | SPEAR | 🔱 | two | D10 | — | 28 |
| `greatsword` | GREATSWORD | ⚔️ | two | 2D6 | — | 32 |

Variants display as e.g. `+2 SPEAR` (D10, +2 ATK, +10% HIT, cost 84g).

### Uncommon uniques

| ID | Name | Emoji | Hand | Die | Stats | Cost |
|---|---|---|---|---|---|--:|
| `sharp_dagger` | SHARP DAGGER | 🗡️ | light | D4 | +1 ATK, +5% HIT | 18 |
| `keen_short` | KEEN SHORTSWORD | ⚔️ | light | D6 | +5% CRIT | 24 |

### Rare uniques

| ID | Name | Emoji | Hand | Die | Stats | Procs | Cost |
|---|---|---|---|---|---|---|--:|
| `venom_dagger` | VENOM DAGGER | 🐍 | light | D4 | — | 25% POISON | 22 |
| `flame_brand` | FLAME BRAND | 🔥 | light | D6 | — | 25% BURN | 26 |
| `piercing_lng` | PIERCING LONGSWORD | ⚔️ | one | D8 | — | 25% IGNORE ARMOR | 34 |

### Legendary uniques

| ID | Name | Emoji | Hand | Die | Stats | Procs / Special | Cost |
|---|---|---|---|---|---|---|--:|
| `bleeding_spear` | BLEEDING SPEAR | 🔱 | two | D10 | — | 25% BLEED | 38 |
| `vampiric_blade` | VAMPIRIC BLADE | 🩸 | light | D6 | +1 ATK | HEAL 2 ON CRIT | 48 |

### Weapon mechanics

- **Damage** — `atkDmg + die_roll − armorUsed`. Crits roll the die **twice**.
- **Armor pierce** — `pierceChance` procs OR `armorPierce` perk on a crit → `armorUsed = 0`.
- **Offhand swings deal 70%** damage. **Master Duelist** perk removes the penalty.
- **Vampiric** — heal 2 HP on crit, fires from either hand.
- **Arcane Sense** (WITS 20) — +5% to every proc chance (poison/burn/bleed/pierce).
- **Brute** (MIGHT 20) — +1 ATK on every swing.

> **▸ Next pass:** Add `rarity` field to all 12 entries; retier named uniques as listed above. Base commons stay common.

---

## 4. Shields

3 base commons, all accept +N → U/R/L variants. No shield uniques yet (design space open — see §16).

`slot:"offhand"` — mutually exclusive with offhand weapon. 2H weapons evict the shield to inventory.

| ID | Name | Emoji | Stats | Cost |
|---|---|---|---|--:|
| `buckler` | BUCKLER | 🛡️ | +1 ARM, +5% DOD | 12 |
| `iron_shield` | IRON SHIELD | 🛡️ | +2 ARM | 18 |
| `tower_shield` | TOWER SHIELD | 🛡️ | +3 ARM, −5% DOD | 26 |

+N adds +N ARM (existing DOD modifiers unchanged).

---

## 5. Armor & accessories

12 base armor pieces across 9 slots, all common, all accept +N → U/R/L variants.

| ID | Name | Emoji | Slot | Stats | +N adds | Cost |
|---|---|---|---|---|---|--:|
| `helmet` | IRON HELMET | 🪖 | helmet | +2 ARM | +N ARM | 16 |
| `mask` | SHADOW MASK | 🎭 | mask | +10% DOD | +5N% DOD | 14 |
| `glasses` | SHARP GLASSES | 🥽 | glasses | +10% HIT | +5N% HIT | 14 |
| `torso` | CHAIN MAIL | 🧥 | torso | +3 ARM | +N ARM | 26 |
| `arms` | POWER BRACERS | 💪 | arms | +2 ATK | +N ATK | 22 |
| `hands` | CRIT GAUNTLETS | 🧤 | hands | +10% CRIT | +5N% CRIT | 18 |
| `belt` | HERO BELT | 🪢 | belt | +4 MAX HP | +2N MAX HP | 16 |
| `leg` | IRON GREAVES | 🦵 | leg | +2 ARM | +N ARM | 18 |
| `eq_cloak` | SHADOW CLOAK | 🧣 | cloak | +15% DOD | +5N% DOD | 20 |
| `dodge_cloak` | DODGE CLOAK | 🥋 | cloak | +20% DOD, −1 ARM | +5N% DOD | 16 |
| `feet` | SWIFT BOOTS | 👢 | feet | +10% DOD | +5N% DOD | 16 |
| `amulet` | MYSTIC AMULET | 📿 | amulet | +5% CRIT, +5% HIT | +5N% HIT + +5N% CRIT | 24 |

Armor uniques (storm helm, ghost cloak, mirror amulet, etc.) — none yet, see wishlist §16.

---

## 6. Rings (`slot:"ring"`)

Up to 5 ring slots. 6 base stat-boost rings (common, accept +N) + 1 unique with a trade-off.

| ID | Name | Stats | +N adds | Cost |
|---|---|---|---|--:|
| `ring_atk` | POWER RING | +1 ATK | +N ATK | 10 |
| `ring_arm` | SHIELD RING | +1 ARM | +N ARM | 9 |
| `ring_hit` | AIM RING | +5% HIT | +5N% HIT | 8 |
| `ring_crt` | CRIT RING | +5% CRIT | +5N% CRIT | 8 |
| `ring_dod` | EVASION RING | +5% DOD | +5N% DOD | 8 |
| `ring_hp` | LIFE RING | +2 MAX HP | +2N MAX HP | 8 |
| `ring_greed` | RING OF GREED | +50% gold, +2 dmg taken | — (unique, no +N) | 15 |

So a `+3 POWER RING` gives **+4 ATK** total (1 base + 3 plus) and sits at legendary tier.

Greed Ring multiplier stacks with **Lucky Coin** (CHARM 20) for the big-coin build.

**Future direction (wishlist §16):** ability-score rings — RING OF ZIP / RING OF MIGHT / RING OF SAVVY etc. bump the underlying ability score by N instead of a derived stat. Bigger lever; design TBD.

---

## 7. Consumables (`SHOP_CONSUMABLES` [249](index.html:249))

18 items. Heal items declare `heal:N` (or `"full"`); cures use `cureX` flags; buffs use `onUse(p,b)` to push to `tempBuffs`. SAV mod added to every heal in `handleUseItem`.

All currently common rarity; ELIXIR may warrant uncommon, multi-battle buffs (`whetstone3`, `scroll3`) may warrant uncommon. Tier balance to playtest.

### Heals & cures

| ID | Name | Effect | Cost |
|---|---|---|--:|
| `potion` | HEALTH POTION | +4 HP (+SAV mod) | 3 |
| `potion2` | MEGA POTION | +8 HP (+SAV mod) | 9 |
| `potion3` | SUPER POTION | +16 HP (+SAV mod) | 15 |
| `elixir` | ELIXIR | FULL HP + cure all status | 20 |
| `antidote` | ANTIDOTE | Cure poison | 3 |
| `ointment` | OINTMENT | Cure burn | 3 |
| `Band-aid` | BANDAID | Cure bleed | 3 |
| `Salt` | SMELLING SALT | Cure stun | 3 |

### Buffs (`tempBuffs` stack — decrement at `startBattle`)

| ID | Name | Effect | Battles | Cost |
|---|---|---|--:|--:|
| `oil` | BATTLE OIL | +10% HIT | 1 | 3 |
| `whetstone` | WHETSTONE | +2 ATK | 1 | 3 |
| `scroll` | CRIT SCROLL | +10% CRIT | 1 | 3 |
| `smokebomb` | SMOKE BOMB | +20% DOD | 1 | 3 |
| `talisman` | GUARD TALISMAN | +2 ARM | 1 | 3 |
| `oil2` | FINE BATTLE OIL | +15% HIT | 2 | 6 |
| `whetstone2` | KEEN WHETSTONE | +3 ATK | 2 | 7 |
| `scroll2` | GREAT CRIT SCROLL | +15% CRIT | 2 | 7 |
| `whetstone3` | MASTER WHETSTONE | +4 ATK, +20% HIT | 3 | 12 |
| `scroll3` | ANCIENT SCROLL | +20% CRIT, +30% DOD (DOD only 2 battles) | 3 | 12 |

> Heal items trigger SAVVY perks: **Reservoir** grants `savMod` Temp HP, **Swift Cure** rolls `savMod × 10%` to skip the enemy turn. See [CLAUDE.md → Stat-20 perks](CLAUDE.md).

> **Naming heads-up:** the legacy `scroll` / `scroll2` / `scroll3` IDs above are CRIT-buff items, NOT the new spell scrolls in §8. Spell scrolls use the `scroll_<spell>_<lv>` ID pattern. Rename buff items to `crit_oil_*` in a future cleanup pass (wishlist §16).

---

## 8. Spell scrolls (NEW)

Single-use consumables that cast a spell from `PLAYER_SPELLS` ([129](index.html:129)). Fills the gap between "I'm not a mage" and "I'd like to throw a fireball once."

### Design rules

- **Spell pool (v1)** — the 6 existing `PLAYER_SPELLS` only. Scroll-exclusive spells parked for future (wishlist §16).
- **Stat gating** — caster must be **literate** (`intMod ≥ 0`). No per-spell stat requirement (a brute can pop a ZAP-ZAP scroll). Illiterate → fizzle log line ("...TOO MANY SQUIGGLES!"), scroll consumed, no effect.
- **Stamped LV** drives spell power, price, and rarity. Spell-LV mechanics match the regular system: each LV above 1 adds **+1 dmg / +1 heal / +1 charm turn**.
- **DC** — flat `10 + floor(waveNumber/2)`. **No caster mod.** Player-cast spells use `8 + caster mod + floor(wave/2)` → a high-WIS mage's own spell IS stronger than the scroll version. Intentional: scrolls are a fallback for non-mages, not a free upgrade for mages.
- **Damage** — drops the spell's `statBonus` (caster stat mod normally added to dmg). Damage = die roll only (+ LV bonus).
- **Heal scrolls** (BOO-BOO FIX, THERE-THERE) — **DO** add SAV mod to the heal, matching potion behavior. A high-WIS hero would still rather cast their own upgraded heal, but a non-caster can still get a meaningful heal off a scroll.
- **Out-of-combat use** — heal scrolls and THERE-THERE's cleanse work out of combat. Damage scrolls (ZAP-ZAP, SICK BURN, HOTFOOT) and the BAT EYES stun are combat-only.

### Tier mapping per spell

Each spell has a **base tier** reflecting its inherent power. Scroll variants at LV1 sit at the base tier; each +1 LV bumps the tier one rung. Tier caps at **legendary** — high-base spells have fewer available LVs.

| Spell | Type | Base | Reach |
|---|---|---|---|
| ZAP-ZAP | dmg (1D4 auto-hit, ignore armor) | **common** | LV1=C, LV2=U, LV3=R, LV4=L |
| SICK BURN | dmg (1D4, SAV shrug, ignore armor) | **common** | LV1=C, LV2=U, LV3=R, LV4=L |
| BOO-BOO FIX | heal (1D8 + SAV) | **uncommon** | LV1=U, LV2=R, LV3=L |
| HOTFOOT | dmg (1D10, burn, ZIP shrug half, ignore armor) | **rare** | LV1=R, LV2=L |
| THERE-THERE | heal+cure (2D4 + SAV) | **rare** | LV1=R, LV2=L |
| BAT EYES | stun (LV turns, SAV shrug) | **rare** | LV1=R, LV2=L |

### Full scroll table (17 entries)

Suggested base cost per tier: 8 / 18 / 35 / 65. Tunable.

| ID | Spell | LV | Tier | Effect | Cost |
|---|---|---|---|---|--:|
| `scroll_zap_1` | ZAP-ZAP | 1 | common | 1D4 auto-hit, ignore armor | 8 |
| `scroll_zap_2` | ZAP-ZAP | 2 | uncommon | 1D4+1 auto-hit, ignore armor | 18 |
| `scroll_zap_3` | ZAP-ZAP | 3 | rare | 1D4+2 auto-hit, ignore armor | 35 |
| `scroll_zap_4` | ZAP-ZAP | 4 | legendary | 1D4+3 auto-hit, ignore armor | 65 |
| `scroll_sick_burn_1` | SICK BURN | 1 | common | 1D4, SAV shrug, ignore armor | 8 |
| `scroll_sick_burn_2` | SICK BURN | 2 | uncommon | 1D4+1, SAV shrug, ignore armor | 18 |
| `scroll_sick_burn_3` | SICK BURN | 3 | rare | 1D4+2, SAV shrug, ignore armor | 35 |
| `scroll_sick_burn_4` | SICK BURN | 4 | legendary | 1D4+3, SAV shrug, ignore armor | 65 |
| `scroll_boo_boo_1` | BOO-BOO FIX | 1 | uncommon | HEAL 1D8 + SAV | 18 |
| `scroll_boo_boo_2` | BOO-BOO FIX | 2 | rare | HEAL 1D8+1 + SAV | 35 |
| `scroll_boo_boo_3` | BOO-BOO FIX | 3 | legendary | HEAL 1D8+2 + SAV | 65 |
| `scroll_hotfoot_1` | HOTFOOT | 1 | rare | 1D10 BURN, ZIP shrug half, ignore armor | 35 |
| `scroll_hotfoot_2` | HOTFOOT | 2 | legendary | 1D10+1 BURN, ZIP shrug half, ignore armor | 65 |
| `scroll_there_there_1` | THERE-THERE | 1 | rare | HEAL 2D4 + SAV, cure 1 status | 35 |
| `scroll_there_there_2` | THERE-THERE | 2 | legendary | HEAL 2D4+1 + SAV, cure 1 status | 65 |
| `scroll_bat_eyes_1` | BAT EYES | 1 | rare | STUN 1 turn, SAV shrug | 35 |
| `scroll_bat_eyes_2` | BAT EYES | 2 | legendary | STUN 2 turns, SAV shrug | 65 |

### Mechanics

- **Type field** — new `type:"scroll"` on the consumable, with `spellId` and `scrollLv` fields.
- **Dispatch** — `handleUseItem` ([1651](index.html:1651)) detects `type:"scroll"` → routes to a new `castScrollSpell(scroll, inBattle)` handler.
- **Casting** — clones the spell def with overrides: `statBonus` stripped (except heals), DC flat at `10 + floor(wave/2)`, level = `scrollLv`. Dispatches through the existing spell resolver so all visuals/log lines reuse normal spell paths.
- **Literacy fizzle** — `isIlliterate(p)` ([502](index.html:502)) → log "...TOO MANY SQUIGGLES!", `removeFromInventory`, no effect. Lines up tonally with wishlist #10 ("YOU DON'T KNOW HOW TO READ"). Could rotate among 3 variants for flavor.
- **Identification** — standard 3-tier gating (§12). Illiterate sees `???`; mid-tier sees "SCROLL" (generic category); fully identified shows the spell name + LV.
- **Stack-by-id** — each `scroll_<spell>_<lv>` is its own stack (no `plus` field — scrolls don't enhance further).

> **▸ Next pass:**
> - Add `SCROLL_DEFS` const after `SHOP_CONSUMABLES` (or merge with `type:"scroll"` markers). Include all 17 entries.
> - Wire into shop pool so scrolls roll alongside potions; tier-weighted via `RARITY_WEIGHTS`.
> - Add `castScrollSpell(p, scroll, ...)` helper that clones the spell def with LV/DC/statBonus overrides, then dispatches into the existing spell resolver.
> - `handleUseItem` ([1651](index.html:1651)) — add early branch: `if (item.type === "scroll") { castScrollSpell(item, inBattle); return; }`.
> - Literacy fizzle path — `addLog("...TOO MANY SQUIGGLES!", "warn")`, consume the scroll, return without effect.
> - Out-of-combat allowlist — heal/cleanse scrolls usable from idle USE ITEM; damage and stun scrolls hidden or grayed.
> - Identify generic-tier label maps `type:"scroll"` to "SCROLL".

---

## 9. Dice progression (`DICE_PROGRESSION` [272](index.html:272))

30 dice sorted by average roll: 1× through 5× of `{D4, D6, D8, D10, D12, D20}`. Each weapon overrides the player's `dieIdx`; equipment can also bump it.

---

## 10. Body slots (`BODY_SLOTS` [322](index.html:322))

13 slot entries with grid `col`/`row` for `BodyLayout` render. Five-ring strip not pinned to a single grid cell.

| Slot | Emoji | col, row |
|---|---|--:|
| weapon | ⚔️ | 1, 2 |
| offhand | 🛡️ | 5, 2 |
| mask | 🎭 | 2, 1 |
| helmet | 🪖 | 3, 1 |
| glasses | 🥽 | 4, 1 |
| amulet | 📿 | 3, 2 |
| arms | 💪 | 1, 3 |
| torso | 🧥 | 3, 3 |
| cloak | 🧣 | 5, 3 |
| hands | 🧤 | 2, 4 |
| belt | 🪢 | 3, 5 |
| leg | 🦵 | 3, 6 |
| feet | 👢 | 3, 7 |

`EMPTY_EQUIP` = `{...slots:null, rings:[]}`.

---

## 11. Equip rules (`canEquipItem` [482](index.html:482))

- Light weapons can sit in offhand IF the player has **Dual Wield** perk AND the main weapon is also light (or empty).
- One-hand weapons need an empty offhand or a shield.
- Two-hand weapons evict any offhand (shield OR off-weapon) to inventory.
- Slotted items always displace the existing occupant to inventory (stack-by-id).
- Mid-fight equip **costs the turn** — enemy gets a free strike, but it lands against your NEW gear stats.

Dual-wield UI: shows a `🛡 → OFFHAND` button on light-weapon shop / inventory entries when `canEquipToOffhand(item)` returns true.

---

## 12. Identify gating (`isIlliterate(p)` / `describeItem` [502](index.html:502)–[504](index.html:504))

| Player state | Label | Description |
|---|---|---|
| `isIlliterate` (WITS < 10, mod < 0) | `???` | `??? — TOO ARCANE TO READ`. Slot/hand tag hidden. **`+N` prefix hidden.** Scroll spell name hidden. |
| `0 ≤ identify < 50%` | Real label | Generic category ("VITAL GEAR" / "DEFENSIVE GEAR" / "COMBAT GEAR" / "ENCHANTED GEAR" / **"SCROLL"**) |
| `identify ≥ 50%` | Real label | Real description, **`+N` prefix shown**, **scroll spell name + LV shown** |

WITS mod adds +15% identify per mod. **Tactician** (WITS 20) forces identify to 100%. Dev mode sets `intelligence:30, identify:95`.

> **▸ Next pass:** `describeItem`/`labelItem` ([504](index.html:504)) — add `+N` prefix at identified tier; add `"SCROLL"` to the generic-category map; hide `+N`/spell-name behind illiterate/mid-tier as above.

---

## 13. Shop math

- **Shop cost** — `ceil(item.cost × (1 − shopDiscount/100))`, floor 1. For +N items, the base cost is multiplied first by the +N cost curve (§2) before CHARM discount.
- **Discount** — CHARM mod × −5% per mod (positive CHA = cheaper). Can go negative (low CHA = markup).
- **Sell value** — `floor(modifiedCost / 2)`, floor 1. Sell one off a stack per click. +N items use the +N-multiplied cost as the basis.
- **Shop composition** — post-boss only. 3 random consumables (`pickRandom(SHOP_CONSUMABLES, 3)`) + 2 random equipment (`pickShopEquipment(avail, 2)`). **Silver Tongue** (CHARM 20) adds +1 equipment slot.
- **Equipment tier roll** — `pickShopEquipment` rolls tier per slot via `RARITY_WEIGHTS` (55/30/12/3), then picks within that tier's pool. For base items in a higher tier, stamps `plus:N` accordingly (uncommon → +1, rare → +2, legendary → +3). Replaces the legacy 25%/75% rare/common roll.
- **Consumables stay in shop after purchase** — stock up on antidotes (and scrolls).

> **▸ Next pass:** Rework `pickShopEquipment` ([518](index.html:518)) tier-first; update `getShopCost` ([498](index.html:498)) and `handleSellItem` ([1753](index.html:1753)) to apply the +N cost multiplier before CHARM math.

---

## 14. Inventory mechanics

- **Stack-by-id** — identical-id items share an inventory entry with a `count` field. UI shows `(×N)` when count > 1. Helpers: `addToInventory(inv, item)` / `removeFromInventory(inv, item)`.
- **+N items stack by composite key.** `+1 SHORTSWORD` and `+2 SHORTSWORD` are distinct stacks. Implementation: stack key = `id + (plus ? "_+" + plus : "")`.
- **All displaced equipment** goes through the same stack path. Useful for selling off your old gear.
- **Consumables, scrolls, and equipment share the inventory state**. The **USE ITEM** selector currently shows all — see wishlist [#8](CLAUDE.md) for the planned split into consumables-only + gear-only flows. Scrolls naturally sit in the consumables view alongside potions and buffs.

---

## 15. Helpers (in `index.html`)

| Fn | Line | Notes | Next-pass change |
|---|---|---|---|
| `canEquipItem(item, equip, p, targetSlot)` | [482](index.html:482) | Validates light-only / Dual Wield / 2H blocks | unchanged |
| `canEquipToOffhand(item)` | [1745](index.html:1745) | Dual-wield visibility check | unchanged |
| `equipToOffhand(item, inBattle)` | [1750](index.html:1750) | Sugar → `handleEquipFromInventory` w/ `targetSlot:"offhand"` | unchanged |
| `handleEquipFromInventory(item, inBattle, targetSlot)` | [1706](index.html:1706) | Equip from inventory; mid-fight = free enemy strike | unchanged |
| `handleUseItem(item, inBattle)` | [1651](index.html:1651) | Heal / buff / cure dispatch. SAVVY perks fire here. | **Add `type:"scroll"` branch + literacy fizzle** |
| `handleSellItem(item)` | [1753](index.html:1753) | `floor(cost/2)` min 1 | **Use +N-multiplied cost as basis** |
| `buyItem(item, targetSlot)` | [2077](index.html:2077) | Cost check + displace-old + add-new | **Cost basis = +N-multiplied + CHARM discount** |
| `getShopCost(item, p)` | [498](index.html:498) | CHARM discount math | **Multiply by +N curve first** |
| `isIlliterate(p)` | [502](index.html:502) | `intMod < 0` | unchanged; reused for scroll fizzle |
| `describeItem(item, p)` / `labelItem` | [504](index.html:504) | 3-tier identify gating | **Prepend `+N` for identified; map `type:"scroll"` to "SCROLL" generic** |
| `getAvailableEquipment(equip)` | [471](index.html:471) | Pool of items not already owned | unchanged |
| `pickRandom(pool, n)` / `pickShopEquipment` | [518](index.html:518) | Shop roll helpers | **Roll tier first via `RARITY_WEIGHTS`; stamp `plus:N` for +N variants** |
| `addToInventory` / `removeFromInventory` | [527](index.html:527) | Stack-by-id math | **Composite key includes `plus`** |
| `applyEquipment(p, equip)` | [370](index.html:370) | Sums `stats:{}` from gear | **Add per-item +N bonus accumulator (slot→stat map per §2)** |
| `castScrollSpell(p, scroll, ...)` | NEW | Scroll spell dispatcher | **Implement: clone spell def, override LV/DC/statBonus, dispatch into resolver** |

---

## 16. Open wishlist (relevant items)

From [CLAUDE.md](CLAUDE.md):
- **[#8 Split equip-gear and use-item UX](CLAUDE.md)** — separate the inventory drawer into consumables-only + gear-only flows; dedicated EQUIP action in combat. Spell scrolls naturally live in the consumables view.
- **[#9 Weapon equip requirements](CLAUDE.md)** — gate weapons behind stat thresholds (Greatsword → MIGHT 18, etc.). **Interacts with +N:** should a `+3 GREATSWORD` raise the MIGHT req beyond 18? Probably yes (e.g., +N adds +1 to req per N), but design TBD. Park until #9 is being scoped.
- **[#10 Shop illiteracy copy](CLAUDE.md)** — "YOU DON'T KNOW HOW TO READ". Bonus relevance: illiteracy now also fizzles spell scrolls; same copy tone fits the fizzle log ("...TOO MANY SQUIGGLES!").

New items raised in this design pass:

- **Ability-score rings** — RING OF ZIP / MIGHT / GUTS / SAVVY / CHARM / WITS, each bumping the underlying ability score by N. Bigger lever than +stat rings (a single +1 mod boundary can cascade). Park as v2 of the ring system; revisit after +N ships.
- **Armor uniques** — no named armor exists yet. Design space wide open. Examples: STORM HELM (cures stun on hit), GHOST CLOAK (+50% DOD vs spells), MIRROR AMULET (50% chance to reflect status onto attacker).
- **Shield uniques** — no shield uniques. Examples: SPIKED BUCKLER (counter dmg on dodge), MIRROR SHIELD (reflect status procs), KITE SHIELD (+1 ARM and 2H-compatible, breaks the offhand-eviction rule).
- **Scroll-exclusive spells** — spells that only exist as scrolls, not in `PLAYER_SPELLS`. Examples: BANISH (remove enemy from initiative for N turns), MASS CURE (cures all status + heals), TRAP DISARM (auto-pass next trap shrug), IDENTIFY (full-identify all carried gear).
- **Buff-item rename cleanup** — rename `scroll` / `scroll2` / `scroll3` (CRIT-buff consumables) to `crit_oil` / `crit_oil2` / `crit_oil3` to free the "scroll" name for spell scrolls. Inventory state migration needed if there are any savefiles in flight (per wishlist #4).

See [CLAUDE.md → Wishlist](CLAUDE.md) for full design notes.
