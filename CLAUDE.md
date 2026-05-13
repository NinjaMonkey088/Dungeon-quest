# Dungeon Quest Project Prompt

Dungeon Quest is a single-file browser RPG built with React 18, Babel standalone, and inline CSS inside `index.html`. It is a retro pixel-style turn-based dungeon crawler inspired by classic Zelda, Rogue, and tabletop fantasy.

The game should feel fast, readable, charming, and arcade-like. Preserve the 8-bit visual language: dark dungeon palette, chunky bordered tile boxes, small tactical text in Press Start 2P, emoji-based characters/items, simple CSS keyframe animations, and punchy combat logs.

## Core Experience

The player explores a persistent dungeon map (small / big / bridge / stairs / shrine rooms), fights enemies in rooms, earns XP and gold, levels up, picks an attribute upgrade or milestone perk, and eventually chooses when to challenge the boss room to advance to the next floor. The dungeon clears at level 21.

Each combat is a turn-based duel: player swings (d20-to-hit, dice damage, optional double/triple strike, optional crit, optional weapon procs), then enemy swings (d20-to-hit, d3 base damage, optional poison/burn/bleed/stun proc, optional counter from the player). Status effects tick at the start of each enemy turn.

Keep the game playable directly from the HTML file unless a requested feature truly requires a build system.

## Mechanics Overview

### Attributes (D&D-style, default 10, mod = floor((score-10)/2))
- `strength` → +ATK and +5% HIT per mod
- `dexterity` → +5% DODGE, +5% CRIT, and +SPD per mod (speed determines turn order — see Planned Features)
- `constitution` → +2 MAX HP and +5% poison resist per +2 score (applied retroactively via `hitDice` history)
- `wisdom` → +15% room awareness per mod (reveals room types and enemies on the map; high WIS prevents surprise — see Planned Features)
- `charisma` → -5% shop prices per mod
- `intelligence` → +15% identify per mod (shows full equipment stats in shop)

### HP System
- `hitDice` is a per-level d8 roll history. Level 1 always starts with `[8]`.
- Each level-up rolls a new d8 and appends to `hitDice`.
- Effective `maxHp = floor((sum(hitDice) + conMod × hitDice.length) / 2) + equipmentBonus`
- CON changes apply retroactively to all past levels.
- Equipment `maxHp` bonuses are summed in `applyEquipment` before `applyAbilityStats` adds the CON-scaled dice result.

### Level-up & Perks
- On each level-up, pick 1 attribute from `STAT_POOL`. Half-max-HP heal on level up. XP needed = `level * 3`.
- Every 5th level (levels 5, 10, 15, 20), a **Milestone Perk** screen appears first (`phase = "perk"`). Pick from `PERK_POOL` — each perk is one-shot and removed from future offers after being picked.
- **PERK_POOL** perks:
  - `doubleStrikeChance` → ★ Double Strike: 25% chance to attack twice
  - `ironWill` → ★ Iron Will: survive a killing blow with 1 HP once per battle
  - `armorPierce` → ★ Armor Pierce: crits ignore all enemy armor
  - `dualWield` → ★ Dual Wield: equip two light weapons, both strike independently each attack

### Weapon System
- Weapons have a `hand` field controlling slot rules:
  - `'light'` — light 1H; dual-wieldable (can also go in offhand with `dualWield` perk)
  - `'one'` — standard 1H; compatible with shields in offhand
  - `'two'` — 2H; **blocks the offhand slot entirely** (shield or weapon)
- Base weapons (in `EQUIPMENT_DEFS`, slot `"weapon"`): Dagger (D4 light), Shortsword (D6 light), Longsword (D8 one), Spear (D10 two), Greatsword (2D6 two)
- Rare weapons (`rare: true`, shop bias 25%): Sharp Dagger, Keen Shortsword, Venom Dagger (poison proc), Flame Brand (burn proc), Piercing Longsword (pierce proc), Bleeding Spear (bleed proc), Vampiric Blade (heal 2 HP on crit)
- Weapon **procs** are defined in `item.procs = { poisonChance, burnChance, bleedChance, pierceChance }`. Procs only trigger on hit (dmg > 0).
- Shields (slot `"offhand"`): Buckler (+1 ARM +5% DOD), Iron Shield (+2 ARM), Tower Shield (+3 ARM −5% DOD). Incompatible with 2H weapons.

