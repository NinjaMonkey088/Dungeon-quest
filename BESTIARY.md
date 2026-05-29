# Bestiary — Dungeon Quest

All creature data — enemies, wave bosses, deep bosses, spawn pools, behaviors. Source of truth for design; live numbers in `ENEMY_DEFS` / `WAVE_BOSSES` / `BOSS_DEFS` / `WAVES` ([index.html:381](index.html:381) onward).

> **Companion docs:** [CLAUDE.md](CLAUDE.md) (mechanics & combat resolver) · [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md) (spell math) · [INVENTORY.md](INVENTORY.md) (items) · [STORY.md](STORY.md) (Baron arc, lieutenant snarls).

---

## 1. Regular enemies (`ENEMY_DEFS` [381](index.html:381))

15 entries. All carry: emoji, niche, `maxHp`, `attack`, `armor`, `hitChance`, `dodgeChance`, `speed`, `xp`, `goldMin/Max`, an `attrs` block (ability scores used as shrug-roll mods), and optionally a `spell` def + `spellChance` and behavior flags.

| Enemy | Emoji | Niche | HP | ATK | ARM | HIT | DOD | SPD | XP | Gold | Spell (chance) | Flags |
|---|---|---|--:|--:|--:|--:|--:|--:|--:|---|---|---|
| Goblin | 👺 | FODDER | 4 | 0 | 0 | 55 | 0 | 8 | 1 | 1–2 | — | — |
| Boar | 🐗 | ACCURATE | 5 | 1 | 0 | 85 | 0 | 10 | 1 | 1–3 | — | — |
| Zombie | 🧟 | TANK | 12 | 1 | 0 | 50 | 0 | 6 | 2 | 2–4 | — | — |
| Giant Lizard | 🦎 | EVASIVE | 5 | 2 | 0 | 65 | 40 | 14 | 2 | 2–4 | — | — |
| Dark Wizard | 🧙 | GLASS CANNON | 3 | 5 | 0 | 80 | 0 | 11 | 2 | 3–5 | FIREBOLT 100% (mod 3, 4 dmg, burn, avoid) | — |
| Fairy | 🦋 | ARMORED | 7 | 2 | 2 | 55 | 0 | 12 | 2 | 3–5 | FAY CHARM 50% (mod 2, 2 dmg, avoid) | — |
| Cursed Sword | 🗡️ | HEAVY ARMOR | 8 | 3 | 4 | 55 | 0 | 7 | 3 | 4–6 | — | — |
| Ghost | 🌪️ | SWIFT | 4 | 3 | 0 | 70 | 40 | 14 | 3 | 3–6 | SPECTRAL TOUCH 40% (mod 2, 3 dmg, avoid) | — |
| Skeleton | 💀 | DEADEYE | 6 | 3 | 1 | 95 | 0 | 10 | 3 | 3–6 | — | — |
| Archer | 🏹 | SKIRMISHER | 4 | 1 | 0 | 90 | 25 | 13 | 2 | 2–4 | — | `fleesWhenLow` |
| Berserker | 🪓 | BERSERKER | 9 | 2 | 0 | 60 | 0 | 9 | 3 | 3–6 | — | `enragesWhenLow` |
| Phantom | 🌑 | CURSED | 6 | 3 | 1 | 70 | 0 | 11 | 2 | 2–5 | DREAD CURSE 50% (mod 3, 3 dmg, bleed, both) | `bleedsOnCrit` |
| Imp | 🦇 | FIRE BAT | 3 | 2 | 0 | 75 | 15 | 15 | 2 | 2–4 | HELLFIRE 50% (mod 2, 3 dmg, burn, both) | `burnsOnHit` |
| Ogre | 🫧 | STUNNING | 5 | 2 | 1 | 70 | 0 | 7 | 2 | 3–5 | — | `stunsOnHit` |
| Witch | 🧙‍♀️ | HEALER | 6 | 1 | 0 | 60 | 0 | 9 | 3 | 3–5 | HEX 60% (mod 3, 1 dmg, poison, status) | `healsAllies` |

Pre-baked `attrs` (ability scores) default to ~10s with thematic bumps (Zombie high GUTS / low ZIP, Dark Wizard high WITS, Witch high SAVVY, etc.). See `ENEMY_DEFS` source for exact values — used as the **defender's mod** on player-cast shrug rolls.

