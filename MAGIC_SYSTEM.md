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

### Pick the right shrug — what the defender is doing

Each shrug stat answers *"how is the target trying NOT to get hit by this thing?"* Match the spell's effect to one of these — wrong-fit shrugs feel arbitrary.

| Shrug | Stat | Defender is… | Use for |
|-------|------|--------------|---------|
| **ZIP** | DEX | dodging the attack | projectiles, lines, blasts, dodgeable AoE |
| **GUTS** | CON | withstanding it | poison, burn, freeze, fatigue, anything that gets *into* the body |
| **WITS** | INT | outsmarting it | illusions, confusion, mind-tricks, mazes |
| **SAVVY** | WIS | zen-ing through it | charm, fear, sleep, calm-under-pressure mental effects |
| **CHARM** | CHA | manipulating their way out | curses, taunts, social pressure, "talk it down" |

A heat-cloud asks for **GUTS** (withstand the burn), not WIT — even though it's a clever caster's spell. The shrug describes the **defender's experience**, not the caster's flavor.

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

Three stats can cast. Each is a **knack** with a clear *specialty* — the archetype it does best — but every knack can dabble in any archetype if the situation demands it. The specialty matches what's already in the shipped code: ZAP-ZAP and HOTFOOT (WITS) are damage, BOO-BOO FIX and THERE-THERE (SAVVY) are heals, BAT EYES (CHARM) is control.

| Stat | Knack | Specialty (best at) | Also dabbles in |
|------|-------|---------------------|-----------------|
| **WITS** | 🤓 **WHIZ-BANG** | direct damage, AoE blasts, DoT zaps, elemental procs | clever tactical control |
| **SAVVY** | 🌿 **COZY** | heals, wards, status cures, prep-defense | off-spec radiant/nature damage |
| **CHARM** | 😏 **SASS** | control — charm, fear, lockdown, debuffs | psychic damage, single-target taunts |

> **Specialty, not exclusivity.** Every knack carries at least one spell of every archetype (bonk · heal · status · AoE · DoT · control) so a single-knack caster can cover any need. The specialty column is where its **strongest, scalable, cheapest** picks live. A 🌿 COZY caster *can* learn a WHIZ-BANG-style zap — it'll just hit softer than the real thing, and want a higher `req` to learn.

A spell gates on its stat via `req` (minimum score to learn) and grows as that score climbs.

**Enemies follow the same scheme.** A caster enemy's archetype is set by its highest mental stat — the Witch (high WIS) heals allies, an INT-heavy foe should be slinging zaps. Details in §5.

---

## 4. Spell roster

All **In-game** values are already halved and ready to use. The **Big roll** column is the author's pre-halve working value — purely an internal scaling note (`in-game ≈ floor(big / 2)`). Spells marked **★ shipped** are implemented in `PLAYER_SPELLS` today; treat [index.html:129](index.html:129) / [CLAUDE.md](CLAUDE.md) as the source of truth for their exact numbers. The rest are designed-and-ready expansion picks.

Each table is sorted **specialty picks first**, then **off-spec** (clearly tagged in the Effect column). Off-spec picks should generally have a higher `req` or smaller dice than a same-tier specialty pick from the other knack.

