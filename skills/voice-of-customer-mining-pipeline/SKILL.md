---
name: voice-of-customer-mining-pipeline
description: >
  Turns raw customer language into a structured insight asset that powers every downstream copy and
  research skill. Feed it review URLs (G2, Capterra, Trustpilot, App Store), support-ticket exports,
  community-forum threads, sales-call transcripts, NPS verbatims, or any free-text dump — and it
  returns: (1) a theme hierarchy with sentiment-weighted frequencies, (2) a persona vocabulary layer
  mapping exact customer phrases to jobs-to-be-done, (3) a pain-to-copy quote library with per-quote
  signal strength scores, and (4) an anti-vocabulary list (how customers do NOT describe the problem,
  which kills marketing that misses the mark). Built on the Jobs-to-Be-Done / Outcome-Driven
  Innovation framework with a Moments-of-Struggle overlay. Downstream skills — icp-persona-builder,
  positioning-messaging-architect, proof-vault, objection-library-builder, headline-hook-generator,
  landing-product-page-copy-writer — can ingest the output file directly instead of starting from
  scratch. Use whenever the user says "mine reviews," "analyze support tickets," "find customer
  language," "what do customers actually say," "extract pain points," "VoC analysis," "pull quotes
  for copy," "review mining," "customer vocabulary," "JTBD from reviews," or pastes a block of
  verbatims and asks what it says about the audience.
---

# Voice-of-Customer Mining Pipeline

Stop guessing how customers describe their problems. This skill mines raw verbatims — reviews, tickets,
forums, transcripts — and returns the exact vocabulary, themes, and signal-bearing quotes your copy
needs. The output is a structured artifact, not a summary: every downstream skill that needs audience
language reads this file instead of re-interviewing the customer base.

Layer 3 of the Growth Skill Library. Feeds Layer 1 (brand/ICP/positioning), Layer 4 (SEO), Layer 5
(content), and every copy-producing skill in the library.

---

## Skills this calls

- **`brand-brain`** (required, first) — loads active brand context: ICP definition, existing persona
  vocabulary, positioning hypotheses, banned words. The VoC run uses the ICP as a lens (not a
  straitjacket — surface what surprises the existing model).
- *(on request or when artifacts exist)* **`icp-persona-builder`** — pass the finished VoC report to
  deepen or create a persona; do not re-mine what this skill already produced.
- *(on request)* **`positioning-messaging-architect`** — quote library + theme hierarchy surface
  messaging gaps vs. current positioning.
- *(on request)* **`proof-vault`** — signal-bearing quotes are proof candidates; hand off rather than
  duplicate.
- *(on request)* **`objection-library-builder`** — negative-sentiment themes map directly to objections;
  feed the output rather than re-deriving.
- *(on request)* **`headline-hook-generator`** or **`landing-product-page-copy-writer`** — paste the
  pain-to-copy quote library as input context.

---

## How a run works

