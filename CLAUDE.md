# Dungeon Quest Project Prompt

Dungeon Quest is a single-file browser RPG built with React 18, Babel standalone, and inline CSS inside `index.html`. It is a retro pixel-style turn-based dungeon crawler inspired by classic Zelda, Rogue, and tabletop fantasy.

The game should feel fast, readable, charming, and arcade-like. Preserve the 8-bit visual language: dark dungeon palette, chunky bordered tile boxes, small tactical text in Press Start 2P, emoji-based characters/items, simple CSS keyframe animations, and punchy combat logs.

## Core Experience

The player navigates a **6-room floor laid out on a 3×2 grid with fog of war**. They walk from tile to tile (adjacency only, with fast-travel through previously-visited tiles), fight one or two enemies in each room, earn XP and gold, level up, and pick an ability-score upgrade (plus a perk at levels 5, 10, 15, and 20). The boss is one of the 6 rooms; defeating it unlocks the shop AND transforms the boss tile into stairs. The player then descends when ready (no early-exit option — you must beat the boss to advance). The dungeon clears at level 21.

Each combat is a turn-based duel mediated by an **initiative roll on speed**. Big rooms may host two enemies fighting alongside each other — the player chooses their target each turn via clickable enemy tiles.
- Player swings (d20-to-hit, weapon-die damage, optional offhand/double/triple strike, optional crit + procs).
- Enemies swing one at a time (d20-to-hit, d3 + attack value, optional poison/burn/bleed/stun proc, optional counter from the player).
- Status effects tick at the start of each enemy phase: enemy statuses first (each enemy's poison/burn/bleed), then the player's once, then alive enemies attack in array order.

Keep the game playable directly from the HTML file unless a requested feature truly requires a build step.

## Mechanics Overview

### Character creation (point-buy)

- New runs open with a **`create` phase** before `idle`. Player picks a **name** (≤16 chars), an **emoji** from `EMOJI_POOL` (12 options), and spends **27 points** across 6 ability scores via the `POINT_COST_STEP` table.
- Base score for every stat is **8** (floor at character creation). Escalating per-step cost: `8→13` = 1 pt/step, `13→14, 14→15` = 2, `15→16` = 3, `16→17` = 4, `17→18` = 5. **18 is the creation cap.**
- `BEGIN ADVENTURE` always works, even with unspent points — they bank on the player via `unspentPoints` state and persist into level-ups. Banked points show as a chip on the idle/exploration screen and inside the level-up panel.
- `draftAdjust(statId, ±1)` refunds the exact cost when decrementing — players can re-allocate freely until they confirm.

### Level-up (per +1 point spending)

- Each level-up grants **+2 unspent points** (added to any banked points). The `levelup` panel shows current STR/DEX/CON/WIS/CHA/INT, total points remaining, and one button per stat (`pickStat` spends one point for `+1`). `DONE` exits and banks any unspent points.
- Per-step cost is **always 1 pt** during level-up (no escalating cost, no 18 cap). Character creation's cost table is creation-only.
- WIS milestone (`WIS ≥ 20` for the first time) still triggers the `wisperk` phase mid-`pickStat`. After picking, the panel re-opens so any remaining points can be spent.

### Attributes (D&D-style, default 8 at creation, mod = floor((score-10)/2))

**Odd-stat rule**: `abilityMod` uses `floor((score-10)/2)`, so 9 = 10's mod, 11 = 10's mod, 13 = 12's mod, etc. Odd scores are "stored progress" toward the next even bump — relevant when spending banked points.

- `strength` → +1 ATK and +5% HIT per mod
- `dexterity` → +5% DODGE, +5% CRIT, and **+1 SPD** per mod
- `constitution` → applied retroactively to every level's hit die roll (see HP system); +5% poison resist per positive mod
- `wisdom` → +15% room awareness per mod (reveals room types and enemy previews on the dungeon map); **+1 HP per mod added to every heal item** (`item.heal + max(0, wisMod)` in `handleUseItem`). At WIS 20 the player picks one **WIS perk** from `WIS_PERKS` (one-time choice, scales with further WIS):
  - **Sage's Reservoir** (`wisPerk === "reservoir"`) — every heal grants `wisMod` Temp HP (capped at `maxHp`). Temp HP is consumed before real HP during enemy attacks (handled inline in `attackOne` via the `curTempHp` local) and cleared on `startBattle`.
  - **Swift Cure** (`wisPerk === "swift"`) — heal items roll `wisMod × 10%` chance to NOT consume the player's turn (skip `runEnemyTurn`).
- `charisma` → -5% shop prices per mod
- `intelligence` → +15% identify per mod (shows full equipment stats in shop at 50%+)

### HP system (hit dice)

`maxHp = floor((sum(hitDice) + conMod × hitDice.length) / 2) + equipmentBonus`

- `hitDice` is a per-level array stored on the player. Level 1 is always max (8).
- Each level-up rolls a fresh d8 and appends. CON modifier is applied retroactively to all dice, so picking CON later still grants partial benefit to earlier levels.
- Half-max-HP heal on level up; the d8 roll is logged.

### Perks

- **Perk pick** every 5th level (L5, L10, L15, L20) from `PERK_POOL`. Once taken, removed from future offers.
  - **Double Strike** — +20% chance to attack again (mainhand)
  - **Triple Strike** — +15% chance to attack again (offhand)
  - **Counter Strike** — +20% chance to counter-attack when hit
  - **Iron Will** — survive one killing blow at 1 HP per battle
  - **Armor Pierce** — crits ignore enemy armor
  - **Dual Wield** — equip a second light weapon in offhand; both swing each turn
  - **Master Duelist** — removes the offhand 50% damage penalty
- XP needed = `level * 3`.

### Combat dice & weapons

- `dieIdx` indexes into `DICE_PROGRESSION` (D2 → 40D10). Each equipped weapon overrides the player's die.
- Weapons are classed by `hand: 'light' | 'one' | 'two'` — light weapons are 1H and dual-wieldable, regular 1H pairs with a shield, 2H locks the offhand slot.
- Damage formula: `atkDmg + die roll − armorUsed` (crits roll the die twice). Pierce procs and the Armor Pierce perk drop `armorUsed` to 0.
- Offhand swings deal 50% damage unless Master Duelist is picked. Bleed bonus damage applies on top.
- Rare weapons (`rare: true`) carry procs: `poisonChance`, `burnChance`, `bleedChance`, `pierceChance` rolled per swing on damaging hits.
- Vampiric weapons heal 2 HP on a crit (works from either hand).

### Initiative & surprise

- `speed = 10 + dexMod` for the player (`applyAbilityStats`). Equipment can stack via `stats.speed`.
- Each enemy has a `speed` value (6–15 typical; bosses vary). Compared at `startBattle`.
- Higher speed acts first; player wins ties.
- **Surprise**: if `room.enemyKnown === false` (the player failed the WIS preview), the enemy gets **+5 SPD for that battle** and the player can be hit before acting. Often enough to lose initiative to a normally-slower foe.
- **Bosses are never surprised**: `bossRoom.enemyKnown` is hardcoded `true` in `makeFloorMap` so the boss tile always shows the foe and the player always gets a fair initiative roll. (A future room-less ambush-boss type — pencilled in for floor 18+ — would be the exception.)

### Status effects

- ☠ **Poison** — 1 dmg / turn for 3 turns. Resisted by CON (poisonResist% chance to ignore application). Cured by Antidote/Elixir.
- 🔥 **Burn** — 2 dmg / turn for 3 turns. Cured by Ointment/Elixir.
- 🩸 **Bleed** — set to `floor(weapon_avg_die + atkDmg)` on a successful bleed proc. That value is both the per-swing damage bonus AND the turn counter (decrements −1 per enemy turn). Re-procs deepen the wound (Math.max). Player-side bleed is a separate +50% incoming multiplier (legacy). Cured by Bandaid/Elixir.
- 💫 **Stun** — skip your next turn (auto-runs another enemy turn). Cured by Smelling Salt/Elixir.

**Persistence rule**: poison / burn / bleed **carry between battles** intentionally — they're cleared only via rest, an item with a cure flag, or restart. **Stun does NOT carry** (cleared at `startBattle`).

### Floor / room structure

- A **floor** has `FLOOR_SIZE` (6) persistent rooms laid out on a **3×2 grid** by `makeFloorMap(floorNum, level, p, revealAll)`. Tiles get a stable `pos:{col,row}` from `FLOOR_POSITIONS`. The entrance (always position `{0,0}`) is always a fight room and is pre-`visited:true` so the player can never spawn on top of the boss. The other five tiles are populated by shuffling the boss + 4 remaining fight rooms across positions 1–5.
- Each room carries an **`enemies` array** (1 or 2 entries pre-rolled). Big rooms have a 50% chance to spawn 2 enemies. The boss room always has exactly 1 enemy (the boss).
- Rooms carry `cleared`, `visited`, and a stable `key`. `resolveVictory` flips `cleared:true` after a kill; `handleRoomEnter` flips `visited:true` the moment the player walks onto the tile.
- Room types in `ROOM_TYPES`: small (clean fight), big (+gold, may spawn 2 foes via `maybeDouble:true`), bridge (swift foe / +gold), shrine (weaker foe / less gold). **Stairs is no longer a room type** — they're created on demand by the boss kill.
- **Boss-as-room**: the boss is one of the 6 rooms on the floor, not forced at a fixed position. Player navigates to it. Defeating the boss **transforms the boss tile into a stairs tile** (`isBoss:false, descend:true, cleared:true, icon:⇣, label:"STAIRS DOWN"`) and unlocks the shop. The rest of the floor persists — the player walks back to the stairs (or clicks DESCEND in the HERE strip) when ready.
- **Stairs descent** (`descend:true` flag): only exists after the boss is dead. `handleRoomEnter` checks `room.descend` before `room.cleared`, so the tile descends on contact regardless of cleared status. `descendStairs()` advances `waveNumber+1`, resets `playerPos` to `{0,0}`, regenerates the next floor.
- Pre-rolled enemies + WIS-rolled `known` / `enemyKnown` flags persist for the lifetime of the floor — toggling dev mode doesn't reshuffle them; `revealAll` simply overrides the gating at render time so the player's exploration progress is preserved.

### Fog-of-war map

- **`playerPos:{col,row}` state** tracks where the hero is standing. Resets to `{0,0}` on every floor regeneration (boss kill / stairs descent / restart).
- **Visibility**: a tile is rendered with its full info if `visited` OR adjacent (Manhattan-1) to any visited tile, OR `revealAll` (dev mode) is on. Everything else renders as a foggy `?` cell.
- **Movement**: clicking a tile fires `handleRoomEnter(room)`. The handler accepts the click only if the tile is visible AND either adjacent to `playerPos` or already visited (visited tiles are always "fast-travel-able" through cleared paths).
- **Click outcomes**:
  - Tile is already cleared → silent walk-through (`▷ YOU CROSS BACK THROUGH X.` log).
  - Tile is uncleared, `descend:true` → `descendStairs()`.
  - Otherwise → `startBattle(room)` against the room's pre-rolled enemies.
- **UI** (in `FloorMap` component, replaces the old vertical button list):
  - 3×2 grid of room tiles. Player avatar (🧝 by default) overlays the current tile.
  - Color coding: current tile (yellow border), cleared (green), boss (red), stairs (blue), reachable (warm brown), foggy (dim).
  - The idle screen also renders a "HERE" detail strip below the map showing the current tile's info and an `ENGAGE` / `DESCEND` button if the tile is uncleared — important for touch users who can't easily click the small map tiles.

### Multi-enemy combat

- **State**: `enemies` (array) replaces single `enemy`. `targetIdx` tracks which enemy the player is aiming at. A derived `enemy` constant points to the alive enemy at `targetIdx` (or the first alive if the target died) so existing single-enemy reads keep working.
- **Player turn**: `handleAction("attack")` strikes the currently-targeted enemy. Damage, procs, and berserker enrage all apply to that enemy in the `enemies` array. When the target dies, `targetIdx` auto-falls-back to the next alive enemy. Victory check is `enemies.every(e => !e || e.hp <= 0)`.
- **Target selection**: each enemy tile in the arena (and in the right-hand stat panel for multi-foe rooms) is clickable during the player phase. A yellow border + background highlight marks the current target. Clicking switches `targetIdx`.
- **Enemy turn** (`runEnemyTurn(p, snapshotEnemies)`): structured in 4 stages per round:
  1. Each enemy's status (poison/burn/bleed) ticks. Enemies can die from status here.
  2. Player status ticks once (poison/burn/bleed). Lethal status ends the run.
  3. Per-enemy archer flee check (`fleesWhenLow`).
  4. Each alive enemy attacks the player. Counter damage routes to the attacking enemy only. Stun, if applied at any point in the round, causes a recursive `runEnemyTurn` call after the round ends (skipping the next player turn).
- **Resolve victory** (`resolveVictory(defeatedList)`): accepts an array of defeated enemies. XP, gold, and kill-count stats are summed across all of them. Fled enemies contribute half rewards. Boss kills (still solo) regenerate the floor + open shop. Backwards-compatible: accepts a single-enemy object too (auto-wraps in array).

### Bosses

- `WAVE_BOSSES` covers waves 1–4: **KING Boar**, **Zombie LORD**, **ARCH Dark_Wizard**, **IRON Cursed_Sword**.
- Waves 5+ cycle `BOSS_DEFS`: **Dragon**, **Giant Cactus**, **PHANTOM Lord**, **Dark_Lord**. Scales with `bonus` and gains "ELDER" prefix after deep cycles.
- Many enemies carry behavior flags: `poisonOnHit`, `burnsOnHit`, `bleedsOnCrit`, `stunsOnHit`, `enragesWhenLow`, `fleesWhenLow`, `immuneToCrit`, `healsAllies` (heals a wounded ally instead of attacking; self-heals when below half HP and alone).

### Equipment & inventory

- **Body slots** (`BODY_SLOTS`): `weapon`, `offhand`, `mask`, `helmet`, `glasses`, `amulet`, `arms`, `torso`, `cloak`, `hands`, `belt`, `leg`, `feet` + 5 ring slots.
- **Weapons** (12 in `EQUIPMENT_DEFS`): 5 commons (dagger, shortsword, longsword, spear, greatsword) + 7 rares (sharp dagger, keen shortsword, venom dagger, flame brand, piercing longsword, bleeding spear, vampiric blade). Hand-classed; only dual-wieldable in the offhand if the perk is taken.
- **Shields** (3 tiers) live in the offhand slot. 2H weapons evict the offhand.
- **Rings** (7 types incl. Ring of Greed: +50% gold but +2 dmg taken).
- **Inventory stacking**: items stack by `id` with a `count` field. UI shows `(×N)` when count > 1. Helpers `addToInventory` / `removeFromInventory` handle the math; both consumables and displaced equipment use the same flow.
- **Buying replaces & banks the old**: buying a slotted item evicts whatever was there into inventory (with stack-by-id), including the offhand when a 2H is equipped. Consumables stay in the shop after purchase — stack up on antidotes.
- **Mid-fight gear swap**: equipping from inventory during the player phase **costs your turn** — the enemy gets a free strike, but it lands against your new gear's stats.
- **Sell**: at the shop, an inventory list lets you sell anything for `floor(cost/2)` (min 1). Sells one at a time off a stack.
- **Identify gating**: shop items show generic labels ("VITAL GEAR" / "DEFENSIVE GEAR" / "COMBAT GEAR" / "ENCHANTED GEAR") until `identify` (INT) reaches 50%. Dev mode bypasses.

### Consumables

- Heal items declare `heal: N` (or `"full"` for elixir). `handleUseItem` adds `max(0, wisMod)` on top, applies the WIS perk effect if any (`reservoir` → Temp HP, `swift` → free-action roll), and only then triggers `runEnemyTurn`. Cure-only items use `curePoison` / `cureBurn` / `cureBleed` / `cureStun` / `cureAll` flags (no `onUse`). Buff items still use `onUse(p,b)` to append to the buffs array.
- **Heal**: HEALTH POTION (+4), MEGA POTION (+8), SUPER POTION (+16), ELIXIR (full HP + cures all status). All accept the wisMod bonus.
- **Cure**: Antidote (poison), Ointment (burn), Bandaid (bleed), Smelling Salt (stun).
- **Buffs** (1–3 battles of stat boost): oils, whetstones, scrolls, smoke bomb, talisman, cloak.

### Rest / Flee

- 75% chance to flee. Flee or rest both move to the rest screen.
- BREAK CAMP from rest: full heal + clears all status. XP is **not** forfeited.

## Technical Constraints

- Dependency-light. No build step. React + ReactDOM + Babel from CDN.
- Plain React state/hooks. No frameworks.
- Read `index.html` before editing — do NOT regenerate the whole file.
- Preserve existing mechanics unless fixing a bug or adjusting balance.
- Effective player stats are computed every render via `getEffectivePlayer(player, equipment, tempBuffs)` which composes `applyEquipment → applyAbilityStats → applyBuffs`. Never mutate `player` with equipment/buff bonuses — they live only on the effective player.
- Follow existing data-driven patterns when adding content:
  - `ENEMY_DEFS`, `WAVES`, `WAVE_BOSSES`, `BOSS_DEFS`
  - `SHOP_CONSUMABLES`, `EQUIPMENT_DEFS`
  - `STAT_POOL`, `PERK_POOL`
  - `BODY_SLOTS`, `ROOM_TYPES`, `STAT_LABEL`, `DICE_PROGRESSION`

## Design Direction

- Retro fantasy dungeon tone.
- UI stays compact and game-like, not a landing page.
- Buttons are clear, chunky, readable (use existing `Btn` + `BTN_VARIANTS`).
- Emoji as sprites/icons.
- Combat log: short, dramatic, easy to scan. Use existing `LOG_COLORS` types (`sys`, `player`, `enemy`, `crit`, `warn`, `level`, `counter`, `dodge`, `miss`, `gold`).
- Show numbers tersely with `STAT_LABEL` abbreviations (ATK, ARM, HIT, DOD, CRT, SPD).

## Combat Flow — Read Before Editing

1. `startBattle(room)`
   - Pulls `baseEnemies` from `room.enemies` (or a fresh 1-element fallback). Big rooms can supply 2.
   - Computes surprise + initiative (highest enemy `finalSpeed` vs `ep.speed`).
   - Clears `stun`. Decrements `tempBuffs`. Sets `targetIdx = 0`.
   - If enemy wins initiative → fires `runEnemyTurn(ep, battleEnemies)` immediately. Otherwise → `setPhase("player")`.
2. `handleAction("attack")`
   - Target = `enemies[targetIdx]` (or first alive via the derived `enemy`).
   - `playerAttack(ep, target)` → `{ total, results, didPoison, didBurn, didBleed, newBleedVal }`. Each weapon swings once; mainhand always, offhand if dual-wielding, double/triple strike if rolled. **Each subsequent swing short-circuits with `enemyDead()` once `total >= e.hp`** so we don't whiff against a corpse.
   - Vampiric heal on crit (works from main or offhand vampiric weapon).
   - Apply proc statuses to the target if it survives. Bleed value is the proc's `newBleedVal`, taking `Math.max` of any existing bleed.
   - Berserker enrage check (per-target).
   - Splice the updated target back into `enemies` via `setEnemies(arr => arr.map(...))`.
   - If target died, auto-fall-back `targetIdx` to the next alive enemy.
   - If `enemies.every(e => !e || e.hp <= 0)` → `resolveVictory(newEnemies)`. Otherwise → `runEnemyTurn(effectivePlayer, newEnemies)`.
3. `runEnemyTurn(p, snapshotEnemies)` schedules a 750 ms `setTimeout`. Inside, in order:
   - **Enemy status ticks** for each enemy: poison drain, burn drain, bleed counter decrement. Multiple enemies can die from status simultaneously.
   - **Player status ticks** once (poison, burn, bleed) — lethal poison/burn ends with `setPhase("lose")`.
   - **Archer flee check** per enemy (`fleesWhenLow`); fled enemies get `hp:0, fled:true`.
   - If all enemies dead/fled here → `resolveVictory(curEnemies)`.
   - **Each alive enemy attacks** in array order via a chained `setTimeout` loop (`attackOne(i)` recurses with a ~700 ms gap). Each swing is its own visible beat: own log line(s), own `triggerArenaAnim`, own `shake("player")`, own `setPlayer` HP commit. The recursive structure means React renders between attacks so the player sees each one land instead of a single batched frame. Each call to `enemyAttack(curP, e)` returns dmgToPlayer, counterDmg, results, gotCrit, didPoison, didBurn, didBleed, didStun. Bleed × 1.5 (player-side bleed) applies to incoming damage. Counter damage routes to the attacking enemy. Player statuses accumulate across attacks. `stunApplied` flag is set if any enemy stunned.
   - Damage applied to player → iron will → lose → otherwise the chain falls through to `finishTurn()`. If `stunApplied`, schedule a recursive `runEnemyTurn` (next round) after 1100 ms; otherwise `setPhase("player")`.
4. `resolveVictory(defeatedList)`
   - Accepts an array of defeated enemies (single-enemy callers can pass an object — auto-wraps).
   - Greed multiplier on gold; per-enemy XP / gold / kill stats aggregated. Fled enemies contribute half rewards.
   - XP gain → possible level up (roll d8, append to hitDice, half-max heal).
   - **If any enemy was a boss**: transform the boss room into stairs (mutate the single room, do NOT regenerate the floor; player stays on the boss tile), open shop (consumables + 2 equipment picks). `waveNumber` is bumped later by `descendStairs` when the player chooses to descend.
   - **If regular room**: flip just that room's `cleared` flag — `setFloorRooms(rooms => rooms.map(r => r.key === roomKey ? {...r, cleared:true} : r))`. Rest of the floor persists.
   - Trigger perk-pick phase at multiples of 5, then stat-pick phase, then `setPhase("win")`.

Status counters (`poison`, `burn`, `bleed`, `stun`) read from closure inside `runEnemyTurn` — that's intentional and represents the "value at turn start". Don't break that pattern unless rewriting the timeout chain.

## Recently Added (last few sessions)

A short list of the latest changes — older additions live in the body of this doc.

- **Character creation + point-buy stat system**: New runs open in a `create` phase (name, emoji from `EMOJI_POOL`, 27-point allocation across 6 stats starting at 8). Cost table is escalating (1/1/1/1/1/2/2/3/4/5 per step from 8 to 18). Cap of 18 at creation. Unspent points bank on the player and carry into level-ups. Level-up now grants **+2 individual points** (no escalating cost during level-up); panel shows current stats and a `DONE` button. The classic STAT_POOL `+2` apply is now `+1` per pick. The WIS-20 perk milestone still fires mid-pick.
- **Live stat effect descriptions** (`statEffectLine` helper, near `abilityMod`): each stat row in character creation shows a one-liner of what that score currently does (`STR 14 → ATK +2 · HIT +10%`, `CON 16 → HP DIE +3 · ☠ RES 15%`, etc.), updating in real time as +/- buttons are clicked. The level-up panel reuses the same helper to show `NOW:` vs `NEXT:` lines so the +1 delta is visible before clicking.
- **Use item during exploration**: The idle/map screen now exposes the `🎒 USE ITEM` button (when inventory is non-empty), mirroring the in-combat selector. Items consumed out of battle don't trigger `runEnemyTurn`.
- **Wisdom heal scaling + WIS perk**: WIS mod is now added to every heal (HEALTH POTION, MEGA, SUPER, ELIXIR overheal). When a stat-up first pushes wisdom to 20+, `pickStat` routes to a new `wisperk` phase where the player chooses one of two perks: **Sage's Reservoir** (heals grant `wisMod` Temp HP, absorbed before real HP, cleared on battle start) or **Swift Cure** (heals roll `wisMod × 10%` to skip the enemy turn). Both perks continue to scale with further WIS picks.
- **Staggered multi-enemy attacks**: `runEnemyTurn`'s attack phase is now a chained `setTimeout` loop instead of a synchronous `for` over enemies. Each alive enemy's swing gets its own ~700 ms beat — own log, own arena animation, own player-HP commit, own shake. Functionally identical (both always attacked) but it now visibly reads as one-vs-many rather than a single batched frame.
- **No-skip floors**: stairs are no longer a separate room type. Boss kill transforms the boss tile into a stairs tile in place (`isBoss:false, descend:true, cleared:true`); shop unlocks as before but the floor persists so the player can revisit cleared tiles or shop before descending. Bosses are also pinned to `enemyKnown:true` — no surprise rounds on bosses.
- **Fog-of-war map**: 3×2 spatial grid replaces the old vertical room button list. `playerPos` state tracks the player's tile; tiles outside visited / adjacent stay foggy. A "HERE" detail strip below the map shows the current room and exposes `ENGAGE` / `DESCEND` for touch-friendly play.
- **Witch enemy** (`healsAllies:true`, waves 4–5). Heals a wounded ally for +3 HP instead of attacking; self-heals +2 if alone and below half HP. Creates a "kill the witch first" priority puzzle in multi-enemy rooms.
- **Multi-enemy combat in big rooms** (50% chance of 2 enemies). `enemies` array + `targetIdx` state; clickable enemy tiles in the arena pick the attack target. `runEnemyTurn` loops per-enemy through statuses, flee checks, then attacks. `ROOM_TYPES.big.enemyMod` dropped its `+HP` rider (the second enemy is the threat now).

## Pending Wishlist

_Empty for now._

## Known Balance Concerns

The user has flagged balance as "a bit broken" and explicitly deferred fixes. Things to watch when you do tune:

- **Bleed scaling**: per-swing bonus = `floor(weapon_avg + atkDmg)`. A Bleeding Spear (D10 avg 5.5) with STR 14 (+2 ATK) can apply bleed for `+10 dmg/swing × 10 turns` — single proc one-shots most mid-game enemies.
- **Multi-enemy rooms**: 2 foes attacking once each per round effectively doubles incoming damage. With the surprise bonus on top, low-WIS players can get blown up at the start of a big-room encounter.
- **Starting HP**: 4 HP (2 hearts) is fragile against any enemy with >2 base attack. The first few rooms before CON can scale up are a high-risk window.
- **Witch healing**: +3 HP per ally turn can exceed the player's per-turn DPS in early game, making big rooms unwinnable without crit luck.
- **Greatsword (2D6)** raw damage outclasses every other weapon: 2-12 + ATK is just better than D10 + 1 ATK from sharp dagger. Two-handed locking the offhand is the only counterweight.

## Testing Note

Toggle the **DEV** button before manual testing. Dev mode disables incoming damage, reveals all room types and enemies, and acts as if you have max WIS + INT (full room awareness + identify gating bypass). Disable dev mode when testing balance.
