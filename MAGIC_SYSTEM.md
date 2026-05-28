# Magic System — Half-Scale Spellcasting

The spellcasting design for **Dungeon Quest**, a bright, goofy, cartoon dungeon crawl. This is the game's **own** magic system — original names, original flavor.

> **Original game — NOT D&D.** Don't reference *Dungeons & Dragons*, its books, class names, or trademarked spells anywhere player-facing. A d20 roll and an ability modifier are generic math (fine to use); spell *names and flavor* are always our own silly originals. See [CLAUDE.md → Naming convention](CLAUDE.md). This doc is the source of truth for spell design; the implemented roster lives in `PLAYER_SPELLS` ([index.html:129](index.html:129)).

One governing rule keeps the numbers cute: **Dungeon Quest runs on a half-scale HP pool** (a goblin has 4 HP, not 40), so spell damage and healing are halved to match. Combat stays readable at a glance — small numbers, big personality.

The engine already builds HP this way — `maxHp = floor((sum(growthDice) + gutsMod × growthDice.length) / 2)` is a deliberately tiny pool. Magic uses the identical halving so the two scales stay in lockstep.

---

## 1. The half-scale rule — author big, ship small

> **Author each spell at a "big" working value — full dice, full modifiers — then ship `floor(total / 2)`. A spell that connects always does at least 1.**

This is the only conversion you ever apply. Do it once, at the very end, after every die and bonus is summed. It's purely a number-scaling step that keeps damage proportional to the low HP pool — nothing more.

- **Damage** halves. A 💥 KABLOOEY rolls big (say 28) → deals **14**.
- **Healing** halves. A ✚ BOO-BOO FIX rolls big (say 9) → restores **4**.
- **Shrug rolls are NOT halved.** A d20 vs a target number is scale-independent — die, target, and modifiers stay full. Halving the shrug would double-dip against the already-tiny HP pool.
- **Turn counts, slows, and "skip a turn" are NOT halved.** Only hit-point numbers shrink. A 3-turn burn is still 3 turns.

### Worked example

🔥 HOTFOOT against a goblin, caster has WITS 16 (+3):

1. Roll the big value: `1d10` → 7, plus the caster's WITS mod (+3) = **10**.
2. Halve it: `floor(10 / 2)` = **5 damage**.
3. The goblin tries to shrug with a ZIP roll vs the spell's target number. If it shrugs, HOTFOOT does half (its `onSave` is `half`) → `floor(5 / 2)` = **2**.

Shrugging halves the *already-halved* number — intentional, so a shrug feels like a real "phew!" at this scale, not a rounding error.

> Shipped numbers are hand-tuned for snappy, low-HP combat. The halve rule is the **default starting point**, not a straitjacket — nudge a spell up or down if it plays better.

---

## 2. Shrug-off math (the engine's defense roll)

Spells don't roll to-hit and they ignore dodge/armor — instead the **defender rolls to shrug it off**. (Internally the field is still `onSave`; player-facing, it's always a **SHRUG**, and the difficulty is a bare **target number**, never "DC".)

**When the player casts at an enemy:**
```
target     = 8 + abilityMod(caster's spell stat) + floor(playerLevel / 2)
enemy roll = d20 + abilityMod(enemy.attrs[shrugStat])
roll > target  → SHOOK IT OFF        roll ≤ target → full hit
```

**When an enemy casts at the player:**
```
target      = 8 + spell.mod + floor(waveNumber / 2)
player roll = d20 + abilityMod(player[spell.ability])
```

Spell hits **bypass dodge and armor** — the shrug *is* the defense. Spells never crit and never trigger a counter. Log it readably, e.g. `GOBLIN ZIP SHRUG 15−2 (13) vs 10 → SHOOK IT OFF!`.

### What "shook it off" means — the `onSave` field

| `onSave`  | Damage on a shrug | Status on a shrug |
|-----------|-------------------|-------------------|
| `avoid`   | 0                 | none              |
| `half`    | `floor(dmg/2)`    | none              |
| `status`  | 0                 | applied           |
| `both`    | `floor(dmg/2)`    | applied           |

`autoHit: true` spells (✨ ZAP-ZAP) skip the shrug entirely and always land.