```
Step 0  Brand context      ──► call brand-brain; load ICP + voice + banned words
Step 1  Source audit       ──► inventory inputs; flag gaps; set scope
Step 2  Ingest + clean     ──► normalize verbatims; strip noise; tag source + sentiment
Step 3  JTBD / ODI coding  ──► code each verbatim to a Job, Struggle, or Outcome
Step 4  Theme hierarchy    ──► cluster → theme → sub-theme; frequency + sentiment weighting
Step 5  Vocabulary layer   ──► extract persona phrases; score signal strength; build anti-vocab
Step 6  Quote library      ──► select, score, map each quote to a copy use-case
Step 7  Write report       ──► save to ./reports/voc-[brand-slug]-[date].md; return digest
Step 8  Offer handoffs     ──► tell the user which downstream skills can consume the report
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill. Use the returned ICP (role, goals, awareness tendency) as the primary
coding lens. If brand-brain is absent, read `~/.brandbrain/brands/.active` + that brand's `brand.md`;
if neither exists, ask for brand slug + a 4-field quick setup before proceeding.

---

## Step 1 — Source audit

Before touching content, map what was provided:

| Source type | Signal type | Known bias |
|---|---|---|
| G2 / Capterra / Trustpilot reviews | Satisfaction poles (very happy / very unhappy) | Self-selection; skews extreme |
| App Store reviews | Friction + delight moments | Mobile/web product only |
| Support tickets / chat logs | Active struggle, failure modes | Channel-filtered (willing to write in) |
| Sales-call transcripts | Pre-purchase vocabulary, objections | Sales-influenced framing |
| Community / forum threads | Peer vocabulary, workarounds | Niche enthusiast bias |
| NPS/CSAT verbatims | Post-experience resonance | Anchored to score band |
| Social listening exports | Unprompted language | Volume + noise tradeoff |

Flag gaps: if only one source type is present, note what's missing and what it might distort. Never
claim representative insight from a single source without a caveat. Suggest additional sources if the
user can access them.

If sources are URLs: fetch them (WebFetch or browser tool). If pasted text: accept as-is.

---

## Step 2 — Ingest and clean

Normalize to a flat verbatim list: each row has `source`, `date` (if available), `sentiment`
(positive / negative / mixed / neutral — inferred from context, not just polarity words), `reviewer
type` (if inferable: persona bucket from the ICP). Strip personally identifiable information.
Flag non-customer verbatims (vendor replies, staff responses) and exclude them from coding.

Minimum viable corpus: 40+ verbatims for theme work; 15–39 is "directional, not definitive" (call this
out). Under 15: surface patterns only, no frequency claims.

---

## The core framework — JTBD + ODI + Moments of Struggle

**Why JTBD / Outcome-Driven Innovation (Ulwick)?** Because customers hire products to do jobs, not to
have features. Mining for features produces a feature list; mining for jobs produces messaging that
lands. ODI gives the job a structure: Functional Job (what they're trying to accomplish) + Emotional
Job (how they want to feel) + Social Job (how they want to be perceived). Desired Outcomes map to those
jobs with a metric direction ("minimize the time it takes to…", "reduce the likelihood that…").

**Moments of Struggle overlay (Competing Against Luck / Christensen):** Every negative verbatim reveals
a Moment of Struggle — the context in which the customer felt the gap between where they were and where
they needed to be. These moments are where switching decisions happen and where copy that says "we get
it" converts.

**Code each verbatim to:**
- **Job statement** (functional / emotional / social)
- **Struggle moment** (trigger context, if present)
- **Desired outcome** (explicit or implied direction + metric)
- **Vocabulary signal** (exact phrase worth preserving verbatim in copy)

---

## Step 4 — Theme hierarchy

Build a three-level hierarchy from the coded verbatims:

```
Theme (L1)  ──  the Job or Struggle category  (e.g. "Speed of setup")
  Sub-theme (L2)  ──  specific outcome direction  (e.g. "Time-to-first-value too long")
    Verbatim evidence (L3)  ──  2–4 representative quotes
```

Weight each L1 theme by **frequency × sentiment intensity** (a single highly-charged quote outweighs
three neutral mentions — flag when this diverges from raw count). Sort descending. Mark the top 5 as
**copy-priority themes**.

Separate positive signal (what they love / hired it for) from negative signal (where it fails / nearly
lost them). Both are copy fuel: positives anchor benefits + social proof; negatives anchor objection-
handling, risk-reversal, and UX copy.

---

## Step 5 — Vocabulary layer

Extract exact phrases — not paraphrases. Two outputs:

**Persona vocabulary (use these):** The specific words real customers use for their problem, their goal,
and the product category. Score each phrase: ★★★ (appears across multiple sources/personas, emotionally
loaded) → ★ (single mention, low affect). Prioritize ★★★ for headlines, subject lines, hero copy.

**Anti-vocabulary (avoid these):** How the brand currently talks about the problem in its marketing
that does NOT appear in customer language. This is the vocabulary gap — where the brand talks past the
buyer. List these explicitly; flag if any appear in the brand's current positioning (`brand.md`).

---

## Step 6 — Pain-to-copy quote library

Select 10–20 signal-bearing quotes. For each:

```
Quote: "[exact verbatim]"
Source: [type + date if known]
Job: [functional / emotional / social job it evidences]
Signal strength: ★★★ / ★★ / ★
Copy use-case: [headline hook | objection handler | social proof | risk-reversal microcopy |
               email subject line | ad hook | testimonial candidate]
