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
| **Voice & tone fix** | The existing copy reads as AI-generated (see §9). Rework voice before adding new text — new beats inherit the new voice, not the old one. | **Done.** Direction locked (§9.1). Prologue + 4 intercom beats + ending + 3 Baron taunts + 7 lieutenant snarls all rewritten and live in index.html, verified in-browser. |
| **Illiteracy variants** | The hero can be illiterate (WIT < 10 → `isIlliterate(p)`, gear labels render as `???`). Story beats must be illiteracy-safe by default, and the comedic asset of "paperwork villain vs. paperwork-immune hero" should be exploited where it lands. See §11. | **In progress.** Adding variants to prologue, Floor 5, Floor 20. |
| **Town & supporting cast** | Marrowfield is named (§10). Still to design: 2–3 recurring NPCs. They are the source of *both* the shopkeeper and the heroes (see Run-handoff). | Town named; cast not started |
| **Hero creation backstory pick** | One-line pick at character creation (Why are you here?) that gets called back to in 2–3 beats. **Literacy is part of the pick** (§11) — Schoolteacher reads, Blacksmith's apprentice doesn't — not just a stat-dump side effect. | Not started |
| **Run-handoff on death** | Player death implies the *next* run is a different villager from Marrowfield who has picked up the flyer. The Town is the literal hero pool. Each villager can have any literacy (§11). | Not started — needs creation-screen support |
| **Death = Baron gloat** | Replace flat `lose` panel with a Baron taunt; ties into Run-handoff ("Next!"). **Two gloats by literacy** (§11) — the illiterate gloat lands *worse* for him. | Not started |
| **Trap rooms = snarky Baron signs** | Each trap room shows a one-liner notice ("NOTICE TO TRESPASSERS — Mgmt") when entered. Traps feel like Baron pettiness, not random hazards. **Literate hero gets `+5` to the ZIP shrug from reading the sign; illiterate hero sees `???` and gets no bonus** (§11). | Not started |
| **Shop = shopkeeper** | Post-boss shop run by a named NPC (from Town & cast), 2–3 lines that rotate per floor. **Shopkeeper switches to gestures and drawings for illiterate customers** (§11). | Not started |
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

### 9.1 Voice direction (locked): Pratchett-wry narrator

The narrator is a person who has noticed things. They take their time. They are charmed by small wrongnesses and willing to tell you about them. They can be cozy and they can be cutting and they often are both inside the same sentence. Pratchett, Douglas Adams, John Finnemore.

**Writing rules**

| Do | Don't |
|----|-------|
| Long sentences that wander toward the joke, with the punchline buried mid-clause, not waiting at the end | Tidy "X. Y? Z." rhythms; tricolons ("landlord, schemer, wearer of a stupid hat") |
| Use proper nouns. The Town has a name. The tavern has a name. The Baron's hat has a brand | "Your town", "the tavern" — abstraction is the enemy |
| Notice small weird specifics (the flyer is badly nailed; a monster came up the laundry chute; the intercom has feedback) | Generic monster-fantasy filler |
| Vary register inside a beat — cozy, cutting, briefly tender, mock-formal | One-note exasperation across every line |
| Drop the joke sometimes; let a line just sit | A punchline at the end of every paragraph |
| Use a short fragment as punctuation after a long sentence, for landing | Short fragments as default rhythm |
| Earn each ALL-CAPS and each `*aside*`. Used sparingly they bite | EMPHASIS on every line; reflexive `*crackle*`-style stage directions |
| **Show one concrete detail, let the reader infer the system.** The yellow NOTICE paper a monster was carrying. The carpet stain the Baron is openly furious about. The intercom feedback that gets worse with each beat. | Name the system in summary ("the monsters were eviction notices"; "the Baron is upset"). If you can replace a sentence with an object or a sound, do |

**The Baron, inside this narrator**

The Baron's voice is still corporate-landlord. The change is *texture*: he reaches for legalese and gets it slightly wrong, he addresses the hero with strained politeness that audibly fails, he has tics (he says "frankly" when he means the opposite, he refers to monsters as "amenities," he digresses mid-threat about carpeting). He is funniest when he is *trying* to be reasonable.