---

## 3. Casting stats → knacks

Three stats can cast. Each is a flavor of magic — a **knack**:

| Stat | Knack | Identity |
|------|-------|----------|
| **WITS** | 🤓 **WHIZ-BANG** | Brainy zaps and gadget-booms — armor-ignoring force & elemental blasts |
| **SAVVY** | 🌿 **COZY** | Heals, sunny pokes, status cures, and friend-buffs |
| **CHARM** | 😏 **SASS** | Single-target control, psychic sass, and debuffs |

A spell gates on its stat via `req` (minimum score to learn) and grows as that score climbs.

---

## 4. Spell roster

All **In-game** values are already halved and ready to use. The **Big roll** column is the author's pre-halve working value — purely an internal scaling note (`in-game ≈ floor(big / 2)`). Spells marked **★ shipped** are implemented in `PLAYER_SPELLS` today; treat [index.html:129](index.html:129) / [CLAUDE.md](CLAUDE.md) as the source of truth for their exact numbers. The rest are designed-and-ready expansion picks.

### 🤓 WHIZ-BANG (WITS)

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| ✨ **ZAP-ZAP** ★ | 3×(1d4+1) | **1d4, auto-hit** | — | — | ignores armor |
| 🥶 **BRAIN FREEZE** | 1d8 | **1d4 + WIT** | — | — | −1 SPD (slow) |
| ⚡ **STATIC CLING** | 1d8 | **1d4 + WIT** | — | — | target can't counter |
| 🔥 **HOTFOOT** ★ | 1d10 (tuned) | **1d10 + WIT** | ZIP | half | burn (3t), ignores armor |
| 🔮 **GUMBALL** | 3d8 | **2d4 + WIT** | — | — | bouncy orb, ignores armor |
| ☄️ **PEW-PEW-PEW** | 3×2d6 | **3d6** (3 pews) | — | — | each pew rolls separately |
| ⚡ **ZIGZAG** | 8d6 | **4d6** | ZIP | half | line — zaps both foes |
| 💥 **KABLOOEY** | 8d6 | **4d6** | ZIP | half | AoE — bonks all foes, burn |

### 🌿 COZY (SAVVY)

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| ✚ **BOO-BOO FIX** ★ | 1d8 + mod | **heal 1d8 + SAV** | — | — | — |
| ✦ **THERE-THERE** ★ | 1d4 + mod | **heal 2d4 + SAV** | — | — | cures 1 status |
| 🔆 **SUNBEAM** | 1d8 | **1d4 + SAV** | ZIP | avoid | ignores armor |
| 🌟 **SPOTLIGHT** | 4d6 | **2d6 + SAV** | — | — | marks foe; your next hit +ACC |
| 🖤 **OWIE** | 3d10 | **3d6 + SAV** | — | — | big poke, ignores armor |
| 🦆 **GUARDIAN DUCKS** | 3d8 | **2d6** | SAV | half | swarm pecks all foes, −1 SPD |
| 🤗 **GROUP HUG** | 3d8 + mod | **heal 2d6 + SAV** | — | — | heals self + all allies |

### 😏 SASS (CHARM)

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| 💢 **SICK BURN** ★ | 1d4 | **1d4 + CHM** | SAV | avoid | ignores armor; −ACC on foe |
| 🌑 **STINK EYE** | 1d10 | **1d6 + CHM** | — | — | force glare, ignores armor |
| 🗣️ **SPOOKY STORY** | 3d6 | **2d4 + CHM** | SAV | half | psychic; foe may flee |
| 🩸 **JINX** | +1d6/hit | **+1d4 dmg** per swing | — | — | curse: extra dmg on your hits |
| 💖 **BAT EYES** ★ | control | **skip turns = spell LV** | SAV | avoid | charm (control) |
| 👁️ **TRASH TALK** | debuff | **−2 to enemy rolls** | CHM | avoid | weakens up to 3 foes |

> Where a ★ shipped spell's number here disagrees with `index.html`, the code wins — sync this doc to it.

---

## 5. Spell scaling (the GROWTH idea, folded into spell level)

Each spell levels up independently (`player.spells = { zap_zap: 2, … }`). Per the engine, **each upgrade adds +1 to the final, post-halved number**:

