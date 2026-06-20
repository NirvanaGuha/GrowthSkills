---
name: experiment-results-analyzer
description: >
  Raw results table (visitors/conversions per variant) → significance verdict, confidence interval,
  uplift, and stakeholder narrative. Takes the numbers from any A/B or multivariate test — a pasted
  table, a screenshot, or a GA4 / Optimizely / VWO export — and runs them through a rigorous
  statistical framework: two-proportion z-test (or chi-squared for MVT), sample-ratio-mismatch
  check, minimum-detectable-effect validation, Bonferroni correction for multi-variant tests, and
  full segment-cut interpretation. Returns a verdict (ship / iterate / call it off), a business
  narrative for stakeholders who don't read p-values, and the next-test recommendation. Catches the
  classic gotchas that make teams ship losing variants: peeking too early, SRM hiding in the data,
  novelty effects, one-sided vs. two-sided confusion, and conflating statistical with practical
  significance. Use when you say "analyze my test results," "did this test win?," "interpret A/B
  outcome," "test significance," "is this uplift real?," "write a results summary," or you paste
  a variant-vs-control table and need an honest verdict.
---

# Experiment Results Analyzer

Give it raw experiment data, get an honest verdict — with the math shown, the gotchas surfaced, and a narrative that non-statisticians can act on.

This skill does not run the experiment, design variants, or set up the platform. Its job starts when the data exists. If you need a test designed before you run it, call `a-b-multivariate-test-designer` first — it sets the sample size, success metric, and guardrail events that this skill then evaluates.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, and voice before writing any stakeholder narrative. Statistical verdicts are brand-agnostic; the narrative is not.
- **`a-b-multivariate-test-designer`** — call it upstream to produce a test brief; call it again after a null result to redesign the follow-up.
- **`funnel-drop-off-analyzer`** — when a winning variant improves the primary metric but a segment cut reveals downstream drop-off, pass the funnel data here for deeper diagnosis.
- **`cta-variant-generator`** — when analysis reveals the CTA was the confounding element, use this to generate a properly structured next test.
- **`stakeholder-update-status-writer`** — for converting the raw results narrative into a Slack post, board slide, or status email.

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain (voice + ICP for the narrative)
Step 1  Ingest & validate    ──► parse data; surface SRM before any stats
Step 2  Run the math         ──► significance, CI, uplift (with practical floor)
Step 3  Segment cuts         ──► flag interactions that change the verdict
Step 4  Interpret & decide   ──► Ship / Iterate / Call it off + next-test rec
Step 5  Write the narrative  ──► stakeholder summary, on brand voice
Step 6  Offer to save        ──► ./experiments/[slug]-results.md
```

---

## Step 0 — Load the brand

**Invoke the `brand-brain` skill** before writing any narrative. Brand voice governs how the stakeholder summary is written — the same 8% uplift reads differently for an enterprise brand vs. a PLG SaaS. Use only real proof from the brand digest; mark any unconfirmed numbers `[verify]`.

Fallback if absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md`, or ask for a 3-question inline mini-setup (brand name, ICP, voice adjectives). Stats are computed either way; the narrative waits for brand context.

---

## Step 1 — Ingest & validate (Sample-Ratio Mismatch first)

Accept any of: pasted table, CSV rows, screenshot description, GA4 export, or platform report text.

Parse into a canonical table:

| Variant | Visitors | Conversions | CVR |
|---|---|---|---|
| Control | n₀ | c₀ | c₀/n₀ |
| Variant A | n₁ | c₁ | c₁/n₁ |
| … | … | … | … |

**SRM check (mandatory before any stats).** Run a chi-squared goodness-of-fit test on the observed traffic split vs. the expected split (usually 50/50 for a two-variant test, or equal fractions for MVT). If χ² p-value < 0.05, **flag SRM immediately and halt interpretation** — the experiment is compromised. Common SRM causes: bot filtering applied unevenly, JS loading order errors, cookie collisions, redirect loops. Surface the most likely cause from the data and recommend a re-run.

Validate: minimum sample size met (if pre-test power analysis exists, check against it); no negative denominators; date range covers full weeks (avoids day-of-week bias).

---

## Step 2 — Statistical framework (Two-Proportion Z-Test / Chi-Squared)

Use a **two-tailed, two-proportion z-test** for A/B. For MVT or >2 variants, use chi-squared omnibus first, then pairwise z-tests with **Bonferroni correction** (divide α by number of comparisons).

**Standard α = 0.05, power = 0.80** unless the test brief specifies otherwise. Always flag if the team was peeking mid-test (sequential testing requires different math — note if this applies).

Compute and display:

| Metric | Formula / Value |
|---|---|
| Observed uplift | (CVR₁ − CVR₀) / CVR₀ × 100 |
| Absolute difference | CVR₁ − CVR₀ |
| Pooled proportion | (c₀ + c₁) / (n₀ + n₁) |
| Standard error | √(p̂(1−p̂)(1/n₀ + 1/n₁)) |
| z-statistic | (CVR₁ − CVR₀) / SE |
| p-value | two-tailed from z |
| 95% CI on uplift | CVR diff ± 1.96 × SE |
| Statistical power | post-hoc at observed effect |

**Practical significance floor.** A result can be statistically significant and commercially irrelevant. Ask (or infer from the test brief): what was the minimum detectable effect (MDE) the team set? If the observed uplift is below the MDE, call it out — it may not be worth shipping even if p < 0.05. If no MDE was set, note it as a gap.

