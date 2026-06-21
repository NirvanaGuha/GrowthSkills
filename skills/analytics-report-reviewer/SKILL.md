---
name: analytics-report-reviewer
description: >
  Takes any analytics or budget-variance report draft — weekly digest, GA4 export narrative,
  channel performance deck, funnel analysis, or paid-media summary — and returns a structured
  editorial review that flags unsupported claims, missing context, weak or hidden assumptions,
  visualization issues, and the classic measurement gotchas that silently corrupt interpretation.
  Works on raw numbers, prose summaries, screenshots, or paste-ins. Composes
  data-qa-measurement-gotcha-checker for data-quality gates, channel-roi-scorecard and
  ltv-cac-payback-calculator for shared math checks, and validity-threat-checker when the report
  includes experiment results. Does NOT rebuild the report — it audits, flags, and prescribes
  fixes so the author can correct it. Use when the user says "review this report," "sense-check
  these numbers," "is this analysis solid," "flag anything wrong in this dashboard writeup,"
  "audit this analytics deck," or hands over a report draft and asks if it's ready to share.
---

# Analytics Report Reviewer

Every analytics report contains at least one silent error. Most contain three: a claim without a
denominator, a metric with no comparison period, and a GA4 gotcha the author didn't know to
question. This skill finds them before the CMO does.

It audits the report you give it against a structured framework — Claim/Evidence alignment, MECE
coverage, measurement validity, visualization honesty, and action-readiness — then returns a
prioritized finding set with severity labels and prescribed fixes. It does not rewrite the report.
It tells you exactly what to fix and why.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, proof standards, and banned
  claims. Ensures the review honors the brand's known metrics baselines and doesn't flag correct
  brand-specific conventions as errors. Fallback if brand-brain is absent or returns no brand: read
  `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user
  for the brand name, key metrics the brand tracks, and any known reporting conventions before
  proceeding.
- **`data-qa-measurement-gotcha-checker`** (required, Step 2) — runs the data-quality gate against
  GA4/measurement gotchas (attribution windows, (not set), sampling, self-referral, SRM, hostname
  spam). Do not re-derive these checks inline; delegate and fold the returned flags into the
  Measurement Validity section.
- **`channel-roi-scorecard`** (compose when report contains channel performance) — validates
  channel-level ROI math; flag mismatches rather than recalculating from scratch.
- **`ltv-cac-payback-calculator`** (compose when report contains LTV/CAC/payback claims) — verifies
  the shared math; flag formula or window inconsistencies.
- **`validity-threat-checker`** (compose when report contains A/B test or experiment results) —
  surfaces SRM, novelty effect, seasonality bias, and instrumentation threats. Returns a threat
  list; fold into the Experimental Validity section.

---

## How a run works

```
Step 0  Load brand context     ──► brand-brain (always first)
Step 1  Intake & scope         ──► classify report type; confirm scope of review
Step 2  Data-quality gate      ──► data-qa-measurement-gotcha-checker
Step 3  Five-axis audit        ──► Claim/Evidence · MECE Coverage · Measurement Validity ·
                                    Visualization · Action-Readiness
Step 4  Compose sub-skills     ──► channel-roi-scorecard | ltv-cac-payback-calculator |
                                    validity-threat-checker (as applicable)