### Behavior flag glossary

- **`fleesWhenLow`** — at ≤30% HP, 35% chance per turn to flee. Fled enemies = half rewards.
- **`enragesWhenLow`** — at ≤50% HP becomes `enraged:true` with +3 ATK.
- **`bleedsOnCrit`** — applies bleed when this enemy crits the player.
- **`burnsOnHit`** — applies burn on every successful hit (not just crits).
- **`stunsOnHit`** — chance to stun the player; player rolls a GUTS shrug to avoid.
- **`healsAllies`** — Witch behavior. See §3.
- **`poisonOnHit`** — applies poison on every successful hit (Giant Cactus, BARON VON BACON).
- **`immuneToCrit`** — crits never fire against this enemy (PHANTOM Lord, BARON VON BACON).

### Witch healer rules (§3)

- Never spawns alone in a room (1+ allies required).
- At most 1 witch per room regardless of size.
- Heal target: most-wounded ally (relative HP%). If alone and < 50% HP, self-heals +2.
- Each successful heal puts the witch on a 2-turn cooldown — she attacks normally during cooldown.
- Witches CAN spawn alongside a boss (boss + minions on deep floors).

---

## 2. Wave bosses (`WAVE_BOSSES` [430](index.html:430))

Bosses for waves 1–4. Each is `isBoss:true`. Get a one-line snarl on combat start (see [STORY.md → §4](STORY.md)).

| Wave | Boss | Emoji | HP | ATK | ARM | HIT | DOD | SPD | XP | Gold | Spell |
|--:|---|---|--:|--:|--:|--:|--:|--:|--:|---|---|
| 1 | KING Boar | 🐗 | 9 | 3 | 1 | 90 | 0 | 9 | 3 | 3–6 | — |
| 2 | Zombie LORD | 🧟 | 20 | 2 | 1 | 100 | 0 | 5 | 4 | 4–8 | — |
| 3 | ARCH Dark Wizard | 🧙 | 7 | 7 | 0 | 100 | 40 | 12 | 4 | 5–9 | FIRESTORM 65% (mod 5, 6 dmg, burn, half) |
| 4 | IRON Cursed Sword | 🗡️ | 15 | 5 | 5 | 100 | 20 | 8 | 5 | 6–11 | — |

---

## 3. Deep bosses (`BOSS_DEFS` [442](index.html:442))

Wave 5+ cycles these four with `bonus` scaling (HP + bonus, ATK + floor(bonus/2)). After several cycles boss gets "ELDER" prefix. Floor 20 = final boss = guaranteed `gamewon`.

| Boss | Emoji | HP | ATK | ARM | HIT | DOD | SPD | XP | Gold | Spell | Flags |
|---|---|--:|--:|--:|--:|--:|--:|--:|---|---|---|
| Dragon | 🐲 | 20 | 7 | 5 | 100 | 15 | 11 | 5 | 8–12 | DRAGON BREATH 40% (mod 4, 5 dmg, burn, half) | — |
| Giant Cactus | 🌵 | 25 | 8 | 4 | 100 | 15 | 6 | 6 | 10–16 | NEEDLE VOLLEY 35% (mod 4, 4 dmg, poison, both) | `poisonOnHit` |
| PHANTOM Lord | 👻 | 30 | 9 | 0 | 100 | 60 | 14 | 7 | 12–18 | SOUL SCREAM 60% (mod 5, 5 dmg, bleed, both) | `immuneToCrit` |
| BARON VON BACON | 🐷 | 40 | 12 | 8 | 100 | 30 | 12 | 10 | 15–25 | ABYSSAL BOLT 50% (mod 6, 7 dmg, poison, half) | `poisonOnHit`, `immuneToCrit` |

> **BARON VON BACON is the final boss** — floor-20 kill triggers `gamewon`. Narrative arc, voice modes (`first`/`elder`/`final`), and lieutenant snarls in [STORY.md](STORY.md).

---

## 4. Boss minion scaling (`bossMinionCount`)

Boss-room composition by floor:

| Floors | Composition |
|---|---|
| f1–9 | solo boss |
| f10–13 | boss + 1 minion |
| f14–17 | boss + 2 minions |
| f18–20 | boss + 3 minions |

