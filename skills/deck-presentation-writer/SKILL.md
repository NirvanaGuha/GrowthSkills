---
name: deck-presentation-writer
description: >
  Bullet-point outline or raw notes → structured slide titles, body copy, and speaker notes
  formatted for Google Slides or PowerPoint. Accepts any input fidelity: a topic + audience,
  a scrappy brain dump, a rough outline, a meeting transcript, or an existing deck that needs
  a rewrite. Applies the Assertion-Evidence framework (claim-first title + evidence body) to
  every slide so each one earns its place. Calls brand-brain for on-voice language, proof
  points, and visual identity; calls positioning-messaging-architect to anchor the narrative
  arc to the brand's current positioning; calls proof-vault for real social proof; calls
  cta-variant-generator for the closing CTA slide. Output is a complete slide-by-slide
  script (title / body / speaker notes) that pastes directly into any deck tool, plus an
  optional slide-count-and-flow audit. Use when the user says "write my deck," "build
  the slides," "turn these notes into a presentation," "write speaker notes," "pitch deck,"
  "sales deck," "board update," "keynote," or hands over an outline and asks to flesh it out.
---

# Deck & Presentation Writer

Turn raw inputs into a complete, ready-to-paste slide script. Every slide has a claim-first title, an evidence body, and speaker notes. The deck earns its argument one slide at a time rather than burying the point in the last bullet.

This skill writes the words and the structure. It does not design the visuals, invent proof, or substitute its judgment for the brand's positioning — those come from the library skills that own them.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, ICP, banned words, visual identity (colors/fonts), and real proof. Call first; do not write a single slide before it returns.
- **`positioning-messaging-architect`** (call when the deck is a pitch, sales, or narrative deck) — anchors the through-line to the brand's current positioning and value prop so the story is not improvised.
- **`proof-vault`** (call when slides need customer evidence, numbers, or testimonials) — surfaces real, sourced proof instead of placeholder copy.
- **`cta-variant-generator`** (call for the closing CTA slide) — writes the final ask with correct awareness-stage and commitment ceiling.

---

## How a run works

```
Step 0  Load the brand        ──► brand-brain (always first)
Step 1  Classify the deck     ──► narrative mode selection
Step 2  Build the arc         ──► Story Spine → slide map
Step 3  Write slide by slide  ──► Assertion-Evidence framework
Step 4  Close the deck        ──► cta-variant-generator for final ask
Step 5  Self-audit + deliver  ──► flow audit, then full script out
```

### Step 0 — Load the brand (always first)

Invoke **`brand-brain`** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency, visual identity (colors, fonts). Do not write any slide copy until it returns.

Obey voice and banned-words as hard overrides. Mark any unconfirmed numbers `[verify]`. Use only proof the brand has confirmed.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if neither exists, ask for the 5-point mini-setup (what it is · ICP + awareness stage · offer + destination · voice adjectives + banned words · 2–3 real proof points), then proceed.

### Step 1 — Classify the deck

Identify the deck type and set the narrative mode:

| Deck type | Primary narrative mode | Positioning call needed? |
|---|---|---|
| Sales / product pitch | Problem → Stakes → Solution → Proof → Ask | Yes |
| Board / investor update | Situation → Complication → Resolution (SCR) | Yes |
| Keynote / thought-leadership | Big idea → Tension → New frame → Implications | Often |
| Internal / status | Context → Progress → Blockers → Next actions | No |
| Webinar / educational | Hook → Why it matters → Framework → Application | No |

If the deck is a pitch or narrative deck, invoke **`positioning-messaging-architect`** to confirm the through-line aligns with current positioning before building the arc.

### Step 2 — Build the arc (Story Spine — Kenn Adams)

Map the deck to a Story Spine before writing any slide:

```
Once upon a time…    [audience's world as it is today]
Every day…           [the status quo and the cost of staying there]
Until one day…       [the inciting disruption or insight]
Because of that…     [why the solution matters now]
Because of that…     [what specifically it does/delivers]
Until finally…       [the transformed outcome]
And ever since…      [the world after; the proof it works]
Then ask:            [the CTA — one clear ask]
```

Compress or expand slides to fit the requested count. A short deck (8–12 slides) needs the spine tight. A long deck (20–30) can expand the "because of that" sections with supporting evidence slides. Confirm the slide count and arc with the user if the request is ambiguous.