---

## Step 3 — Segment cuts

Even a clean omnibus result can hide problems. Check the following when data permits:

| Cut | Why it matters |
|---|---|
| Device type (mobile/desktop/tablet) | A variant that wins on desktop and loses on mobile is not a winner |
| New vs. returning visitors | Novelty effect: returning visitors may flip 2–4 weeks in |
| Traffic source / channel | Paid and organic visitors often convert at very different base rates |
| Geography | Winners in one region may confuse users in another |
| Cohort/date | A mid-test platform change or external event can invalidate the whole window |

Flag any segment where the directional result reverses (interaction effect). These drive the "Iterate" verdict — you have a winner for a segment, not site-wide.

---

## Step 4 — Verdict framework

Apply this decision tree to reach one of three verdicts:

```
SRM detected?           → CALL IT OFF — rerun with clean traffic split
p > 0.05?               → NULL — no detectable effect; see sample size note
p ≤ 0.05 AND uplift ≥ MDE AND no harmful segment reversals?  → SHIP
p ≤ 0.05 AND (uplift < MDE OR segment reversal exists)?       → ITERATE
```

**SHIP:** State the uplift, the CI, and the implementation recommendation. Note any guardrail metrics (bounce, downstream conversion, revenue-per-visitor) and whether they moved.

**ITERATE:** Name the specific segment or magnitude gap. Recommend what changes — narrow the audience, adjust the variant, or extend runtime for more power.

**NULL:** Distinguish three sub-types — (a) truly no effect, (b) underpowered (ran too short), (c) test design was confounded. Recommend accordingly.

**CALL IT OFF:** SRM or fundamental integrity failure. Do not interpret the numbers; the data is not trustworthy.

---

## Step 5 — Stakeholder narrative

Write two outputs using the brand's voice from brand-brain:

**1. One-paragraph executive summary** (~80 words): what was tested, what happened, the verdict in plain English, and the one-sentence recommendation. No p-values; translate to business impact (e.g., "at this traffic volume, the uplift projects to ~X additional sign-ups per month — [verify with revenue team]").

**2. Technical appendix** (for the analyst or PM): the full stats table, segment-cut summary, gotchas flagged, and the pre/post power analysis. This is the audit trail.

---

## Classic gotchas — always check

| Gotcha | Signal | Correct move |
|---|---|---|
| **Peeking** | Test "stopped when it looked good" | Results are inflated; note sequential-testing requirements |
| **SRM** | Traffic split deviates from expected | Halt. Do not interpret. |
| **Novelty effect** | Returning-visitor CVR diverges from new-visitor | Extend runtime or segment the result |
| **Multiple comparisons** | >2 variants, no Bonferroni | Apply correction; re-evaluate each pair |
| **One-sided thinking** | Team only cares if variant wins | Report both directions; a statistically significant loss matters |
| **Stats ≠ practical sig** | p < 0.05 but uplift is 0.2% on a 1% base | Check the MDE; don't ship for a rounding error |
| **Interaction effects** | Mobile loses, desktop wins | Not a site-wide winner — scope the rollout or iterate |
| **GA4 sampling / model blending** | GA4 applies Bayesian blending for low-traffic properties | Cross-check with raw event counts; note if sampling warnings appear |
| **Attribution window mismatch** | Test ran 7 days; conversion window is 30 days | Results are premature; extend observation period |

---

## Principles

- **SRM check is non-negotiable.** Run it before any statistics. If it fails, nothing else is valid.
- **Two-tailed by default.** Losses matter as much as wins. One-sided tests require explicit pre-registration.
- **Bonferroni for multi-variant.** False-discovery rate climbs fast; correct for it.
- **Distinguish statistical from practical significance.** Always reference the MDE; note if none was set.
- **Segment cuts are not fishing.** Checking device and source is due diligence; just don't make a segment cut the primary endpoint post-hoc without correction.
- **Brand voice governs the narrative, not the stats.** The math is objective; the recommendation needs the brand's context and ICP to be actionable.
- **Truth discipline.** Business-impact projections are estimates — mark them `[verify with revenue team]` unless the conversion value is confirmed.

---

## What not to do

- Don't interpret data before running the SRM check.
- Don't declare a winner on p = 0.049 when the uplift is below the pre-set MDE.
- Don't run one-sided tests without surfacing that choice to the stakeholder.
- Don't skip Bonferroni on multi-variant tests — five variants at α = 0.05 gives a ~23% false-positive rate.
- Don't project revenue impact without flagging it as an estimate needing confirmation.
- Don't rewrite the test brief or redesign variants — call `a-b-multivariate-test-designer` for that.
- Don't write the stakeholder narrative before `brand-brain` returns.

---

## Quality checklist

- `brand-brain` called and digest loaded before writing any narrative?
- SRM check run first, before z-test or chi-squared?
- Two-tailed test used (or one-sided explicitly justified)?
- Bonferroni applied for >2 variants?
- Observed uplift compared to MDE (noted as missing if absent)?
- At least mobile/desktop and new/returning segment cuts checked?
- Verdict is one of four: SHIP / ITERATE / NULL / CALL IT OFF — with a clear one-sentence rationale?
- Business-impact projection marked `[verify]`?
- Technical appendix (stats table + gotchas) separate from the executive summary?
- Offer to save to `./experiments/[test-slug]-results.md`?