### Dual Wield
- Requires the `dualWield` perk AND a `hand: 'light'` main weapon equipped.
- `applyEquipment` sets `out.weapons = [main, offhand]` when both are light weapons.
- `playerAttack` iterates `p.weapons`, each swinging with its own die and procs. An "OFFHAND STRIKE!" log line precedes the second swing.
- `canEquipItem` gates offhand weapon purchases: requires `dualWield` perk + light main weapon; 2H main blocks all offhand.

### Combat Dice
- `dieIdx` indexes into `DICE_PROGRESSION` (D2 → 40D10, ~170 entries).
- Player damage = `atkDmg + dice roll − enemy.armor`; crit doubles the dice roll (both rolls printed in verbose/dev mode).
- Enemy damage = `attack + d3 − player.armor + greedPenalty`.
- `verbose=true` (enabled in devMode) adds dice detail to log lines.

### Status Effects (all decay over turns)
- ☠ **Poison** — 1 dmg / turn for 3 turns. Applied by Manhandla boss, Ganon boss, or Venom Dagger proc. Resisted by CON (`poisonResist`).
- 🔥 **Burn** — 2 dmg / turn for 3 turns. Applied by Keese enemy (`burnsOnHit`) or Flame Brand proc.
- 🩸 **Bleed** — +50% incoming damage for 3 turns. Applied by Cursed enemy on crit (`bleedsOnCrit`) or Bleeding Spear proc. Bleed amplifies damage dealt **to the bleeder** (enemy or player).
- 💫 **Stun** — skip your next turn (auto-runs another enemy turn). Applied by Like-Like (`stunsOnHit`, 30% chance).
- Enemy status ticks happen **before** the enemy attacks in `runEnemyTurn` (enemy poison → enemy burn → enemy bleed decrement). Player status ticks follow immediately after (player poison → player burn → player bleed decrement), also before the enemy attack roll.

### Enemies
Full list in `ENEMY_DEFS` (name → niche):
- `MOBLIN` (Fodder), `BOKOBLIN` (Accurate), `GIBDO` (Tank), `LIZALFOS` (Evasive)
- `WIZZROBE` (Glass Cannon), `PATRA` (Armored), `DARKNUT` (Heavy Armor)
- `PEAHAT` (Swift), `STALFOS` (Deadeye)
- `ARCHER` (Skirmisher) — `fleesWhenLow`: flees at ≤30% HP (35% chance), grants half reward
- `BERSERKER` (Berserker) — `enragesWhenLow`: at ≤50% HP gains +3 ATK and `enraged` flag (shown in UI)
- `CURSED` (Cursed) — `bleedsOnCrit`: applies 3-turn bleed on critical hits
- `KEESE` (Fire Bat) — `burnsOnHit`: applies 3-turn burn on every hit
- `LIKE_LIKE` (Stunning) — `stunsOnHit`: 30% chance to stun on hit

Enemy flags also available: `poisonOnHit`, `immuneToCrit`.

### Wave Structure
- 5 rooms per wave (WAVE_LENGTH = 5), room 5 is always a boss.
- Defeating a boss: advance wave, `wavePos` resets to 0, shop unlocks.
- Wave enemy pools (weighted) defined in `WAVES[0..4]`; enemies beyond wave 5 keep using wave 5 pool with scaling.
- **WAVE_BOSSES** (waves 1–4): King Bokoblin, Mummy Lord, Arch Wizzrobe, Iron Darknut.
- **BOSS_DEFS** (waves 5+, cycling): Gleeok, Manhandla (poison), Phantom Ganon (immuneToCrit), Ganon (poison + immuneToCrit). Each cycle past the list adds HP/ATK/gold scaling and "ELDER" prefix after 2+ repeats.

### Enemy Scaling
- `pickEnemy(level)` selects from `WAVES[clamp(level-1, 0, 4)]` weighted pool.
- `scaleHit` ramps enemy `hitChance` toward 100 by level 15, then +1/level above 15 (counters player dodge accumulation).
- Enemies beyond wave 5's pool gain +1 maxHp per over-level, +1 ATK per 5 over-levels.

