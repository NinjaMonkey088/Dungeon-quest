# Story — The Baron Von Bacon Arc

The narrative spine of **Dungeon Quest**. The game is a 20-floor descent against **🐷 BARON VON BACON**, a landlord-shaped pig who is expanding his dungeon UP into the player's town. Monsters are "eviction notices with teeth." All narrative content currently lives in [index.html:164–241](index.html:164) (the `STORY` / `BARON_TAUNTS` / `LIEUTENANT_SNARLS` tables) and is rendered by the `story` phase ([index.html:2601](index.html:2601)).

> **Tone:** soft, cartoonish, whimsical, silly — never grim. Goofy banter, gentle stakes, exclamation marks welcome. Matches the wooden game-UI-kit aesthetic. (See [CLAUDE.md → Editing conventions](CLAUDE.md).)
>
> **Original game — NOT D&D.** No references to *Dungeons & Dragons*, its books, classes, or trademarked content. The villain, the town, and every named beat is our own.

---

## 1. Premise

A flyer is nailed to the tavern door:

> "WANTED — 1 (one) HERO. Must like stairs.
>   Monster problem. Pls hurry. — The Town"

Monsters are pouring **up** out of the dungeon. Behind it all is **BARON VON BACON 🐷** — landlord, schemer, wearer of a truly stupid hat. His scheme: expand his dungeon UPWARD into the player's town. The monsters are his eviction notices.

Twenty floors stand between the hero and the Baron's office. Down they go.

Source: `STORY.prologue` ([index.html:169](index.html:169)).

---

## 2. The villain — Baron Von Bacon

A pig in a stupid hat who runs the dungeon as if it were a building he is the landlord of. Speaks in **lease-and-eviction language**: "section 12 of the lease you do NOT have," "I'll discuss your security deposit," "EVICTION TIME!" He heckles the hero over a **"dungeon intercom"** (`*crackle*`) at floor milestones, getting progressively more rattled.

He **recurs as a wave-5+ boss** — every 4 floors his stat block cycles back around with bonus scaling. In-fiction, that's him "taking the express stairs down" after each defeat, retreating one tier deeper. Last stand is floor 20.

### Baron's three voice modes (`BARON_TAUNTS`, [index.html:227](index.html:227))

| Mode  | Trigger | Line |
|-------|---------|------|
| `first` | First time facing the Baron | "Ah — the hero from the flyer. EVICTION TIME!" |
| `elder` | Battered rematch (ELDER-prefixed cycle) | "You AGAIN? Ugh, fine. I've been doing squats." |
| `final` | Floor 20 | "My LAST floor. Make this quick, I have a 4 o'clock." |

Routing lives in `startBattle` ([index.html:1769](index.html:1769)) — picks `final` if `waveNumber >= 20`, else `elder` if the boss name starts with `ELDER`, else `first`.

---

## 3. Floor beats (the intercom heckles)

Triggered by `descendStairs` ([index.html:1656](index.html:1656)) when arriving on floors 5/10/15/20 — sets `storyBeat` and switches to the `story` phase, which renders a full-screen dialogue panel ([index.html:2601](index.html:2601)) with a single button to continue.

| Floor | Title | Vibe | Source |
|------:|-------|------|--------|
| **0** (start) | 📜 A FLYER ON THE TAVERN DOOR | Setup — the job, the villain, the stakes | `STORY.prologue` ([index.html:169](index.html:169)) |
| **5** | 📢 DUNGEON INTERCOM — FLOOR 5 | First heckle — Baron tries the "you don't have a lease" angle, gives up mid-line | `STORY.beats[5]` ([index.html:189](index.html:189)) |
| **10** | 📢 DUNGEON INTERCOM — FLOOR 10 | Annoyed — "you were meant to be a tasteful smear by now" | `STORY.beats[10]` ([index.html:195](index.html:195)) |
| **15** | 📢 DUNGEON INTERCOM — FLOOR 15 | Tired — admits he just keeps taking the stairs down | `STORY.beats[15]` ([index.html:201](index.html:201)) |
| **20** | 📢 FLOOR 20 — THE CORNER OFFICE | The summons — "the penthouse of the basement" | `STORY.beats[20]` ([index.html:207](index.html:207)) |
| **WIN** | (ending, no title) | "The stupid hat tumbles off. The Monster Maker sputters, coughs, and quits." | `STORY.ending` ([index.html:214](index.html:214)), rendered at [index.html:2749](index.html:2749) |