### Step 3 — Write slides (Assertion-Evidence framework)

Apply the Assertion-Evidence framework (Alley & Neely, 2005) to every content slide:

- **Title = assertion.** A full declarative sentence that stands alone as the slide's claim. Not a topic label ("Customer Data"), not a vague noun phrase — a testable statement ("Churn drops 34% when subscribers receive a triggered campaign within 24 hours of intent signal"). The audience should know the point before reading the body.
- **Body = evidence.** 2–4 supporting items (data, examples, mechanism, quote) that prove the title assertion. Use bullets only when items are genuinely parallel. Prefer short prose when there are fewer than 3 items. Never use bullets to smuggle a whole argument the title should be making.
- **Speaker notes = narrative bridge.** 3–6 sentences. What to say (not read), how to transition into the next slide, and any timing/interaction cues. Notes are the presenter's script, not a restatement of the body.

**Slide types in the script:**

| Tag | Purpose | Title style |
|---|---|---|
| `[COVER]` | Title + brand + event/date | Evocative or hook question |
| `[SECTION]` | Arc divider | Transition statement |
| `[CONTENT]` | Core argument slides | Assertion (full sentence) |
| `[PROOF]` | Customer story, stat, quote | Outcome claim |
| `[VISUAL CUE]` | Chart/image/demo placeholder | Note what visual is needed |
| `[CTA]` | Closing ask | cta-variant-generator output |

### Step 4 — Close with a CTA slide

For the closing slide, invoke **`cta-variant-generator`** (Skill tool), passing the brand context digest and the deck's awareness stage. Use the returned primary CTA as the [CTA] slide title and the microcopy as the body. Do not invent this.

If `proof-vault` is installed and the deck needs social proof slides, invoke it before writing [PROOF] slides. Use returned real proof verbatim (do not paraphrase into something that sounds bigger than it is).

---

## Output format

Deliver a complete script, one block per slide:

```
## Slide [N] — [TAG]
**Title:** [assertion or heading]
**Body:**
- [line 1]
- [line 2]
- [line 3]
**Speaker notes:** [3–6 sentences. What to say, how to land it, transition cue.]
```

For decks ≤15 slides: deliver all slides in one pass.
For decks >15 slides: confirm the arc and outline first, then write in sections on request.

Save full scripts to `./decks/[brand-slug]-[deck-type]-deck.md` when the user asks. Quick inline outputs stay in chat.

---

## Principles

- **Brand-brain first.** No slides before the brand loads. Voice + banned-words are hard overrides.
- **Assertion titles, always.** A topic label is not a title. Every content slide title is a full claim the audience can agree or disagree with.
- **Evidence that proves the title.** The body exists to prove the assertion, not to list related thoughts.
- **One argument per slide.** If a slide is making two points, split it.
- **Real proof or `[verify]`.** Never invent a customer name, number, or outcome. Pull from `proof-vault` or mark clearly.
- **Speaker notes are a script, not a restatement.** Notes say what to say aloud; they do not repeat the body bullets.
- **Compose, don't duplicate.** Never re-derive positioning, proof, or the CTA — call the skills that own them.

## What Not to Do

- Don't write slides before `brand-brain` returns the active brand.
- Don't use topic-label titles ("Overview," "Our Solution," "Agenda") as content slide titles — those are section dividers, not argument slides.
- Don't pack more than one assertion into a slide title.
- Don't invent proof, testimonials, or metrics — use `proof-vault` or `[verify]`.
- Don't reimplement brand scanning, positioning work, or proof curation here.
- Don't ship a deck without a [CTA] slide and a clear single ask.
- Don't write speaker notes that just read the bullets back to the presenter.

## Quality Checklist (self-review before delivering)

- `brand-brain` called and digest loaded before first slide?
- Voice + banned-words honored throughout; all unconfirmed numbers marked `[verify]`?
- Every [CONTENT] slide title is a full declarative assertion (not a label)?
- Body bullets prove the title assertion (not free-floating related facts)?
- Arc follows Story Spine or SCR — does the deck tell a single coherent story?
- Positioning-messaging-architect called for pitch/board decks?
- [PROOF] slides use real sourced proof from `proof-vault` (or marked `[verify]`)?
- [CTA] slide written via `cta-variant-generator` with correct awareness ceiling?
- Speaker notes are narrative bridges, not restatements?
- Slide count appropriate to deck type; no filler slides?