Persona fit: [ICP bucket, if multiple personas present]
```

Flag any quote that is outstanding proof and should be routed to `proof-vault`. Mark any that contain
PII or competitive claims requiring legal review.

---

## Step 7 — Save the report

Save to `./reports/voc-[brand-slug]-[YYYY-MM-DD].md` (create `./reports/` if needed).

Report structure:
1. **Run metadata** — brand slug, sources used, corpus size, date, known biases
2. **Theme hierarchy** — L1/L2 with frequency + sentiment weights
3. **Copy-priority themes** (top 5 flagged)
4. **Persona vocabulary** (★★★ first)
5. **Anti-vocabulary / vocabulary gap**
6. **Pain-to-copy quote library**
7. **Downstream handoff notes** — which skills can consume this file and what they'll use from it

Return a digest to the user (summary of top 3 themes + 3 standout quotes + vocabulary gap finding).
Point to the saved path.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No mining output before the active brand is loaded. The ICP is the coding
  lens; it does not filter out surprises — surface what challenges the current model.
- **Verbatim, not paraphrase.** The value is the customer's exact words. Paraphrasing destroys the
  signal. Quote, attribute source type, preserve the specific phrasing.
- **Corpus integrity.** State corpus size, source mix, and known biases before claiming patterns.
  "Directional" and "representative" mean different things; use the right word.
- **Frequency × intensity, not frequency alone.** A single quote dripping with emotion in a Moment of
  Struggle outweighs five mild mentions. Weight accordingly; show both counts.
- **Anti-vocabulary is the finding.** The gap between how the brand talks and how customers talk is
  often more valuable than the theme hierarchy. Name it explicitly.
- **No invention.** Every theme, phrase, and quote must trace to an actual verbatim in the input.
  Never synthesize a "representative" quote.

---

## What Not to Do

- Don't start theming before loading brand context — you'll mis-code jobs without the ICP lens.
- Don't summarize quotes into paraphrases — the exact phrase is the deliverable.
- Don't claim representativeness from fewer than 40 verbatims; label directional work as such.
- Don't conflate source types — a support ticket and a 5-star review are different signals even if
  they say similar things; keep source provenance in the library.
- Don't pass a raw dump to downstream skills — the structured report is the handoff, not the input.
- Don't write to `brand.md` — vocabulary and persona updates belong in the skills that own those
  files (`brand-brain`, `icp-persona-builder`); pass them the report instead.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; ICP loaded and used as the coding lens?
- Source audit complete: types inventoried, biases named, gaps flagged?
- Corpus size stated; "directional" label applied if under 40 verbatims?
- Every verbatim coded to a Job, Struggle moment, and Desired Outcome direction?
- Theme hierarchy has 3 levels; weighted by frequency × sentiment intensity?
- Top 5 copy-priority themes explicitly flagged?
- Persona vocabulary uses exact customer phrases (★★★ scored); anti-vocabulary gap named?
- Quote library has 10–20 entries; each has source type, Job, signal strength, and copy use-case?
- PII stripped; proof candidates flagged for `proof-vault`; legal flags noted?
- Report saved to `./reports/voc-[slug]-[date].md`; digest returned with path?
- Downstream handoff notes present so the user knows which skill to run next?
