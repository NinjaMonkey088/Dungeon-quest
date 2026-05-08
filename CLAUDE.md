# Dungeon Quest Project Prompt

Dungeon Quest is a single-file browser RPG built with React 18, Babel standalone, and inline CSS inside `index.html`. It is a retro pixel-style turn-based dungeon crawler inspired by classic Zelda, Rogue, and tabletop fantasy.

The game should feel fast, readable, charming, and arcade-like. Preserve the 8-bit visual language: dark dungeon palette, chunky bordered tile boxes, small tactical text in Press Start 2P, emoji-based characters/items, simple CSS keyframe animations, and punchy combat logs.

## Core Experience

The player chooses a room (small / big / bridge / stairs / shrine) from a dungeon map, fights a single enemy in that room, earns XP and gold, levels up, picks an attribute upgrade, eventually faces a boss at the end of the wave, and unlocks the shop. The dungeon clears at level 21.

Each combat is a turn-based duel: player swings (d20-to-hit, dice damage, optional double/triple strike, optional crit), then enemy swings (d20-to-hit, d3 base damage, optional poison/burn/bleed/stun proc, optional counter from the player). Status effects tick at the start of each enemy turn.

Keep the game playable directly from the HTML file unless a requested feature truly requires a build system.

## Mechanics Overview

- **Attributes** (D&D-style, default 10, mod = floor((score-10)/2)):
  - `strength` → +ATK and +5% HIT per mod
  - `dexterity` → +5% DODGE and +5% CRIT per mod
  - `constitution` → +2 MAX HP and +5% poison resist per +2 score
  - `wisdom` → +15% room awareness per mod (reveals room types on the dungeon map)
  - `charisma` → -5% shop prices per mod
  - `intelligence` → +15% identify per mod (shows full equipment stats in shop)
- **Level-up**: pick 1 attribute from the full STAT_POOL list. Half-max-HP heal on level up. XP needed = `level * 3`.
- **Perks** (`PERK_POOL`): At levels 5, 10, 15, and 20, pick one perk you haven't already taken. Choices: Double Strike (25% chance to swing twice), Iron Will (survive one killing blow at 1 HP), Armor Pierce (crits ignore enemy armor), Dual Wield (equip a second light weapon in offhand — both swing each turn).
- **Combat dice**: `dieIdx` indexes into `DICE_PROGRESSION` (D2 → 40D10). Player damage = `atkDmg + dice roll - enemy.armor`; crit doubles the dice roll. Enemy damage = `attack + d3 - player.armor + greedPenalty`.
- **Status effects** (all decay over turns, applied by specific enemies):
  - ☠ Poison — 1 dmg / turn for 3 turns. Resisted by CON.
  - 🔥 Burn — 2 dmg / turn for 3 turns.
  - 🩸 Bleed — +50% incoming damage for 3 turns. Applied by Cursed on crit.
  - 💫 Stun — skip your next turn (auto-runs another enemy turn).
- **Wave structure**: 5 rooms per wave, room 5 is a boss. Defeating a boss advances the wave, unlocks the shop, and reshuffles room types.
- **Bosses**: `WAVE_BOSSES` for waves 1–4 (King Bokoblin, Mummy Lord, Arch Wizzrobe, Iron Darknut). Waves 5+ cycle through `BOSS_DEFS` (Gleeok, Manhandla, Phantom Ganon, Ganon) with scaling and "ELDER" prefix at deep cycles.
- **Equipment**: 13 body slots (`weapon`, `offhand`, mask/helmet/glasses, amulet, arms/torso/cloak, hands, belt, leg, feet) + 5 ring slots. Weapons span 5 commons (dagger, shortsword, longsword, spear, greatsword) tagged `hand: 'light' | 'one' | 'two'`, plus 7 rares with procs/riders (sharp dagger, keen shortsword, venom dagger, flame brand, piercing longsword, bleeding spear, vampiric blade). The `offhand` slot holds shields (3 tiers) by default, or a second light weapon when Dual Wield is unlocked + main is light. Armor pieces, two cloak variants (Shadow vs Dodge tradeoff), and 7 ring types including Ring of Greed (+50% gold, +2 dmg taken).
- **Consumables**: HP potions, antidote (cures all status), elixir, and timed buff items (oils, whetstones, scrolls, smoke bomb, talisman, cloak) granting 1–3 battles of stat boosts.
- **Rest / Flee**: 75% chance to flee a battle. Flee or rest forfeits current XP and full-heals + clears all status.
- **Identify gating**: Equipment in the shop shows a generic category label ("VITAL GEAR" / "DEFENSIVE GEAR" / "COMBAT GEAR" / "ENCHANTED GEAR") until your `identify` (from INT) reaches 50%.
- **Awareness gating**: Room types on the map start hidden as `?` — WIS reveals them.

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
- More equipment and item synergies (follow `EQUIPMENT_DEFS` shape; special-case behavior via flags like `vampiric` handled in `handleAction`).
- Better balance across waves, bosses, and item costs.
- Clearer status effect feedback (icons in the header, on the player tag, in the stat panel, and via `*Pulse` keyframes on the player sprite).
- More satisfying boss encounters.
- New room types in `ROOM_TYPES` with `enemyMod` functions.
- Quality-of-life improvements for inventory, gear panel, and combat logs.
- Small animations or visual polish that does not make the UI noisy.

## Combat Flow — Read Before Editing

1. `handleAction("attack")` → calls `playerAttack(ep, enemy)` → applies vampiric heal → updates enemy HP → if alive, calls `runEnemyTurn(effectivePlayer, next)`.
2. `runEnemyTurn(p, e)` schedules a `setTimeout` (750ms). Inside the callback, in order:
   - Poison tick (1 dmg, decrement)
   - Burn tick (2 dmg, decrement)
   - Bleed decrement (effect already amplifies incoming damage below)
   - Archer flee check (`fleesWhenLow`)
   - `enemyAttack(curP, e)` → returns `{dmgToPlayer, counterDmg, results, gotCrit, didPoison, didBurn, didBleed, didStun}`
   - Bleed amplifies dmg (×1.5) before applying
   - Apply status effects (`setPoison/setBurn/setBleed/setStun`)
   - Counter damage to enemy (may resolve victory)
   - Damage to player; iron will check; lose check; phase back to "player" (or recursive `runEnemyTurn` if stunned)
3. `resolveVictory(defeatedEnemy)` → gold (+greed multiplier) → XP → maybe level up → if boss, advance wave + open shop → either `triggerLevelUp` or `setPhase("win")`.

Be careful with React state updates during combat turns. Status-effect counters (`poison`, `burn`, `bleed`) are read from closure inside `runEnemyTurn` — that's intentional and represents the "value at turn start". Do not break this pattern unless reworking the timeout chain.

## Testing Note

Toggle the **DEV** button before manual testing. Dev mode disables incoming damage and reveals all room types, so you can reliably reach the end. Disable dev mode when testing difficulty or balance changes.
