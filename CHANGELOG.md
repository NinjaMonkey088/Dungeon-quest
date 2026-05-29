# Changelog — Dungeon Quest

Historical record of shipped feature additions and balance tunings. Newest at top of each section. See [CLAUDE.md](CLAUDE.md) for current mechanics, [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md) for spell design, [STORY.md](STORY.md) for narrative.

---

## Feature additions

- **Trap rooms — disguised + sub-typed** — trap tiles now render as a random non-big fight type (small/bridge/shrine) with full fake enemies. Two independent awareness rolls drive the deception: `trapAware` (flips the tile + HERE to `⚠ TRAP!`) and `enemyKnown` (shows the fake enemies). Each trap rolls a `trapEffects` array: primary uniform from `TRAP_EFFECTS` (spike/status/mimic/ambush, status expanding into poison/burn/bleed), 25% chance of a different secondary on top. `triggerTrap` now applies each effect in sequence (ambush sorted last); spike scales hardest with floor, status chip damage and mimic gold-theft scale gently, ambush scales via enemy tier and gets +5 SPD via `enemyKnown:false`.
- **Boss minions on deep floors** — `bossMinionCount(floorNum)` ramps the entourage: solo on f1–9, +1 on f10–13, +2 on f14–17, +3 on f18–20. `makeFloorMap` now builds the boss room as `[pickBoss(...), ...bossMinions]`. Minions are pulled via `pickEnemy(level)` from `WAVES`, with the same one-Witch-per-room cap as fight rooms (a single Witch CAN spawn alongside the boss — "kill the healer first" decision). Boss `enemyKnown:true` preserved → boss tile still never surprises. Minion XP/gold contributes via the normal `resolveVictory` aggregation.
- **Big rooms scale with depth** — `bigRoomCountRange(floorNum)` ramps the big-room foe count from 2 on f1 up to 5–6 on f20 (f1 → 2, f2–4 → 2–3, f5–7 → 2–4, f8–10 → 3–4, f11–13 → 3–5, f14–16 → 4–5, f17–19 → 4–6, f20 → 5–6, uniform within range). `rollEnemyCount` now takes `(type, floorNum)`. Non-big fight rooms still 35% chance of 2 else 1. Witch-spawn cap of 1 per room still applies regardless of size.
- **Reservoir + Vital Surge stack** — `endRound`'s Vital Surge block now also triggers Sage's Reservoir if the player has both perks: each tick grants `wisMod` Temp HP (capped at `maxHp`), logged as `✦ SAGE'S RESERVOIR! +N TEMP HP` after the `✦ VITAL SURGE — +N HP` line. Mirrors the existing heal-item Reservoir path.
- **End-of-round on the winning round** — extracted `endRound(p, es, tHp, onDone)` (right above `runQueueFrom`) that runs enemy DOTs → player DOTs → Vital Surge regen, and called it from THREE places: `runQueueFrom`'s queue-exhaust, `handleAction`'s victory check, and `handleCastSpell`'s victory check. Moved the existing DOT ticks out of `beginRound`'s top-of-round block (Vital Surge moved out of `runQueueFrom`'s end-of-queue block). Player DOT damage still kills on the winning round — death overrides the win. Flee checks stay at top of round.
- **UI softening pass** — swapped Press Start 2P for rounded **Fredoka**, warm wood-gradient `body`, `TileBox` → rounded beveled wooden panel, `Btn`/`BTN_VARIANTS` → rounded candy buttons (`.dq-btn` hover/press), and rounded/warmed secondary chrome (floor map + tiles, XP bar, combat arena box, create-screen controls, info strips, DEV button).
- **Camp anytime** — `🏕 MAKE CAMP` on the idle/map screen (not only the post-combat `win` screen); both route through `enterRest`.
- **Gear-modal stats rebuilt** — the `equipOpen` modal's stat block is now a readable cartoon card layout (combat chips + one card per attribute using `STAT_FULL` names + `statEffectLine`, plus perk/special badges), replacing the old cramped raw-`STR/DEX/CON…` text wall (which also broke the no-D&D naming rule).
- **Per-round initiative** — combat re-rolls `speed + d10` for the player + each enemy every round and interleaves turns (`beginRound`/`runQueueFrom`/`enemyAct`/`resumeAfterPlayer` via `combatRef`). Replaced the old `runEnemyTurn` player-phase/enemy-phase split. Log shows `⚡ ROUND N — 🧝19 ▸ 👺9`.
- **Cartoon rename** — stats MIGHT/ZIP/GUTS/SAVVY/CHARM/WITS (`STAT_LABEL`/`STAT_FULL`/`statLine`); spells ZAP-ZAP/HOTFOOT/BOO-BOO FIX/THERE-THERE/SICK BURN/BAT EYES; "shrug-off" instead of saves. Internal keys unchanged.
- Dual-wield equip flow rebuilt — explicit `→ OFFHAND` button via `canEquipToOffhand`. `buyItem` + `handleEquipFromInventory` + `canEquipItem` all accept `targetSlot` arg.
- All 6 stats have score-20 milestone perks (`STAT_PERKS`). Generic `pickStatPerk` + `statperk` phase. Legacy `wisperk` flow preserved.
- Map scales with depth (`getFloorLayout`). `FloorMap` derives grid dims from rooms.
- Game ends at floor 20 (`gamewon` from `resolveVictory`).
- Character creation point-buy (27 pts, escalating cost, 18 cap).
- `statEffectLine` real-time previews in create + level-up panels.
- `🎒 USE ITEM` in idle/map screen (out-of-combat, no enemy turn).
- WIS heal scaling (`+wisMod` on every heal) + WIS perks (Reservoir, Swift).
- Boss kill transforms tile to stairs (no separate stairs room type).
- Fog-of-war map + HERE strip replacing vertical room buttons.
- Witch (`healsAllies`) added to waves 4–5.
- Multi-enemy combat (`enemies[]` + `targetIdx`, big-room 50% double).

---

## Balance tuning

- Witch on 2-turn `healCooldown` after each heal.
- Witches never spawn alone or two-to-a-room (`pickEnemy({allowHealer})` filter).
- Light weapons buffed: Dagger +10% crit, Shortsword +5% crit.
- Offhand penalty 50% → 30% (offhand swings do 70%).
- Surprise rounds gated to floor 3+.