### Room Types
`ROOM_TYPES` (small, big, bridge, stairs, shrine) each provide an optional `enemyMod(e)` function applied via `applyRoomToEnemy`:
- **Small Room**: standard tight fight, no modifier
- **Big Room**: enemy +2 HP, +1–3 gold; has a chance to hold multiple enemies simultaneously (see Planned Features)
- **Bridge**: enemy +15% dodge, +2 gold max
- **Stairs**: leads to a new dungeon floor (see Planned Features) — currently grants enemy +1 XP
- **Shrine**: enemy −10% hit (min 30), −1 gold min

Room and enemy identity on the map are hidden unless the player's `awareness` (from WIS) or `devMode` reveals them.

### Shop
- Unlocked after each boss kill.
- Items: 3 random consumables + up to 2 equipment pieces (biased 75% common / 25% rare via `pickShopEquipment`).
- Equipment already equipped in the same slot is filtered out (weapons/shields can be replaced; other slots only offer once per slot).
- Dual-wield aware: if `dualWield` perk is active and player has a light main weapon, `pickShopEquipment` re-tags ~40% of light weapon picks as offhand variants.
- `canEquipItem` greys out and shows reason text for illegal purchases (2H blocks offhand, offhand weapon requires dual-wield).
- **Identify gating**: equipment shows a generic category ("COMBAT GEAR", "DEFENSIVE GEAR", etc.) until `identify ≥ 50%` (from INT). `describeItem(item, p)` handles this.
- **Shop discount**: `getShopCost(item, p)` applies `p.shopDiscount` (from CHA) as a percentage reduction, minimum cost 1.

### Consumables
HP potions (4/8 HP), Elixir (full HP + cure poison), Antidote (cures all status), and timed buff items (oils, whetstones, scrolls, smoke bomb, talisman) granting 1–3 battles of stat boosts. Buffs tracked in `tempBuffs` as `{ stat, val, battles }` and decremented on each `startBattle`.

### Rest / Flee
- **Flee**: 75% chance to succeed. On success, forfeits all current XP, calls `enterRest()`. On fail, triggers an enemy turn immediately.
- **Rest (Make Camp)**: forfeits current XP, fully heals, clears all status effects, returns to `"idle"`.

### Game Stats & High Scores
- `gameStats` state tracks: `biggestHit`, `biggestHitTaken`, `killCounts` (by enemy name), `itemsUsed` (by item label). Displayed on the game-won screen.
- High scores (top 5 by gold) are persisted to `localStorage` as `"dq_scores"` via `saveHighScore`. Shown on the title screen.

## Technical Constraints

- Keep the project dependency-light. No build step. React + ReactDOM + Babel from CDN.
- Prefer plain React state/hooks over adding frameworks.
- Keep changes scoped and understandable. Always read `index.html` before editing — do not regenerate the whole file.
- Preserve existing mechanics unless fixing a bug or improving balance.
- When adding new content, follow the existing data-driven patterns:
  - `ENEMY_DEFS`, `WAVES`, `WAVE_BOSSES`, `BOSS_DEFS`
  - `SHOP_CONSUMABLES`, `EQUIPMENT_DEFS`
  - `STAT_POOL`, `PERK_POOL`
  - `BODY_SLOTS`, `ROOM_TYPES`, `STAT_LABEL`, `DICE_PROGRESSION`
- Effective player stats are computed every render via `getEffectivePlayer(player, equipment, tempBuffs)` which composes `applyEquipment → applyAbilityStats → applyBuffs`. Do NOT mutate `player` with equipment/buff bonuses.
- `ep` = effective player with temp buffs (used during combat). `idlePlayer` = effective player without buffs (used for shop pricing and gear display). `roomPlayer` = `idlePlayer` with boosted awareness/identify in devMode (used for room/shop text gating — never stored to state).

## UI Components

| Component | Purpose |
|---|---|
| `App` | Root component; all state lives here |
| `TileBox` | Bordered tile container; `highlight` adds gold glow |
| `Btn` | Styled button; `variant` keys into `BTN_VARIANTS` |
| `Hearts` | Half-heart HP display |
| `BodyLayout` | 5×8 grid of body slot icons; `onSelect` makes slots clickable |
| `EquipDisplay` | Compact horizontal equipment icon strip |
| `GearDetail` | Slot detail panel (item name, desc, stats, cost) |
| `DungeonMap` | Wave progress grid + room choice icons |
| `ItemButtons` | Renders consumable use buttons |