### 🤓 WHIZ-BANG (WITS) — damage

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| ✨ **ZAP-ZAP** ★ | 3×(1d4+1) | **1d4, auto-hit** | — | — | ignores armor |
| 🔥 **HOTFOOT** ★ | 1d10 (tuned) | **1d10 + WIT** | ZIP | half | burn (3t), ignores armor |
| 🥶 **BRAIN FREEZE** | 1d8 | **1d4 + WIT** | GUTS | half | −1 SPD (slow) |
| ⚡ **STATIC CLING** | 1d8 | **1d4 + WIT** | ZIP | half | target can't counter |
| 🔮 **GUMBALL** | 3d8 | **2d4 + WIT** | — | — | bouncy orb; ignores armor |
| ☄️ **PEW-PEW-PEW** | 3×2d6 | **3d6** (3 pews) | — | — | each pew rolls separately |
| ⚡ **ZIGZAG** | 8d6 | **4d6** | ZIP | half | line — zaps both foes |
| 💥 **KABLOOEY** | 8d6 | **4d6** | ZIP | half | AoE — bonks all foes, burn (3t) |
| ❄️ **FROSTY FIZZ** | 3d8 | **2d4 + WIT** | GUTS | half | AoE — all foes, −1 SPD |
| 🌋 **MELT ZONE** | 2d6 | **1d6 + WIT** | GUTS | both | AoE burn (3t); cloud lingers |
| 🤢 **TUMMY ACHE** | 1d4 | **1d4 + WIT** | GUTS | both | poison (3t); ignores armor |
| 🪞 **MIRROR MAZE** | — | **stunned 1t** | WIT | avoid | off-spec control; foe confused |
| 🧊 **POPSICLE** | — | **skip turns = spell LV** | GUTS | avoid | off-spec control; frozen solid |

### 🌿 COZY (SAVVY) — heal & defense

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| ✚ **BOO-BOO FIX** ★ | 1d8 + mod | **heal 1d8 + SAV** | — | — | — |
| ✦ **THERE-THERE** ★ | 1d4 + mod | **heal 2d4 + SAV** | — | — | cures 1 status |
| 🩹 **BAND-AID** | 1d4 + mod | **heal 1d4 + SAV** | — | — | small patch; in-or-out of combat |
| 🌟 **WAKEY-WAKEY** | 1d6 + mod | **heal 1d4 + SAV** | — | — | also cures 1 status |
| 🍵 **SOOTHING BREW** | — | **heal 1 + SAV each turn × 3t** | — | — | heal-over-time buff |
| 🤗 **GROUP HUG** | 3d8 + mod | **heal 2d6 + SAV** | — | — | heals self + all allies |
| 🩺 **CHECK-UP** | — | **cure ALL status** | — | — | full cleanse, no heal |
| 🛡️ **BUBBLE** | 1d8 + mod | **+1d4 + SAV temp HP** | — | — | absorb shield, 1 battle |
| ⛑️ **HARD HAT** | — | **+2 ARM, 1 battle** | — | — | armor buff |
| 🪟 **GLASS HOUSE** | — | **reflect next spell at caster** | — | — | one-shot spell counter |
| 🌿 **THORNS** | — | **self-buff 3t** | — | — | attackers gain bleed (3t) |
| 🔆 **SUNBEAM** | 1d8 | **1d4 + SAV** | ZIP | avoid | off-spec; ignores armor |
| 🌟 **SPOTLIGHT** | 4d6 | **2d6 + SAV** | — | — | off-spec; marks foe, your next hit +ACC |
| 🖤 **OWIE** | 3d10 | **3d6 + SAV** | — | — | off-spec; big poke, ignores armor |
| 🦆 **GUARDIAN DUCKS** | 3d8 | **2d6** | ZIP | half | off-spec AoE; pecks all foes, −1 SPD |
| 🐍 **SNAKE SPIT** | 1d4 | **1d4 + SAV** | GUTS | both | off-spec AoE poison (3t); ignores armor |
| 🐝 **BEE SWARM** | 2d6 | **1d6 + SAV** | GUTS | half | off-spec AoE; poison (3t) |

### 😏 SASS (CHARM) — control

