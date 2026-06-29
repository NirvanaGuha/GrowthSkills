---
name: sample-size-calculator
description: >
  Given a baseline conversion rate, minimum detectable effect (MDE), confidence level, and
  statistical power, computes the required sample size per variant and an estimated runtime
  in days based on your daily traffic. Works for two-variant A/B tests and multi-variant (MVT)
  setups. Applies the standard two-proportion z-test (as popularized by Evan Miller's calculator), flags
  every validity threat that could invalidate the result before you start — SRM risk, novelty
  effect window, attribution lag, GA4 sampling thresholds, minimum traffic floor — and outputs
  a launch-ready test spec you can hand to engineering or drop into your CRO backlog.
  Composes `validity-threat-checker` for pre-launch threat review and connects forward to
  `a-b-multivariate-test-designer` for the full hypothesis/design doc and `post-test-learning-logger`
  for structured result capture. Use when the user says "how big does my sample need to be,"
  "how long should I run this test," "is my traffic enough to A/B test," "calculate sample size,"
  "what MDE can I detect," "will my test be underpowered," or pastes a conversion rate and asks
  how long to run.
---

# Sample Size Calculator

Before you launch any A/B test, you need two numbers: how many visitors per variant, and how many days that takes. Get them wrong — too few and the test is underpowered; cut early and you chase noise. This skill computes both correctly using the two-proportion z-test, flags every validity threat that could make the numbers irrelevant, and hands you a ready-to-use test spec.

It does not design the hypothesis, write copy variants, or analyze results. Those live in the sibling skills this one feeds into.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's context: product category, funnel stage, and any known baseline metrics on file. Does not implement brand scanning or storage; that lives in `brand-brain`, once.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their product category, the page/funnel step being tested, and any historical conversion rate benchmarks before proceeding.
- **`validity-threat-checker`** (Step 4) — takes the test design and flags novelty effect, SRM risk, seasonality, instrumentation bias, and other threats before launch. Synthesize inline when absent.
- **`data-qa-measurement-gotcha-checker`** (Step 4) — data-quality gate: confirms the measurement setup can actually detect the effect (GA4 sampling, event tracking, attribution windows). Synthesize inline when absent.
- **`a-b-multivariate-test-designer`** *(forward handoff)* — use after this skill to build the full hypothesis doc, variant spec, and QA checklist.
- **`post-test-learning-logger`** *(forward handoff)* — use after the test concludes to log the result, decision, and next action as a structured learning card.
- **`experiment-results-analyzer`** *(forward handoff)* — for significance calculation and decision framework once results are in.

---

## How a run works

```
Step 0  Load brand context   ──► brand-brain (or fallback)
Step 1  Gather inputs        ──► baseline CVR · MDE · α · power · variants · daily traffic
Step 2  Compute sample size  ──► two-proportion z-test → N per variant → total N
Step 3  Estimate runtime     ──► N / daily traffic per variant (with holdout factor)
Step 4  Flag validity threats ──► validity-threat-checker + data-qa-measurement-gotcha-checker
Step 5  Output test spec     ──► table + launch criteria + forward handoffs
```

---

## Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest to anchor the test spec — product category informs realistic MDE ranges; known proof points inform variant framing; funnel stage informs which baseline CVR benchmark to apply if the user does not supply one.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their product category, the page/funnel step being tested, and any historical conversion rate benchmarks before proceeding.

---

## Step 1 — Gather inputs

Ask for the five inputs if not provided. Never proceed with assumed values without flagging them.

| Input | Symbol | Default if omitted | Notes |
|---|---|---|---|
| Baseline conversion rate | p₁ | *ask* | The current CVR at the test surface (last 30 days, same segment) |
| Minimum detectable effect | MDE | *ask* | Relative or absolute — clarify which; relative is the standard |
| Statistical significance | α | 5% (two-tailed) | 95% confidence; one-tailed only if you can justify directionality |
| Statistical power | 1-β | 80% | 90% is better for high-stakes tests; flag the tradeoff |
| Number of variants | k | 2 (A/B) | Increases N per variant for MVT (Bonferroni correction) |
| Daily unique visitors to the test surface | n/day | *ask* | Do not use total site traffic — scope to the exact funnel step |

**On MDE:** if the user names a revenue goal instead of an MDE, back-calculate the minimum CVR lift needed to hit the goal, then use that as the MDE. Show the working.

---

## Step 2 — Compute sample size (two-proportion z-test)

### The formula

This skill uses the standard two-proportion z-test formulation (as implemented in Evan Miller's calculator at [https://www.evanmiller.org/ab-testing/sample-size.html](https://www.evanmiller.org/ab-testing/sample-size.html) [verify]):

```
p₂ = p₁ × (1 + MDE_relative)   [or p₁ + MDE_absolute]

p̄  = (p₁ + p₂) / 2

N per variant = (z_α/2 + z_β)² × [p₁(1-p₁) + p₂(1-p₂)]
                ─────────────────────────────────────────
                              (p₂ - p₁)²
```

Z-score reference (exact values):

| Confidence | α | z_α/2 |
|---|---|---|
| 90% | 0.10 | 1.645 |
| 95% | 0.05 | 1.960 |
| 99% | 0.01 | 2.576 |

| Power | β | z_β |
|---|---|---|
| 80% | 0.20 | 0.842 |
| 90% | 0.10 | 1.282 |
| 95% | 0.05 | 1.645 |

**MVT Bonferroni correction:** for k variants (including control), multiply N per variant by `k − 1` comparisons using adjusted α: `α_adjusted = α / (k − 1)`.

### Show the full working

Always display: p₁, p₂ (derived), the z-scores used, the N-per-variant result, and total N across all variants. Round N up to the nearest integer — never down.

### Output table

```
── Sample Size Summary ─────────────────────────────────────────────
  Baseline CVR (p₁):         [x]%
  Target CVR (p₂):           [x]%   ([+x]% relative MDE)
  Confidence:                [x]%   (α = [x], two-tailed)
  Power:                     [x]%   (β = [x])
  Variants:                  [k] (including control)

  Required N per variant:    [N] visitors
  Total N across all variants: [N×k] visitors
────────────────────────────────────────────────────────────────────
```

---

## Step 3 — Estimate runtime

```
Days to significance = N per variant ÷ (daily traffic to test surface ÷ k)
```

**Apply these floors and flags — they override the math:**

1. **Minimum 1 full business cycle.** Never less than 7 days regardless of traffic. 14 days is the preferred minimum to smooth weekday/weekend behavior. Flag if the math says fewer.
2. **Novelty effect buffer.** First 3–5 days of a UI/copy test often show inflated engagement from returning users seeing something new. If the test involves a visible page change, add a 5-day discard window and restate the runtime.
3. **Attribution window.** If the conversion event has a meaningful lag (e.g., free trial → paid conversion at 14 days), runtime must extend past the last enrolled user's attribution window. Flag this explicitly.
4. **GA4 sampling threshold.** GA4 applies sampling to Explorations at ~10M events/property/month [verify]. If the test surface generates event volumes near this threshold, flag it and recommend a GA4 360 or BigQuery export path.
5. **Traffic floor check.** If total daily traffic to the test surface is below ~200 visitors/day, flag the test as low-power regardless of the math and recommend either widening the test surface, increasing MDE, or reducing confidence to 90%.

```
── Runtime Estimate ────────────────────────────────────────────────
  Daily visitors to test surface: [n]
  Traffic split per variant:      [n/k]/day
  Days to reach N per variant:    [days]

  ⚠ Minimum runtime:  [max(calculated, 14)] days
  [Any flags: novelty buffer / attribution lag / GA4 sampling / low traffic]
────────────────────────────────────────────────────────────────────
```

---

## Step 4 — Validity threat review

Invoke `validity-threat-checker` passing the test design (surface, variants, runtime, traffic split, measurement tool). If absent, run inline:

- **SRM (Sample Ratio Mismatch):** confirm the traffic split mechanism (random hashing on a stable ID — cookie, user ID, or session — not page load). A/A test recommendation: run a 7-day A/A before the real test if you've never validated the split.
- **Novelty effect:** flag if the test changes something visible that existing users have learned to ignore.
- **Seasonality:** flag if the runtime window crosses a known promotional event, holiday, or release.
- **Instrumentation:** confirm the conversion event fires once per user, not per session or page view.
- **Holdout contamination:** if variants share a page or funnel step with other active tests, flag interaction risk.

Invoke `data-qa-measurement-gotcha-checker` for the GA4 / measurement-layer gate. If absent, check inline: Is the conversion event a GA4 key event? Is it deduped to one fire per user per session? Is direct/self-referral excluded from attribution? Is the (not set) dimension rate on the test page below 5%?

---

## Step 5 — Output the test spec

Produce a single, pasteable test spec. Save to `./experiments/[test-name]-sample-size.md` if a test name was provided or if the user asks to save.

```markdown
# [Test Name] — Sample Size Spec
**Date calculated:** [today]
**Brand:** [slug, from brand-brain]
**Test surface:** [page/flow step]

## Inputs
| Parameter | Value |
|---|---|
| Baseline CVR | [x]% |
| MDE (relative) | [+x]% |
| Confidence | [x]% |
| Power | [x]% |
| Variants | [k] |
| Daily traffic (test surface) | [n] |

## Results
| Metric | Value |
|---|---|
| N per variant | [N] |
| Total N | [N×k] |
| Estimated runtime | [days] days |
| Recommended launch date | [today + buffer] |
| Recommended stop date (earliest) | [launch + days] |

## Pre-launch validity flags
[Bullet list from Step 4]

## Launch criteria (don't call it early)
- [ ] Minimum [days] days elapsed
- [ ] N per variant reached for all variants
- [ ] No SRM detected (chi-square p > 0.05)
- [ ] No active conflicting tests on this surface
- [ ] Attribution window closed for all enrolled users

## Next steps
- Design the full test → `a-b-multivariate-test-designer`
- Log results after stopping → `post-test-learning-logger`
- Analyze significance → `experiment-results-analyzer`
```

---

## MDE sensitivity table (when asked or when traffic is borderline)

When a user's traffic is low or they're unsure what MDE to target, produce a sensitivity table showing the runtime at multiple MDE levels. This reframes the question from "will my test work" to "what effect size can I realistically detect given my traffic."

```
MDE (relative) | N per variant | Days to significance
    +5%        |   [N]         |   [days]
   +10%        |   [N]         |   [days]
   +15%        |   [N]         |   [days]
   +20%        |   [N]         |   [days]
```

Flag the "minimum meaningful MDE" — the smallest effect that would actually change a business decision. Testing for a 2% relative lift on a 1% CVR page is almost always a waste of traffic.

---

## Principles

- **Show the math.** Always display the formula inputs and intermediate values — a black-box answer is not trustworthy for a test decision.
- **Never round N down.** Always round up; underpowered tests produce false negatives.
- **7-day floor is inviolable.** No runtime recommendation below 7 days. 14 days is the standard.
- **Attribution window extends runtime.** Cutting a test before delayed conversions clear is one of the most common ways CRO teams invalidate results.
- **Distinguish "not significant" from "proven no effect."** An underpowered null result is not evidence — flag it.
- **Scope traffic correctly.** N per variant draws from visitors to *the test surface*, not total site sessions.
- **SRM is silent and fatal.** A split that drifts even 3% from 50/50 corrupts the result. Flag the mechanism, not just the warning.

## What Not to Do

- Don't invent a baseline CVR — ask; or pull from brand.md if on file; mark any benchmark-derived value `[benchmark — verify against your actual data]`.
- Don't recommend running a test on fewer than 200 daily visitors without a strong caveat.
- Don't call a test early because "the line is moving" — state the stop date and enforce the launch criteria checklist.
- Don't confuse relative and absolute MDE — clarify which the user means before computing.
- Don't run a one-tailed test without documenting why directionality is known in advance.
- Don't skip the validity threat review — the number is useless if the measurement is broken.

## Quality Checklist

- `brand-brain` called (or fallback executed) before any numbers produced?
- All five inputs gathered or flagged as assumed?
- Formula displayed with full working — p₁, p₂, z-scores, N per variant, total N?
- Runtime respects the 7-day floor, novelty buffer, and attribution window?
- GA4 sampling threshold, SRM mechanism, and instrumentation verified (or flagged)?
- MDE sensitivity table offered when traffic is borderline?
- Test spec artifact includes launch criteria checklist and forward handoffs?
- No N rounded down; no CVR invented; no relative/absolute MDE ambiguity unresolved?