`BTN_VARIANTS`: attack, flee, neutral, stat, gold, shop.
`LOG_COLORS` types: sys, player, enemy, crit, warn, level, counter, dodge, miss, gold.

## CSS & Animations

All CSS is in the `CSS` string constant injected via `<style>{CSS}</style>`. Keyframes:
- `blink`, `shake` — general utility
- `playerSwing`, `enemySwing` — attack lunge animations
- `getHit`, `getCrit` — damage-received flashes
- `enemyDodge`, `playerDodge` — dodge sidestep animations
- `bossGlow` — persistent golden glow on boss sprite
- `missSwing` — sword swipe on a miss
- `poisonPulse`, `burnPulse`, `stunPulse` — continuous status-effect tints on the player sprite

Animations are driven by `arenaAnim = { player, enemy }` state set in `triggerArenaAnim(p, e, dur)` and cleared after `dur` ms. Status-effect pulses are always-on when the counter > 0, overriding arena idle.

## Game Phase State Machine

`phase` drives the UI branch rendered:
- `"title"` → start screen
- `"idle"` → dungeon map / room selection
- `"player"` → player action (attack, use item, flee)
- `"enemy"` → waiting for the 750ms enemy-turn setTimeout
- `"perk"` → milestone perk pick (every 5th level)
- `"levelup"` → attribute stat pick
- `"win"` → post-battle victory options
- `"shop"` → merchant screen
- `"rest"` → campfire rest screen
- `"lose"` → death screen
- `"gamewon"` → victory / stats screen

## Design Direction

- Retro fantasy dungeon tone.
- UI should stay compact and game-like, not become a landing page.
- Buttons should be clear, chunky, and readable (use existing `Btn` + `BTN_VARIANTS`).
- Use emoji as sprites/icons where appropriate.
- Combat log language should be short, dramatic, and easy to scan (use existing `LOG_COLORS` types: sys, player, enemy, crit, warn, level, counter, dodge, miss, gold).
- Avoid cluttering the screen with long explanations — show numbers tersely with `STAT_LABEL` abbreviations (ATK, ARM, HIT, DOD, CRT).

## When Improving The Game

Good additions include:

- More enemies with distinct mechanics (use existing flags pattern: `poisonOnHit`, `burnsOnHit`, `bleedsOnCrit`, `stunsOnHit`, `enragesWhenLow`, `fleesWhenLow`, `immuneToCrit`).
- More equipment and item synergies (follow `EQUIPMENT_DEFS` shape; `procs` object for weapon on-hit chances; special flags like `vampiric` handled explicitly in `handleAction`).
- Better balance across waves, bosses, and item costs.
- Clearer status effect feedback (icons in the header, on the player tag, in the stat panel, and via `*Pulse` keyframes on the player sprite).
- More satisfying boss encounters.
- New room types in `ROOM_TYPES` with `enemyMod` functions.
- Quality-of-life improvements for inventory, gear panel, and combat logs.
- Small animations or visual polish that does not make the UI noisy.

## Combat Flow — Read Before Editing

1. `handleAction("attack")` → calls `playerAttack(ep, enemy)` → applies vampiric heal (if weapon has `vampiric` and a crit landed) → updates enemy HP → if enemy has procs (poison/burn/bleed), applies them → if enemy `enragesWhenLow` and crosses threshold, sets `enraged` flag → if alive, calls `runEnemyTurn(effectivePlayer, next)`.
2. `runEnemyTurn(p, e)` schedules a `setTimeout` (750ms). Inside the callback, in order:
   - Enemy poison tick (1 dmg, decrement) — may resolve victory
   - Enemy burn tick (2 dmg, decrement) — may resolve victory
   - Enemy bleed decrement (only decrements; damage bonus applied on player swings against a bleeding enemy)
   - **Player** poison tick (1 dmg, decrement) — may cause lose
   - **Player** burn tick (2 dmg, decrement) — may cause lose
   - **Player** bleed decrement
   - Archer flee check (`fleesWhenLow`, 35% pct at ≤30% HP) — resolves with half reward
   - `enemyAttack(curP, e)` → returns `{dmgToPlayer, counterDmg, results, gotCrit, didPoison, didBurn, didBleed, didStun}`
   - Bleed amplifies `dmgToPlayer` (×1.5 if `bleed > 0`, devMode zeroes it)
   - Apply status effects to player (`setPoison/setBurn/setBleed/setStun`)
   - Counter damage to enemy (may resolve victory)
   - Damage to player; iron will check; lose check; phase back to `"player"` (or recursive `runEnemyTurn` after 1.1s if stunned)
