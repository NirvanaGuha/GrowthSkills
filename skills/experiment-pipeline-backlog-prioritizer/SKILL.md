---
name: experiment-pipeline-backlog-prioritizer
description: >
  Takes a growth problem, funnel stage, and a raw list of test ideas — however messy — and turns
  them into a fully structured, ICE/RICE/PIE-scored experiment backlog with hypothesis briefs,
  effort estimates, and a sequenced run-order. Works as a standing backlog manager: add ideas
  any time, re-score on cadence, and always know what to run next. It does NOT run significance
  math on results (that is experiment-results-analyzer's job) or design the full test spec (that
  is a-b-multivariate-test-designer's job) — it prioritizes and structures so those downstream
  skills start from a ranked brief, not a pile of sticky notes. Use when someone says "prioritize
  our tests," "what should we run next," "clean up the CRO backlog," "which experiments have the
  highest ROI," "score these ideas," "build a hypothesis," "our testing is ad hoc and scattered,"
  or hands over a list of ideas and asks where to start.
---

# Experiment Pipeline & Backlog Prioritizer

A test you don't run has zero expected value. A test run in the wrong order delays the ones that matter. This skill turns a pile of growth ideas into a ranked, hypothesis-structured, execution-ready backlog so the next sprint starts with the highest-leverage work, not the most recently remembered idea.

Layer: L2 Strategy & Planning. Senior strategist voice; no sugar-coating; flags gaps a junior would miss.

---

## Skills this calls

- **`brand-brain`** (required first) — loads active brand context: ICP, funnel shape, offer mechanics, positioning. Grounds prioritization scores in real audience + funnel data, not generic weights.
- **`funnel-drop-off-analyzer`** — if the user has GA4 funnel data available, call this to surface the highest-leverage drop-off points before scoring; it turns guesswork into evidence.
- **`a-b-multivariate-test-designer`** — hand off a prioritized hypothesis brief to get the full test spec (variants, success metric, feature-flag spec, platform setup).
- **`experiment-results-analyzer`** — after a test closes, pass the results table here for significance verdict and stakeholder narrative.
- **`landing-page-heuristic-live-cro-auditor`** — when ideas are thin or vague, call this first to generate evidence-backed hypotheses from the actual page.
- **`growth-diagnostic-deep-dive`** — when the user doesn't have a clear growth problem yet, call this first; it surfaces the lever list that the backlog should target.
- **`data-qa-measurement-gotcha-checker`** — before any test launches, flag measurement pitfalls that would invalidate results.

---

## How a run works

```
Step 0  Load brand context        ──► call brand-brain
Step 1  Clarify scope (if needed) ──► funnel stage, goal metric, traffic volume, platform
Step 2  Collect + normalize ideas ──► extract hypothesis elements; flag thin ideas
Step 3  Score with chosen method  ──► ICE (default) | RICE | PIE
Step 4  Sequence the backlog      ──► stack-rank, apply constraints, assign run-order
Step 5  Write hypothesis briefs   ──► one per top-tier test
Step 6  Save + present            ──► write to ./plans/experiment-backlog.md
```

---

## Step 0 — Load brand context (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`) before touching the backlog. Use the returned digest to calibrate: Which funnel stage is the ICP in? What is the primary conversion goal? What offer mechanics and destinations exist? What proof points are available for variant copy?

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none exists, ask: (1) primary conversion goal, (2) funnel stage under test, (3) monthly unique visitors or qualified sessions at that stage, (4) current baseline conversion rate.

---

## Step 1 — Clarify scope

Ask only what is missing. You need:
- **Growth problem** — "checkout drop-off is 74%," "trial-to-paid is 8%," "homepage bounce is 69%"
- **Funnel stage** — top (awareness/acquisition), middle (activation/consideration), bottom (conversion/retention)
- **Goal metric** — the one number that moves. One metric per backlog run; secondary metrics are diagnostics.
- **Traffic volume** — estimated weekly sessions at the test URL/step. Informs minimum detectable effect.
- **Test platform / feature-flag setup** — sets feasibility ceiling for effort scoring.

If the user provides a raw idea dump, extract these implicitly where possible before asking.

---

## Step 2 — Normalize raw ideas into hypothesis skeletons

For each incoming idea, extract or construct:

```
Hypothesis:  If we [change X] for [audience segment] at [funnel stage],
             then [goal metric] will [increase/decrease] by [~magnitude],
             because [mechanism / behavioral rationale].
```

A hypothesis without a mechanism is a guess. If the "because" is missing, write `[mechanism unknown — requires research]` and flag it; these ideas score lower on Impact until the mechanism is validated.

---

## Step 3 — Score with the right framework

Default to **ICE**. Switch to **RICE** when the backlog has items at very different reach scales (e.g. homepage vs. niche segment email). Switch to **PIE** for pure CRO/landing-page backlogs where ease-of-implementation is the key constraint.

### ICE (default — fast, daily-driver)

| Dimension | 1–10 | Calibration |
|---|---|---|
| **Impact** | Score on goal metric uplift × funnel leverage. 10 = moves the needle materially; 1 = marginal. Anchor to baseline: e.g., 74% drop-off means even a 5-pt lift is significant. |
| **Confidence** | How strong is the evidence? Prior A/B data → 8–10. Qualitative/heuristic → 4–6. Gut → 1–3. |
| **Ease** | Inverse of effort. Dev-free (copy/color) = 9–10. Two-sprint build = 3–4. Requires data pipeline work = 1–2. |

**ICE score = (Impact + Confidence + Ease) / 3** — use the average, not the product, to prevent any one 10 from overriding real constraints.

### RICE (when reach varies significantly)

**Score = (Reach × Impact × Confidence) / Effort**
- Reach: estimated users who will see the change per period.
- Impact: 0.25 / 0.5 / 1 / 2 / 3 multiplier.
- Confidence: 100% / 80% / 50% expressed as decimal.
- Effort: person-weeks.

### PIE (for pure CRO page backlogs — Bryan Eisenberg)

| Dimension | 1–10 | Calibration |
|---|---|---|
| **Potential** | How much room is there to improve? High current friction = high score. |
| **Importance** | How much traffic / revenue passes through this page/step? |
| **Ease** | Same as ICE Ease. |

**PIE score = (Potential + Importance + Ease) / 3**

---

## Step 4 — Sequence the backlog

Scoring is necessary but not sufficient. Apply these sequencing constraints after ranking:

1. **Velocity tax.** A test that takes 3 sprints to build costs 3 sprints of opportunity — downweight heavy builds unless Impact is extreme.
2. **Mutual exclusivity.** Flag test pairs that fight for the same page section; they must run serially.
3. **Learning dependency.** If Test B's hypothesis depends on Test A's result, mark A as a prerequisite.
4. **Traffic budget.** Low-traffic stages need higher-lift tests to reach significance in reasonable time; flag if the test's MDE is unreachable in under 4 weeks at current volume. Formula: `n ≈ 16σ²/δ²` (two-sided, 80% power, α=0.05) [verify exact constant for your target power].
5. **Seasonal / campaign interference.** Flag tests that should not run during peak promotions (confounded reads).

Output: a numbered run-order with tier labels (Tier 1 = run now / Tier 2 = queue / Tier 3 = park).

---

## Step 5 — Hypothesis briefs (Tier 1 only)

For each Tier 1 experiment, produce:

```markdown
### [#N] [Short test name]
**Hypothesis:** If we [change] for [segment] at [stage], then [metric] will [direction] ~[magnitude], because [mechanism].
**Score:** ICE [I:_/C:_/E:_] = [total] | Method: ICE/RICE/PIE
**Goal metric:** [primary] | Secondary: [diagnostics]
**Estimated traffic needed:** [n per variant] (~[X weeks] at current volume)
**Effort:** [T-shirt: XS / S / M / L / XL] | Owner: [role]
**Variants:** Control vs. [Variant description]
**Dependencies / conflicts:** [none | test #N must run first | conflicts with #M]
**Pass to:** a-b-multivariate-test-designer for full test spec
```

---

## Step 6 — Save and present

Write the full scored backlog to `./plans/experiment-backlog.md` (create `./plans/` if absent). Include:
- Run metadata: brand slug, funnel stage, goal metric, scoring method, date generated.
- Full scored table (all ideas, ranked).
- Tier-labelled run-order.
- Tier 1 hypothesis briefs.
- Parked ideas with the reason parked (low confidence, traffic blocked, mechanism unknown).

Confirm save path in one line. Present a summary inline — top 3 Tier 1 picks with scores and the one-line "why now" for each.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No scoring before brand context loads. Prioritization anchored to the real funnel, not generic CRO platitudes.
- **Hypothesis has a mechanism.** "Change the button color" is a tactic. "Change the button color to reduce visual competition with the primary CTA because contrast drives fixation" is a hypothesis. Demand the because.
- **Score honestly; flag thin evidence.** A Confidence of 8 on a gut feeling is a lie that corrupts the stack-rank. Mark gut-level ideas at ≤4.
- **Sequence ≠ score.** A top-scoring idea that blocks three others, takes 6 weeks to build, and runs during Black Friday should not be Test #1.
- **Traffic sanity.** Never recommend a test that cannot reach significance in a reasonable window without flagging it explicitly.
- **One goal metric per run.** Multi-metric optimization is the graveyard of experiment programs. Pick one; make the rest secondary/diagnostic.
- **Parked is not killed.** Low-scoring ideas go to Tier 3 with a reason, not the trash. Conditions change; evidence accumulates.

---

## What Not to Do

- Don't re-derive brand context — call `brand-brain`; it owns that.
- Don't run significance math on results — that is `experiment-results-analyzer`'s job.
- Don't produce a full test spec (variants × MVT combinations × feature-flag config) — hand the brief to `a-b-multivariate-test-designer`.
- Don't present a ranked list without a run-order layer — score rank and run order are different.
- Don't score Confidence above 6 for any idea without cited prior data (internal test, reliable benchmark, or qualitative research). Gut is 1–4.
- Don't let a heavy-lift Tier 1 block the queue — flag it and recommend a parallel fast-test that generates learning while the big build is in flight.
- Don't invent traffic estimates or baseline conversion rates — ask the user or mark `[verify]`.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; ICP, funnel stage, offer mechanics loaded and reflected in the scores?
- Every idea has a normalized hypothesis with a mechanism (or an explicit `[mechanism unknown]` flag)?
- Scoring method chosen and calibrated to the backlog's range (ICE / RICE / PIE with rationale)?
- Confidence scores reflect actual evidence level — no gut-level ideas above 4?
- Run-order applied after scoring — velocity tax, mutual exclusivity, learning dependencies, traffic budget, seasonal flags addressed?
- Tier 1 items have full hypothesis briefs with estimated traffic, effort, variants, and `pass to a-b-multivariate-test-designer`?
- Backlog saved to `./plans/experiment-backlog.md` with run metadata?
- Top 3 picks presented with scores and a "why now" rationale?