### Implementation notes
- The villain-voice color is **pink** (`#f078c8`); a friendly-narrator beat is **gold** (`#f8e000`). Field: `storyBeat.villain` ([index.html:2603](index.html:2603)).
- Every beat has a custom `button` label (`▶ ENTER THE DUNGEON`, `▶ KEEP DESCENDING`, `▶ FACE THE BARON`) — defaults to `▶ ONWARD`.
- Beats are gated to descent only (not on first floor entry, not on rests). Trap rooms, shops, level-ups, and combat have **zero narrative wiring** today.

---

## 4. Lieutenant snarls (per-boss one-liners)

Every boss except the Baron is one of his hired goons. When a boss battle starts, `startBattle` ([index.html:1770](index.html:1770)) emits a single italicized snarl from `LIEUTENANT_SNARLS` ([index.html:233](index.html:233)). All bend the dialogue back toward the Baron, even when the boss has no other personality.

| Boss | Snarl |
|------|-------|
| **KING Boar** 🐗 | "BARON VON BACON said I could keep your boots!" |
| **Zombie LORD** 🧟 | "Tenant... of the month... braaains..." |
| **ARCH Dark Wizard** 🧙 | "The Baron's rent is due — I collect in FIREBALLS." |
| **IRON Cursed Sword** 🗡️ | "No one evicts the Baron's floor. Not today." |
| **Dragon** 🐲 | "The Baron promised me the top three floors. MOVE." |
| **Giant Cactus** 🌵 | "I'm the Baron's landscaping. Prickly, isn't it?" |
| **PHANTOM Lord** 👻 | "The Baron haunts the lease. I just haunt." |

Fallback if a boss has no entry: `"✦ One of BARON VON BACON's lieutenants blocks the stairs!"`

---

## 5. The ending

Triggered when the Floor 20 Baron dies. `resolveVictory` ([index.html:2047](index.html:2047)) sets `phase = "gamewon"`; the `gamewon` screen prints `STORY.ending`:

> The stupid hat tumbles off. The **Monster Maker**
> sputters, coughs, and quits. The dungeon stops growing.
>
> The ceiling holds. Your town stays a town —
> and decidedly NOT the Baron's newest basement.
>
> Far above, a tavern erupts in cheers
> (and quietly opens a tab in your name).

The "Monster Maker" is mentioned **only here** — it has no prior setup in the prologue or any intercom beat.

---

## 6. What the story currently does NOT cover

These are gaps a story pass would address — not bugs in what exists, just unwritten territory:

- **15 of 20 floors are narratively silent.** Beats only fire on floors 5/10/15/20.
- **The hero has no characterization.** They have a name, an emoji, and a stat block. No backstory option at creation, no reaction lines in combat or after beats.
- **The Town is invisible.** Mentioned in the flyer and the ending, never anywhere between.
- **No supporting cast.** No companions, no shopkeeper, no recurring NPC. The shop (post-boss) is a list of items, not a person.
- **The "Monster Maker" is a deus ex machina.** Named only in the ending payoff — never foreshadowed.
- **No second act / no twist.** The Baron is the entire conflict; the arc is linear "go down, then go down more."
- **Shops, traps, rests, and trap rooms have no flavor wiring.** They're purely mechanical screens.
- **Death has no narrative.** The `lose` phase ([index.html:1421](index.html:1421)) just shows a game-over panel — the Baron doesn't get a victory gloat.
- **The win-screen tab joke is the entire emotional payoff.** No epilogue scene, no specific named townsfolk reaction, no callbacks to anything the player did in the run.

---

## 7. Editing conventions