3. `resolveVictory(defeatedEnemy)` → gold (+greed multiplier if Ring of Greed equipped) → XP → maybe level up (roll d8 hitDice, half-HP heal) → if boss, advance wave + open shop → either `triggerLevelUp` or `setPhase("win")`.

Be careful with React state updates during combat turns. Status-effect counters (`poison`, `burn`, `bleed`) are read from closure inside `runEnemyTurn` — that's intentional and represents the "value at turn start". Do not break this pattern unless reworking the timeout chain.

## Planned Features & Design Goals

These are confirmed design directions for the game. When implementing any of them, read this section first and implement against the intent described here rather than the current placeholder behaviour.

---

### 1. Persistent Dungeon Map with Visible Room Sizes

**Goal**: The map should show all rooms on the current floor laid out visually, with size clearly communicated. Choosing the small room should not consume the big room — both remain available.

**How it should work**:
- Rooms are generated once per floor and stored in state. They do not regenerate when the player returns to the map.
- Each room on the map displays its size/type icon prominently (▦ small, ▣ big, ═ bridge, ⇣ stairs, ✚ shrine, ▰ boss).
- Rooms have two states: `cleared` (enemy defeated, re-entry is safe/empty) and `uncleared` (enemy still present).
- The player can freely enter any uncleared room, defeat the enemy, then return to the map and enter a different uncleared room. No room is locked out by entering another.
- Cleared rooms are visually distinct (dimmed, ✓ marker) but still enterable — re-entering just shows an empty room with no combat.
- The map should render all rooms for the floor simultaneously, not only the 3 current choices. The `DungeonMap` component needs to show a spatial layout rather than a simple linear progress bar.

**Data shape to add per room** (extend current room object):
```js
{ id, type, icon, label, desc, enemy, enemyMod, cleared: false, known, enemyKnown }
```

---

### 2. Stairs → New Dungeon Floor

**Goal**: Entering a stairs room transitions to a deeper floor rather than just granting +1 XP.

**How it should work**:
- When the player enters a stairs room and clears the enemy (or the room is already cleared), a "DESCEND?" prompt appears.
- Descending generates a fresh set of rooms for the new floor (new enemies, new layout). Floor number increments.
- Enemies on deeper floors use the next wave tier / higher scaling — stairs are the primary difficulty ramp mechanism beyond boss waves.
- The shop does **not** auto-unlock on a floor transition; only boss kills unlock the shop.
- Floor number is displayed in the HUD alongside wave number.
- Stairs should be less common than other room types to prevent trivial skipping of boss rooms.

---

### 3. Boss Room: Optional Entry, Not Forced

**Goal**: The boss room is visible on the map but the player decides when to fight it. It should not trigger automatically.

**How it should work**:
- The boss room always appears on the floor map (last position or marked with ▰ BOSS).
- The player can clear other rooms first to level up and heal, then choose to enter the boss room when ready.
- Entering the boss room shows a confirmation prompt: "THE GUARDIAN WAITS. ENTER?" before starting combat.
- The boss room cannot be skipped to reach the next floor — descending via stairs or advancing requires either defeating the boss or finding another path forward (TBD).
- `wavePos` should not auto-increment just from entering the boss room; the boss fight itself must complete.

---

### 4. Big Rooms: Multi-Enemy Encounters

**Goal**: Big rooms have a chance (~40%) to contain two enemies that fight simultaneously.

