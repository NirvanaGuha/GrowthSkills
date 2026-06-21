---
name: validity-threat-checker
description: >
  Audits an experiment design doc (or brief) against a canonical taxonomy of validity threats —
  internal threats (SRM, instrumentation bias, novelty/primacy effects, contamination/spillover,
  regression to the mean, multiple comparisons inflation) and external threats (seasonality,
  selection bias, sample representativeness, generalizability ceiling) — and returns a
  prioritized threat register with severity, evidence, and actionable mitigations before any
  experiment launches or result is called. Powered by Shadish, Cook & Campbell's four-validity
  framework (statistical conclusion validity, internal validity, construct validity, external
  validity) combined with industry-standard SRM detection and GA4/analytics gotchas. Composes
  `a-b-multivariate-test-designer` (the upstream design the checker audits) and
  `analytics-report-reviewer` (the downstream report it guards). Does NOT redesign the experiment;
  it flags and mitigates, then defers execution decisions to the experimenter. Use whenever
  the user says "check my experiment for threats," "audit this A/B test design," "validity
  issues," "SRM check," "is this test sound," "pre-launch experiment review," "why did this
  test fail," "did novelty effect distort my results," or hands over an experiment brief, test
  plan, or results summary for a sanity check.
---

# Validity Threat Checker

Give it an experiment design or results doc, get a complete threat register. Every major threat class is interrogated — SRM, novelty, seasonality, instrumentation drift, contamination, multiple comparisons, selection bias — with severity ratings and mitigations you can act on today. It finds the holes before bad data drives a wrong call.

This skill audits and flags. It does not redesign the experiment, call a winner, or rewrite the test plan. If the design is too broken to audit meaningfully, it says so and routes you to `a-b-multivariate-test-designer`.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand context so threat severity is calibrated against the brand's real traffic volume, seasonality profile, and funnel. Does NOT reimplement brand scanning or storage.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their approximate daily unique experiment traffic, typical conversion rate, and any known seasonal peaks before proceeding.
- **`a-b-multivariate-test-designer`** *(optional, upstream)* — if the user has a design draft, it likely came from here. Load it to pull the hypothesis, variants, and success metrics directly.
- **`analytics-report-reviewer`** *(optional, downstream)* — when the input is a results summary rather than a pre-launch design, compose with this to catch weak claims and missing context in the write-up.
- **`data-qa-measurement-gotcha-checker`** *(optional)* — delegate the instrumentation and GA4-specific gotchas pass (attribution windows, sampling, (not set), self-referral, broken page_view) to this sibling rather than re-deriving them here.
- **`sample-size-calculator`** *(optional)* — when the pre-test power calculation is missing or wrong, route to this for correct sample size and runtime estimates.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (calibrates threat severity to brand scale)
Step 1  Parse the input     ──► extract: hypothesis, variants, metric(s), traffic, duration, instrumentation
Step 2  Run the threat scan ──► apply the 4-validity framework + SRM + GA4 gotchas
Step 3  Build threat register ► severity × evidence × mitigation per threat
Step 4  Surface the critical path ► P0 blockers, P1 fixes, P2 monitors
Step 5  Self-review, then present
```

### Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Use the returned digest to calibrate:
- **Traffic volume** — low-traffic sites have elevated SRM and power risks; set severity accordingly.
- **Seasonality profile** — if the brand has known peaks (BFCM, launch windows), test duration overlapping them is a P0 external-validity threat.
- **Conversion baseline** — needed to assess whether the stated MDE is achievable in the planned runtime.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their approximate daily unique experiment traffic, typical conversion rate, and any known seasonal peaks before proceeding.

---

## The four-validity framework (Shadish, Cook & Campbell)

All threats map to one of four validity types. Call it out for every finding.

| Validity type | "Did the stat hold up?" | Typical threats |
|---|---|---|
| **Statistical conclusion** | Are the numbers trustworthy? | Underpowered, multiple comparisons, peeking, high variance |
| **Internal** | Did X really cause Y? | SRM, novelty effect, instrumentation drift, contamination, regression to mean |
| **Construct** | Did we measure what we think? | Wrong proxy metric, tracking gaps, attribution window mismatch |
| **External** | Does it generalize? | Seasonality, selection bias, novelty ceiling, segment-locked results |

---

## Threat catalog (scan these in order)

### P0 — Launch blockers (fix before running)

**SRM (Sample Ratio Mismatch)**
Definition: observed variant split deviates from expected by >1% (chi-squared p < 0.01).
Pre-launch signals: unequal bucketing in the assignment layer, missing filters, cache-layer bypasses.
Post-launch detection: run chi-squared on day 1 and day 3 samples; flag immediately if p < 0.01.
Mitigation: fix assignment layer, add a holdout check, document expected split in the test plan.

**Underpowered design**
Definition: planned runtime reaches < 80% power at the stated MDE.
Check: derive required n = 2 × ((z_α + z_β)² × p(1−p)) / MDE² per variant (two-tailed, α=0.05, β=0.20 by default). If the plan doesn't include this, flag and route to `sample-size-calculator`.
Mitigation: extend runtime, widen MDE, reduce metric variance (use ratio metrics), or accept lower power with declared risk.

**Instrumentation gaps / broken tracking**
Definition: the primary metric isn't firing correctly for all variants — missing event params, misfired triggers, GA4 (not set) inflating a dimension.
Route to `data-qa-measurement-gotcha-checker` for the full GA4-specific pass.
Pre-launch gate: QA the event in GA4 DebugView for every variant state before traffic goes live.

### P1 — Fix before calling results

**Novelty / primacy effect**
Definition: short-term behavioral change driven by unfamiliarity with the variant, not genuine preference.
High-risk signals: UI treatment changes, new feature exposure, first-time users in the test.
Mitigation: run for ≥2 full business cycles (typically ≥2 weeks); segment by new vs returning users; look for convergence in the time series.

**Peeking (sequential testing without correction)**
Definition: checking results and making ship/kill decisions before planned runtime.
Mitigation: lock the analysis date in the test plan; if continuous monitoring is needed, use a sequential testing method (e.g., always-valid inference / mSPRT) rather than standard t-test.

**Multiple comparisons inflation (α inflation)**
Definition: running k simultaneous tests at α=0.05 gives a family-wise error rate of 1−(0.95)^k.
Threshold: flag when k ≥ 3 concurrent metrics or ≥ 3 simultaneous test variants.
Mitigation: Bonferroni or Holm–Bonferroni correction; pre-register exactly one primary metric; treat secondary metrics as exploratory.

**Contamination / spillover**
Definition: control group is exposed to the treatment signal (e.g., shared sessions, social proof of variant B, server-side cache serving wrong variant).
Mitigation: verify client-side vs server-side bucketing; check that variant assignment is sticky per user (not per session); use holdout groups for high-contamination risk tests.

### P2 — Monitor during / document post

**Seasonality overlap**
Definition: test window spans a known behavioral anomaly (sale event, holiday, product launch, algorithm update).
Check against brand's seasonality profile from brand-brain.
Mitigation: exclude the anomaly window from analysis OR document and apply a correction factor; never launch a test that *starts* during a peak if you can avoid it.

**Regression to the mean**
Definition: variants selected because they looked anomalously high/low will drift toward the mean during the test, creating a false negative or false positive.
High-risk signal: test idea was generated by spotting a recent metric spike or trough.
Mitigation: establish a stable baseline window (4+ weeks pre-test) before measuring lift.

**Selection bias / non-representative sample**
Definition: test traffic is a non-random slice — geo filter, logged-in only, one device type — and results are being generalized to the full population.
Mitigation: state the scope explicitly in the test plan; flag any generalization claim that exceeds the tested population.

**Novelty ceiling / external validity cap**
Definition: the winning variant's lift is real but is bounded by the test audience; it won't replicate at full rollout (different segments, markets, or device mix).
Mitigation: document tested segment; plan a phased rollout with monitoring; compare segment-level results before full ship.

---

## How to read the threat register output

```
## Validity Threat Register — [Experiment name / ID]
Audited: [date]   Brand: [slug, via brand-brain]   Input type: [design doc / results]

