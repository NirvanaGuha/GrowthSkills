---
name: brand-voice-codifier
description: >
  Turns sample copy plus a few brand inputs into a structured, enforceable voice and tone guide —
  the part of the brain that makes every other copy skill sound like the brand instead of like AI.
  It places the brand on the 12 Pearson/Mark archetypes and the four Nielsen Norman tone dimensions,
  builds a do/don't lexicon, and EXTRACTS a real banned-word list from the brand's own sample copy
  (not a generic hype list). It is a brand-brain COMPONENT: callable standalone (it invokes
  `brand-brain` to load the active brand) or called by `brand-brain` during bootstrap/refresh
  (it uses the passed context and returns its sections — no recursion). It owns exactly three
  brand.md sections: "Voice & tone," "Banned words / phrases," and "Preferred lexicon." Use when the
  user says "codify voice," "brand voice," "tone of voice," "voice guide," "make this sound like us,"
  hands over sample copy and asks "what's our voice," or wants a banned-word / preferred-word list.
---

# Brand Voice & Tone Codifier

Feed it the brand's real copy; get back a voice guide a junior writer can execute without taste. It pins the brand to named frameworks — archetype, four tone dials, a do/don't lexicon, and a banned list mined from the brand's own words — so "on-brand" stops being a vibe and becomes a checklist. It writes only the three voice sections of `brand.md`; it never touches positioning, offer, ICP, or proof.

This is a codifier, not a copywriter. It does not write headlines, CTAs, or body copy — it writes the rules those skills obey.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode only) — resolves the active brand and loads its current `brand.md` for context. This skill never implements brand resolution, scanning, or storage itself; that lives in `brand-brain`, once. When `brand-brain` calls *this* skill, the context is already passed in — do not call back (no recursion).
- No other skills. Voice codification is self-contained.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED (context passed in) | STANDALONE (resolve via brand-brain)
Step 1  Gather inputs    ──► sample copy + 3 voice adjectives + any existing voice notes
Step 2  Codify           ──► archetype → tone dials → lexicon → banned-word extraction
Step 3  Return or persist
```

### Step 0 — Detect the mode

- **CALLED BY brand-brain.** The request already carries the active **slug**, the current `brand.md` content, and the scanned raw inputs (sample copy, persona files, prior notes). Use that context. Do the work. **RETURN** the three section blocks for `brand-brain` to fold in. **Do not invoke `brand-brain`** — that recurses.
- **STANDALONE.** No brand context in the request. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. It bootstraps on first use. Do the work, then **persist** (Step 3). **Fallback if `brand-brain` isn't installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly, or ask the user. Always prefer the call.

### Step 1 — Gather inputs

You need real material to codify a *real* voice. Pull, in order:
1. **Sample copy** — the single most important input. Homepage, top emails, best-performing posts, founder writing, docs. 3–5 samples beats 1. If the caller passed it, use it; standalone, ask for it (or pull what the brand-brain scan found).
2. **Existing voice notes** — any prior "Voice & tone" content in `brand.md`, a style doc, a Thoth persona's archetype/tone. Refine, don't ignore.
3. **Three voice adjectives** — if not already known, ask. They anchor the archetype + tone read.

**No usable sample copy?** Codify from the adjectives + positioning + one founder paragraph, mark the result `confidence: low`, and say plainly that the banned/preferred lists are provisional until real copy exists. Never fabricate sample copy to mine.

### Step 3 — Return or persist

- **CALLED:** return the three section blocks (below) as your output. Note `brand-voice-codifier` as the source for them. Done.
- **STANDALONE:** update **only** your three owned sections in `brand.md` — read the file, replace just those three blocks, bump `updated` to today, append `brand-voice-codifier (YYYY-MM-DD)` to `sources`. Leave every other section byte-for-byte unchanged. Confirm in one line where you saved (`~/.brandbrain/brands/<slug>/brand.md` — the data root `brand-brain` resolves; a per-project `./.brandbrain/` takes precedence when present). No companion file.

---

## The craft

### 1) Archetype — what the brand *is* (Pearson/Mark, 12 archetypes)

The archetype is the personality engine; tone is how it speaks in a given moment. Pick **one primary** (the dominant motivation) and **at most one secondary** (the flavor). Three+ archetypes means you haven't decided — push back.

| Archetype | Core desire | Voice signature | Reach-for verbs |
|---|---|---|---|
| Innocent | safety, simplicity | warm, plain, optimistic | trust, simple, easy |
| Sage | truth, understanding | precise, evidence-led, calm | learn, understand, prove |
| Explorer | freedom, discovery | restless, first-person, bold | discover, break free, go |
| Outlaw | disruption, revolution | blunt, provocative, anti-status-quo | break, disrupt, refuse |
| Magician | transformation | visionary, "imagine if," catalytic | transform, unlock, reimagine |
| Hero | mastery, courage | direct, challenge-driven, decisive | win, conquer, achieve |
| Lover | intimacy, belonging | sensory, warm, you-focused | love, savor, connect |
| Jester | joy, play | witty, irreverent, light | play, laugh, enjoy |
| Everyman | belonging, realism | down-to-earth, inclusive, no-jargon | join, get, share |
| Caregiver | service, protection | nurturing, reassuring, generous | care, support, protect |
| Ruler | control, order | authoritative, premium, exacting | lead, command, set the standard |
| Creator | self-expression, craft | imaginative, original, maker-minded | build, design, craft |

State the choice as a sentence: *"Sage primary (we win on rigor), Everyman secondary (no jargon, talk like a peer)."* That sentence governs every tone call below.

### 2) Tone dials — how it speaks (Nielsen Norman, 4 dimensions)

Tone is contextual; archetype is constant. Place the brand on each axis as a default, then note where it *shifts* by context. Use a 1–5 scale (1 = far left).

| Dimension | 1 ◄────────► 5 | Default | Shifts to… |
|---|---|---|---|
| Formality | formal ↔ casual | _n_ | e.g. legal/security copy → more formal |
| Humor | serious ↔ funny | _n_ | e.g. error states → drop the jokes |
| Respect | respectful ↔ irreverent | _n_ | e.g. competitor takes → more irreverent |
| Enthusiasm | matter-of-fact ↔ enthusiastic | _n_ | e.g. launch copy → more enthusiastic |

Two rules that keep this honest: **the four numbers must agree with the archetype** (a Sage at humor-5 / respect-5 is a contradiction — resolve it), and **enthusiasm is the one most brands over-dial** — default it lower than instinct unless the sample copy earns it.

### 3) Do/don't lexicon

Three columns, drawn from the archetype + samples, not invented:
- **Favor (verbs & framings)** — the brand's reach-for verbs and the possessives/second-person framings that put the reader in the scene ("your subscribers," "ship it today").
- **Use precisely (domain terms)** — the metric and category words this brand owns and how it spells/capitalizes them (e.g. "web push," not "browser notifications").
- **Avoid (soft no)** — weaker-but-not-banned words to steer away from. (Hard bans go in the banned list, below.)

### 4) Banned-word extraction (from the brand's OWN copy + the universal slop list)

This is the section that separates a real guide from a template. Two passes:

1. **Universal slop pass** — ban the AI/marketing tells regardless of brand: *leverage, utilize, seamless, robust, unlock, elevate, revolutionary, game-changer, cutting-edge, world-class, supercharge, delve, in today's fast-paced world, it's not just X — it's Y, the em-dash-heavy "not only… but also."* Plus punctuation tics: exclamation marks and emojis unless the archetype/tone explicitly allow them.
2. **Brand-specific extraction** — read the brand's sample copy and pull words/phrases that *clash with the chosen archetype/tone*: hype the Sage brand would never use, jargon the Everyman brand should drop, a competitor's trademarked term, an off-positioning frame. Quote the offending phrase and give the on-voice replacement.

Output as a table so it's enforceable downstream:

| Banned | Why | Use instead |
|---|---|---|
| "revolutionary" | hype; clashes with Sage rigor | "measurably better," with the number |
| "leverage" (verb) | slop tell | "use" |

If `brand-brain` already passed a banned list, **merge** — keep its entries, add yours, dedupe, never silently drop a user-confirmed ban.

---

## The three sections you write (and only these)

Return/persist exactly these `brand.md` blocks. Match the template's headings verbatim.

```markdown
## Voice & tone  (core)
Archetype: <Primary> primary / <Secondary> secondary — <one-line why>.
Tone dials (NN/g, 1–5): Formality <n> · Humor <n> · Respect <n> · Enthusiasm <n>.
Context shifts: <axis → direction in which context>.
POV & sentence style: <person; sentence length/rhythm>.
Sounds like: "<one sentence in the brand's actual voice>."

