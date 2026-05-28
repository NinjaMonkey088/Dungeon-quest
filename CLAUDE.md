# Dungeon Quest — `index.html` Reference

Single-file React 18 (Babel standalone, CDN) turn-based dungeon crawler. No build step. **Soft, cartoonish, whimsical** vibe — rounded **Fredoka** font, warm wood-toned panels, beveled candy buttons, emoji sprites, bouncy CSS, goofy combat banter. (Originally an 8-bit/pixel look with Press Start 2P; softened toward a wooden cartoon game-UI-kit aesthetic.) All code lives in `index.html` (~2800 lines). Edit it in place — never regenerate.

**Game goal**: clear 20 floors. Floor-20 boss kill → `setPhase("gamewon")`.

> **Original game — NOT D&D.** Dungeon Quest is its own silly little world. Never reference *Dungeons & Dragons*, its rulebooks, class names, or any trademarked/Product-Identity content in code, UI, or docs. Game *mechanics* (a d20 roll, an ability modifier, a half-scale HP pool) are generic and fair to use; *names* are always reskinned to original cartoon terms (see [Naming convention](#naming-convention--cartoon-reskin-not-dd)). When in doubt, make it sillier and more original — that is also the safest legal direction.

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
| `ENEMY_DEFS` | [173](index.html:173) | 15 enemies. Each: emoji, niche, maxHp, attack, armor, hitChance, dodgeChance, speed, xp, goldMin/Max, optional spell/spellChance/attrs, behavior flags |
| `WAVES` | [213](index.html:213) | Spawn pool weights per wave 1–5 (wave 5+ reuses last) |
| `WAVE_BOSSES` | [222](index.html:222) | 4 wave bosses (waves 1–4) |
| `BOSS_DEFS` | [234](index.html:234) | 4 deep bosses (wave 5+ cycle, scales with `bonus`, "ELDER" prefix at deep cycles) |
| `SHOP_CONSUMABLES` | [249](index.html:249) | 18 items. Heal/cure flags or `onUse(p,b)` for buffs |
| `DICE_PROGRESSION` | [272](index.html:272) | 30 dice (1–5 of D4/6/8/10/12/20) sorted by avg roll |
| `EQUIPMENT_DEFS` | [279](index.html:279) | 35 items: 12 weapons (5 common + 7 rare), 3 shields, 13 armor/accessory, 7 rings |
| `BODY_SLOTS` | [322](index.html:322) | 13 slot defs with grid col/row positions for `BodyLayout` |
| `EMPTY_EQUIP` | [338](index.html:338) | `{...slots:null, rings:[]}` — initial equipment state |
| `STAT_LABEL` | [339](index.html:339) | Abbreviation map: atkDmg→ATK, hitChance→HIT, etc. **Use in UI strings.** |
| `ROOM_TYPES` | [556](index.html:556) | 4 fight types: small, big (`maybeDouble:true`), bridge, shrine. Stairs is NOT a room type (created on boss kill) |
| `TRAP_ROOM` | [575](index.html:575) | Trap room def (`isTrap:true`, no enemies). `rollEnemyCount(type)` + `TRAP_ROOM_CHANCE` (20) drive room composition |
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
| `enterRest()` / `descendStairs()` | [1351](index.html:1351) | Flee/rest; descend bumps wave+1, regenerates floor, `playerPos={0,0}` |
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
With the round system, **DOTs tick once at the top of each round** (in `beginRound`) — enemy poison (−1 HP), burn (−2 HP), bleed (counter −1); then the player's poison/burn/bleed. Lethal player DOT → `setPhase("lose")`. Flee checks also run at round top. This replaces the old "tick at start of the enemy phase" timing. Player-side `bleed > 0` still applies a ×1.5 incoming-damage multiplier inside `enemyAct`.

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

### Spells (cartoon names — `PLAYER_SPELLS` [129](index.html:129))

| id | Name | Stat / req / cost | Effect |
|---|---|---|---|
| `zap_zap` | ZAP-ZAP | WITS 12 / 1 | 1d4, always hits, ignores armor |
| `hotfoot` | HOTFOOT | WITS 14 / 2 | 1d10 + WIT, burn, ZIP shrug → half, ignores armor |
| `boo_boo_fix` | BOO-BOO FIX | SAVVY 12 / 1 | heal 1d8 + SAV |
| `there_there` | THERE-THERE | SAVVY 14 / 2 | heal 2d4 + SAV, cures 1 status |
| `sick_burn` | SICK BURN | CHARM 12 / 1 | 1d4 + CHM, SAV shrug → avoid, ignores armor |
| `bat_eyes` | BAT EYES | CHARM 14 / 2 | SAV shrug → skip turn(s) = spell LV |

Stored as level map: `player.spells = { zap_zap: 2, boo_boo_fix: 1 }`. To reach LV N the relevant ability must be at `spell.req + N − 1`. Each upgrade adds **+1 dmg** (attack), **+1 heal HP** (heal), or **+1 charm turn** (control). Cost is the spell's `cost` field (1–2 pts). Damage/heal is a **die roll** (`spell.die`) + `statBonus` mod + `levelBonus`.

Shrug-off math (player caster): `target = 8 + abilityMod(player[spell.stat]) + floor(playerLevel/2)`. Enemy rolls `d20 + abilityMod(enemy.attrs[saveStat])`. `total > target` = shook it off. Log reads `Zombie ZIP SHRUG 15-2 (13) vs 10 → SHOOK IT OFF!`. `autoHit` spells (ZAP-ZAP) skip the roll.

Shrug-off math (enemy caster): `target = 8 + spell.mod + floor(waveNumber/2)`. Player rolls `d20 + abilityMod(player[spell.ability])`. **Shrug-offs bypass dodge and armor.** No crit, no counter on spells.

| `onSave` | Saved dmg | Saved status |
|---|---|---|
| `avoid` | 0 | none |
| `half` | `floor(dmg/2)` | none |
| `status` | 0 | applied |
| `both` | `floor(dmg/2)` | applied |

### Status effects

| Status | Effect | Cure | Persists between battles? |
|---|---|---|---|
| ☠ Poison | 1 dmg/turn × 3 turns | Antidote / Elixir | **Yes** |
| 🔥 Burn | 2 dmg/turn × 3 turns | Ointment / Elixir | **Yes** |
| 🩸 Bleed | `floor(weapon_avg + atkDmg)` per swing AND turn counter; deepens via Math.max on re-proc; player-side bleed = +50% incoming dmg | Bandaid / Elixir | **Yes** |
| 💫 Stun | Lose your initiative slot this round | Smelling Salt / Elixir | **No** (cleared at `startBattle`) |

### Dice & weapons

- `dieIdx` indexes `DICE_PROGRESSION` (D2 → 40D10).
- Damage: `atkDmg + die roll − armorUsed`. Crits roll the die **twice**.
- Pierce procs / `armorPierce` perk → `armorUsed = 0`.
- **Offhand swings deal 70%** damage. Master Duelist removes the penalty.
- Weapon `hand`: `light` (dual-wieldable, +built-in crit on dagger/shortsword), `one` (shield-compatible), `two` (locks offhand).
- Rare procs per swing on damaging hits: `poisonChance`, `burnChance`, `bleedChance`, `pierceChance`. Arcane Sense (INT 20) adds +5% to each.
- Vampiric weapons: heal 2 HP on crit (either hand).

### Initiative

`player.speed = 10 + dexMod` (+ equipment `stats.speed`). Compared at `startBattle`. Higher acts first; **player wins ties**. Surprise → +5 enemy SPD (gated to floor 3+). Bosses pinned `enemyKnown:true` — never surprised.

### Identify gating (`describeItem` / `labelItem` [504](index.html:504))

| Player state | Label | Description |
|---|---|---|
| `isIlliterate` (INT < 10, mod < 0) | `???` | `??? — TOO ARCANE TO READ`. Slot/hand tag hidden. |
| `0 ≤ identify < 50%` | Real label | Generic category ("VITAL GEAR" / "DEFENSIVE GEAR" / "COMBAT GEAR" / "ENCHANTED GEAR") |
| `identify ≥ 50%` | Real label | Real description |

Dev mode bypasses (sets `intelligence:30, identify:95` in `roomPlayer`).

### Floor scaling

| Floors | Grid | Tiles |
|---|---|---|
| 1–3 | 3×2 | 6 |
| 4–7 | 3×3 | 9 |
| 8–13 | 4×3 | 12 |
| 14–20 | 4×4 | 16 |

Each floor: 1 boss room + (size−1) other rooms. Entrance pinned to `{0,0}`, pre-`visited`. Boss tile shuffled into one of the others. Boss kill → tile transforms to stairs in place; shop unlocks; player walks back to descend.

**Room composition** (in `makeFloorMap`):
- **Entrance (i===0)** — always a single-foe fight. Never a trap, never a swarm (no floor-1 ambush).
- **Every other room** — `TRAP_ROOM_CHANCE` (20%) to be a **trap** instead of a fight; otherwise a fight whose foe count comes from `rollEnemyCount(type)`: **big rooms always ≥2** (30% chance of 3), all other types 35% chance of 2. `pickRoomEnemies` allows at most one healer (Witch) per room.

### Trap rooms (`triggerTrap` [~1432](index.html:1432))

Walking onto an uncleared `isTrap` room (routed in `handleRoomEnter` before `startBattle`) forces a **ZIP (dexterity) shrug-off**: `roll(20) + zipMod + (aware ? 5 : 0)` vs `target = 10 + floor(waveNumber/2)`. `aware` = `room.trapAware` (the awareness/WIS roll at floor-gen) OR devMode. Fail → `roll(6) + floor(waveNumber/2)` damage + 30% poison-dart rider (3 turns); lethal traps end the run. Pass → no damage. Resolves instantly, marks the room `cleared` (shows "SPRUNG"). Map tile + HERE strip render amber with a "TRAP SPOTTED (+5)" hint when aware. Trap rooms count toward the floor's "ROOMS CLEARED" tally.

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

### Recently tuned

- Witch on 2-turn `healCooldown` after each heal ([1893](index.html:1893)).
- Witches never spawn alone or two-to-a-room (`pickEnemy({allowHealer})` filter).
- Light weapons buffed: Dagger +10% crit, Shortsword +5% crit.
- Offhand penalty 50% → 30% (offhand swings do 70%).
- Surprise rounds gated to floor 3+ ([1405](index.html:1405)).

### Recent feature additions

- **UI softening pass** — swapped Press Start 2P for rounded **Fredoka**, warm wood-gradient `body`, `TileBox` → rounded beveled wooden panel, `Btn`/`BTN_VARIANTS` → rounded candy buttons (`.dq-btn` hover/press), and rounded/warmed secondary chrome (floor map + tiles, XP bar, combat arena box, create-screen controls, info strips, DEV button). See [UI tone](#editing-conventions).
- **Camp anytime** — `🏕 MAKE CAMP` on the idle/map screen (not only the post-combat `win` screen); both route through `enterRest`.
- **Gear-modal stats rebuilt** — the `equipOpen` modal's stat block is now a readable cartoon card layout (combat chips + one card per attribute using `STAT_FULL` names + `statEffectLine`, plus perk/special badges), replacing the old cramped raw-`STR/DEX/CON…` text wall (which also broke the no-D&D naming rule).
- **Per-round initiative** — combat re-rolls `speed + d10` for the player + each enemy every round and interleaves turns (`beginRound`/`runQueueFrom`/`enemyAct`/`resumeAfterPlayer` via `combatRef`). Replaced the old `runEnemyTurn` player-phase/enemy-phase split. Log shows `⚡ ROUND N — 🧝19 ▸ 👺9`.
- **Trap rooms** (`TRAP_ROOM`, `triggerTrap`) — 20% of non-entrance rooms; ZIP shrug-off, +5 if `trapAware`.
- **Tougher rooms** — big rooms always ≥2 foes (30% chance of 3); other fight rooms 35% chance of 2; entrance always solo.
- **Cartoon rename** — stats MIGHT/ZIP/GUTS/SAVVY/CHARM/WITS (`STAT_LABEL`/`STAT_FULL`/`statLine`); spells ZAP-ZAP/HOTFOOT/BOO-BOO FIX/THERE-THERE/SICK BURN/BAT EYES; "shrug-off" instead of saves. Internal keys unchanged.
- Dual-wield equip flow rebuilt — explicit `→ OFFHAND` button via `canEquipToOffhand` ([1745](index.html:1745)). `buyItem` + `handleEquipFromInventory` + `canEquipItem` all accept `targetSlot` arg.
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
