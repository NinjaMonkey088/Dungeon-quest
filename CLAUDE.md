# Dungeon Quest — `index.html` Reference

Single-file React 18 (Babel standalone, CDN) turn-based dungeon crawler. No build step. **Soft, cartoonish, whimsical** vibe — rounded **Fredoka** font, warm wood-toned panels, beveled candy buttons, emoji sprites, bouncy CSS, goofy combat banter. (Originally an 8-bit/pixel look with Press Start 2P; softened toward a wooden cartoon game-UI-kit aesthetic.) All code lives in `index.html` (~2800 lines). Edit it in place — never regenerate.

**Game goal**: clear 20 floors. Floor-20 boss kill → `setPhase("gamewon")`.

> **Original game — NOT D&D.** Dungeon Quest is its own silly little world. Never reference *Dungeons & Dragons*, its rulebooks, class names, or any trademarked/Product-Identity content in code, UI, or docs. Game *mechanics* (a d20 roll, an ability modifier, a half-scale HP pool) are generic and fair to use; *names* are always reskinned to original cartoon terms (see [Naming convention](#naming-convention--cartoon-reskin-not-dd)). When in doubt, make it sillier and more original — that is also the safest legal direction.

**Companion docs:** [BESTIARY.md](BESTIARY.md) (enemies, bosses, spawns) · [INVENTORY.md](INVENTORY.md) (weapons, armor, consumables, dice) · [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md) (spells, shrug-off math, half-scale rule) · [STORY.md](STORY.md) (Baron arc, voice, beats, illiteracy variants) · [CHANGELOG.md](CHANGELOG.md) (shipped features & balance tunings).

---

## File map — `index.html`

| Lines | Section |
|------|---------|
| 1–18 | `<head>`, CDN scripts, base `<style>` |
| 20–339 | **Constants & data tables** (PLAYER_BASE, pools, enemy/boss defs, equipment, dice, room types) |
| 341–569 | **Helpers** (roll/clamp/pct, abilityMod, applyEquipment/AbilityStats/Buffs, getEffectivePlayer, item helpers) |
| 574–685 | **Floor / enemy generation** (getFloorLayout, makeFloorMap, pickEnemy, pickBoss, scaleHit) |
| 687–869 | **Combat resolvers** (playerAttack, enemyAttack, enemySpellAttack) — pure functions, return result shapes |
| 871–1107 | **UI primitives** (Hearts, TileBox, Btn, EquipDisplay, BodyLayout, FloorMap, GearDetail, ItemButtons) |
| ~1154–1173 | Inline `CSS` const — Fredoka `@import`, warm wood-gradient `body`, `.dq-btn` hover/press classes, keyframes (blink, shake, hit, crit, dodge, attack, status pulses, pop) |
| 1131–end | **`App`** root component — state, handlers, render |
| 1131–1200 | App state declarations + derived `ep` / `enemy` / `XP_NEEDED` |
| 1202–1300 | Level-up, perk-pick, spell-pick handlers |
| 1303–1370 | Creation + rest/descend handlers |
| 1372–1442 | `handleRoomEnter`, `startBattle` |
| 1444–1650 | `handleAction` (attack/flee), `handleCastSpell` |
| 1651–1759 | `handleUseItem`, `handleEquipFromInventory`, dual-wield helpers, `handleSellItem` |
| ~1600–1830 | **Initiative rounds**: `beginRound` / `runQueueFrom` / `enemyAct` / `resumeAfterPlayer` (per-round re-rolled turn order) |
| 1995–2110 | `resolveVictory`, `buyItem` |
| 2112–end | `restart`, full render tree (title / create / idle / combat / shop / rest / win / lose / gamewon) |

---

## Data tables — anchor map

| Const | Line | Shape / notes |
|------|------|--------------|
| `PLAYER_BASE` | [20](index.html:20) | Starting hero: hp:4, hitDice:[8], armor:1, atkDmg:3, dieIdx:0, all stats:8, perks/spells empty |
| `POINT_COST_STEP(score)` | [48](index.html:48) | Creation cost table — 1/1/1/1/1/2/2/3/4/5 for 8→18 steps; 18 cap |
| `POINT_COST_TOTAL(score)` | [58](index.html:58) | Sum-from-8 helper |
| `EMOJI_POOL` | [65](index.html:65) | 12 character creation emojis |
| `STAT_POOL` | [69](index.html:69) | 6 stat-bump options for level-up (`+1` apply) |
| `STAT_PERK_FIELD` | [83](index.html:83) | `{statId → "<short>Perk"}` map for milestone dispatch |
| `STAT_PERKS` | [87](index.html:87) | Score-20 milestone perks — 2 options per stat |
| `WIS_PERKS` | [114](index.html:114) | Legacy alias = `STAT_PERKS.wisdom` |
| `PLAYER_SPELLS` | [129](index.html:129) | 6 spells. Fields: stat, req, cost, type, die, statBonus, autoHit, saveStat, onSave, effect, cureOne, stunTarget, ignoreArmor |
| `PERK_POOL` | [154](index.html:154) | 7 perks for L5/10/15/20 picks — one-shot pool |
| `STORY` | [164](index.html:164) | Narrative beats — `prologue`, `beats[5/10/15/20]`, `ending` (Baron Von Bacon arc). All copy lives here. See [STORY.md](STORY.md). |
| `ENEMY_DEFS` | [173](index.html:173) | 15 enemies. See [BESTIARY.md](BESTIARY.md). |
| `WAVES` | [213](index.html:213) | Per-wave spawn pools (wave 5+ reuses last). See [BESTIARY.md](BESTIARY.md). |
| `WAVE_BOSSES` | [222](index.html:222) | 4 wave bosses (waves 1–4). See [BESTIARY.md](BESTIARY.md). |
| `BARON_TAUNTS` | [227](index.html:227) | Three Baron voice modes (`first` / `elder` / `final`). See [STORY.md](STORY.md). |
| `LIEUTENANT_SNARLS` | [233](index.html:233) | Per-boss one-liner on combat start. See [STORY.md](STORY.md). |
| `BOSS_DEFS` | [234](index.html:234) | 4 deep bosses (wave 5+ cycle, `bonus` scaling, "ELDER" prefix). See [BESTIARY.md](BESTIARY.md). |
| `SHOP_CONSUMABLES` | [249](index.html:249) | 18 items (heals / cures / buffs). See [INVENTORY.md](INVENTORY.md). |
| `DICE_PROGRESSION` | [272](index.html:272) | 30 dice sorted by avg roll. See [INVENTORY.md](INVENTORY.md). |
| `EQUIPMENT_DEFS` | [279](index.html:279) | 34 items (12 weapons + 3 shields + 12 armor/acc + 7 rings). See [INVENTORY.md](INVENTORY.md). |
| `BODY_SLOTS` | [322](index.html:322) | 13 slot defs (`BodyLayout` grid). See [INVENTORY.md](INVENTORY.md). |
| `EMPTY_EQUIP` | [338](index.html:338) | `{...slots:null, rings:[]}` — initial equipment state. |
| `STAT_LABEL` | [339](index.html:339) | Abbreviation map: atkDmg→ATK, hitChance→HIT, etc. **Use in UI strings.** |
| `ROOM_TYPES` | [556](index.html:556) | 4 fight types: small, big (`maybeDouble:true`), bridge, shrine. Stairs is NOT a room type (created on boss kill) |
| `TRAP_ROOM` | [575](index.html:575) | Bare trap stub (`{id:"trap", isTrap:true}`). `makeFloorMap` spreads it then layers a random non-big disguise (`icon`/`label`/`desc`/`enemies`) + a rolled `trapEffects` array. `rollEnemyCount(type, floorNum)` + `bigRoomCountRange(floorNum)` + `TRAP_ROOM_CHANCE` (20) still drive whether-a-trap and overall composition. |
| `TRAP_EFFECTS` / `TRAP_STATUS_KINDS` / `TRAP_EFFECT_LABEL` | — | Sub-effect pool (`spike`/`status`/`mimic`/`ambush`), the three DOTs `status` expands into, and the pretty labels for log + HERE strip. |
| `BTN_VARIANTS` | [939](index.html:939) | Per-variant button theme. **Object shape**: `{fill:[topHex,botHex], border, text, edge}` (`edge` = the chunky 3D bottom-shadow color). Add new variants in this shape — `Btn` reads it. Fills stay dark so caller-supplied sub-text stays legible. |
| `LOG_COLORS` | [919](index.html:919) | Hex map per log type: sys, player, enemy, crit, warn, level, counter, dodge, miss, gold |

---

## Function index

### Pure helpers (line 341–569)

| Fn | Line | Returns / Purpose |
|---|---|---|
| `roll(n)` | [343](index.html:343) | `1..n` |
| `clamp(v,lo,hi)` | [344](index.html:344) | bounded |
| `pct(c)` | [345](index.html:345) | bool — c% chance |
| `getDie(i)` | [346](index.html:346) | `DICE_PROGRESSION[i]` capped |
| `rollDiceV(idx)` | [348](index.html:348) | Sum of N dice |
| `d20Hit(chance)` | [354](index.html:354) | `{hit, crit, fumble}` (nat 20 / nat 1) |
| `dmgRange(atk, die)` | [359](index.html:359) | `{min,max}` for previews |
| `applyBuffs(p, buffs)` | [364](index.html:364) | Adds `tempBuffs` stat deltas |
| `applyEquipment(p, equip)` | [370](index.html:370) | Sums `stats:{}` from all worn gear + rings |
| `abilityMod(score)` | [386](index.html:386) | `floor((score-10)/2)` — odd-stat rule lives here |
| `statEffectLine(statId,score,ctx)` | [393](index.html:393) | UI string: `STR 14 → ATK +2 · HIT +10%` |
| `applyAbilityStats(p)` | [413](index.html:413) | Converts STR/DEX/CON/WIS/CHA/INT scores + perks into atkDmg/hit/dodge/crit/speed/armor/etc. **Edit here when adding stat-driven effects.** |
| `computeMaxHpFor(con,lvl,hd,eq)` | [459](index.html:459) | HP preview for creation/level-up |
| `getEffectivePlayer(p,equip,buffs)` | [466](index.html:466) | Pipeline: `applyEquipment → applyAbilityStats → applyBuffs`. **Always read effective via this — never mutate `player`.** |
| `getAvailableEquipment(equip)` | [471](index.html:471) | Pool of items the player doesn't already own |
| `canEquipItem(item,equip,p,targetSlot)` | [482](index.html:482) | Validates light-only / Dual Wield / 2H blocks |
| `getShopCost(item, p)` | [498](index.html:498) | `ceil(cost × (1 − discount/100))`, floor 1 |
| `isIlliterate(p)` | [502](index.html:502) | `intMod < 0` → all gear labels become `???` |
| `describeItem(item, p)` / `labelItem` | [504](index.html:504) | Three-tier identify gating |
| `addToInventory` / `removeFromInventory` | [527](index.html:527) | Stack-by-id with `count` field |
| `pickRandom(pool, n)` / `pickShopEquipment` | [518](index.html:518) | Shop roll helpers (25% rare/75% common bias) |

### Floor & enemy gen (line 574–685)

| Fn | Line | Notes |
|---|---|---|
| `getFloorLayout(floorNum)` | [574](index.html:574) | Returns `{cols,rows}` — 3×2 (f1–3), 3×3 (f4–7), 4×3 (f8–13), 4×4 (f14–20) |
| `floorPositions(floorNum)` | [580](index.html:580) | Flat `[{col,row}…]` array |
| `isAdjacentPos(a,b)` | [588](index.html:588) | Manhattan-1 |
| `makeFloorMap(floorNum,level,p,revealAll)` | [590](index.html:590) | Builds rooms. Entrance pinned to `{0,0}` and `visited:true`. Boss is shuffled among the others, always `enemyKnown:true` |
| `applyRoomToEnemy(e, room)` | [628](index.html:628) | Runs `room.enemyMod` if present |
| `scaleHit(base, level)` | [635](index.html:635) | Ramps to 100 by L15, then +1/level |
| `pickEnemy(level, {allowHealer})` | [638](index.html:638) | Rolls from `WAVES[wave-1]`; filters Witch when `allowHealer:false` |
| `pickRoomEnemies(level, count)` | [658](index.html:658) | Calls `pickEnemy` N times. Second enemy disallows healer (no double-witch) |
| `pickBoss(waveNumber, level)` | [665](index.html:665) | Wave 1–4 → `WAVE_BOSSES`; wave 5+ cycles `BOSS_DEFS` with bonus scaling + "ELDER" prefix |
| `bossMinionCount(floorNum)` | — | 0 on f1–9, 1 on f10–13, 2 on f14–17, 3 on f18–20. Boss-room `enemies` array prepends the boss then appends N minions via `pickEnemy(level)` in `makeFloorMap`. |

### Combat resolvers (line 687–869) — **pure**

| Fn | Line | Returns |
|---|---|---|
| `playerAttack(p, e, verbose)` | [687](index.html:687) | `{total, results, didPoison, didBurn, didBleed, newBleedVal}`. Walks each weapon (main + offhand if dual-wield) + double/triple strikes. Short-circuits via `enemyDead()` once `total >= e.hp` |
| `enemyAttack(p, e, verbose)` | [786](index.html:786) | `{dmgToPlayer, counterDmg, results, gotCrit, didPoison, didBurn, didBleed, didStun}` |
| `enemySpellAttack(p, e, floor)` | [831](index.html:831) | Same shape as `enemyAttack`. DC = `8 + spell.mod + floor(waveNumber/2)`. Bypasses dodge/armor — save IS the defense. No crit. No counter (no melee swing). |

### UI primitives (line 871–1107)

| Component | Line | Notes |
|---|---|---|
| `Hearts` | [871](index.html:871) | HP rendering |
| `TileBox` | [931](index.html:931) | Wooden panel — rounded corners, warm dark-wood gradient, beveled border, soft drop shadow. `highlight` adds a golden frame + glow. Fills stay dark so light-on-dark text still reads. |
| `Btn` | [948](index.html:948) | Rounded beveled gradient button; `className="dq-btn"` adds hover/press feedback. Reads `BTN_VARIANTS`. |
| `EquipDisplay` | [921](index.html:921) | Sidebar gear list |
| `BodyLayout` | [938](index.html:938) | Click-to-select slot grid |
| `FloorMap` | [965](index.html:965) | Fog-of-war grid. Derives dims from rooms' max `pos`. Tile minHeight shrinks on 4-wide floors. **HERE strip** rendered alongside on idle screen |
| `GearDetail` | [1044](index.html:1044) | Selected-slot inspector |
| `ItemButtons` | [1080](index.html:1080) | Inventory entry renderer with dual-wield `→ OFFHAND` button (gated by `canEquipOffhand`) |

### `App` handlers (line 1131+)

| Fn | Line | Purpose |
|---|---|---|
| `addLog(text, type)` | [1202](index.html:1202) | Appends to log (capped 60 entries). `type` ∈ `LOG_COLORS` keys |
| `shake(who)` | [1203](index.html:1203) | 350 ms shake animation |
| `triggerArenaAnim(p, e, dur)` | [1205](index.html:1205) | Player + enemy arena animation state |
| `saveHighScore(finalGold)` | [1210](index.html:1210) | LocalStorage `dq_scores` |
| `triggerLevelUp(newLevel, currentPlayer)` | [1217](index.html:1217) | Routes to `perk` (L5/10/15/20) or `levelup`. Tops up `unspentPoints += 2` |
| `pickStat(stat)` | [1232](index.html:1232) | Spends 1 point. **Checks `STAT_PERK_FIELD`** — score ≥20 + field null → `statperk` phase with `milestoneStat` set |
| `finishLevelUp()` | [1247](index.html:1247) | Banks unspent points, → `win` |
| `pickSpell(spell)` | [1254](index.html:1254) | Learns at level 1 |
| `upgradeSpell(spell)` | [1265](index.html:1265) | +1 level (validates `player[stat] >= req + currentLevel`) |
| `pickPerk(perk)` | [1277](index.html:1277) | Applies perk, removes from pool, → `levelup` |
| `pickWisPerk(perk)` | [1284](index.html:1284) | **Legacy** — kept for back-compat |
| `pickStatPerk(perk)` | [1293](index.html:1293) | Stores on `<stat>Perk`, → back to `levelup` |
| `beginCreation()` | [1303](index.html:1303) | Inits `draft`, `unspentPoints=27`, → `create` |
| `draftAdjust(statId, delta)` | [1316](index.html:1316) | +/− with exact refund — uses `POINT_COST_STEP` |
| `finishCreation()` | [1336](index.html:1336) | Commits draft, computes maxHp, sets hp=maxHp, → `idle` |
| `enterRest()` / `descendStairs()` | [1351](index.html:1351), [1656](index.html:1656) | Flee/rest; descend bumps wave+1, regenerates floor, `playerPos={0,0}`. `descendStairs` also triggers the `story` phase on f5/10/15/20 via `STORY.beats[n]` — see [STORY.md](STORY.md). |
| `handleRoomEnter(room)` | [1372](index.html:1372) | Adjacency/visited check. Branches: cleared (silent walk), descend, else `startBattle` |
| `startBattle(room)` | [1392](index.html:1392) | Initiative + surprise (waveNumber ≥ 3 && enemyKnown===false → +5 SPD). Resets stun, decrements tempBuffs, tempHp=0 |
| `handleAction(key)` | [1444](index.html:1444) | `"attack"` or `"flee"` (75% success). Runs `playerAttack`, applies procs, Cleaving Blow spill, victory check, else `resumeAfterPlayer` (advances the initiative queue) |
| `handleCastSpell(spell)` | [1537](index.html:1537) | Player spell dispatch. `levelBonus = lvl − 1` adds to dmg/heal/charm |
| `handleUseItem(item, inBattle)` | [1651](index.html:1651) | Heal (+wisMod), buff, or cure. WIS perks (`reservoir`→tempHp, `swift`→skip enemy turn). Slotted items route to `handleEquipFromInventory` |
| `handleEquipFromInventory(item, inBattle, targetSlot)` | [1706](index.html:1706) | Equip. Displaced → inventory. Mid-fight: enemy gets free strike against NEW gear |
| `canEquipToOffhand(item)` | [1745](index.html:1745) | Dual-wield visibility check |
| `equipToOffhand(item, inBattle)` | [1750](index.html:1750) | Sugar — calls handleEquip with `targetSlot:'offhand'` |
| `handleSellItem(item)` | [1753](index.html:1753) | `floor(cost/2)` min 1, one off a stack |
| `beginRound(round, curP, curEnemies, curTempHp)` | — | Top of each round: DOT ticks + flee + **re-rolls initiative** (`speed + d10`), logs `⚡ ROUND N — …`, runs the queue |
| `runQueueFrom(i, …)` | — | Walks the initiative queue; player slot pauses for input, enemy slot → `enemyAct` |
| `enemyAct(i, curP, curEnemies, curTempHp, done)` | — | One enemy's turn (charm/heal/spell/melee/counter/tempHP/Iron Will) |
| `resumeAfterPlayer(curP, curEnemies)` | — | Player handlers call this to advance the queue past the player's slot |
| `resolveVictory(defeatedList)` | [1995](index.html:1995) | XP/gold aggregation. Fled = half. **Wave 20 boss → `gamewon`**. Else boss → transform tile to stairs + open shop. Triggers level-up chain |
| `buyItem(item, targetSlot)` | [2077](index.html:2077) | Displaces old slot → inventory. Consumables stay in shop. 2H evicts offhand |
| `restart()` | [2112](index.html:2112) | Full state reset → `title` |

---

## `App` state (line 1131–1187)

| State | Init | Purpose |
|---|---|---|
| `player` | `{...PLAYER_BASE}` | Persistent player — base stats only, never mutate with equipment/buffs |
| `enemies` | `[]` | Active battle enemies (1 or 2) |
| `targetIdx` | `0` | Which enemy player is aiming at |
| `log` | initial msg | Combat log (capped 60 entries) |
| `phase` | `"title"` | State machine — see Phases |
| `ironUsed` | `false` | Iron Will fires once per battle |
| `level` / `xp` | 1 / 0 | `XP_NEEDED = level * 3` |
| `gold` | 0 | |
| `levelUpOpts` / `pickedStats` / `perkOpts` | `[]` | Pick menus |
| `waveNumber` | 1 | Current floor (1–20) |
| `floorRooms` | `makeFloorMap(1,1,…)` | Persistent room layout |
| `playerPos` | `{col:0,row:0}` | Fog-of-war position |
| `shopItems` / `shopUnlocked` | | Post-boss shop |
| `equipment` | `{...EMPTY_EQUIP}` | All worn gear |
| `inventory` | `[]` | Stack-by-id consumables + displaced gear |
| `tempBuffs` | `[]` | `{stat,val,battles}[]` — decrements at `startBattle` |
| `selectingItem` / `selectingSpell` | false | UI toggles |
| `arenaAnim` | `{player:'idle',enemy:'idle'}` | Per-frame animation state |
| `devMode` | false | Reveals all + 0 damage + maxes WIS/INT (via `roomPlayer` at [1191](index.html:1191)) |
| `equipOpen` / `selectedSlot` | | Body panel |
| `poison` / `burn` / `bleed` / `stun` | 0 | Player status counters (poison/burn/bleed PERSIST between battles; stun does NOT) |
| `tempHp` | 0 | Sage's Reservoir absorption. Cleared on `startBattle` |
| `unspentPoints` | 0 | Banked stat points. +27 on create, +2/level |
| `milestoneStat` | null | Set while `statperk` phase is active |
| `draft` | null | Working state during `create` |
| `storyBeat` | null | Current beat (`{title, villain, lines, button, next}`) during `story` phase. Set by `descendStairs` / `finishCreation` / `gamewon`. See [STORY.md](STORY.md). |
| `gameStats` | `{biggestHit,biggestHitTaken,killCounts,itemsUsed}` | Stats panel |
| `highScores` | localStorage | `dq_scores` |

Derived (every render): `ep` ([1189](index.html:1189)), `idlePlayer` ([1190](index.html:1190)), `roomPlayer` ([1191](index.html:1191)), `enemy` ([1196](index.html:1196)).

---

## Phases (state machine)

| Phase | Entered from | UI |
|---|---|---|
| `title` | Initial / `restart()` | Title screen + START + high scores |
| `create` | `beginCreation` ([1303](index.html:1303)) | Point-buy hero forge |
| `idle` | `finishCreation` ([1336](index.html:1336)), shop exit, rest | Floor map + HERE strip |
| `player` | `runQueueFrom` reaching the player's initiative slot | Player-phase action panel |
| `enemy` | `beginRound` (rolling) / `runQueueFrom` enemy slot | Round-resolve + animated enemy attack |
| `win` | `resolveVictory` ([1997](index.html:1997)) and end of pick chain | "X DEFEATED" + continue button |
| `lose` | Status death ([1806](index.html:1806), [1817](index.html:1817)) or HP death ([1981](index.html:1981)) | Game over + high-score |
| `gamewon` | Floor-20 boss kill ([2047](index.html:2047)) | Victory screen |
| `levelup` | `triggerLevelUp` and after sub-pickers | +1 stat / spell learn / upgrade panel |
| `perk` | L5/10/15/20 in `triggerLevelUp` | 2-of-N perk pick from `PERK_POOL` |
| `wisperk` | **Legacy** — kept for old saves | WIS-specific milestone picker |
| `statperk` | `pickStat` ([1240](index.html:1240)) when stat hits 20 | Generic stat milestone picker (uses `milestoneStat`) |
| `shop` | "ENTER SHOP" button | Shop UI |
| `rest` | `enterRest` ([1354](index.html:1354)) | Rest screen (full heal + clear status on confirm) |
| `story` | `descendStairs` ([1656](index.html:1656)) on f5/10/15/20; `finishCreation` (prologue); floor-20 win | Full-screen dialogue panel rendered at [2601](index.html:2601). Renders `storyBeat`. See [STORY.md](STORY.md). |

---

## Combat flow

### `startBattle(room)` ([1392](index.html:1392))
1. Pull `baseEnemies` from `room.enemies` (1–3). Big rooms always ≥2 (sometimes 3); other fight rooms 35% chance of 2.
2. Surprise check: `waveNumber >= 3 && room.enemyKnown === false` → +5 SPD per enemy (biases their initiative rolls all battle).
3. Reset `stun=0`, `tempHp=0`, decrement `tempBuffs`.
4. Kicks off `beginRound(1, ep, battleEnemies, 0)` — initiative is rolled per round, not once.

### Initiative rounds (`beginRound` / `runQueueFrom` / `enemyAct`) — **re-rolled every round**
Combat is no longer "player phase then enemy phase". Each **round**, every living combatant (player + each enemy) rolls initiative and acts in descending order, so turns interleave (e.g. `👺 ▸ 👺 ▸ 🧝 ▸ 👺`). Round state threads through `combatRef` (`{queue, ptr, round, curP, curTempHp}`) to survive the async enemy chain + input-gated player slot.

- **`rollInit(speed)`** = `speed + roll(10)`. Player wins ties (tiebreak), then random.
- **`beginRound(round, curP, curEnemies, curTempHp)`** (600ms beat): ticks enemy DOTs + player DOTs (once each), runs flee checks, commits state, checks victory/lose, then rolls the queue, logs `⚡ ROUND N — 🧝19 ▸ 👺9 ▸ …`, and calls `runQueueFrom(0)`.
- **`runQueueFrom(i, curP, curEnemies, curTempHp)`**: skips dead-enemy slots. At end-of-queue → Vital Surge regen → `beginRound(round+1)`. Player slot → if `stun`, clear + skip; else commit state + `setPhase("player")` + pause (handlers resume via `resumeAfterPlayer`). Enemy slot → 600ms beat → `enemyAct` → chain.
- **`enemyAct(i, curP, curEnemies, curTempHp, done)`**: one enemy's turn — charm-skip, witch heal (2-turn cooldown), spell-vs-melee, status procs, counter, temp-HP absorb, Iron Will, death checks. Calls `done(curP, curEnemies, curTempHp, playerDied)`.
- **`resumeAfterPlayer(curP, curEnemies)`**: player action handlers call this instead of the old `runEnemyTurn`; advances the queue from the player's stored `ptr`.

### `handleAction("attack")` ([~1790](index.html:1790))
1. Target = derived `enemy` (alive at `targetIdx`, or first alive).
2. `playerAttack(ep, target)` runs each weapon (main, offhand-if-dual, double/triple) — short-circuits on `enemyDead()`.
3. Vampiric heal on crit (mainhand or offhand).
4. Apply procs to survivor: `poison:3`, `burn:3`, `bleed: max(newBleedVal, existing)`.
5. Berserker enrage (`enragesWhenLow` + hp ≤ half) → +3 ATK.
6. Splice target into `enemies`.
7. **Cleaving Blow** (MIGHT 20 `cleave`): 25% chance to spill `floor(total/2)` to another live foe.
8. Target died → auto-fall-back `targetIdx` to next alive.
9. All dead → `resolveVictory`. Else → `resumeAfterPlayer(effectivePlayer, newEnemies)`.

### Status / DOT timing
**DOTs tick once at the *end* of each round** inside `endRound` — enemy poison (−1 HP), burn (−2 HP), bleed (counter −1); then the player's poison/burn/bleed. Lethal player DOT → `setPhase("lose")` (a winning-round DOT can still kill — death overrides the win). `endRound` also fires Vital Surge regen and is called from `runQueueFrom` queue-exhaust *and* from `handleAction`/`handleCastSpell` when the player kills mid-queue, so end-of-round effects never get skipped by a quick win. Flee checks still run at the top of each round in `beginRound`. Player-side `bleed > 0` still applies a ×1.5 incoming-damage multiplier inside `enemyAct`.

### Stun
If the player is `stun`ned when their initiative slot comes up, they **lose that slot** (log "STUNNED — YOU SKIP YOUR TURN!", clear stun, advance the queue). Enemies still act in their own slots. (Old behavior gave the enemies a whole extra phase; now it's just one lost player action.)

### `resolveVictory(defeatedList)` ([1995](index.html:1995))
1. Filter null/falsy. Empty → `setPhase("win")` + return.
2. Greed mult: `(Greed Ring ? 1.5 : 1) × (chaPerk==="lucky" ? 1.25 : 1)`.
3. Per enemy: half rewards if `fled`. Sum XP + gold, patch `killCounts`.
4. XP overflow → level up: roll d8, append to `hitDice`, half-effective-max heal.
5. **Boss kill**:
   - `waveNumber >= 20` → `saveHighScore`, `setPhase("gamewon")`. Return.
   - Else: transform boss room in-place (`isBoss:false, descend:true, cleared:true, icon:"⇣", label:"STAIRS DOWN"`). Open shop with `pickRandom(SHOP_CONSUMABLES,3) + pickShopEquipment(avail, 2 + (chaPerk==="silver"?1:0))`.
6. Regular kill: flip `cleared:true` on the room.
7. Chain: `triggerLevelUp` (if leveled) else `setPhase("win")`.

---

## Mechanics cheat sheet

### Naming convention — cartoon reskin (NOT D&D)

To stay clear of the classic tabletop six, the **internal keys** stay `strength/dexterity/constitution/wisdom/charisma/intelligence` (used everywhere in code: `player.x`, `enemy.attrs.x`, `spell.stat`), but every **player-facing label** is reskinned via `STAT_LABEL` (3-letter) and `STAT_FULL`:

| Internal key | Full (`STAT_FULL`) | Code (`STAT_LABEL`) |
|---|---|---|
| `strength` | MIGHT | MGT |
| `dexterity` | ZIP | ZIP |
| `constitution` | GUTS | GUT |
| `wisdom` | SAVVY | SAV |
| `charisma` | CHARM | CHM |
| `intelligence` | WITS | WIT |

Use `statLine(src)` to render the compact `MGT 12 · ZIP 10 · …` row; never hardcode stat labels. Likewise "saving throw / DC" terminology is reskinned to **"SHRUG (off)"** with a bare target number (no "DC"), and "hit die" → **"GROWTH DIE"**.

### Ability mods & odd-stat rule

`abilityMod = floor((score-10)/2)`. So 11/9 share 10's mod (+0); 13/12 share +1; 14/15 share +2. **Odd scores are stored progress.** Negative mods apply raw (no `Math.max(0)` floor) — `clamp` floors `awareness/identify/poisonResist` at 0, but `shopDiscount` and heal `wisMod` can go negative. `statEffectLine` always shows the SIGNED value.

### Stat effects (`applyAbilityStats` [413](index.html:413)) — keyed by internal name
- **strength (MIGHT)** — +1 ATK, +5% HIT per mod.
- **dexterity (ZIP)** — +5% DODGE, +5% CRIT, +1 SPD per mod.
- **constitution (GUTS)** — applied retroactively to every hit die roll in `hitDice`; +5% poisonResist per positive mod.
- **wisdom (SAVVY)** — +15% awareness per mod. **All healing scales off SAVVY**: heal items add `wisMod` (`handleUseItem`); heal spells (BOO-BOO FIX, THERE-THERE) add `wisMod` to their die; Sage's Reservoir temp-HP = `wisMod`; and the Vital Surge perk regens `max(1, 1+wisMod)`/round.
- **charisma (CHARM)** — −5% shop prices per mod.
- **intelligence (WITS)** — +15% identify per mod (50%+ = full shop labels; mod < 0 = illiterate `???`).

### HP

`maxHp = floor((sum(hitDice) + conMod × hitDice.length) / 2) + equipmentBonus`. L1 always rolls max (8); each level appends a fresh d8. CON applied retroactively. Half-effective-max heal at every level-up. **Player starts at full HP** — `finishCreation` sets `hp = maxHp`.

### Stat-20 milestone perks (`STAT_PERKS` [87](index.html:87))

| Stat | Perk A | Perk B |
|---|---|---|
| STR | **brute** — +1 ATK every swing (passive in `applyAbilityStats`) | **cleave** — 25% on damaging hit → spill `floor(total/2)` to another live foe (in `handleAction` [1507](index.html:1507)) |
| DEX | **reflex** — +5% dodge AND raises dodge cap 100→110% | **flurry** — +10% double, +5% triple strike (passive) |
| CON | **vital** — regen `max(1, 1 + savvyMod)` HP at round end (all healing scales off SAVVY; in `runQueueFrom` end-of-queue) | **hide** — +1 ARM permanent (passive) |
| WIS | **reservoir** — every heal grants `wisMod` Temp HP (consumed at [1954](index.html:1954)) | **swift** — heals roll `wisMod × 10%` to NOT consume turn ([1695](index.html:1695)) |
| CHA | **silver** — +1 extra shop equipment slot ([2069](index.html:2069)) | **lucky** — +25% gold (in `greedMult` [2000](index.html:2000)) |
| INT | **tactician** — `identify=100` regardless of INT (passive) | **arcane** — +5% to every weapon proc chance (per-swing in `playerAttack`) |

Storage: `<stat>Perk` field on player (e.g. `strPerk`). Once chosen, locked for the run. Routing: `pickStat` ([1232](index.html:1232)) checks `STAT_PERK_FIELD` and routes to `statperk`.

### Spells & shrug-off math

Full spell roster, shrug-off formula (player & enemy casters), `onSave` semantics, and the half-scale design rule live in **[MAGIC_SYSTEM.md](MAGIC_SYSTEM.md)**. Implemented spells = `PLAYER_SPELLS` ([129](index.html:129)). Spell-level storage: `player.spells = { id: levelNum }`; to reach LV N the caster stat must be at `spell.req + N − 1`; each level adds **+1 dmg / +1 heal / +1 charm turn** post-halve.

**Trap shrug uses a different formula** — `DC = 10 + waveNumber` (full waveNumber, not halved). Steeper than mod-attacker DCs by design. See the Trap rooms section below.

### Status effects

| Status | Effect | Cure | Persists between battles? |
|---|---|---|---|
| ☠ Poison | 1 dmg/turn × 3 turns | Antidote / Elixir | **Yes** |
| 🔥 Burn | 2 dmg/turn × 3 turns | Ointment / Elixir | **Yes** |
| 🩸 Bleed | `floor(weapon_avg + atkDmg)` per swing AND turn counter; deepens via Math.max on re-proc; player-side bleed = +50% incoming dmg | Bandaid / Elixir | **Yes** |
| 💫 Stun | Lose your initiative slot this round. **`stunsOnHit` melee procs (Ogre-style) trigger a `GUTS` shrug-off**: `d20 + gutMod` vs `DC = 8 + abilityMod(e.attrs.strength) + floor(waveNumber/2)`. Spell-based stuns go through the normal spell shrug-off. Other status procs (poison/burn/bleed) stay guaranteed — only stun gets a defense roll because chained stun-lock is the worst failure mode. | Smelling Salt / Elixir | **No** (cleared at `startBattle`) |

### Dice & weapons

Full weapon roster, hand-class rules, procs, dice progression, and offhand math live in **[INVENTORY.md](INVENTORY.md)**. Quick refs: damage = `atkDmg + die − armorUsed`; crits roll the die twice; offhand swings = 70% damage (Master Duelist removes penalty); weapon `hand` ∈ {`light`, `one`, `two`}.

### Initiative

`player.speed = 10 + dexMod` (+ equipment `stats.speed`). Compared at `startBattle`. Higher acts first; **player wins ties**. Surprise → +5 enemy SPD (gated to floor 3+). Bosses pinned `enemyKnown:true` — never surprised.

### Identify gating

Three tiers (illiterate → generic category → full description), keyed off WITS mod + the player's `identify` percentage. Dev mode bypasses with `intelligence:30, identify:95`. Full table → [INVENTORY.md](INVENTORY.md).

### Floor scaling

| Floors | Grid | Tiles |
|---|---|---|
| 1–3 | 3×2 | 6 |
| 4–7 | 3×3 | 9 |
| 8–13 | 4×3 | 12 |
| 14–20 | 4×4 | 16 |

Each floor: 1 boss room + (size−1) other rooms. Entrance pinned to `{0,0}`, pre-`visited`. Boss tile shuffled into one of the others. Boss kill → tile transforms to stairs in place; shop unlocks; player walks back to descend.

**Room composition** (in `makeFloorMap`):
- **Entrance (i===0)** — always a single-foe fight. Never a trap, never a swarm (no floor-1 ambush). Its `type` is **rolled from `ROOM_TYPES.filter(t => !t.maybeDouble)`** so it can never be labeled BIG (preserves the "big rooms always ≥2" promise — a single-enemy BIG ROOM tile was a bug here).
- **Every other room** — `TRAP_ROOM_CHANCE` (20%) to be a **trap** instead of a fight; otherwise a fight whose foe count comes from `rollEnemyCount(type, floorNum)`: **big rooms scale with depth** via `bigRoomCountRange` (f1 → 2, f2–4 → 2–3, f5–7 → 2–4, f8–10 → 3–4, f11–13 → 3–5, f14–16 → 4–5, f17–19 → 4–6, f20 → 5–6, uniform within range); all other types 35% chance of 2 else 1. `pickRoomEnemies` allows at most one healer (Witch) per room regardless of total count.

### Trap rooms (`triggerTrap` [~1432](index.html:1432))

**Disguise.** Trap rooms render as a random non-big fight type (small/bridge/shrine) — their surface `icon`/`label`/`desc` are copied from the disguise at `makeFloorMap` time, plus a full set of fake `enemies` rolled via `pickRoomEnemies`. The tile only flips to the amber `⚠ TRAP!` look when `room.trapAware || revealAll || cleared`. Two independent awareness rolls govern the deception: `trapAware` (spot the trap) and `enemyKnown` (see the fake enemies) — both use `pct(awareness)` at floor-gen. So a high-WIS hero can: spot both, spot one, or spot neither.

**Sub-effects.** Each trap rolls a primary sub-effect uniformly from `TRAP_EFFECTS` (`spike`/`status`/`mimic`/`ambush`); `status` expands to one of `TRAP_STATUS_KINDS` (`poison`/`burn`/`bleed`). 25% chance of a different secondary on top → ~74% pure traps, ~26% combos (15 possible pairs). Stored as `room.trapEffects` (e.g. `["spike"]`, `["burn","mimic"]`).

**Walking onto an uncleared `isTrap` room** (routed in `handleRoomEnter` before `startBattle`) forces a **ZIP shrug-off**: `roll(20) + zipMod + (aware ? 5 : 0)` vs `target = 10 + waveNumber` (steeper scaling than other DCs — traps are meant to make awareness/WIS feel mandatory at depth). `aware = room.trapAware || devMode`. Pass → no damage, room cleared. Fail → effects fire in order (`ambush` always last so other stings land first):
- `spike` — `roll(3) + floor(waveNumber/3)` HP, ignores armor.
- `poison` / `burn` / `bleed` — chip damage `1 + floor(waveNumber/8)` (1 on f1–7, 2 on f8–15, 3 on f16–20), then sets the matching counter to 3.
- `mimic` — steals `max(5, floor(gold * 0.25))` capped at current gold; logs "empty pockets" if broke.
- `ambush` — `pickEnemy(level)` and call `startBattle` with a synthetic `{isTrap:false, enemyKnown:false}` room (so startBattle's surprise check applies the +5 SPD bonus on f3+). Room is cleared up-front so the trap can't re-fire on victory.

Lethal damage during the effect chain ends the run; ambush short-circuits any remaining (already-sorted-last) effects. Map tile and HERE strip pivot from disguise to amber `TRAP!` (+sub-type label) when revealed; cleared traps show "TRAP ROOM (SPRUNG)". Trap rooms count toward the floor's "ROOMS CLEARED" tally.

### Visibility / movement

Tile visible if `visited` OR adjacent to any visited tile OR `revealAll` (dev). `handleRoomEnter` accepts the click only if visible AND (adjacent to `playerPos` OR already visited). Cleared visited tiles = fast-travel.

### Perks (`PERK_POOL` [154](index.html:154))

Offered at L5/10/15/20. One-shot — removed from pool after pick. `XP_NEEDED = level * 3`.

- **Double Strike** +20% chance for a 2nd (main-hand) swing.
- **Triple Strike** +15% chance for a 3rd swing — **chained**: only rolled if a Double Strike fired that turn (even if the Double *missed*); never guaranteed. **Gated**: not offered in the perk pool until the player already has Double Strike capability (`doubleStrikeChance > 0`, from the perk or DEX flurry). The 3rd swing uses the offhand weapon (with its penalty) when dual-wielding, else a no-penalty bonus main-hand swing. Logic in `playerAttack` ([~806](index.html:806)); pool gate in `triggerLevelUp` ([~1282](index.html:1282)).
- **Counter Strike** +20% counter on hit
- **Iron Will** survive one killing blow at 1 HP per battle
- **Armor Pierce** crits ignore armor
- **Dual Wield** equip a second light weapon
- **Master Duelist** removes the 30% offhand damage penalty (offhand swings deal full damage). **Gated**: not offered until the player has Dual Wield — it's meaningless without an offhand. Gate in `triggerLevelUp` ([~1286](index.html:1286)).

### Rest / Flee

75% flee success. **Camp is reachable any time from the idle/map screen** (`🏕 MAKE CAMP` button) as well as the post-victory `win` screen — both call `enterRest` → `rest` phase → BREAK CAMP fully heals + clears all status. XP is **not** forfeited. (`restUsed`/`restXpLost` state exists but currently gates nothing — camp is unlimited.)

---

## Editing conventions

- **No build step.** React + ReactDOM + Babel from CDN. No external imports.
- **Read `index.html` before editing.** Don't regenerate the whole file. Edit in place.
- **Never mutate `player` with equipment/buff bonuses.** They live only on the effective player from `getEffectivePlayer`.
- **Follow data-driven patterns** when adding content. Add a row to `ENEMY_DEFS` / `EQUIPMENT_DEFS` / etc., wire it into the relevant pool, never branch on names.
- **UI tone**: soft, cartoonish, whimsical — silly and charming, never grim, gritty, or edgy; think the warm wooden game-UI kit, not 8-bit pixel art. Rounded corners, warm wood/amber palette, beveled `Btn` + `BTN_VARIANTS`, emoji as sprites, bouncy/squishy animations. Build new panels with `TileBox` and new buttons with `Btn` so the theme stays consistent; prefer `borderRadius` + warm browns (`#6b4a28`/`#4a371d`) over sharp black `outline`s. Combat log is short, punchy, and goofy (puns, "BONK!", "+4 HP — all better!", exclamation marks welcome), color-coded via `LOG_COLORS`.
- **Original naming only.** No D&D / *Dungeons & Dragons* references anywhere player-facing. Stats, spells, saves, and dice are reskinned to cartoon terms (see [Naming convention](#naming-convention--cartoon-reskin-not-dd) and [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md)). If you spot a legacy label in code (`MAGIC MISSILE`, `FIRE BOLT`, `SAVING THROW`, raw `STR/DEX/…`), rename it — those strings are exactly what we're moving away from.
- **Tersely show numbers** with `STAT_LABEL` abbreviations.
- **Dev button** disables incoming damage, reveals all rooms, sets WIS+INT to 30. Toggle OFF when balance testing.

### Open balance issues

- **Bleed scaling**: `floor(weapon_avg + atkDmg)` per swing × bleed counter is degenerate at mid-game. Bleeding Spear + STR 14 = `+10 dmg × 10 turns` on one proc. Untuned.
- **Starting HP fragility**: 8-CON L1 = 3 HP. CON 18 mitigates (→ 6 HP) but a dump build is thin.

### Changelog

Shipped features and balance tunings live in **[CHANGELOG.md](CHANGELOG.md)**. When an item from [Wishlist](#wishlist--design-todos) ships, move it into CHANGELOG.md (newest at top).

---

## Wishlist — design TODOs

Forward-looking changes the user has flagged. Mix of bug-fixes and feature additions — pick one off the top, scope it, ship it, then move the entry into [CHANGELOG.md](CHANGELOG.md) when done. Each item leans on the current code state (per-round initiative, existing trap rooms, `🏕 MAKE CAMP` anywhere) — skim [CHANGELOG.md](CHANGELOG.md) before scoping so you don't re-do work that already shipped.

### 1. SUPPLIES — a camping resource

Add a new consumable `supplies` (emoji 🥖 or 📦). Owning ≥1 enables BREAK CAMP at the rest screen; the camp consumes one. No supplies → BREAK CAMP is disabled with a `NEED SUPPLIES` label.

The existing `🏕 MAKE CAMP` entry (idle/map screen) still routes the player to the rest screen — supplies gate the camp ACTION, not WALKING to the campfire.

Sources:
- **Boss kill** — always drops N supplies (suggest 1–2 guaranteed, +1 from wave 10+).
- **Shop** — rarely stocks 1 supply at a high-ish price (~1 in 3 shops at most). Tune via `pickRandom(SHOP_CONSUMABLES,3)` weight or a small dedicated pool.

State: simplest is a player counter (`player.supplies: 0`). Or treat as a stack-by-id inventory item matching the existing consumable pattern. Either way, surface the count as a chip on the idle/map HUD so the player isn't surprised at the campfire.

### 2. Hide enemy stats — only HP + status visible

The combat panel currently shows the enemy's full block (ATK/ARM/HIT/DOD/SPD/MAX HP, attrs, niche). Strip it down. The player sees ONLY:
- HP bar / current HP / max HP
- Status icons (☠ poison · 🔥 burn · 🩸 bleed · 💫 stun · 💖 charm) with turn counters

Hide ATK, ARM, HIT, DOD, SPD, attrs, niche, and behavior flags from the combat UI. `devMode:true` still reveals everything for testing. The fog-of-war "enemy known" room preview gets the same treatment — emoji + name only, no stats.

Tactical info now comes from the **bestiary** (#3) — players learn enemy stats by fighting them.

### 3. Bestiary on the title screen

A new persistent panel reachable from the title screen, showing every enemy the player has ever fought, with progressive reveal gated by kill count. Data source: see **[BESTIARY.md](BESTIARY.md)** for the full enemy roster.

Tiered reveal (suggest):
- **0 kills** — silhouette + `???`.
- **1 kill** — emoji + name + niche.
- **5 kills** — HP, ATK, ARM.
- **10 kills** — HIT, DOD, SPD, attrs.
- **20 kills** — behavior flags (`poisonOnHit`, `fleesWhenLow`, `healsAllies`, `enragesWhenLow`, `bleedsOnCrit`, …), spell defs, and a sample log line.

Persistence: store as `localStorage["dq_bestiary"]` — a **separate** counter map from `gameStats.killCounts` (which resets on restart). Increment on every enemy kill via a helper at `resolveVictory`. Bosses included — `WAVE_BOSSES` and `BOSS_DEFS` get their own gated entries (their reveal tiers can be lower since you'll typically only kill each one a couple times per run).

Title-screen UI: add a `📖 BESTIARY` button alongside START / HIGH SCORES. Open in a scrollable list with one card per `ENEMY_DEFS` + boss entry, grouped by floor band where they first appear.

Pairs tightly with #2 — the bestiary is the only path to enemy stats once combat hides them.

### 4. SAVE & QUIT from the campfire

Add a `💾 SAVE & QUIT` button next to BREAK CAMP on the rest screen. Serializes the full run to `localStorage["dq_save"]` and returns to title.

State to serialize:
- `player`, `level`, `xp`, `gold`, `waveNumber`, `floorRooms`, `playerPos`
- `equipment`, `inventory`, `tempBuffs`
- Status counters (`poison` / `burn` / `bleed` / `stun` / `tempHp`)
- `unspentPoints`, `milestoneStat` (if mid-pick), `draft` (if mid-create — probably moot since you can't camp during create)
- `gameStats`, `ironUsed` (reset on load — it's per-battle)
- `restUsed` (reset on load — you're effectively starting fresh from the next encounter)

Loading: title-screen detects the save and shows a `▶ CONTINUE RUN` button alongside START. Selecting it deserializes, restores state, and lands the player on `idle` at the saved floor + position. Save is cleared on completion (death, `gamewon`, or an explicit "ABANDON RUN" option from title).

Edge cases:
- Save only allowed from the rest screen — gear is settled, no `enemies[]` mid-fight to worry about.
- Versioning: stamp the save with a schema version (`{v: 1, ...}`) so future schema changes can migrate or invalidate cleanly.
- One save slot for now — a second SAVE & QUIT overwrites the existing save (prompt "OVERWRITE?" if a save exists).

### 5. All stats start at 5 (was 8)

Drop the `PLAYER_BASE` ability-score floor from 8 to 5. Default heroes are now *meaningfully* below-average across the board — builds matter more, dump stats bite harder, and the point-buy phase carries real weight.

Knock-on changes to plan for:
- **`POINT_COST_STEP`** — extend the cost table down to 5→6, 6→7, 7→8 (suggest 1 pt each, matching the cheap 8→13 tier). New floor for `draftAdjust` decrement is 5.
- **HP fragility** — at GUTS 5, `gutsMod = −3`; L1 `maxHp = floor((8 − 3) / 2)` = **2**. The existing [Open balance issues](#open-balance-issues) note already flags GUTS 8 = 3 HP as thin; GUTS 5 = 2 HP is brittle to the point of one-shot territory. Mitigation options to pick from:
  - (a) raise the base growth-die size (D8 → D10?),
  - (b) floor `maxHp` at a baseline minimum (3 or 4),
  - (c) bake in a flat `+N maxHp` before the formula,
  - (d) lift the creation point budget so GUTS 13+ becomes affordable.
- **Identify floor** — `isIlliterate` triggers at `intMod < 0`, which means a default WIT-5 hero is illiterate (all gear labels `???`) until they spend ~5 points on WITS. Confirm intended, or shift the illiteracy threshold to stay where it was player-experience-wise.
- **Shop discount** — CHM 5 (mod −3) → +15% shop markup baseline. Real pinch on starting gold; tighter early game.
- **Creation point budget** — 27 pts may be too tight at the new floor. From 5 to max 18 is roughly 24 pts (3 for 5→8 + 21 for 8→18), so 27 barely lets you max one stat with 3 leftover. Consider bumping to 32–36 pts, lowering the creation cap to 16, or both.
- **UI / `statEffectLine`** — any preview text that hardcodes "base 8" or shows a default delta should be re-checked. The character-creation panel's neutral baseline shifts visibly downward.

This is a balance overhaul, not a knob twist — pencil in a playtest pass right after.

### 6. Enemy attacks that drain base stats

A new enemy ability that hits the **persistent** player stats (`player.strength` / `dexterity` / `constitution` / `wisdom` / `charisma` / `intelligence`), not the effective player. Stat damage survives equipment swaps and combat end — a real "you got *worse*" moment that makes encountering one of these foes feel dangerous.

Design questions to settle before wiring it up:
- **Persistence** — battle-only, until rest/camp, until next floor, or until the run ends? (Persistent within the run is the scariest version; rest-restorable is the most forgiving.)
- **Magnitude** — −1 per hit (stacks across multiple hits) or fixed −1 per encounter?
- **Stat picked** — random, tied to enemy archetype (a brainsucker drains WITS, a charm-sapper drains CHARM, a spectre drains GUTS), or player-targeted (drains your highest stat)?
- **Shrug** — SHRUG against the matching stat? (A WITS-drainer demands a WITS shrug; thematically harsh because the very stat being drained also defends.) Or against a related-but-different stat.
- **Recovery** — heal items can't restore base stats. Options: camping/rest restores, a new shop consumable (`RESTORATIVE TONIC`), level-up restores, or no recovery (you live with the loss until the run ends).
- **Floor gate** — which floors do drainers appear on? Probably f10+ so a fresh hero isn't crippled on floor 1.

Wire as a new enemy spell type with `effect:"drain"` plus a `drainStat` field, dispatched in the spell resolver and gated by the standard shrug roll. Surface the drain prominently in the combat log (`💀 WITS DRAINED! WIT 12 → 11`) and again on the next idle/HUD render so the player notices the loss.

Pairs with **#3 (bestiary)** — gating which stat each drainer targets behind kill count rewards repeat encounters (and lets a well-read player route around them).

### 7. Mid-game difficulty curve

The early game (f1–4) reads well — tight HP, point-buy choices matter, every fight feels like a question. By the mid-game (~f5–13) the curve flattens: accumulated level-ups, perks, rare-weapon procs, and stacked stat investment outpace enemy scaling, and fights stop posing real questions. By f14+ deep bosses start asserting themselves again, but the 6–10 floor stretch in between coasts. Goal: keep the mid-game sharp without hand-wavingly cranking enemy HP.

Several layers to consider — **pick two or three and playtest**, not all at once. The temptation is to globally multiply enemy stats, which only punishes the early game without solving the flattening.

**Scale regular enemies with floor depth.** `pickBoss` already takes a `bonus` from `waveNumber` (HP/ATK/ARM), but `pickEnemy` / `pickRoomEnemies` does not — regular foes stay at their `ENEMY_DEFS` base from f1 to f20. Apply a smaller per-floor bonus to regulars starting at f6: e.g., `+1 HP per 2 floors`, `+1 ATK per 4 floors`, `+5% hitChance per 2 floors`. The same Goblin name on f10 actually poses a question, without growing the roster.

**Elite variants.** Add an `elite` roll (~10% at f5, scaling to ~20% at f15+) that takes any enemy from the wave pool and stamps a modifier: +50% HP, +1 ATK, and one bonus behavior flag from `enragesWhenLow` / `bleedsOnCrit` / `burnsOnHit` / `poisonOnHit`. Prefix the name with ⭐ and pay +50% XP/gold. Cheap variety injection — one helper function and the existing `ENEMY_DEFS` becomes 2× as deep.

**Mid-game archetypes that punish power builds.** By f8 a player typically has Double Strike + Dual Wield + a proc-weapon stack. Drop in enemy types that *answer* those tools rather than just out-stat them:
- **Counter-attackers** — auto-riposte on every player swing (taxes double/triple/extra-strike builds).
- **Status reflectors** — bounce a freshly-applied poison/burn back onto the player.
- **Resistance variants** — Fire-Eater (immune to burn), Iron Hide (ignore the first 2 dmg of every swing — taxes light-weapon dual-wielders into a wall), Pestilent (immune to poison).
- **Armor piercer** — bypass the player's armor entirely (taxes shield / heavy-armor builds).
- **Phase-shifters** — two HP bars; reaching 0 triggers a second phase with a fresh buff (e.g., a Berserker that enrages a second time at half of its second bar).

Pairs with **#6 (stat drainers)** — that's already a mid-game power-cap mechanic in this bucket.

**Fix bleed scaling (also in [Open balance issues](#open-balance-issues)).** This is the single biggest mid-game flattener. A single Bleeding Spear proc + STR 14 = `+10 dmg × 10 turns` and auto-clears the next ~3 fights. Re-tune: flat per-weapon bleed value (e.g., `floor(weapon_avg / 2)`), or cap the bleed counter at 5 turns regardless of weapon, or both. If a player rolls into Bleeding Spear at f6, the rest of the act is currently a no-op — fixing this alone reclaims meaningful combat for f6–11.

**Tighten the heal economy.** Mid-game inventories overflow with potions and HP attrition stops mattering. Options:
- Cap consumable stacks at 5 per type.
- Past f5, drop the shop's heal-potion stock — fewer SUPER POTIONs, more situational consumables.
- Tie BREAK CAMP to a SUPPLIES item (per wishlist **#1**) so resting becomes a finite resource that the player has to spend deliberately, not a free between-fight reset.
- Reduce the level-up heal from half-max to one-third-max past L8 so leveling stops doubling as a free full-heal.

**Bend the curve, don't crank the knob.** Layered, small changes (a touch of scaling + elite chance + 1–2 new archetypes + bleed fix) read to the player as *"the dungeon is escalating, and my build still matters"*. Globally bumping enemy HP/ATK reads as *"the numbers are bigger now"* — which is the wrong feel and breaks new-player onboarding upstream.

### 8. Split equip-gear and use-item UX

Today the `🎒 USE ITEM` selector mixes consumables (potions, antidotes, buffs) and slotted equipment (weapons, shields, armor, rings) in one list, because `handleUseItem` ([1651](index.html:1651)) auto-routes anything with a `slot` field over to `handleEquipFromInventory`. It works but reads as a junk drawer — players hunting for an antidote scroll past five weapons, and the only way to swap a weapon mid-fight is to dig through the consumable button.

Two cleanups:

**Equipment lives in the view-gear panel.** Make the `equipOpen` modal (`BodyLayout` + `GearDetail` + the inventory rail) the canonical home for ALL equip/swap actions, in *and* out of combat. The inventory rail inside the modal shows ONLY slotted items (weapons, shields, armor, rings) with the standard equip / `🛡 → OFFHAND` / sell buttons. Out of combat, this is already where players go to inspect gear — confirm it's the *only* equip surface and drop gear from the USE ITEM selector entirely.

**Dedicated EQUIP action in combat.** Add a new action button to the combat panel alongside ATTACK / CAST / USE ITEM / FLEE — something like `⚔ SWAP GEAR` (or simply open the existing gear modal directly). It surfaces the slotted-items list and lets the player pick a weapon / shield / armor to equip mid-fight; the swap still routes through `handleEquipFromInventory(item, inBattle:true)` so the enemy still gets a free strike against the newly-equipped stats (per the existing rule). The USE ITEM selector becomes consumables-only.

Implementation notes:
- `ItemButtons` ([1080](index.html:1080)) currently renders everything; add a `filter` arg (`"consumable"` | `"equipment"` | `"all"`) and have call sites pass what they want.
- Combat panel: add `selectingGear` state mirroring `selectingItem`. Opens a gear-only list using the same `ItemButtons` with `filter:"equipment"`. Dual-wield affordance (`canEquipToOffhand` + the `🛡 → OFFHAND` button) stays exactly as wired.
- `handleUseItem` ([1651](index.html:1651)) can drop its `if (item.slot)` re-route to `handleEquipFromInventory` since it'll never receive a slotted item once the lists are filtered — keep the guard as a defensive belt-and-suspenders.
- Idle-screen USE ITEM: also filter to consumables; gear management always goes through the gear modal there.
- Optional polish: an `🎒 OPEN GEAR` button in combat that opens the full modal (pause-the-fight feel) is the heaviest option; the quick `⚔ SWAP GEAR` selector is the lighter, more battle-snappy one. Probably want the lighter one for combat, full modal only out of combat.

Net effect: USE ITEM = potions / cures / buffs only (the actual junk drawer is sized right); the gear modal = the one true place to swap weapons and armor; mid-fight weapon swaps are a clear, discoverable action instead of buried in the consumables list.

### 9. Weapon (and heavy gear) equip requirements

Today any hero can equip any weapon — a WIT-maxed mage can swing the Greatsword as effectively as a beefcake. Add a stat gate so heavy weapons want MIGHT, light/finesse weapons want ZIP, and magical gear wants WITS/SAVVY. Builds become identities; the point-buy phase carries even more weight; rare drops feel like rewards for the right hero, not free upgrades for everyone. Weapon list: see **[INVENTORY.md → Weapons](INVENTORY.md)**.

User's anchor example: **Greatsword requires 18+ MIGHT.**

**Suggested requirement table** (tune in playtest):

| Weapon | Req | Rationale |
|---|---|---|
| DAGGER | ZIP 8 | universal starter |
| SHORTSWORD | ZIP 10 | basic finesse |
| LONGSWORD | MIGHT 12 | one-hand steel |
| SPEAR | MIGHT 13 | reach + thrust |
| GREATSWORD | **MIGHT 18** | user's anchor — the wall |
| SHARP DAGGER | ZIP 10 | rare finesse |
| KEEN SHORTSWORD | ZIP 12 | rare finesse |
| VENOM DAGGER | ZIP 12 | rare finesse |
| FLAME BRAND | WITS 12 | magical light |
| PIERCING LONGSWORD | MIGHT 14 | rare steel |
| BLEEDING SPEAR | MIGHT 16 | heavy rare |
| VAMPIRIC BLADE | SAVVY 14 | bloodbound |
| BUCKLER | ZIP 10 | active block |
| IRON SHIELD | MIGHT 12 | weight |
| TOWER SHIELD | MIGHT 16 | dead weight |

Armor pieces could follow the same pattern (CHAIN MAIL → MIGHT 12, heavy boots → GUTS 10, etc.) — but worth shipping weapons first and seeing how it feels.

**Open design questions:**

- **Hard gate vs. soft penalty.** Hard gate (can't equip below req) is cleanest and matches how `canEquipItem` ([482](index.html:482)) already validates 2H / Dual Wield / light-only rules. Soft penalty (equip with malus, e.g. −1 ATK and −10% hit per missing point) lets a desperate player still swing the Greatsword in a pinch — more flexible but adds tuning surface. **Recommend hard gate for v1**, soft penalty as a later option if it feels too restrictive.
- **Multi-stat requirements?** Vampiric Blade could want SAVVY *and* MIGHT, gating it behind hybrid builds. Keep schema flexible (`req: { strength: 14, wisdom: 12 }`) even if every weapon launches with just one entry.
- **Shop visibility.** Show the requirement on every shop entry — RED if unmet, GREEN if met. Don't block purchase (player might be saving for a future level), just visually warn. Identify gating still applies: at INT-mod < 0 the req is hidden as `???` along with the rest of the label.
- **Stat-drain interaction (#6).** If a drainer drops MIGHT below the equipped weapon's req, what happens? Options: (a) auto-unequip into inventory with a log line, (b) keep equipped but apply a "wielding above your means" malus until you re-stat, (c) keep equipped at full power (drain doesn't auto-unequip). **(a) or (b)** read more naturally for the "you got worse" feeling — pick whichever pairs with the gate vs. penalty choice above.
- **Stats-start-at-5 interaction (#5).** With base stat 5, default heroes can't equip almost anything at the gate values above — the point-buy phase becomes "earn your starting weapon." Want to playtest #5 and this item (#9) together; might need to lower the table by 2–3 across the board so a thoughtful build can actually wield something on f1.

**Implementation hooks:**

- New field `req: { <statKey>: number }` on `EQUIPMENT_DEFS` entries ([279](index.html:279)). Use internal stat keys (`strength`/`dexterity`/`…`) — the UI maps them to cartoon labels (MIGHT/ZIP/…) via `STAT_FULL`/`STAT_LABEL`.
- Extend `canEquipItem(item, equip, p, targetSlot)` ([482](index.html:482)) — iterate `item.req` and return false if any `p[stat] < req[stat]`. Returns a reason string for the UI? Or a separate `unmetReqs(item, p)` helper that the shop / gear modal can call to render the chip.
- Shop list entry: render a `MIGHT 18 ⚠` chip in red when unmet; tooltip lists exactly which stat(s) are short. Identify gating hides it behind `???` for illiterate heroes.
- `GearDetail` ([1044](index.html:1044)): show the req as a top-row badge in the slot inspector.
- Stat-drain (#6) hook: in the drain resolver, after applying the drain, walk equipped slots and either auto-unequip or apply the malus (whichever was chosen).

### 10. Shop illiteracy copy — "YOU DON'T KNOW HOW TO READ"

The shop currently shows a `WITS TOO LOW` message (or similar stat-jargon) when the hero can't read item labels. Rephrase to the more on-tone, on-character: **`YOU DON'T KNOW HOW TO READ`** (or `YOU CAN'T READ THIS YET`, `STILL LEARNING YOUR LETTERS`, etc.).

The mechanic stays exactly as wired in `isIlliterate(p)` ([502](index.html:502)) — `intMod < 0` → labels become `???`. Only the *user-facing string* changes: drop the stat name from the message entirely. The player feels the loss as "my hero can't read," not as "my INT score is mathematically too low" — which reads warmer, sillier, and stays clear of the D&D-style "INT/WITS dump" framing.

Hunt down every shop message that names WITS / INT as the gate (the `??? — TOO ARCANE TO READ` line at minimum, plus any new req-mismatch copy from **#9**). Equip-requirement chips for *other* stats (MIGHT/ZIP/GUTS/etc.) still name the stat, since those are real strength/agility/etc. demands rather than literacy.

Optional polish:
- Show the line as a little speech bubble from the shopkeeper, not as a stat error — leans further into the cartoon vibe.
- Variants for variety: rotate among 3–4 lines so it doesn't feel like a hard fail (`"...HMM, GOT TO LEARN THOSE SQUIGGLES."`, `"...WORDS ARE TRICKY!"`, `"...MAYBE THE SHOPKEEPER WILL READ IT TO ME?"`).

### 11. Turn-order visualization — make initiative tactical

Combat re-rolls per-round initiative (`speed + d10` per actor in `beginRound`), and the order is currently surfaced only in the round-start log line (`⚡ ROUND N — 🧝19 ▸ 👺9`). That's text the player has to *read* — by the time they've scanned it, they've already lost the tactical thread. The goal of this item: make turn order so visually obvious that the player can *plan* to drop an enemy before its turn — and feel smart when they do.

**Initiative tray above the arena.** A horizontal strip showing this round's queue left-to-right: each actor as their emoji + a small init number underneath (`🧝19` / `👺17` / `🧙14` / `🐗11`). Big and chunky to match the cartoon-UI vibe — beveled tiles, rounded corners, warm wood palette.

State machinery the tray needs to read:
- **Current actor**: glowing yellow border + a subtle bounce/pulse animation.
- **Acted already**: greyed out + slight rotation or "✓" stamp so the player can see they've passed.
- **Upcoming**: normal tile, full color.
- **Dead mid-round**: animate the tile out (puff/💨) and slide remaining tiles left. *This is the tactical money shot* — when the player kills the witch before her turn, she literally vanishes from the queue and the player can see they earned that.
- **Skipping turn**: stunned/charmed actors render struck-through with the status emoji (`💫 / 💖`) overlaid; tooltip says "SKIPS".

**Per-enemy speed badge.** Also stamp each enemy's `speed + d10` roll as a small chip on the enemy tile itself (next to the HP bar), so a player choosing a target can compare quickly without scrolling up to the tray. Highlight the badge yellow on the current actor.

**"YOU GO IN N" label.** A loud, friendly cue below the tray (or on the player's tile): `▶ YOUR TURN!` / `⏳ YOU GO IN 2` / `⏳ YOU GO NEXT`. Reinforces tactical decisions even when the tray is dense. Reads to new players who haven't internalized the tray yet.

**Round-start animation.** At `beginRound`, animate the tray building left-to-right — each tile pops in with its init number rolling up from 0 like a slot machine. ~600 ms total; pleasant, telegraphs that initiative IS re-rolled each round (otherwise players assume a fixed order).

**Tactical clarity is the goal.** A player should be able to read three things at a glance:
1. Whose turn is it RIGHT NOW.
2. Who goes between now and the player's next action.
3. Which enemy is most worth killing first to remove the biggest upcoming threat (heaviest hitter? next to act? caster about to fire?).

#3 is where the tray earns its keep. Pair with **#2** (hide enemy stats): even with stats hidden, the tray gives the player real, actionable info — *threat ordering by initiative* — without leaking damage numbers. The bestiary (#3) handles "how scary is this archetype"; the tray handles "and they're acting NEXT".

**Implementation hooks:**

- `beginRound` already builds an ordered queue — surface it as a state slot (`roundQueue: [{actor, init, acted}]`) the React tree can subscribe to.
- New `<InitiativeTray>` component placed above (or stacked with) the arena. Reads `roundQueue` + the in-flight current actor.
- `enemyAct` / `resumeAfterPlayer` / kill-mid-round paths all need to update the queue (mark acted, splice out dead actors). Splice should animate.
- Stun/charm: if the dispatcher already skips them mid-queue, just stamp their tile in the tray as struck-through; if they're filtered out of `roundQueue` entirely, render them in the tray anyway with the `SKIPS` overlay so the player understands why no one's at that slot.
- Multi-enemy rooms (big-room scaling is shipped; up to 5–6 foes on f17–20 via `bigRoomCountRange`): test the tray with 7 tiles (player + 6 enemies) and confirm it doesn't crowd the arena on phone widths. May need to shrink tile size or wrap to two rows past 4 actors.
