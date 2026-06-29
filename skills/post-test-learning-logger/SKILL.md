---
name: post-test-learning-logger
description: >
  Converts raw experiment outputs — test brief, stat-sig results, analyst commentary, and a launch
  decision — into a structured, searchable learning card that your team can actually act on.
  Uses our house ICE + CATE rubric (an ICE prioritization score paired with CATE-style result framing —
  two distinct tools we combine here, not one established framework): captures the causal claim cleanly (Hypothesis), the evidence
  quality honestly (Confidence flags for SRM, sample size, novelty effect, measurement window),
  the business Impact in revenue/lift terms, and the Compounded Action (winning-variant launch
  checklist OR kill/park reasoning). Calls data-qa-measurement-gotcha-checker to gate on GA4
  measurement quality before encoding a finding as fact. Calls validity-threat-checker when the
  test doc is present but no threat review has been run. Saves every card to a project-relative
  test log (./experiments/learning-log.md) so the library compounds — one run feeds the next.
  Use when the user says "log this test," "write up our experiment results," "A/B test learning
  card," "post-test debrief," "what did we learn from this test," "should we ship the winner,"
  "winning variant launch checklist," or pastes in test results and asks what to do next.
---

# Post-Test Learning Logger

Turn a completed experiment into a reusable, honestly-rated learning card — not a one-pager that
gets lost in a Notion graveyard. Every card encodes the hypothesis, the evidence, a confidence
grade, and the exact next action. Over time, the log becomes the team's shared institutional
memory for what actually moves the needle.

This skill logs and advises on launch. It does not design the test — use `a-b-multivariate-test-designer`
for that. It does not crunch statistical significance — provide the p-value or confidence interval;
if you need to validate sample size, call `sample-size-calculator` first.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, ICP, and proof so the card's
  business framing and win narrative use real positioning, not generic copy.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for brand name, ICP one-liner, and north-star
  metric before proceeding.
- **`data-qa-measurement-gotcha-checker`** (quality gate) — runs GA4 / tracking gotcha check before
  encoding a number as a confirmed finding; call when results come from GA4 or any pixel-based stack.
- **`validity-threat-checker`** (optional, called when no prior threat review exists) — surfaces SRM,
  novelty effect, seasonality, and instrumentation bias threats before the card grades confidence.
- **`analytics-report-reviewer`** (optional) — peer-reviews the written card's claims before saving;
  invoke when the card will be shared with execs or a board.
- **`experiment-pipeline-backlog-prioritizer`** (optional) — feed the card's "Next test ideas" output
  directly into the pipeline so learnings compound into the roadmap.
- **`a-b-multivariate-test-designer`** — call to design the follow-up test the card recommends;
  do not re-derive test design here.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain
Step 1  Ingest inputs        ──► test brief + results + decision
Step 2  Quality gates        ──► data-qa-measurement-gotcha-checker, validity-threat-checker
Step 3  Grade confidence     ──► ICE + CATE rubric
Step 4  Build the card       ──► structured learning card
Step 5  Launch checklist     ──► if winning variant ships, emit checklist
Step 6  Save + surface       ──► append to ./experiments/learning-log.md
```

### Step 0 — Load brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned digest for ICP framing,
positioning language, and the brand's north-star metric. Every number in the card's Impact section
must be anchored to a real metric the brand tracks.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand name, ICP one-liner, and north-star
metric before proceeding.

### Step 1 — Ingest inputs

Ask for (or extract from the paste):
- **Test brief** — hypothesis, variants, audience, traffic split, duration.
- **Results** — primary metric + delta, p-value or confidence interval, secondary metrics, sample sizes per variant.
- **Decision already made or requested** — Ship winner / Kill / Iterate / Hold for replication.

If any are missing, ask before proceeding. Do not infer results from vague prose — "it improved"
is not a result; ask for the number.

### Step 2 — Quality gates (run before grading)

- If results come from GA4 or any pixel stack: call `data-qa-measurement-gotcha-checker` with the
  metric name, funnel step, and any known setup notes. Incorporate its flags into the card's
  Confidence section verbatim.
- If no validity-threat review has been done (the user hasn't mentioned one): call
  `validity-threat-checker` with the test design. Capture any HIGH threats in Confidence.

### Step 3 — Grade confidence (ICE + CATE rubric)

Score each dimension 1–5. Show the score inline:

| Dimension | Score | What it means |
|---|---|---|
| **I — Impact** | 1–5 | Magnitude of the lift × strategic importance of the metric |
| **C — Confidence** | 1–5 | Statistical rigor + absence of validity threats + measurement quality |
| **E — Ease to ship** | 1–5 | Engineering effort, legal/brand risk, rollback complexity |

**Confidence automatic downgrades (apply before scoring):**
- p > 0.05 or CI crosses zero → cap Confidence at 2
- SRM detected → cap at 2; flag as "do not ship without re-run"
- Novelty effect likely (< 2 weeks runtime or new UI paradigm) → cap at 3; recommend hold for
  validation window
- Measurement gotcha flagged by `data-qa-measurement-gotcha-checker` → cap at 3, describe flag
- Single-metric win but secondary metrics negative → note explicitly; do not bury
- Sample size below `sample-size-calculator` minimum → cap at 2

ICE total = I + C + E (max 15). Use as a priority signal for "ship now vs. queue."

**CATE framing** — in the card's Impact field, express the Conditional Average Treatment Effect in
plain language: "For [segment], [variant] produced [X% / $Y / N seconds] lift in [metric] over
[window]." Resist over-generalizing from a single segment or traffic slice to all users.

---

## The learning card format

```markdown
## [Test name] — [YYYY-MM-DD closed]