- **Attack spells:** +1 damage per spell level.
- **Heal spells:** +1 HP healed per spell level.
- **Control spells:** +1 turn of effect per spell level.

To reach spell level **N**, the caster's stat must be at `req + (N − 1)`. So a `req 12` spell wants WITS 12 for LV1, WITS 13 for LV2, and so on — odd ability scores are stored progress toward the next spell rank (same odd-stat rule as everything else).

> **Why +1-per-level instead of adding dice?** Adding a die then halving averages a fractional gain that rounds inconsistently at this tiny scale. A flat +1 to the halved total is predictable and keeps the cute low numbers readable. If you ever want dice-based growth, add the extra die to the **big roll** *before* halving — never after.

### Optional power bands for damage zaps

Scales the zappy at-will attack spells across the 20 floors in four bands:

| Player level | Big dice | In-game (e.g. HOTFOOT) |
|--------------|----------|------------------------|
| 1–4   | 1 die  | 1d10 + WIT |
| 5–10  | 2 dice | 2d10 + WIT, then halve |
| 11–16 | 3 dice | 3d10 + WIT, then halve |
| 17–20 | 4 dice | 4d10 + WIT, then halve |

---

## 6. Status effects (reuse the engine's four)

Spells apply the same statuses weapons do. Damage-over-time is **already at half scale** — do not re-halve it.

| Status | Effect | Persists between battles? |
|--------|--------|---------------------------|
| ☠ Poison | 1 dmg / turn × 3 turns | Yes |
| 🔥 Burn | 2 dmg / turn × 3 turns | Yes |
| 🩸 Bleed | scaling dmg / turn (see combat rules) | Yes |
| 💫 Stun / Charm | skip turn(s) | No (cleared at battle start) |

---

## 7. Wiring a spell into `index.html`

Spells live in the `PLAYER_SPELLS` array ([index.html:129](index.html:129)). The halving is applied by the combat resolver, not stored on the spell — you author the **big dice** and the engine halves the rolled total. Field shape:

```js
{ id:"zigzag", name:"ZIGZAG", emoji:"⚡", stat:"intelligence", req:14, cost:2,
  desc:"4D6 ZAP · HITS BOTH FOES · ZIP SHRUG FOR HALF · IGNORES ARMOR",
  type:"attack", die:{count:4,size:6}, saveStat:"dexterity", onSave:"half",
  ignoreArmor:true, hitsAll:true }
```

| Field | Meaning |
|-------|---------|
| `id` / `name` | unique key / cartoon display name (ALL-CAPS, no D&D terms) |
| `stat` | caster knack — `intelligence` (WITS) \| `wisdom` (SAVVY) \| `charisma` (CHARM) *(internal keys stay classic; labels reskin — see [CLAUDE.md](CLAUDE.md))* |
| `req` | minimum score to learn (LV1); `+1` per further level |
| `cost` | spell-point cost to learn (1–2) |
| `type` | `attack` \| `heal` \| `control` |
| `die` | `{count, size}` — the **big dice**; engine halves the total |
| `statBonus` | ability whose mod is added to the roll *before* halving |
| `autoHit` | skips the shrug, always lands (ZAP-ZAP) |
| `saveStat` / `onSave` | defender's shrug ability + result (`avoid`/`half`/`status`/`both`) |
| `effect` | `poison` \| `burn` \| `bleed` \| `stun` to apply on hit |
| `ignoreArmor` | bypass armor (most spells do) |

**Design conventions**

- Author every spell at its **big dice**; let the resolver halve. Never pre-halve a die in the data table — keeping the big value visible makes the scaling auditable.
- A connecting damage spell floors at **1**. Healing floors at **1**.
- A shrug halves the already-halved total (§1) — that double application is intended.
- Shrug rolls, turn counts, and slows are full-scale — never halve them.
- Keep `desc` short and ALL-CAPS for the retro UI; lead with the dice (`"4D6 ZAP · …"`).
- **Names stay original and silly.** No legacy labels (`MAGIC MISSILE`, `FIRE BOLT`, `CURE WOUNDS`, …) — if you find one in code, rename it to its cartoon name from §4.
