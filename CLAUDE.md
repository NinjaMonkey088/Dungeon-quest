# Dungeon Quest Project Prompt

Dungeon Quest is a single-file browser RPG built with React 18, Babel, and inline CSS inside an HTML file. It is a retro pixel-style turn-based dungeon crawler inspired by classic fantasy adventure games.

The game should feel fast, readable, charming, and arcade-like. Preserve the 8-bit visual language: dark dungeon colors, chunky buttons, small tactical text, emoji-based characters/items, simple animations, and punchy combat logs.

## Core Experience

The player fights enemies in waves, earns XP and gold, levels up, chooses stat upgrades, unlocks milestone powers, buys consumables and equipment, manages status effects, and tries to survive until the dungeon is cleared.

Keep the game playable directly from the HTML file unless a requested feature truly requires a build system.

## Technical Constraints

- Keep the project dependency-light.
- Prefer plain React state/hooks over adding frameworks.
- Keep changes scoped and understandable.
- Avoid large rewrites unless specifically requested.
- Preserve existing mechanics unless fixing a bug or improving balance.
- When adding new content, follow the existing data-driven patterns:
  - `ENEMY_DEFS`
  - `SHOP_CONSUMABLES`
  - `EQUIPMENT_DEFS`
  - `STAT_POOL`
  - `THRESHOLD_UNLOCKS`
  - `WAVES`
  - boss definitions

## Design Direction

- Retro fantasy dungeon tone.
- UI should stay compact and game-like, not become a landing page.
- Buttons should be clear, chunky, and readable.
- Use emoji as sprites/icons where appropriate.
- Combat log language should be short, dramatic, and easy to scan.
- Avoid cluttering the screen with long explanations.

## When Improving The Game

Good additions include:

- More enemies with distinct mechanics.
- More equipment and item synergies.
- Better balance across levels and waves.
- Clearer status effect feedback.
- More satisfying boss encounters.
- Quality-of-life improvements for inventory, gear, and combat logs.
- Small animations or visual polish that does not make the UI noisy.

## Important

Before making changes, understand how combat flow moves through:

- `handleAction`
- `playerAttack`
- `runEnemyTurn`
- `enemyAttack`
- `resolveVictory`

Be careful with React state updates during combat turns, especially delayed enemy turns and status effects.

be sure to select the dev button before running your tests so you can reliable reach the end unless we are running a test on difficullty or balance