**Hypothesis:** [If we [change], then [metric] will [direction] because [mechanism].]
**Variants:** Control: [desc] · Treatment: [desc]
**Audience:** [segment, % of total, N per variant]
**Run window:** [start] → [end] ([N days])

### Result
| Metric | Control | Treatment | Delta | p / CI | Sig? |
|--------|---------|-----------|-------|--------|------|
| Primary: [metric] | | | | | ✓/✗ |
| Secondary: [metric] | | | | | ✓/✗ |

**CATE statement:** [For X audience, Treatment produced Y lift in Z metric over W days.]

### Confidence grade: [C score]/5
[Bullet list of threats, flags, and gotchas from gates. "Clean" if none.]

### ICE scores
Impact [I/5] · Confidence [C/5] · Ease [E/5] · **Total [N/15]**

### Decision: [Ship / Kill / Iterate / Replicate]
**Rationale:** [1–2 sentences. Tie to brand's north-star metric and ICP stage.]

### What we learned (the durable insight)
[1–3 sentences in present tense — what is now true about this audience/surface/message that was
not known before. Avoid tautology ("the winner won"); extract the mechanism.]

### Next test ideas
1. [Specific hypothesis that follows from this result]
2. [...]

### Winning-variant launch checklist
*(Omit section if decision is Kill or Replicate.)*
- [ ] QA variant in staging for [device/browser targets]
- [ ] Confirm tracking parity — event names unchanged, goals firing
- [ ] Roll out to [%] of traffic for [N days] before full launch
- [ ] Set revert trigger: if [metric] drops [threshold], roll back within [hours]
- [ ] Update brand's brand.md proof section with confirmed lift `[verify if not yet in prod]`
- [ ] Notify [owner] to deprecate control assets
- [ ] Schedule post-ship validation pull at [date]
```

---

## Saving and compounding

After the card is finalized:
1. **Append** to `./experiments/learning-log.md` (create if absent; never overwrite prior entries).
2. **Emit a one-line index entry:** `[YYYY-MM-DD] [test name] | ICE [N/15] | [Ship/Kill/Iterate] | [primary metric delta]`
3. Offer to feed "Next test ideas" into `experiment-pipeline-backlog-prioritizer`.
4. If the decision is Ship and the lift is brand-proof-worthy, prompt the user to update `brand.md`
   via `brand-brain refresh` — mark the number `[verify]` until live data confirms it.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No card before brand context loads. ICP framing and metric anchoring come from the brand, not from inference.
- **Quality gates before grading.** Never encode a measurement artifact as a real lift. Run `data-qa-measurement-gotcha-checker`; surface every flag in Confidence.
- **CATE over averages.** Always scope the result to the actual tested audience; never generalize silently.
- **Honest confidence, hard ceilings.** SRM or p > 0.05 caps Confidence at 2 and blocks "Ship" by default. Don't paper over a bad test with a clever narrative.
- **Durable insight, not a trophy.** The "What we learned" field must name a mechanism — not just "Treatment B won." If you can't name a mechanism, flag it and recommend a follow-up.
- **Compound the log.** Every card appended to `./experiments/learning-log.md`; the log is the asset.
- **Launch checklist is not optional when shipping.** No winning-variant decision without the checklist.

## What Not to Do

- Don't log a result without running the measurement quality gate — a ghost conversion is not a win.
- Don't generalize a segment-level CATE to all users without explicitly noting the scope.
- Don't mark a test as Ship when SRM is detected — require a re-run.
- Don't skip secondary metrics; a primary-only win with negative secondaries is a partial loss.
- Don't invent mechanism explanations — if the "why" is unknown, say so and write a test to find out.
- Don't overwrite prior entries in learning-log.md — append only.
- Don't reimplement brand resolution or experiment design here; call the right siblings.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; ICP and north-star metric anchor the card's framing?
- `data-qa-measurement-gotcha-checker` run when results are GA4-sourced; flags surfaced in Confidence?
- `validity-threat-checker` run if no prior threat review; SRM / novelty / window flags captured?
- Confidence auto-downgrade rules applied; score honest (no bumping a p=0.06 to a 3)?
- CATE statement scoped correctly to the tested audience — no silent over-generalization?
- "What we learned" names a mechanism, not just the winner?
- Winning-variant launch checklist present if decision is Ship?
- Card appended to `./experiments/learning-log.md`; index line emitted?
- "Next test ideas" linked to `experiment-pipeline-backlog-prioritizer` offer made?