| Spell | Big roll | In-game (halved) | Shrug | onSave | Effect |
|-------|---------:|------------------|-------|--------|--------|
| 💖 **BAT EYES** ★ | control | **skip turns = spell LV** | SAV | avoid | charm |
| 👁️ **TRASH TALK** | debuff | **−2 to enemy rolls** | CHM | avoid | weakens up to 3 foes |
| 😱 **YIKES!** | — | **flee chance 50% × 2t** | SAV | avoid | fear |
| 🎭 **PUPPET STRINGS** | — | **foe attacks ally 1t** | WIT | avoid | if alone, skip turn |
| 😴 **NIGHTY-NIGHT** | — | **skip turns = spell LV** | SAV | avoid | sleep; only on foes ≤ half HP |
| 🌳 **ROOT WIGGLE** | — | **stunned 1t** | GUTS | avoid | rooted; ignores armor |
| 🤐 **ZIP-IT** | — | **no spells, 2t** | CHM | avoid | silences a caster foe |
| 🥱 **SNOOZER** | — | **drowse all foes 1t** | SAV | avoid | AoE skip-turn (weakened sleep) |
| 💢 **SICK BURN** ★ | 1d4 | **1d4 + CHM** | SAV | avoid | off-spec; ignores armor, −ACC on foe |
| 🌑 **STINK EYE** | 1d10 | **1d6 + CHM** | — | — | off-spec; force glare, ignores armor |
| 🗣️ **SPOOKY STORY** | 3d6 | **2d4 + CHM** | SAV | half | off-spec psychic; foe may flee |
| 🩸 **JINX** | +1d6/hit | **+1d4 dmg** per swing | — | — | off-spec; curse: extra dmg on your hits |
| 🤡 **HEX-A-FOE** | 1d4 | **1 dmg + poison + burn + bleed** | SAV | both | off-spec; triple-DoT, ignores armor |
| 👃 **STINK BOMB** | 1d6 | **1d4 + CHM** | GUTS | both | off-spec AoE poison (3t) |
| 🎺 **KAZOO BLAST** | 3d6 | **2d4 + CHM** | CHM | half | off-spec AoE psychic; bleed |

> Where a ★ shipped spell's number here disagrees with `index.html`, the code wins — sync this doc to it.

### Quick archetype lookup

Same roster, sorted by intent. A spell can appear under more than one — many AoE spells also tag a status. **Bold knack** = that archetype's specialty home.

**💥 BONK** (direct damage)
- **🤓 WHIZ-BANG**: ✨ ZAP-ZAP · 🔥 HOTFOOT · 🥶 BRAIN FREEZE · ⚡ STATIC CLING · 🔮 GUMBALL · ☄️ PEW-PEW-PEW · ⚡ ZIGZAG · 💥 KABLOOEY · ❄️ FROSTY FIZZ · 🌋 MELT ZONE · 🤢 TUMMY ACHE
- 🌿 COZY: 🔆 SUNBEAM · 🌟 SPOTLIGHT · 🖤 OWIE · 🦆 GUARDIAN DUCKS · 🐍 SNAKE SPIT · 🐝 BEE SWARM
- 😏 SASS: 💢 SICK BURN · 🌑 STINK EYE · 🗣️ SPOOKY STORY · 🩸 JINX

**💚 HEAL & DEFENSE** (restore HP, prevent damage)
- **🌿 COZY**: ✚ BOO-BOO FIX · ✦ THERE-THERE · 🩹 BAND-AID · 🌟 WAKEY-WAKEY · 🍵 SOOTHING BREW · 🤗 GROUP HUG · 🩺 CHECK-UP · 🛡️ BUBBLE · ⛑️ HARD HAT · 🪟 GLASS HOUSE · 🌿 THORNS (counter-defense)

**🎯 CONTROL** (skip-turn lockdown, charm, fear)
- **😏 SASS**: 💖 BAT EYES · 😱 YIKES! · 🎭 PUPPET STRINGS · 👁️ TRASH TALK · 😴 NIGHTY-NIGHT · 🌳 ROOT WIGGLE · 🤐 ZIP-IT · 🥱 SNOOZER
- 🤓 WHIZ-BANG: 🪞 MIRROR MAZE · 🧊 POPSICLE