Step 5  Collate & prioritize   ──► severity-tagged finding list (P1/P2/P3)
Step 6  Output the review      ──► structured audit report; save to ./reports/[slug]-review.md
```

### Step 0 — Load brand context (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). Use the returned digest to anchor
the review to the brand's real proof standards, known baselines, and banned unverified claims.
Do not flag a brand convention as an error.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for the brand name, key metrics the
brand tracks, and any known reporting conventions before proceeding.

### Step 1 — Intake & scope

Identify the report type from the input:
- **Performance narrative** (weekly/monthly digest, exec summary)
- **GA4/data export** (raw numbers + minimal prose)
- **Channel breakdown** (paid, organic, email, push)
- **Funnel/conversion analysis**
- **Budget variance / P&L commentary**
- **Experiment readout** (A/B test results deck)

Confirm scope if ambiguous. State what you're reviewing in one line at the top of the output.

### Step 2 — Data-quality gate

**Delegate to `data-qa-measurement-gotcha-checker`**. Pass the report input. Fold its findings
into the Measurement Validity axis (Step 3c). Do not duplicate this check; surface the returned
flags, not a re-derivation.

### Step 3 — Five-axis audit

Run each axis in sequence. Tag every finding **P1** (blocks sharing), **P2** (must fix before
acting on the data), or **P3** (cleanup).

#### (a) Claim / Evidence Alignment

The single most common report failure: a claim that floats free of the numbers beneath it.

| Check | Flag if… |
|---|---|
| Every "up X%" claim has a stated baseline and period | Baseline missing or ambiguous |
| Percentage changes accompany absolute values | "Conversions up 40%" with n=5 |
| Superlatives ("best ever," "all-time high") have evidence | Unverified best-evers |
| Causation language ("because," "drove," "resulted in") has a mechanism, not just correlation | Correlation presented as causation |
| Attribution is correctly scoped to the model used | "Email drove $X" without stating the attribution model |

#### (b) MECE Coverage

The report should cover all relevant dimensions without overlap or silence.

| Check | Flag if… |
|---|---|
| All channels/segments contributing >5% of the metric are represented | Silent contributors |
| Period comparisons are symmetric (same calendar days, same session window) | "This month vs last month" straddling a 28/31-day gap |
| Negative results and flat trends are disclosed, not just wins | Cherry-picked periods or segments |
| Denominator is stated for every rate (CVR, CTR, open rate) | Rate without denominator |
| New vs. returning split present for conversion/revenue metrics | Aggregated sessions masking new-user drop |

#### (c) Measurement Validity

Built from `data-qa-measurement-gotcha-checker` output + these mandatory GA4/measurement gotchas.
Flag each that applies:

- **Attribution window mismatch** — compare ad platform windows vs GA4 session/event attribution;
  discrepancies must be disclosed and reconciled or explicitly noted as unreconciled.
- **GA4 sampling** — GA4 Explorer reports sample above ~500K sessions; flag if the report uses
  an Exploration surface without noting potential sampling.
- **`(not set)` volume** — if `(not set)` exceeds ~3% of a key dimension (source/medium, landing
  page, campaign), data quality is compromised; flag.
- **Self-referral / hostname pollution** — GA4 counts cross-domain self-referral as a traffic
  source; flag if the brand has multiple domains and the report includes direct or referral
  breakdowns without confirming a cross-domain measurement setup.
- **Sample Ratio Mismatch (SRM)** — for experiment readouts: flag if control/variant split
  deviates >1% from intended allocation (compose `validity-threat-checker`).
- **SDK↔direct phantom flip-flop** — known GA4 artifact; sessions switching between SDK and
  direct can inflate direct or app traffic; flag for mobile-heavy brands.
- **Conversion event misconfiguration** — verify the reported conversion event maps to the correct
  GA4 key event; "purchases" may include micro-conversions if misconfigured.
- **Spam hostname traffic** — GA4 properties without hostname filters can include bot/iframe
  traffic; flag if the report includes raw Sessions without a hostname filter applied.

#### (d) Visualization Issues

Review any included charts, tables, or referenced dashboards.

| Check | Flag if… |
|---|---|
| Y-axis starts at zero for absolute metrics | Truncated axis exaggerating change |
| Dual-axis charts label both axes | Unlabeled right axis with different scale |
| Color usage is consistent (red = bad, green = good) | Same color for unrelated metrics |
| Tables have a sort logic and a "total" row | Unsorted tables with no rollup |
| Trend lines span enough periods to be meaningful | 2-point "trend" |
| Legend is present when ≥2 series shown | Unlabeled series |

#### (e) Action-Readiness

A report that surfaces facts without directing action is half-finished.

| Check | Flag if… |
|---|---|
| Each key finding maps to a named next action | Finding with no "so what" |
| Owner/deadline assigned per action (or explicitly noted as TBD) | Floating recommendations |
| Confidence level stated for forward-looking projections | Projections presented as certainties |
| Anomalies are explained (or explicitly noted as unexplained) | Spike/drop with no hypothesis |
| Audience is identified and report depth matches their decision-making level | Board-level deck with row-level data |

### Step 4 — Compose sub-skills (conditional)

- **Channel breakdowns present** → invoke `channel-roi-scorecard`. Pass channel data. Flag math
  discrepancies returned; fold into Claim/Evidence findings.
- **LTV/CAC/payback claims present** → invoke `ltv-cac-payback-calculator`. Validate formula and
  window. Flag formula deviations.
- **Experiment results present** → invoke `validity-threat-checker`. Pass the test design details.
  Fold returned threats into Measurement Validity (SRM, novelty effect, seasonality, instrumentation).

### Step 5 — Collate & prioritize

Group findings by severity:
- **P1 — Blocks sharing:** a finding that, if shared as-is, will produce a wrong business
  decision or publicly embarrassing factual error (missing denominator, causation-as-correlation,
  unacknowledged SRM).
- **P2 — Must fix before acting:** a finding that won't embarrass but will cause the team to
  act on flawed data (attributionwindow mismatch, (not set) >3%, cherry-picked period).
- **P3 — Cleanup:** a finding that improves trust and professionalism but doesn't invalidate
  the conclusions (visualization, missing owner, unlabeled axis).

### Step 6 — Output

```markdown
## Analytics Report Review — [Report title or "Untitled"] · [date]
Brand: [slug, from brand-brain]
Report type: [type from Step 1]
Overall verdict: READY TO SHARE | REVISE BEFORE SHARING | DO NOT SHARE

