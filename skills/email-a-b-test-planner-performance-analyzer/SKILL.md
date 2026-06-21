---
name: email-a-b-test-planner-performance-analyzer
description: >
  Two-mode email and push A/B testing skill. Planning mode: takes a campaign goal and
  current baseline metrics, produces a statistically valid test design doc — one clear
  hypothesis, one variable isolated, sample-size calculation, runtime, segment split,
  success metric, and pre-mortem. Analysis mode: takes an ESP or push platform
  performance export and produces a consolidated performance narrative covering open rate,
  CTOR, CTR, revenue/conversion, and statistical significance verdict, with plain-English
  so-what and one next action. Framework: the PIER cycle (Plan → Isolate → Execute →
  Report). Calls brand-brain for voice context, subject-line-preview-text-optimizer for
  subject-line variants, cta-variant-generator for CTA variants, sample-size-calculator
  for the math, and lifecycle-email-push-copy-reviewer to QA the test copy.
  Use whenever the user says "A/B test my email," "design an email split test,"
  "analyze my test results," "which subject line won," "is this significant," "plan an
  email experiment," "performance results," "open rate analysis," or hands over an ESP
  CSV/screenshot asking what it means.
---

# Email A/B Test Planner & Performance Analyzer

Design tests that teach something. Analyze results that drive the next action. Every test lives and dies by a single hypothesis, one isolated variable, and a pre-calculated sample — otherwise you're just coin-flipping with brand equity.

This skill runs two modes: **Plan** (design a valid A/B test before you send) and **Analyze** (interpret results after). Both anchor to the PIER cycle. It does not write brand context, subject lines, CTAs, or copy from scratch — it calls the sibling skills that already do that.

---

## Skills this calls

- **`brand-brain`** (required, first) — loads voice, ICP, offer, proof, and banned words. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, voice adjectives, and banned words before proceeding.
- **`subject-line-preview-text-optimizer`** (Plan mode) — generates the variant subject/preview pair; do not rewrite subject lines inline.
- **`cta-variant-generator`** (Plan mode, when the test variable is the CTA) — generates the CTA variants.
- **`sample-size-calculator`** (Plan mode) — calculates required N per variant and estimated runtime; do not re-implement the math here.
- **`lifecycle-email-push-copy-reviewer`** (Plan mode, optional) — QAs the test copy for voice, CTA strength, and character-limit compliance before the design doc is finalized.
- **`post-test-learning-logger`** (Analyze mode, optional) — logs the structured learning card after a result is called.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (always first)
Step 1  Detect the mode ──► Plan | Analyze (or both, if user provides goal + results together)
Step 2  Execute the mode
Step 3  Self-review against the quality checklist, then present
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice, banned words, ICP + awareness tendency, offer mechanics, proof points, and the path to `brand.md`. Do not produce any test design or analysis copy before it returns.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, voice adjectives, and banned words before proceeding.

### Step 1 — Detect the mode

- **Plan mode** — triggered by a campaign goal, a test idea, "what should I test," or a new campaign that hasn't sent yet.
- **Analyze mode** — triggered by an ESP export, a screenshot, a metrics paste, "which won," "is this significant," or post-send results.
- **Both** — if the user provides a test design doc AND results in the same pass, run Plan review first (validate the design was sound), then Analyze.

---

## The PIER Cycle (shared framework)

PIER is the operating spine of all email experimentation:

| Phase | Job | Gate |
|---|---|---|
| **P**lan | One hypothesis, one variable, valid sample, defined winner | Don't send until all four pass |
| **I**solate | Everything identical except the test element | Segment split is random + concurrent |
| **E**xecute | Send, monitor deliverability, don't peek early | No early calls; watch for SRM |
| **R**eport | Statistical verdict, plain-English so-what, one next action | Log in `post-test-learning-logger` |

---

## Plan mode

### The hypothesis contract

Every test must have one hypothesis in this form:

> "Changing **[one variable]** from **[control]** to **[variant]** will **[increase/decrease]** **[primary metric]** because **[reason from ICP or brand data]**."

If the user can't complete this sentence, stop and help them build it before generating variants. A vague hypothesis wastes the send.

### Variables worth testing (pick exactly one per test)