### 9.2 Before / after samples

**Narrator — prologue opening**

> **Before** (current — [index.html:172](index.html:172)):
>
> "WANTED — 1 (one) HERO. Must like stairs. Monster problem. Pls hurry. — The Town"
>
> The monsters are pouring up out of the dungeon, and behind it all sits BARON VON BACON 🐷 — landlord, schemer, wearer of a truly stupid hat. His scheme: expand his dungeon UP into your town. The monsters? Eviction notices. With teeth.

> **After** (proposed — pending Town name):
>
> Monsters had been coming up out of the dungeon for about a fortnight when somebody finally got around to writing a flyer. The flyer was, the tavernkeep noted, badly nailed; the monster problem had presumably been more pressing than the carpentry.
>
> The dungeon belonged to one Baron Von Bacon. He was a pig. He wore a hat that was one size too small and several decades out of fashion, and was, in the strictly technical sense, a landlord.
>
> Every monster that had so far been struck down was found to be carrying a small yellow slip of paper. One had been retrieved from the teeth of a goblin and pinned beside the flyer. It read:
>
> > **NOTICE TO TENANT** — *Removal scheduled forthwith. Vacate upper premises by sundown. Failure to comply will be enforced by Mgmt.*
>
> Nobody in Marrowfield had ever signed anything. This was, the tavernkeep gathered, the heart of the disagreement.

The change from the earlier draft: the "tell" line ("The monsters were his eviction notices") is gone. We see one actual NOTICE pinned to the door. The reader works out the rest — that monsters deliver these things, that "Mgmt" is the Baron, that nobody signed a lease. The Town name fills the final beat, which is why the prologue rewrite is pending §10.

---

## 10. Marrowfield — the Town

**Name:** Marrowfield. (Named for the marrow squashes grown there; the Watney family pumpkin patch is the unofficial town center.)

**Vibe:** "Marrowfield has been quiet, on average, for the better part of two centuries, and the residents have always taken this to mean it would continue being quiet. The first sign that this might no longer be true was a goblin standing very visibly lost in the Watney family's pumpkin patch on a Tuesday morning."

**Established Marrowfield specifics (canon, reuse these):**
- The **Watney family pumpkin patch** is the running geographic anchor. It is the first place monsters appeared.
- The **tavern** is where the flyer is nailed (and badly). It is also where the Baron's NOTICE has been pinned next to the flyer as evidence.
- The **tavernkeep** is the de facto narrator-observer. They have already opened a tab in the hero's name.
- Marrowfield has been **quiet for two centuries** — the monster problem is genuinely unprecedented. The residents are not equipped for this; that's the joke.

Marrowfield is the source of:
- the **villager pool** — per the run-handoff death system, each new run is a different Marrowfield resident picking up the flyer
- the **shopkeeper** (post-boss shop NPC — concept TBD)
- the **campfire voice** at rest (TBD)

Cast roster, backstory pick options, and run-handoff lore all live here once drafted.

---

## 11. Illiteracy & the illiterate hero

### The mechanic

`isIlliterate(p)` ([index.html:670](index.html:670)) returns true when `abilityMod(p.intelligence) < 0` — i.e. WITS < 10. The default character is WITS 8 → illiterate. Illiterate heroes see every gear label and description as `???` ("TOO ARCANE TO READ"). A player can choose to be illiterate by dumping WIT at creation, and they can climb out of it by raising WIT to 10+ via level-ups.

### The narrative principle

Two rules:

1. **Never make a story beat REQUIRE the hero to read.** Have a narrator or another character read for the audience, or have the hero hear/see something non-textual. Default writing should be illiteracy-safe — most beats need no variant at all.
2. **When literacy IS the joke, make it land.** The Baron is a paperwork villain. The illiterate hero is paperwork-immune. The biggest threat in the dungeon is a man waving a clipboard at someone who cannot, in any meaningful sense, see a clipboard. Lean in.

### Implementation pattern

Beats that branch on literacy are **functions of the player**; beats that don't branch are either plain objects OR trivial functions (for uniformity, this pass makes all `STORY.beats[n]` functions).