### P0 — Blockers (do not run / do not call until resolved)
| # | Threat | Validity type | Evidence in this design | Mitigation |
|---|---|---|---|---|

### P1 — Fix before calling results
| # | Threat | Validity type | Evidence | Mitigation |
|---|---|---|---|---|

### P2 — Monitor or document
| # | Threat | Validity type | Evidence | Action |
|---|---|---|---|---|

### Verdict
[One paragraph: overall soundness, critical path item, and whether results (if present) are callable or should be flagged as compromised]
```

Save to `./experiments/[test-id]-validity-threats.md` if asked; inline otherwise.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Threat severity is relative to the brand's traffic volume and seasonality — load it before scoring anything.
- **Compose, don't duplicate.** Instrumentation/GA4 gotchas go to `data-qa-measurement-gotcha-checker`; power math goes to `sample-size-calculator`; don't re-derive.
- **Severity is honest.** An underpowered test is a P0 blocker even if the team is excited to launch. Call it.
- **Flag, don't redesign.** This skill identifies and mitigates; if the design needs structural surgery, route to `a-b-multivariate-test-designer`.
- **Pre-register primacy.** Multiple-metric designs always need one declared primary; flag when it's missing.
- **Real proof or nothing.** Every severity rating is grounded in the input doc; no invented claims.

## What Not to Do

- Don't call a winner or loser — that's `experiment-results-analyzer`.
- Don't redesign the experiment — that's `a-b-multivariate-test-designer`.
- Don't re-implement the GA4/tracking audit — route to `data-qa-measurement-gotcha-checker`.
- Don't assign P0 severity to every finding — reserve it for genuine blockers that would invalidate the result.
- Don't skip brand-brain on the grounds that the experiment "isn't copy-related" — traffic volume and seasonality live there.
- Don't produce a threat register without a verdict paragraph; the register without interpretation is raw data, not a deliverable.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and traffic/seasonality context loaded (or fallback inputs collected)?
- All eight catalog threats interrogated — each either raised or explicitly cleared?
- SRM check spec present (expected split, chi-squared gate, day-1 QA)?
- Power calculation present or routed to `sample-size-calculator`?
- Threats correctly bucketed P0 / P1 / P2 — no severity inflation?
- Mitigations are actionable, not generic ("run longer" is not a mitigation; "extend from 7 to 14 days to capture 2 full business cycles given brand's Mon–Fri conversion pattern" is)?
- Verdict paragraph present — verdict is callable/not-callable, not just a summary of the table?
- If instrumentation gaps flagged, `data-qa-measurement-gotcha-checker` composed or explicitly called out for the follow-up?