**🟢 STATUS APPLIERS** (one tag, three-turn tick)
- **🤓 WHIZ-BANG**: 🔥 HOTFOOT (burn) · 🤢 TUMMY ACHE (poison) · 🌋 MELT ZONE (burn) · 💥 KABLOOEY (burn)
- 🌿 COZY: 🐍 SNAKE SPIT (poison) · 🐝 BEE SWARM (poison) · 🌿 THORNS (counter-bleed)
- 😏 SASS: 🤡 HEX-A-FOE (all three) · 👃 STINK BOMB (poison) · 🎺 KAZOO BLAST (bleed)

**🌪️ AoE** (line, swarm, all foes)
- **🤓 WHIZ-BANG**: ☄️ PEW-PEW-PEW · ⚡ ZIGZAG · 💥 KABLOOEY · ❄️ FROSTY FIZZ · 🌋 MELT ZONE
- 🌿 COZY: 🤗 GROUP HUG (allies) · 🦆 GUARDIAN DUCKS · 🐍 SNAKE SPIT · 🐝 BEE SWARM
- 😏 SASS: 👁️ TRASH TALK · 👃 STINK BOMB · 🎺 KAZOO BLAST · 🥱 SNOOZER

**🔁 DoT** (lingering tick damage — the actual ticks live in §6)
- **🤓 WHIZ-BANG**: 🔥 HOTFOOT · 💥 KABLOOEY · 🌋 MELT ZONE · 🤢 TUMMY ACHE
- 🌿 COZY: 🍵 SOOTHING BREW (heal-over-time) · 🐍 SNAKE SPIT · 🐝 BEE SWARM · 🌿 THORNS
- 😏 SASS: 🤡 HEX-A-FOE · 👃 STINK BOMB · 🎺 KAZOO BLAST

> Designing a new DoT? **Keep the upfront damage low** — three-turn ticks compound hard at half-scale HP. A status-applying spell should usually do *less* immediate damage than a same-tier pure-bonk spell. 🤡 HEX-A-FOE is the extreme example: 1 dmg upfront, then three statuses stack — gated behind a SAV shrug and CHM 16+ for a reason.

---

## 5. Enemy casters — same scheme, different spells

The specialty framework applies to enemies too. A caster's primary mental stat sets its archetype, mirroring §3:

| Enemy's high stat | Knack | Specialty |
|-------------------|-------|-----------|
| **INT** | 🤓 WHIZ-BANG | damage |
| **WIS** | 🌿 COZY | heal / ally support |
| **CHA** | 😏 SASS | control / debuff |

Enemy spells live on each `ENEMY_DEFS` entry as a `spell` field (see [CLAUDE.md](CLAUDE.md)). Shape is simpler than `PLAYER_SPELLS` — flat damage, one optional status, no upgrade levels — and `spellChance` gates cast-vs-melee per turn. Keep them simple.

> **Most enemies use some kind of magic.** Dungeon Quest's monsters aren't pure-physical brutes by default — almost every spawn brings *some* trick. Pure melee (a goblin with a rusty knife, a mindless slime, a magic-null golem) is the exception. Default question when designing a new enemy: **what magic does it bring?** Non-magical foes have to earn the "just claws" tag (mindless · pure beast · magic-null).

### Engine reality

Today's `spell` field only resolves **damage** (plus an optional status rider). COZY ally-heals and SASS control debuffs need engine extension or per-enemy hooks. The shipped Witch already takes this route — `healsAllies: true` is checked separately at [index.html:1893](index.html:1893), parallel to the `spell` field. New COZY/SASS effects should follow the same pattern (parallel flags), not overload the damage resolver.

> **For the enemy-specific roster, audit, and implementation plan, see [BESTIARY.md](BESTIARY.md).** This doc keeps the magic-design principles general; the bestiary tracks who casts what.

---

## 6. Spell scaling (the GROWTH idea, folded into spell level)

Each spell levels up independently (`player.spells = { zap_zap: 2, … }`). Per the engine, **each upgrade adds +1 to the final, post-halved number**:

- **Attack spells:** +1 damage per spell level.
- **Heal spells:** +1 HP healed per spell level.
- **Control spells:** +1 turn of effect per spell level.

