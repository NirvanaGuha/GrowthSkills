---
name: quote-polisher
description: >
  Transforms a raw exec transcript snippet or rough attributed quote into three polished,
  ready-to-publish variants — punchy, neutral, and expansive — that preserve the exec's
  actual voice rather than flattening it into press-release boilerplate. Works for press
  releases, award submissions, investor updates, media kits, keynote pull-quotes, LinkedIn
  exec posts, case studies, and analyst briefings. Loads the brand via brand-brain first so
  every variant honors the brand voice and never introduces banned words. Optionally calls
  de-slop-humanize-pass to scrub any residual corporate gloss. Output is three labeled quote
  variants plus a usage guide showing which fits which placement, ready to paste. Use whenever
  the user says "polish this quote," "make this exec quote usable," "clean up this transcript,"
  "turn this into a pull-quote," "make it sound less like a press release," "quote variants,"
  "exec attribution copy," or hands over a transcript excerpt and asks for a quotable.
---

# Quote Polisher

Raw exec transcript or rough quote → three polished variants (punchy / neutral / expansive) that sound like the exec, not a press release.

The job is voice preservation, not voice replacement. Press-release speak — passive constructions, synergy vocabulary, filler nominalizations — erodes trust and readability. This skill removes the corporate gloss while keeping the speaker's real personality, rhythm, and genuine claim. It delivers three tonal variants because different placements have different character limits and register requirements.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice profile, banned words, positioning, and real proof so variants stay on-brand and clean.
- **`de-slop-humanize-pass`** (optional, Step 3) — called when the source quote is especially jargon-dense or when the user asks for a "slop pass"; removes corporate clichés programmatically before the polishing layer.
- **`proof-vault`** (optional) — if a claim in the quote needs a real number or customer reference to land, check `proof-vault` before fabricating; mark unverified numbers `[verify]`.
- **`content-repurposer-atomizer`** (optional) — if the user needs all three variants atomized into LinkedIn exec post, X pull-quote card, and press release simultaneously, delegate to `content-repurposer-atomizer` after polishing is complete.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (required)
Step 1  Diagnose the source  ──► voice fingerprint + failure inventory
Step 2  Slop pass (if needed) ──► de-slop-humanize-pass or inline
Step 3  Write three variants  ──► Punchy · Neutral · Expansive
Step 4  Usage guide  ──► placement map + A/B pick if one placement
Step 5  Self-review  ──► quality checklist
```

---

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, ICP, positioning line, and real proof.

Obey voice adjectives and banned words as hard overrides. Use only real proof from the brand digest; mark any unverified numbers `[verify]`. Do not write a single variant before `brand-brain` returns.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for three things: (a) three voice adjectives for this exec/brand, (b) any banned words or phrases, (c) one positioning sentence. Prefer the skill call.

---

### Step 1 — Diagnose the source

Before rewriting, extract the signal and name the problems. Do this silently (one compact block in the output) so the user sees your reasoning.

**Voice fingerprint the exec.** In two to three sentences: characteristic sentence length, cadence (direct/conversational/formal), vocabulary register (technical/commercial/vernacular), any recurring structure. Pull this from the transcript itself — not from the brand brief. If the transcript is too short (fewer than ~30 words), ask for a second sample or note the limitation.

**Inventory the failures.** Tag every problem present in the source with one of:

| Tag | Failure |
|-----|---------|
| `[SLOP]` | Corporate cliché, buzzword, or nominalization ("leverage," "synergize," "journey," "solution") |
| `[PASSIVE]` | Passive construction burying the subject |
| `[VAGUE]` | Claim without a number, name, or concrete mechanism |
| `[LONG]` | Sentence that buries the punchline or can split |
| `[HEDGE]` | Unnecessary qualification that undercuts the claim |
| `[BANNED]` | Word or phrase on the brand's banned list |
| `[UNVERIFIED]` | Statistic or customer claim not in `proof-vault` or brand digest |

Surface only the tags — no rewrite yet. This makes the polish decisions traceable.

---

### Step 2 — Slop pass (when needed)

If the source has ≥3 `[SLOP]` or `[BANNED]` tags, invoke `de-slop-humanize-pass` (Skill tool) on the raw source before writing variants. Pass it the raw quote plus the brand's banned-word list. Use its clean output as the base for Step 3.

If `de-slop-humanize-pass` is unavailable or the slop count is low, apply the replacement table inline:

| Replace | With |
|---------|------|
| leverage (verb) | use, apply, put to work |
| solutions | products, tools, the software, [specific name] |
| journey | path, move, shift |
| synergize / synergies | work together, combine |
| space (as in "in the X space") | delete or name the category |
| empower | let, help, enable (or cut entirely) |
| robust | strong, deep, reliable (be specific) |
| seamless | smooth, fast, without friction |
| take it to the next level | [name the level specifically] |
| best-in-class | [cite the evidence or cut] |

---

### Step 3 — Write three variants

The Kelleher–Simmons Voice-Fidelity Framework: the goal is to produce the quote the exec *would have said* if they had twenty minutes to craft it, not the quote a PR intern wrote for them. Every variant must clear three gates:

1. **Authenticity gate.** Would this exec plausibly say this, based on their fingerprint? A CTO who uses technical shorthand should not suddenly sound like a CMO.
2. **Claim gate.** Every claim must be as specific or more specific than the original, never vaguer. Replace a vague claim with the real number from `proof-vault` or `[verify]` it.
3. **Brand gate.** No banned words; voice adjectives honored; positioning coherent.

Deliver exactly three variants, each under a labeled H3:

#### Punchy (≤35 words)
One to two sentences. The pull-quote format. Opens with the sharpest claim or the most surprising number. No hedges. Active voice throughout. Designed for: social cards, press release pull-out boxes, award submissions, keynote slides, media pitches.

#### Neutral (50–80 words)
Two to three sentences. Balanced register — not cold, not breathless. States the claim, gives one piece of evidence, closes with a forward-looking implication. Designed for: press releases, investor updates, analyst briefings, media kit bios.

#### Expansive (90–130 words)
Three to four sentences. Includes context, one brief narrative detail from the transcript if present, and the brand's relevant proof. Appropriate for: case study testimonials, long-form features, whitepapers, award submissions requiring extended quotes.

For each variant, add one line of attribution guidance: name, title, and any flagged `[verify]` items that need fact-checking before publishing.

---

### Step 4 — Usage guide

After the three variants, output a compact placement map:

```
## Usage guide
Punchy    → press release sidebar / social card / award box / media pitch
Neutral   → press release body / investor update / analyst Q&A / media kit
Expansive → case study / feature article / whitepaper / award long-form