### P1 — Blocks Sharing
[numbered findings, each: axis · claim · why it matters · prescribed fix]

### P2 — Fix Before Acting
[numbered findings, same format]

### P3 — Cleanup
[numbered findings, same format]

### Measurement Validity (from data-qa-measurement-gotcha-checker)
[paste or summarize the data-qa flags]

### What passes
[brief list of axes that check out — omit if everything is P1/P2]
```

Save to `./reports/[brand-slug]-analytics-review-[YYYY-MM-DD].md`.

---

## The Analytics Report Audit Framework

This skill applies the **CEMVA** framework: Claim/Evidence, Exclusions/MECE, Measurement Validity,
Visualization, Action-Readiness. Reviewers/auditors in this library (this skill,
`validity-threat-checker`, `ad-to-landing-page-message-match-auditor`) review siblings' output —
they don't rebuild it. Each axis has a specific failure mode list; every finding gets a severity
and a fix, not a vague suggestion.

The five most common P1 failures in growth marketing reports, in order of frequency:
1. Percentage change with no stated baseline (n too small to be meaningful)
2. Causation language over correlation data
3. Attribution model unstated on revenue/conversion claims
4. (not set) >5% in source/medium silently absorbing untagged campaigns
5. Experiment readout with SRM never checked

---

## Principles

- **Reviewer, not rewriter.** Prescribe the fix; don't rewrite the report. The author owns it.
- **Brand-brain first.** No review before context loads — known baselines and brand conventions
  must inform what counts as a real finding vs. an expected pattern.
- **Compose, don't re-derive.** Data-quality checks live in `data-qa-measurement-gotcha-checker`;
  experiment validity lives in `validity-threat-checker`; call them, fold their output in.
- **Severity discipline.** P1 means "do not share until fixed." Don't dilute it. P3 is not a
  compliment — it is a real finding at lower urgency.
- **Real proof or `[verify]`.** Every benchmark cited in a finding (e.g., "(not set) >3% is
  problematic") must be a real industry standard or marked `[verify]`.
- **Silence on what's solid.** The "What passes" section exists, but keep it brief — the job is
  finding what's wrong, not celebrating what's right.

---

## What Not to Do

- Don't rewrite the report or produce a "corrected" version — prescribe fixes, not prose.
- Don't skip `brand-brain`; without brand context, you'll flag correct brand conventions as errors.
- Don't delegate GA4 gotcha checks to inline reasoning — compose `data-qa-measurement-gotcha-checker`.
- Don't invent benchmarks (e.g., "good CTR is 2%") without sourcing or `[verify]` marking.
- Don't conflate P2 and P3 severity to soften the review — a wrong business decision is P1.
- Don't produce a review longer than the report itself; if the input is a 200-word digest,
  the review should be focused, not encyclopedic.
- Don't mark everything P1 — severity inflation makes the review useless.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` invoked and brand context loaded (or fallback applied)?
- `data-qa-measurement-gotcha-checker` delegated and its output folded into Measurement Validity?
- All five CEMVA axes covered, even those with no findings (note "passes" for each clean axis)?
- Every finding has: axis · claim · why it matters · prescribed fix · severity label?
- Sub-skills composed for channel ROI, LTV/CAC, and experiment results where present?
- Overall verdict (READY / REVISE / DO NOT SHARE) stated at the top?
- Output saved to `./reports/[brand-slug]-analytics-review-[YYYY-MM-DD].md`?
- No invented benchmarks — everything cited is real or `[verify]`'d?