| Variable | Primary metric | Secondary metric |
|---|---|---|
| Subject line | Open rate | List health (unsubscribe) |
| Preview text | Open rate | Open-to-click rate |
| From name | Open rate | Reply rate |
| Send time / day | Open rate | CTOR |
| CTA label | CTOR | Revenue |
| CTA placement | CTOR | Scroll depth / heatmap |
| Body copy angle | CTOR | Revenue |
| Offer / incentive | Revenue / conversion | List fatigue |
| Email length | CTOR | Unsubscribe |
| Personalization token | Open rate or CTOR | Depends on placement |

Do not test two variables simultaneously and call it an A/B test. That's a multivariate test (call `a-b-multivariate-test-designer` instead) or a confounded mess.

### Sample and runtime (call `sample-size-calculator`)

Pass `sample-size-calculator` the user's:
- Current baseline conversion rate for the primary metric (e.g., open rate 0.22)
- Minimum detectable effect (MDE) — default to 10% relative lift if unstated (e.g., 0.22 → 0.242)
- Confidence level — default 95%; power — default 80%

It returns required N per variant and estimated days to reach it given list size and send cadence. Surface both in the design doc. If the list is too small to reach significance in a reasonable window, say so and propose a proxy metric or a longer accumulation window instead of recommending a test that will be underpowered.

### Variant generation

- For **subject/preview tests**: call `subject-line-preview-text-optimizer` with the campaign goal and ICP context from `brand-brain`.
- For **CTA tests**: call `cta-variant-generator` in Battery mode; tell it the placement and awareness stage.
- For **copy angle tests**: sketch the angle contrast (e.g., benefit-led vs. fear-of-missing-out vs. social proof) — do not re-implement a full copy skill here; flag that the user should draft variants and return them for review.

### QA gate (optional — call `lifecycle-email-push-copy-reviewer`)

Before finalizing the test design doc, pass the control and variant through `lifecycle-email-push-copy-reviewer`. A test copy flaw (banned word, broken CTA, off-voice line) invalidates the test result after the fact. Catch it before sending.

### Test design doc output

```
## A/B Test Design — [Campaign / Send Name]
**Brand:** [slug, via brand-brain]
**Date drafted:** [today]

### Hypothesis
[Full PIER hypothesis sentence]

### Variable tested
[One variable + control vs. variant description]

### Primary metric  |  Secondary metric
[Metric + baseline]  |  [Metric]

### Sample
- Required N per variant: [from sample-size-calculator]
- Estimated runtime: [days, given send cadence + list size]
- Split: 50/50 random (or specify holdout % if list is large enough)
- Segment: [audience definition — do NOT mix segments across variants]

### Subject line / CTA / copy variants
Control: [exact string]
Variant: [exact string]
[Source: subject-line-preview-text-optimizer / cta-variant-generator / manual]

### Winner declaration rule
Call winner at [N per variant] reached with p < 0.05 (two-tailed) on [primary metric].
If no significance after [2× estimated runtime]: declare inconclusive, do not ship the "winning" variant.

### Pre-mortem: what could invalidate this test?
- [e.g., send-time bleed between cohorts]
- [e.g., seasonal event during test window]
- [e.g., SRM if opens are bot-inflated]

### Next action on completion
Log in post-test-learning-logger → [path: ./experiments/[slug]-[date].md]
```

Save test design docs to `./experiments/[brand-slug]-[test-slug]-design.md`.

---

## Analyze mode

### What the user must provide

- ESP/push export (CSV, pasted table, or screenshot) OR a verbal metrics summary with: send date, list size, variant split, opens, clicks, unsubscribes, and revenue/conversions if tracked.
- The original hypothesis and control/variant definitions, if available. If not: infer what was tested from metric deltas and flag that the design was either not logged or not shared.

### Metrics hierarchy (in order of reliability)

| Metric | What it measures | Reliability caveat |
|---|---|---|
| **Open rate** | Subject/preview + from-name resonance | MPP (Apple Mail Privacy Protection) inflates iOS opens since Sep 2021; [verify] whether ESP separates machine vs. human opens |
| **CTOR** (clicks ÷ opens) | Body + CTA effectiveness, conditional on open | More reliable than raw CTR for content quality; still MPP-contaminated on open denominator |
| **CTR** (clicks ÷ sends) | True engagement signal | Best combined metric post-MPP |
| **Conversion / revenue** | What actually matters | Requires proper UTM attribution; confirm tracking before calling this definitive |
| **Unsubscribe rate** | List health signal | A "winning" variant with higher unsub rate may be net-negative |

Always compute CTOR alongside CTR. A variant with higher open rate but lower CTOR may have a misleading subject line — it burns trust over time.

### Statistical significance check

