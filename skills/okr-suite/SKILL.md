---
name: okr-suite
description: >
  Turns strategic priorities and actuals into a complete OKR tree — Objectives that move the
  brand, Key Results with measurable baselines and targets, and a health critique that catches
  the most common OKR failure: all lagging, no leading. On subsequent runs with actuals,
  produces a RAG (Red/Amber/Green) progress update with a commentary layer that distinguishes
  causal contributors from confounders. Two modes: Write (new-quarter OKR drafting from
  priorities) and Review (progress scoring on existing OKRs). Opinionated about KR quality —
  it will flag output-KRs masquerading as outcome-KRs and push back until the tree is honest.
  Brand context comes from brand-brain (voice and ICP shape Objective language). Use when the
  user says "write our OKRs," "score our OKRs," "OKR health check," "draft Q[n] objectives,"
  "our OKRs need KPIs," "make our KRs measurable," "what's our OKR status," or hands you a
  strategy doc and asks for the planning output.
---

# OKR Suite

Strategic priorities in, honest OKRs out. Objectives that describe the destination in the
brand's language. Key Results that pass the "so what" test. A health critique that catches
a tree full of lagging indicators before the quarter starts. And a RAG update that tells the
story behind the numbers — not just whether you're green.

This skill writes and scores. It does not set strategy (that is the operator's job), and it does
not invent proof or actuals. Every baseline, target, and current value either comes from you or
carries `[verify]`.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, and voice. Objective
  language borrows from positioning; ICP awareness shapes which outcomes matter.
- **`experiment-results-analyzer`** — when a KR is backed by an A/B result, call this to get a
  significance verdict before treating the lift as a confirmed baseline or current value.
- **`growth-diagnostic-deep-dive`** — when the user has GA4 / GSC data and wants this skill to
  derive baselines from real metrics rather than estimates, delegate the data pull here.
- **`prioritization-framework-suite`** — when the user's strategy input is an undifferentiated
  list of initiatives, call this first to rank and focus before drafting Objectives.
- **`ltv-cac-payback-calculator`** — when a KR involves unit-economics targets (CAC, LTV, payback),
  call this to anchor the target in real cohort math rather than a round-number wish.
- **`data-qa-measurement-gotcha-checker`** — when the user provides a data export as the basis for
  baselines or actuals, run it through here to catch attribution and measurement pitfalls first.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain
Step 1  Pick the mode       ──► Write (new OKRs) | Review (score existing OKRs)
Step 2  Do the work         ──► framework execution below
Step 3  Health critique      ──► lagging/leading audit + KR quality flags
Step 4  Save artifact        ──► ./plans/okrs-[brand]-[quarter].md
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest:
voice adjectives, ICP, positioning line, offer mechanics, real proof. Use positioning language
in Objective phrasing; ICP outcomes shape which KRs are worth tracking. Do not draft a single
Objective until `brand-brain` returns.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly. If none exists, ask the user for three things: positioning line, ICP
job-to-be-done, and the brand's 2–3 voice adjectives. Proceed only with those in hand.

---

## Mode A — Write (new OKRs)

**Inputs needed:** strategic priorities (or a strategy doc), the quarter/period, and optionally
current baselines for candidate metrics. If the user hands you a flat list of initiatives
without clear focus, call `prioritization-framework-suite` before drafting Objectives.

### 1. Draft Objectives (the "where are we going?" layer)

Apply the **Objective quality check** (the standard OKR Objective qualities): an Objective is a
qualitative, inspiring, direction-setting statement — not a metric, not a project, not an output.
Each Objective should:

- Describe a meaningful *change in state* for the brand or the customer.
- Be achievable in the quarter but non-trivially ambitious.
- Be unambiguous: a neutral observer could tell at quarter-end whether you achieved it.
- Use the brand's voice (adjectives from `brand-brain`); not corporate filler.

Limit: 3–5 Objectives per quarter. More is dilution.

### 2. Draft Key Results (the "how will we know?" layer)

For each Objective, write 2–4 Key Results using the **KR quality gate** (our working checklist) and
the **lagging-vs-leading audit** (see below):

| KR quality gate | Pass criteria |
|---|---|
| Measurable | Numeric target with a unit and a direction ("increase X from A to B") |
| Outcome, not output | Describes a changed state in the world, not a task completed |
| Attributable | The team can move this metric; not 100% externally driven |
| Baselined | Has a real current value (or is marked `[verify]` if unknown) |
| Time-boxed | Target date = end of the quarter unless noted |

**Mandatory: write the baseline.** Format for every KR:
```
KR: [Metric] from [baseline] → [target] by [date]
Measurement: [how it is tracked / data source]
Owner: [role, not a name]
```

If the user has not provided a baseline, mark it `[verify — baseline needed]` and note the
recommended data source. A KR without a baseline is a wish.

### 3. Initiative column (optional but encouraged)

For each KR, list 1–3 initiatives (the "how" — projects/bets) that are expected to move it.
These are *inputs* to the OKR, not the OKRs themselves. Keeping them separate prevents the
most common OKR anti-pattern: task lists dressed as strategy.

---

## Mode B — Review (RAG progress update)

**Inputs needed:** existing OKRs (pasted or at a file path) and current actuals for each KR.

### 1. Compute progress scores

For each KR, compute progress as:
```
progress = (current − baseline) / (target − baseline) × 100%
```
RAG assignment (our working thresholds — note these differ from Google's canonical 0.0–1.0 OKR
grading scale, where ~0.7 is the target sweet spot):

| Score | Status | RAG |
|---|---|---|
| ≥ 70 % of target with time remaining | On track | Green |
| 40–69 % or concern about trajectory | At risk | Amber |
| < 40 % or already missed | Off track | Red |
| KR is a milestone (binary) | Pass/Fail | Green / Red |

Adjust for time elapsed: a KR at 50% progress with 90% of the quarter gone is Red, not Amber.
State the time-elapsed assumption explicitly.

### 2. Commentary layer (not just a scorecard)

For each Red or Amber KR, write:
- **Root cause hypothesis:** what is most likely driving underperformance (one sentence, hedged
  if not confirmed — no invented explanations).
- **Causal vs. confounded:** flag whether the shortfall is attributable to the team's initiatives
  or to an external shift (algorithm, seasonality, market). This is where most RAG updates lie.
- **Recommended action:** continue / accelerate / pivot / close. One clear call.

For Green KRs: one line confirming what is working, or whether the target was set too conservatively
(sandbagging is a real failure mode even when everything is green).

### 3. Objective health summary

Roll up KR statuses into an Objective-level verdict:
- **Achieved / On Track:** majority of KRs Green, no Red KR that is load-bearing.
- **At Risk:** one or more Amber KRs on the critical path.
- **Off Track:** any Red KR that is load-bearing, or multiple Ambers.

---

## Health Critique (runs at the end of both modes)

This is where most OKR sets earn their honest grade. Apply every gate:

### Lagging-vs-leading balance

A KR is **lagging** if it measures an outcome that you cannot influence mid-quarter (revenue
recognized, NPS score collected, annual churn). A KR is **leading** if it measures a signal that
moves *before* the lagging outcome and is actionable now (activation rate, trial starts, product
adoption depth, pipeline coverage).

A healthy KR set has at least one leading indicator per Objective. An all-lagging set is a
post-mortem, not a management tool. Flag every KR as L (lagging) or Ld (leading) and report
the ratio.

### Anti-pattern flags

| Anti-pattern | Diagnosis | Fix |
|---|---|---|
| Output KR | "Launch X," "Ship Y," "Publish Z" — describes a task | Rewrite as the *outcome* launching X is meant to produce |
| Vanity metric | Pageviews, impressions, follower counts with no conversion link | Replace with a metric tied to a business outcome |
| No baseline | Target with no current value | Mark `[verify]` and name the data source |
| Sandbagged target | "Grow X by 2%" when recent run-rate is 15%/mo | Call it out; offer a stretched target |
| Disconnected KR | KR that cannot plausibly be moved by the listed initiatives | Flag the logical gap |
| Too many KRs | More than 4 KRs per Objective | Prioritize — more KRs ≠ more rigor |

---

## Output format

```markdown
# OKRs — [Brand] · [Quarter/Period]
Generated: [date]  |  Brand: [slug, via brand-brain]  |  Mode: Write | Review

## Objective 1: [Objective text in brand voice]
**Why this quarter:** [one-sentence strategic rationale]

| # | Key Result | Baseline | Target | Owner | L/Ld | [Review: Current | % | RAG] |
|---|---|---|---|---|---|
| KR1.1 | ... | ... | ... | ... | Ld | |
| KR1.2 | ... | ... | ... | ... | L | |

**Initiatives (inputs, not KRs):**
- ...

[Review mode: Commentary for each Amber/Red KR]

---
[repeat for each Objective]

## Health Critique
- Lagging/leading ratio: [n leading / n total]
- Anti-patterns flagged: [list or "none"]
- Overall grade: [Solid / Needs refinement / Restructure advised]
- Top recommendation: [single most important fix]

[Review mode: Objective-level RAG summary table]
```

Save to `./plans/okrs-[brand-slug]-[quarter].md`. Confirm path in the final line.

---

## Principles

- **Brand-brain first.** No OKRs before `brand-brain` returns. Voice and ICP shape Objective language.
- **Outcomes over outputs.** A task completed is not a Key Result. If the KR disappears when the
  project ships, it was never a KR.
- **Baselines are not optional.** A target without a baseline is a guess with a number attached.
  Mark every unknown `[verify]` and name the data source.
- **Leading indicators are not optional.** Every Objective needs at least one KR you can see move
  before the quarter ends.
- **Honest over comfortable.** Flag sandbags, all-lagging trees, and vanity metrics even if the
  user likes the current draft. The job is an honest OKR set, not a flattering one.
- **Real proof only.** Baselines and actuals from the user's data or `[verify]`. Never synthesize
  numbers.

## What Not to Do

- Don't draft OKRs before `brand-brain` returns (or the fallback mini-setup completes).
- Don't reimplement brand scanning or voice derivation — call `brand-brain`.
- Don't let output KRs ("launch X") pass without flagging and offering a rewrite.
- Don't produce a RAG update when actuals are missing — ask for them; don't estimate.
- Don't write the OKR artifact to the skill folder — always `./plans/`.
- Don't accept an all-lagging KR set without flagging it, even if the targets are ambitious.
- Don't conflate the initiative list with the Key Results — keep the "how" and the "what we'll
  measure" as separate columns.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any Objective was drafted?
- Every Objective qualitative, inspiring, direction-setting — no metrics in the O-layer?
- Every KR: numeric, outcome (not output), baselined (or `[verify]`), time-boxed, attributable?
- At least one leading KR per Objective?
- Health critique run: lagging/leading ratio reported, anti-patterns flagged?
- Review mode: progress computed against time-elapsed, not just raw %; commentary for every
  Amber/Red KR; causal vs. confounded distinction made?
- Artifact saved to `./plans/okrs-[brand]-[quarter].md` and path confirmed?