If you only need one placement: [name the recommended variant and one-line why]
```

If the user named a specific placement (e.g., "this is for a G2 case study"), lead with the recommended variant prominently and suppress the others unless asked.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No variant before `brand-brain` returns. Its voice adjectives and banned words override all style instincts.
- **Preserve, don't replace.** The exec's actual vocabulary and sentence rhythm survive polishing. You are a line editor, not a ghostwriter starting from scratch.
- **Specificity over atmosphere.** A vague positive ("impressive growth") is always worse than a concrete one ("39% retention lift"). Use real proof or flag `[verify]`.
- **Authenticity gate is non-negotiable.** If the variant wouldn't survive an exec reading it aloud and saying "yes, that's me," rewrite it.
- **Three variants, three distinct registers.** Punchy, neutral, and expansive must differ by more than length — they differ in structure, emphasis, and placement fit.
- **No invented proof or flattery.** Never fabricate a customer name, number, or claim. Mark everything unconfirmed `[verify]`.
- **Honest diagnostic.** If the source quote has no genuine claim — only filler — say so and ask for the real point the exec was making before polishing.

---

## What Not to Do

- Don't write variants before `brand-brain` returns the active brand.
- Don't produce three near-identical variants that differ only in word count — that is not a tonal range.
- Don't introduce proof, names, or numbers that aren't in the brand digest or `proof-vault`.
- Don't homogenize the exec's distinctive speech patterns into generic LinkedIn prose — that defeats the job.
- Don't apply the brand's marketing voice directly to the exec quote. The exec has a personal voice that *complements* the brand — reconcile them, don't override one with the other.
- Don't silently strip hedges that are legally or contractually important (e.g., forward-looking statements). Flag them as `[legal hedge — do not remove without counsel review]`.
- Don't save output inside the skill folder. Polished quotes save to `./pr/quotes/[exec-slug]-[date].md` when persistence is requested.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand context loaded before any variant was written?
- Voice fingerprint extracted from the actual transcript (not from the brand brief alone)?
- Failure tags surfaced (`[SLOP]`, `[PASSIVE]`, `[VAGUE]`, `[LONG]`, `[HEDGE]`, `[BANNED]`, `[UNVERIFIED]`)?
- Slop pass applied (via skill or inline) when ≥3 `[SLOP]`/`[BANNED]` tags present?
- Punchy ≤35 words; Neutral 50–80; Expansive 90–130?
- Each variant clears all three gates: authenticity, claim, brand?
- No banned words in any variant; no invented proof; unverified claims marked `[verify]`?
- Attribution line on each variant (name, title, flag)?
- Usage guide present with placement map and a recommended pick if a single placement was named?