To reach spell level **N**, the caster's stat must be at `req + (N − 1)`. So a `req 12` spell wants the relevant knack-stat at 12 for LV1, 13 for LV2, and so on — odd ability scores are stored progress toward the next spell rank (same odd-stat rule as everything else).

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

## 7. Status effects (reuse the engine's four)

Spells apply the same statuses weapons do. Damage-over-time is **already at half scale** — do not re-halve it.

| Status | Effect | Persists between battles? |
|--------|--------|---------------------------|
| ☠ Poison | 1 dmg / turn × 3 turns | Yes |
| 🔥 Burn | 2 dmg / turn × 3 turns | Yes |
| 🩸 Bleed | scaling dmg / turn (see combat rules) | Yes |
| 💫 Stun / Charm | skip turn(s) | No (cleared at battle start) |

---

## 8. Wiring a spell into `index.html`

Spells live in the `PLAYER_SPELLS` array ([index.html:129](index.html:129)). The halving is applied by the combat resolver, not stored on the spell — you author the **big dice** and the engine halves the rolled total. Field shape:

```js
{ id:"hotfoot", name:"HOTFOOT", emoji:"🔥", stat:"intelligence", req:14, cost:2,
  desc:"1D10 ZAP · BURN (3T) · ZIP SHRUG FOR HALF · IGNORES ARMOR",
  type:"attack", die:{count:1,size:10}, statBonus:"intelligence",
  saveStat:"dexterity", onSave:"half", effect:"burn", ignoreArmor:true }
```

| Field | Meaning |
|-------|---------|
| `id` / `name` | unique key / cartoon display name (ALL-CAPS, no D&D terms) |
| `stat` | caster knack — `intelligence` (WITS → 🤓 WHIZ-BANG) \| `wisdom` (SAVVY → 🌿 COZY) \| `charisma` (CHARM → 😏 SASS) *(internal keys stay classic; cartoon labels reskin — see [CLAUDE.md](CLAUDE.md))* |
| `req` | minimum score to learn (LV1); `+1` per further level. Off-spec spells should sit 2+ higher than a same-tier specialty pick. |
| `cost` | spell-point cost to learn (1–2) |
| `type` | `attack` \| `heal` \| `control` |
| `die` | `{count, size}` — the **big dice**; engine halves the total |
| `statBonus` | ability whose mod is added to the roll *before* halving |
| `autoHit` | skips the shrug, always lands (✨ ZAP-ZAP) |
| `saveStat` / `onSave` | defender's shrug ability + result. `saveStat` ∈ `dexterity` (ZIP) \| `constitution` (GUTS) \| `intelligence` (WIT) \| `wisdom` (SAV) \| `charisma` (CHM). Pick the one that matches what the defender is *doing* (see §2). |
| `effect` | `poison` \| `burn` \| `bleed` \| `stun` to apply on hit |
| `ignoreArmor` | bypass armor (most spells do) |

**Design conventions**

- Author every spell at its **big dice**; let the resolver halve. Never pre-halve a die in the data table — keeping the big value visible makes the scaling auditable.
- A connecting damage spell floors at **1**. Healing floors at **1**.
- A shrug halves the already-halved total (§1) — that double application is intended.
- Shrug rolls, turn counts, and slows are full-scale — never halve them.
- **Pick the shrug stat by defender experience, not caster flavor** (§2 table). A wizard's poison cloud still asks for GUTS.
- **Off-spec picks pay rent.** A heal cast off WHIZ-BANG or damage cast off COZY should be weaker, costlier, or higher-`req` than the same archetype on its specialty knack. The specialty has to mean something.
- Keep `desc` short and ALL-CAPS for the retro UI; lead with the dice (`"4D6 ZAP · …"`).
- **Names stay original and silly.** No legacy labels (`MAGIC MISSILE`, `FIRE BOLT`, `CURE WOUNDS`, …) — if you find one in code, rename it to its cartoon name from §4.