Minions pulled from `pickEnemy(level)` (current wave's `WAVES` pool). 1-Witch-per-room cap still applies — a single Witch CAN spawn alongside the boss ("kill the healer first" decision). Boss `enemyKnown:true` preserved — boss tile never surprises.

---

## 5. Spawn pools (`WAVES` [421](index.html:421))

Per-wave weighted pool. Waves 5+ reuse the wave-5 list. `pickEnemy(level)` rolls from this; `pickEnemy({allowHealer:false})` filters out Witch.

| Wave | Pool (entries are `[name, weight]`) |
|--:|---|
| 1 | Goblin × 10 |
| 2 | Boar × 5, Goblin × 3, Archer × 2 |
| 3 | Zombie × 3, Giant Lizard × 3, Fairy × 2, Boar × 2, Archer × 2, Berserker × 1, Skeleton × 1 |
| 4 | Dark Wizard × 3, Imp × 3, Fairy × 2, Zombie × 1, Giant Lizard × 1, Berserker × 2, Phantom × 2, Skeleton × 1, Ogre × 1, Witch × 1 |
| 5+ | Cursed Sword × 3, Ghost × 3, Dark Wizard × 2, Fairy × 2, Imp × 2, Berserker × 2, Phantom × 2, Skeleton × 1, Ogre × 2, Witch × 2 |

---

## 6. Spell casters — quick reference

Casters carry a `spell` def + `spellChance`. Each turn they roll `pct(spellChance)` to cast instead of swing melee. Full spell math (DC, shrug, onSave) → [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md). Per-caster summary:

| Enemy | Stat | Spell | Chance | Effect | onSave |
|---|---|---|--:|---|---|
| Dark Wizard | INT | FIREBOLT | 100% | 4 dmg, burn | avoid |
| Fairy | CHA | FAY CHARM | 50% | 2 dmg | avoid |
| Ghost | WIS | SPECTRAL TOUCH | 40% | 3 dmg | avoid |
| Phantom | CHA | DREAD CURSE | 50% | 3 dmg, bleed | both |
| Imp | CHA | HELLFIRE | 50% | 3 dmg, burn | both |
| Witch | WIS | HEX | 60% | 1 dmg, poison | status |
| ARCH Dark Wizard | INT | FIRESTORM | 65% | 6 dmg, burn | half |
| Dragon | CHA | DRAGON BREATH | 40% | 5 dmg, burn | half |
| Giant Cactus | WIS | NEEDLE VOLLEY | 35% | 4 dmg, poison | both |
| PHANTOM Lord | CHA | SOUL SCREAM | 60% | 5 dmg, bleed | both |
| BARON VON BACON | INT | ABYSSAL BOLT | 50% | 7 dmg, poison | half |

> Player counter-play: invest in the matching ability to widen the shrug margin. WITS-heavy heroes shrug off Dark Wizards; CHARM shrugs Phantoms / Imps; SAVVY shrugs Witches / Ghosts.

---

## 7. Magic specialty plan

Layers the [MAGIC_SYSTEM.md](MAGIC_SYSTEM.md) knack framework onto this roster — tags current casters by specialty, calls out off-spec damage, and queues new caster adds per knack. Keep enemy spells **simple** — one effect per spell.

### Specialty cheat sheet

| High stat | Knack | Specialty | Engine-ready? |
|-----------|-------|-----------|---------------|
| INT | 🤓 WHIZ-BANG | damage zaps, DoTs, AoE | ✅ today (current `spell` field) |
| WIS | 🌿 COZY | ally heal / shield / buff | ⚠️ needs hooks beyond `spell` |
| CHA | 😏 SASS | control, fear, debuff | ⚠️ needs hooks beyond `spell` |

The Witch's existing `healsAllies` flag is the precedent — non-damage effects ride **parallel flags**, not the `spell` field.

### Tagging the current casters (uses §6)

- **🤓 WHIZ-BANG on-spec ★** — Dark Wizard, ARCH Dark Wizard, BARON VON BACON. All INT-stat damage spells. ✓
- **🌿 COZY** — Witch's `healsAllies` ★ is the only true on-spec ally-support today; her HEX is off-spec damage. Ghost (WIS) and Giant Cactus (WIS) cast damage — ⚠ off-spec until COZY hooks land.
- **😏 SASS** — all currently off-spec damage. Fairy's FAY CHARM is *named* "charm" but resolves as 2 dmg. Phantom, Imp, Dragon, PHANTOM Lord same story. Reskin paths in §7.5 below.

Caster-vs-pure-melee ratio: **11 / 23** — slightly under the "most enemies use magic" target ([MAGIC_SYSTEM.md §5](MAGIC_SYSTEM.md)).

### Engine extensions needed

Two batches of hooks unlock COZY and SASS as true specialties. Both live **parallel to `spell`**, like `healsAllies`.

**COZY ally hooks**
- `healsAllies: { amount, target }` — generalize the existing flag's hardcoded +3
- `shieldsAllies: { armor, turns }`
- `buffsAllies: { stat, val, turns }`
- `raisesDead: { hp }` — revive once per battle

**SASS control hooks**
- `controls: { type, turns }` — stun / charm; mirror player path
- `debuffs: { stat, val, turns }` — −ATK / −HIT / −DODGE on player
- `silences: turns` — player can't cast spells
- `frightens: { chance, turns }` — flee-attempt chance per turn

Both batches plug into `enemyAct` ([index.html:2212](index.html:2212)) alongside the existing spell-vs-melee dispatch. A single per-turn "ally-support / debuff-aura" check runs before the cast-vs-melee decision (mirroring how `healsAllies` already gates at [index.html:2229](index.html:2229)).

### New caster additions

One-liner each. Full `ENEMY_DEFS` rows TBD in the dedicated bestiary chat.

**🤓 WHIZ-BANG (wireable today)**

| Enemy | High stat | Spell | Effect |
|-------|----------|-------|--------|
| 💀 Skeleton Mage | INT 14 | BONE DART · 3 dmg, ZIP half | dart volley |
| 🐍 Slime Lord | INT 12 | ACID SPRAY · 2 dmg, GUTS both, poison | corrosive cloud |
| 🔥 Fire Sprite | INT 14 | EMBER COUGH · 2 dmg, GUTS both, burn | tiny burn DoT |
| ⚡ Storm Elemental | INT 16 | STORM JAB · 5 dmg, ZIP half | high single-target |
| 🦟 Swarm Caller | INT 12 | BUG ZAP · 1 dmg, ZIP half, hits both | weak AoE |
| 🕸️ Spider Queen | INT 12 | WEB SHOT · 1 dmg, ZIP both, stun | trap + tiny damage |

**🌿 COZY (after engine work)**

| Enemy | High stat | Hook | Effect |
|-------|----------|------|--------|
| 🦎 Lizard Shaman | WIS 14 | `shieldsAllies` | +2 ARM on ally, 3t |
| 🐺 Wolf Alpha | WIS 12 | `buffsAllies` | +1 ATK all allies, 2t |
| ⚰️ Necromancer | WIS 16 / INT 14 | `raisesDead` + small bone zap | revive once + 3 dmg |
| 🙏 Cultist Priest | WIS 14 | `healsAllies` (generalized to +4) | bigger heal than Witch |

**😏 SASS (mostly after engine work)**

| Enemy | High stat | Hook | Effect |
|-------|----------|------|--------|
| 👁️ Banshee | CHA 16 | `controls` | HYPNO STARE — skip 1 turn |
| 🩻 Wraith | CHA 14 / WIS 12 | `controls` | BONE GRIP — stun 1t |
| 💢 Ogre Captain | CHA 12 / STR 16 | `debuffs` | INTIMIDATE — −2 player ATK, 2t |
| 🔇 Hex Hag | CHA 16 | `silences` | SILENCE GLARE — block player spells 2t |
| 💀 Lich Apprentice | CHA 14 / WIS 14 | (today's bleed-DoT shape — wireable now) | DEATH WHISPER · 1 dmg, bleed |
| 👻 Spectral Mocker | CHA 14 | `frightens` | SPOOK — 25% flee × 2t |

Lich Apprentice is wireable today as a bleed-flavored damage spell — quick win.

### Reskinning current CHA casters

Once SASS hooks land, two paths for Fairy / Phantom / Imp:

- **Path A — true SASS:** Fairy's FAY CHARM becomes actual charm (`controls`); Phantom's DREAD CURSE becomes a debuff (`debuffs`); Imp's HELLFIRE stays as off-spec burn damage (fire is its identity).
- **Path B — leave as off-spec damage:** mirror the player roster, where SASS off-spec damage exists (SICK BURN, STINK EYE, etc.).

**Recommended:** Path B for shipped Fairy / Phantom / Imp (they play fine), Path A for new SASS adds so the control specialty actually exists in-game. Dragon and PHANTOM Lord (bosses) stay as-is.

### Implementation phases

- **Phase 1 — WHIZ-BANG fill-out** (no engine work). Add Skeleton Mage, Slime Lord, Storm Elemental, plus Lich Apprentice to mid-floor waves.
- **Phase 2 — Engine hooks.** Generalize `healsAllies`; add `shieldsAllies` / `buffsAllies` / `raisesDead` / `controls` / `debuffs` / `silences` / `frightens`.
- **Phase 3 — COZY + SASS additions.** Wire in Lizard Shaman, Wolf Alpha, Necromancer, Cultist Priest, Banshee, Wraith, Ogre Captain, Hex Hag, Spectral Mocker. Reskin Fairy (Path A from §7.5).
- **Phase 4 — Pure-melee gap-fill** (optional). Give Skeleton a tiny WHIZ-BANG zap to nudge the magic-default ratio above 50%.

### Open questions

- **Caster saturation per wave.** Adding 16 new casters risks oversaturating — pace by floor introduction.
- **Reskin shipped CHA casters?** Path A vs B for Fairy / Phantom / Imp (see §7.5).
- **Boss diversity.** All four `BOSS_DEFS` are damage casters or brutes. Room for a COZY (heals minions) or SASS (locks player) boss?
- **Magic-null slot.** A "magic-eater" enemy that drains player spell-points or counters spells, fleshing out the magic-null exception?
- **Naming.** Banshee / Wraith / Hex Hag / Necromancer lean straight-fantasy; game tone is goofy — refine in the dedicated chat (Hex Hag → something punchier in the KABLOOEY family).

---

## 8. Surprise & initiative

- Player `speed = 10 + zipMod` vs enemy `speed`. Higher acts first; **player wins ties**.
- Surprise: `room.enemyKnown === false && waveNumber >= 3` → enemies get **+5 SPD** for the battle (initiative-rolled, not instant attack).
- Bosses are pinned `enemyKnown:true` — never surprised.
- See [CLAUDE.md → Combat flow](CLAUDE.md) for round-by-round mechanics (per-round init, `beginRound` / `runQueueFrom` / `enemyAct`).

---

## 9. Helpers (in `index.html`)

| Fn | Line | Notes |
|---|---|---|
| `pickEnemy(level, {allowHealer})` | [964](index.html:964) | Rolls from `WAVES[wave-1]`; filters Witch when `allowHealer:false` |
| `pickRoomEnemies(level, count)` | [984](index.html:984) | N enemies for a room. Second+ enemies use `allowHealer:false` |
| `pickBoss(waveNumber, level)` | [996](index.html:996) | Waves 1–4 → `WAVE_BOSSES`; wave 5+ cycles `BOSS_DEFS` with bonus scaling + "ELDER" prefix |
| `bossMinionCount(floorNum)` | [816](index.html:816) | See §4 |
| `applyRoomToEnemy(e, room)` | [935](index.html:935) | Runs `room.enemyMod` if present (big rooms = +gold; bridge = +SPD/gold; shrine = −hit/gold) |
| `scaleHit(base, level)` | [961](index.html:961) | Enemy hitChance ramps to 100 by L15, then +1/level |

Combat resolvers (pure): `enemyAttack` ([1124](index.html:1124)) returns `{dmgToPlayer, counterDmg, results, gotCrit, didPoison, didBurn, didBleed, didStun}`; `enemySpellAttack` ([1170](index.html:1170)) returns the same shape. See [CLAUDE.md → Function index](CLAUDE.md) for full handler map.

---

## 10. Open wishlist (relevant items)

- **[#3 Bestiary on the title screen](CLAUDE.md)** — in-game progressive-reveal panel keyed to kill count. This doc is the data source.
- **[#6 Enemy attacks that drain base stats](CLAUDE.md)** — new caster archetype.
- **[#7 Mid-game difficulty curve](CLAUDE.md)** — elite variants, archetype additions, scaling tweaks.

See [CLAUDE.md → Wishlist](CLAUDE.md) for full design notes.