**How it should work**:
- When a big room spawns with `multiEnemy: true`, two separate enemy objects are generated via `pickEnemy`.
- Both enemies are tracked in state (e.g. `enemies: [enemyA, enemyB]` instead of a single `enemy`).
- **Player turn**: the attack action UI shows target buttons — the player picks which enemy to hit. Alternatively, attacks hit a random living enemy (simpler first pass).
- **Enemy turn**: each living enemy takes an independent turn, rolling attack separately. Both attack the player in sequence within the same 750ms window.
- Status effects (poison, burn, bleed) are tracked per-enemy.
- Killing one enemy does not end the fight — the surviving enemy continues.
- XP and gold are awarded per enemy on death, with a multi-enemy bonus (e.g. +1 gold for each enemy beyond the first).
- Reward on full clear: standard resolveVictory called once all enemies are dead.
- **Combat log** prefixes each enemy action with its name to distinguish attackers.
- The arena should display both enemy sprites side by side when multiple enemies are alive.

**Implementation note**: This is the most complex planned feature. The current single `enemy` state and `runEnemyTurn(p, e)` chain must be extended to support an `enemies[]` array. Keep the single-enemy path working; only big rooms with the flag trigger the multi-enemy path.

---

### 5. Speed Stat & Turn Order

**Goal**: Add a `speed` stat derived from DEX that determines who acts first in combat. Currently the player always attacks first.

**How it should work**:
- `speed` is computed in `applyAbilityStats`: base 10 + DEX mod × 2 (tentative formula).
- At `startBattle`, compare `ep.speed` vs `enemy.speed`. If the enemy is faster, the first turn is an enemy turn (`runEnemyTurn` fires immediately before the player gets input).
- Display turn order in the combat HUD: "YOU GO FIRST" vs "ENEMY STRIKES FIRST!".
- Enemies have a `speed` field in `ENEMY_DEFS` (default 10; fast enemies like PEAHAT, LIZALFOS should be higher).
- Speed ties go to the player.
- Add `speed` to `STAT_LABEL` as `SPD`.

**Surprise condition**:
- If the player's WIS was **not** high enough to reveal the enemy's identity (`enemyKnown === false` on the room choice), the enemy gains a surprise speed bonus (+5 to +10 SPD) at the start of that specific battle.
- This represents being caught off-guard. The combat log shows "⚡ SURPRISE! ENEMY STRIKES FIRST!" when this triggers.
- WIS investment is therefore doubly rewarded: reveals enemies on the map AND prevents getting surprised.
- The surprise bonus only applies to the first round; subsequent turns use base speeds.

---

### 6. Old Weapon Moves to Inventory on Replace

**Goal**: Buying a new weapon from the shop should send the previously equipped weapon to the player's bag, not discard it silently.

**How it should work**:
- In `buyItem`, when `item.slot === 'weapon'` (or `'offhand'`) and `equipment[item.slot]` is already set, move the current weapon into `inventory` before equipping the new one.
- This applies to shields/offhand items too.
- Inventory items are usable from the bag (for consumables) or can be re-equipped via the gear panel (for equipment — future feature: gear panel swap).
- The shop should offer a **Sell** option for inventory equipment: sell price = 50% of base `item.cost` (before any discount), rounded down, minimum 1 gold.
- Selling is only available at the shop, not from the inventory mid-battle.
- The combat log should note "⚔ OLD WEAPON STORED IN BAG" when a weapon is displaced.

---

### 7. Double Strike Early Exit (Bug Fix Goal)

**Goal**: If the first attack swing kills the enemy, skip the double/triple strike roll entirely. Currently the game can roll extra swings against a 0-HP enemy.

**How it should work**:
- In `playerAttack`, after accumulating `total` from the primary swing(s), check if `total >= e.hp` before the double-strike roll.
- If the enemy would already be dead, return early without the extra swings.
- This is a small code fix in the double-strike block inside `playerAttack`:
  ```js
  // Only roll double strike if enemy would survive the primary swing
  if (total < e.hp && (dbl >= 100 || (dbl > 0 && pct(dbl)))) { ... }
  ```
- Triple strike follows the same rule — check after the double swing if the enemy is still alive.

---

## Testing Note

Toggle the **DEV** button before manual testing. Dev mode:
- Disables all incoming damage (`dmgToPlayer = 0`)
- Forces `verbose=true` in combat (dice detail shown in logs)
- Reveals all room types and enemy identities on the map
- Boosts `roomPlayer` awareness/identify to near-max so room descs and shop stats show

Disable dev mode when testing difficulty or balance changes.