```js
// Prologue — branches on the tail
prologue: (heroName, p) => ({
  title: "📜 …",
  villain: false,
  lines: [
    ...sharedOpening,
    ...(isIlliterate(p) ? altTail(heroName) : litTail(heroName)),
  ],
  button: "▶ ENTER THE DUNGEON",
}),

// Beat — branches inside the lines array
5: (p) => ({
  title: "📢 …", villain: true, button: "▶ KEEP DESCENDING",
  lines: isIlliterate(p) ? [...altLines] : [...litLines],
}),

// Beat — no variant, trivial function for uniformity
10: (p) => ({ title: "📢 …", villain: true, button: "▶ …", lines: [...] }),
```

Call sites:
```js
// finishCreation — pass the just-built draft (it has the stats)
setStoryBeat({ ...STORY.prologue(heroName, draft), next: "idle" });

// descendStairs — pass the current effective idlePlayer
const beatFn = STORY.beats[nextFloor];
if (beatFn) { setStoryBeat({ ...beatFn(idlePlayer), next: "idle" }); setPhase("story"); }
```

### Current variants (this pass)

| Beat | Variant? | What changes |
|------|----------|--------------|
| Prologue | **Yes — tail** | Literate: "The tavernkeep has, with some foresight, opened a tab…" / Illiterate: "The tavernkeep, having gathered ${heroName} is not much of a reader, read the flyer aloud twice and the NOTICE TO TENANT once, with finger-pointing…" |
| Floor 5 | **Yes — inline clause** | Baron's "per Schedule C, subsection iv" routine breaks mid-sentence with "…and which I am now realizing you also could not READ if you did have, given the look on your face…" |
| Floor 10 | No — Baron lists amenities; no reading required | — |
| Floor 15 | No — Baron's barbed-wire complaint; no reading | — |
| Floor 20 | **Yes — tail** | Literate: "Let's discuss your security deposit." / Illiterate: "Let's discuss your security depo— oh. You don't even know what that IS, do you. Fine. Let's just FIGHT." |
| Ending | No — narrator describes the tavernkeep writing PAID; hero is far below | — |
| Baron taunts | No — short combat-log lines; no reading | — |
| Lieutenant snarls | No — short combat-log lines; no reading | — |

### Planned hooks for illiteracy (not in this pass)

These are noted in §8 alongside their parent features:

- **Trap room signs:** literate hero gets `+5` to ZIP shrug from reading the warning; illiterate hero sees `???` and gets no bonus. Optional: alternate detection path via pictogram (a wobbly drawing of a man falling through a floor).
- **Shopkeeper:** when serving an illiterate customer, switches to gestures and drawings — points at items, mimes their use, rotates 1–2 lines like "*(the shopkeeper draws a small picture of an apple on the counter and points at the apple, then at you)*."
- **Death gloat:** two Baron gloats by literacy. The illiterate gloat is *condescending* — and lands worse for him, because the hero who couldn't read his eviction notice has just outlasted the one who wrote it.
- **Backstory pick:** literacy is part of the pick, not just a stat-dump side effect. Picks should make the player choose their relationship to text deliberately.

**Baron — Floor 5 intercom**

> **Before** (current — [index.html:189](index.html:189)):
>
> *crackle* "Ahem. Welcome to floor FIVE.
> Per section 12 of the lease you do NOT have,
> kindly EXIT the premises. Upward. Immediately.
> ...You're not leaving, are you. Fine. FINE."

> **After** (proposed):
>
> *(the intercom takes a moment to clear its throat)*
>
> "Yes. Hello. This is the Management. Welcome to floor five, which — per Schedule C, subsection iv of your lease — you don't actually have a lease, do you, you simply *don't* — kindly turn around and walk back upstairs. Mind the railings on your way out. They bite now."
>
> *(a pause. The intercom does not switch off.)*
>
> "...you're not leaving. Right. Fine."

The Baron's after-voice keeps his existing energy (landlord-pedantic, runs out of patience mid-line) but adds: a wandering sentence with the joke buried mid-clause, a digression about the railings that does the work the old `*crackle*` was meant to, and a pause that lands without an exclamation point.