- **All beats live in the `STORY` object** ([index.html:168](index.html:168)) — `prologue`, `beats[N]`, `ending`. Adding floor 7's intercom = add `beats[7]: { title, villain, lines, button }`.
- **`villain: true`** = pink heckle text. **`villain: false`** = gold narrator text.
- **Lines are an array of strings.** Empty string `""` = blank line (renders at half-height — see [index.html:2605](index.html:2605)).
- **Keep lines ≤ ~50 characters** — they're hand-wrapped, no word-wrap. Look at existing beats for length.
- **ALL-CAPS for emphasis** matches the in-game UI; "the Baron" stays mixed-case in narrator beats.
- **Stay in voice.** Baron = lease/eviction/corporate-landlord. Narrator = warm, slightly snarky, charming. Both are silly.
- **Lieutenant snarls go in `LIEUTENANT_SNARLS`** ([index.html:233](index.html:233)), keyed by exact boss `name`. Add a new boss → add its snarl, or it falls through to the generic line.
- **Never reference D&D** — no spell names like FIREBALL outside of in-character flavor (the wizard's snarl is fine because it's his joke), no class names, no trademarked terms.

---

## 8. Current pass — scope & decisions

Scope label: **Light polish** — no rewrite of the existing Baron arc. Existing prologue, intercom beats, ending, taunts, and snarls all stay. New material plugs into what's there.

### In scope (this pass)

| Plan | What | Status |
|------|------|--------|
| **Voice & tone fix** | The existing copy reads as AI-generated (see §9). Rework voice before adding new text — new beats inherit the new voice, not the old one. | Pending tone-direction pick |
| **Town & supporting cast** | Give the Town a name; design 2–3 recurring NPCs. They are the source of *both* the shopkeeper and the heroes (see Run-handoff). | Not started |
| **Hero creation backstory pick** | One-line pick at character creation (Why are you here? — revenge / paid / dragged into it / etc.) that gets called back to in 2–3 beats. | Not started |
| **Run-handoff on death** | Player death implies the *next* run is a different villager from the Town who has picked up the flyer. The Town is the literal hero pool. This makes death narratively meaningful and naturally amplifies the Town & cast plan. | Not started — needs creation-screen support |
| **Death = Baron gloat** | Replace flat `lose` panel with a Baron taunt; ties into Run-handoff ("Next!"). | Not started |
| **Trap rooms = snarky Baron signs** | Each trap room shows a one-liner notice ("NOTICE TO TRESPASSERS — Mgmt") when entered. Traps feel like Baron pettiness, not random hazards. | Not started |
| **Shop = shopkeeper** | Post-boss shop run by a named NPC (from Town & cast), 2–3 lines that rotate per floor. | Not started |
| **Rest = campfire voice** | Make Camp shows a short reflective line per rest. Source TBD (narrator / inner voice / companion). | Not started |

### Parked for later

| Idea | Reason parked |
|------|---------------|
| Fill the 15 silent floors with beats | Hold until voice is settled — don't 4× the AI-toned text and then rewrite it all. Revisit once voice and Town/cast exist. |
| Foreshadow the Monster Maker | Hold for a future pass. Belongs with a Heavy-scope rewrite of the arc, not a Light polish. |
| Mid-game twist / second-act complication | Same — out of scope for Light polish. |

---

## 9. Voice & tone — moving away from the default AI register

**The problem.** The existing copy (prologue, beats, ending) reads as AI-generated. Some specific tells:

| Tell | Where it shows up | Why it reads as AI |
|------|-------------------|-------------------|
| Symmetric "X. Y? Z." rhythm | "His scheme: expand his dungeon UP. The monsters? Eviction notices. With teeth." | Pattern repeats across paragraphs. Human comedy varies tempo. |
| Lists of three | "landlord, schemer, wearer of a truly stupid hat." | Tidy tricolons are an LLM rhythm. Use occasionally, not always. |
| Every line lands a joke | Each beat ends on a quip. | Real characters drop the bit sometimes; LLMs perform constantly. |
| Punctuation-as-tone | EMPHASIS CAPS + ! on most lines. | Diluted by repetition — when everything is loud, nothing is. |
| Abstract proper nouns | "your town", "the Town", "the tavern" — none are named. | LLMs default to abstract because specifics carry risk. Names anchor voice. |
| Parenthetical asides | "(and quietly opens a tab in your name)" | Cute writerly tic, overused. |
| One register only | Baron is uniformly "exasperated landlord." | Real voices shift — smug, tired, defensive, weirdly tender. |
| Stage directions on default | `*crackle*` | Fine occasionally; reads as filler when reflexive. |

**The plan.** Pick a voice direction (see question below), then rewrite the existing 5 beats + ending against it *before* writing any new material. New Town/cast/shop/rest/death copy follows the chosen direction from day one.