Use the two-proportion z-test for open/click rates. The formula:

> z = (p1 − p2) / sqrt( p̄(1−p̄) × (1/n1 + 1/n2) )

where p̄ = (x1+x2)/(n1+n2). At z > 1.96 → p < 0.05 (two-tailed, 95% confidence).

If the user did not pre-specify a sample size, surface the achieved power and note whether the test was underpowered. An underpowered result with p > 0.05 is **inconclusive**, not a tie and not a control win — say so explicitly. Never declare a winner on underpowered data.

**MPP caveat:** if the ESP does not separate machine opens from human opens and the test variable is the subject line, flag that open rate significance may be inflated. Recommend CTOR or CTR as the primary metric for future subject-line tests.

### Performance narrative output

```
## A/B Test Results — [Campaign / Send Name]
**Brand:** [slug]
**Test variable:** [what was tested]
**Send date:** [date]  |  **List size:** [N]

### Metric summary
| Metric | Control | Variant | Delta | Significant? |
|---|---|---|---|---|
| Open rate | x% | y% | +/−z pp | Yes / No / Inconclusive |
| CTOR | x% | y% | +/−z pp | Yes / No / Inconclusive |
| CTR | x% | y% | +/−z pp | Yes / No / Inconclusive |
| Conversions / Revenue | x | y | +/−z | Yes / No / Inconclusive |
| Unsubscribes | x% | y% | +/−z pp | — |

### Statistical verdict
[Plain statement: "Variant [wins / loses / inconclusive] on [primary metric] at 95% confidence.
Achieved p = [x]. Sample per variant: [n1] / [n2]. Power: [~x%] [verify if underpowered]."

### MPP / attribution caveats
[List any data-quality issues — MPP inflation, missing UTMs, bot-click inflation, etc.]

### So what
[2–3 sentences: what this result means for the brand's audience and messaging strategy.
Do NOT claim causation beyond the variable tested.]

### Next action
[One concrete next step: ship the winner / run a follow-on test / declare inconclusive and retest with larger sample / log in post-test-learning-logger]
```

Save analysis outputs to `./experiments/[brand-slug]-[test-slug]-results.md`.

---

## Principles (Non-Negotiable)

- **One variable, one hypothesis.** A test that changes two things teaches nothing. Block multi-variable "tests" and route to `a-b-multivariate-test-designer` if the user insists.
- **Pre-specify everything.** Sample size, runtime, winner rule, and primary metric must be locked before sending — not chosen after seeing the numbers (HARKing: Hypothesizing After Results are Known).
- **Power before significance.** A p < 0.05 on an underpowered test is a false positive waiting to be re-tested. Call out low power explicitly.
- **MPP is real.** Open rate is a noisy signal post-Sep 2021. Default to CTOR or CTR as the primary metric when the test variable is subject line, unless the ESP separates human from machine opens.
- **Truth only.** Real metrics or `[verify]`; never invent baselines or benchmarks. Industry averages (e.g., "2.5% CTR is typical for SaaS") must be cited with source or marked `[verify]`.
- **Brand-brain first.** Voice and banned-words from brand-brain override any copy produced in test variants.

## What Not to Do

- Don't declare a winner before the pre-specified sample is reached.
- Don't test two variables and call it A/B.
- Don't write subject lines or CTAs from scratch — call `subject-line-preview-text-optimizer` and `cta-variant-generator`.
- Don't use open rate as the sole success metric for subject-line tests without flagging MPP.
- Don't apply a result from one segment to a different segment without retesting.
- Don't invent industry benchmarks — `[verify]` or omit.
- Don't ship the variant just because it "looks better" — only on a statistically significant win against a pre-specified metric.

## Quality Checklist (self-review before presenting)

**Plan mode:**
- `brand-brain` called; voice + banned-words loaded?
- One hypothesis in the full PIER sentence form?
- Exactly one variable isolated?
- `sample-size-calculator` called; N per variant and runtime stated?
- Variant strings generated via `subject-line-preview-text-optimizer` or `cta-variant-generator`, not inline?
- Pre-mortem includes at least two invalidation risks?
- Winner-declaration rule stated before send?

**Analyze mode:**
- Metric table includes open rate, CTOR, CTR, and conversion/revenue (or explicit note that it's unavailable)?
- Statistical significance computed with explicit p-value and power note?
- MPP / attribution caveats surfaced?
- Verdict is "winner / loser / inconclusive" — not just "variant performed better"?
- One next action named?
- Output saved to `./experiments/` path?