## Banned words / phrases  (core)
| Banned | Why | Use instead |
| ... (universal slop + brand-extracted, merged with any existing) |
Punctuation: <exclamation/emoji/em-dash policy>.

## Preferred lexicon
Favor (verbs/framings): <list>.
Use precisely (domain terms): <term → spelling/usage>.
Avoid (soft no): <list>.
```

**Suggest, don't write, neighbors.** If the sample copy reveals a positioning frame, a proof point, or an ICP nuance, note it as a one-line *suggestion to brand-brain* — never write into "Positioning," "Proof," or "ICP" yourself.

---

## Principles

- **Codify from real copy.** Voice is extracted from how the brand already sounds, not imagined. No samples → say so and lower confidence.
- **One archetype, four numbers.** A primary archetype (+ optional secondary) and a number on each NN/g dial. No mush.
- **Internal consistency.** The dials must agree with the archetype; resolve contradictions before persisting.
- **Banned list earns its place.** Every ban has a reason and a replacement; brand-specific bans quote real offending copy.
- **Own three sections, period.** Voice & tone, Banned words / phrases, Preferred lexicon. Everything else is someone else's to write.
- **Truth only.** Mark unconfirmed reads `[verify]`; never invent how the brand "feels."
- **Standalone persists surgically.** Replace only your three blocks; never rewrite the file.

## What not to do

- Don't write CTAs, headlines, or body copy — you write the rules, not the copy.
- Don't touch any `brand.md` section other than your three; don't overwrite the whole file.
- Don't call `brand-brain` when you were *called by* it — that recurses.
- Don't ship a generic banned list with no brand-extracted entries when sample copy exists.
- Don't pick three+ archetypes or leave a tone dial blank — that's an undecided voice.
- Don't silently drop a user-confirmed banned word on merge.
- Don't invent sample copy to analyze, or assert a voice trait the samples don't support.

## Quality checklist (self-review before returning/persisting)

- Mode detected correctly? (Called → return, no `brand-brain` call; standalone → `brand-brain` loaded, then persist.)
- Codified from real sample copy? (If none, flagged + `confidence: low`.)
- Exactly one primary archetype (+ ≤1 secondary), stated as a sentence?
- All four NN/g dials have a number, with context shifts, and they agree with the archetype?
- Banned list = universal slop **plus** brand-extracted entries, each with reason + replacement; existing bans merged not dropped?
- Preferred lexicon has favor / use-precisely / avoid columns drawn from the samples?
- Wrote ONLY the three owned sections; neighbors handled as suggestions to brand-brain?
- Standalone: replaced only those blocks, bumped `updated`, appended self to `sources`, confirmed the path?
