---
name: aha-moment-activation-metric-definer
description: >
  Product description + usage data → documented aha-moment definition, activation metric formula,
  and dashboard spec. Takes any combination of event logs, cohort retention tables, onboarding
  flow descriptions, user interviews, or product analytics exports and surfaces the single
  behavioral signal that predicts long-term retention — then translates it into a measurable
  activation milestone, a tracking formula, and a dashboard spec the engineering or analytics
  team can implement without further interpretation. Opinionated about method: uses the
  correlation-over-opinion framework (Reforge / Sean Ellis-era growth method) for aha-moment discovery and the
  North Star Metric structure for metric design. Does NOT guess from product descriptions alone
  when data is available; calls `funnel-drop-off-analyzer` and `growth-diagnostic-deep-dive`
  when more investigation is needed. Use when the user says "define our aha moment,"
  "what is our activation metric," "why aren't users converting to active," "onboarding isn't
  sticking," "define the north star for activation," or pastes cohort data and asks what the
  signal means.
---

# Aha-Moment & Activation Metric Definer

Most teams measure activation by gut feel or by whatever the first PM wrote in a launch doc. This skill replaces that with a documented, data-grounded definition: the exact behavior that predicts long-term retention, turned into a metric formula, an instrumentation spec, and a dashboard that tells you on any given day whether new users are hitting the moment or sliding past it.

Brand context loads first. The activation metric must be calibrated to the ICP and offer mechanics the brand has already validated — not to generic SaaS benchmarks.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand context: ICP, offer mechanics, positioning, proof. The aha moment must be real-value-delivery for *this* ICP, not a proxy.
- **`funnel-drop-off-analyzer`** (call when onboarding funnel data is available) — surfaces where users fall out before or after the candidate aha moment; feeds Step 2 correlation.
- **`growth-diagnostic-deep-dive`** (call when macro-level retention or activation trend data is available) — contextualizes whether an activation problem exists before defining the metric.
- **`lifecycle-journey-mapper`** (call when the full customer lifecycle is unclear) — maps the pre-activation journey so the aha-moment window is correctly bounded.
- **`usage-triggered-message-sequencer`** (downstream) — once the activation event is defined, hands the event definition to this skill to wire up trigger-based onboarding nudges.
- **`a-b-multivariate-test-designer`** (downstream) — once a candidate activation metric is chosen, use this to design the experiment validating that optimizing for it moves retention.
- **`experiment-results-analyzer`** (downstream) — reads the activation experiment results and confirms or refutes the metric choice.
- **`data-qa-measurement-gotcha-checker`** (downstream) — audits the tracking implementation before the metric goes live.

---

## How a run works

```
Step 0  Brand context     ──► call brand-brain; load ICP + offer mechanics
Step 1  Inputs audit      ──► assess what data is present; set the discovery path
Step 2  Discover          ──► correlation analysis OR structured inference; name the aha moment
Step 3  Define the metric ──► formula, time-window, cohort logic, segmentation cuts
Step 4  Dashboard spec    ──► tiles, KPIs, filters, alert thresholds
Step 5  Activation brief  ──► one-page doc saved to ./reports/
```

---

## Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool) before any analysis. The returned digest anchors two decisions:

- **ICP alignment check.** The aha moment must correspond to the ICP's core job-to-be-done, not a superficial product interaction. If the ICP is mid-market eCommerce retention teams, "created a segment" is more meaningful than "visited the dashboard."
- **Offer mechanics gate.** If the product has a trial or freemium gate, the activation window is bounded by that limit. Load the real offer structure; do not assume.

Fallback: read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if absent, ask the user for ICP + offer mechanics before proceeding.

---

## Step 1 — Inputs audit

Classify what the user has provided and route accordingly:

| Input present | Discovery path |
|---|---|
| Cohort retention table (retained vs. churned) + event log | **Path A — Correlation Analysis** (most rigorous) |
| Funnel drop-off report or step-by-step conversion data | Call `funnel-drop-off-analyzer`; synthesize with Path B |
| Product description + onboarding steps only (no data) | **Path B — Structured Inference** (explicit; mark outputs `[verify with data]`) |
| Mix of partial data | Combine paths; note confidence per claim |

If the user has data but hasn't attached it, ask once: "Can you paste the cohort retention table or event frequency data? Even a summary table speeds this up."

Never conflate the two paths. Label structured-inference outputs as hypotheses requiring data validation.

---

## Step 2 — Discover the aha moment

### Path A — Correlation analysis (Reforge / Sean Ellis-era growth method)

The aha moment is the behavioral signal with the highest correlation to long-term retention — not the moment users *feel* delighted, but the moment they *do* the thing that predicts they will stay.

1. **Split the cohort.** Retained users (still active at Day 30 / 60 / 90 — pick the brand's natural retention horizon) vs. churned users.
2. **Map candidate events.** List 5–12 discrete actions users can take in the first session or first week. Score each on two axes:
   - **Retention lift:** % of Day-N retainers who performed this event in the first [window] vs. % of churned users.
   - **Causality plausibility:** does this event represent genuine value delivery, not just product depth (e.g., "used the product's core job," not "clicked settings")?
3. **Apply the 3-test filter:**
   - Retention lift > 2× between doers and non-doers (directional; exact threshold is brand-specific) [verify].
   - Event is completable in the first session or first meaningful use window (onboarding window).
   - Event maps to ICP's core job-to-be-done (from brand-brain).
4. **Name the aha moment** in plain language: *"Users who [specific action] within [time window] retain at [X]× the rate of those who don't."* Mark numbers `[verify]` unless from user-supplied data.

### Path B — Structured inference (when data is absent)

Use when only a product description and onboarding flow are available. Be explicit that this is a hypothesis backlog, not a validated finding.

1. **Map the value ladder.** What is the earliest moment the user experiences the product's core value? (Not "saw the feature" — "completed the loop once.")
2. **Apply the Value-First heuristic.** The aha moment is the first time the product does its job for the user, not the first time the user completes a setup step.
3. **Generate 3 candidate moments** ranked by proximity to core value delivery.
4. **Mark all three** `[verify with cohort data]` and recommend Path A as the validation step.

---

## Step 3 — Define the activation metric

Once the aha moment is identified, translate it into a measurable metric. Use the **North Star Metric structure** (amplitude.com / Reforge convention [verify]):

### The formula template

```
Activation Rate (Day-[N]) =
  Users who completed [aha-moment event(s)] within [time window] after [start trigger]
  ÷ Users who [start trigger]
  × 100
```

Fill in each slot:

- **Aha-moment event(s).** Exact event name(s) from the analytics schema, not colloquial descriptions. If the schema is unknown, specify the behavior and note `[map to event schema]`.
- **Time window.** The onboarding window — typically first session, Day 1, Day 3, or Day 7, depending on product cadence. Default to Day 7 unless the product's natural usage cycle is shorter. Justify the choice.
- **Start trigger.** Signup confirmed, trial started, first login — whichever marks the start of the onboarding window. Be precise; misaligned start triggers inflate or deflate activation rates artificially.
- **Segmentation cuts (mandatory minimums):** by acquisition channel, plan/tier, ICP fit (if scored), and device/platform. Aggregate activation rates hide the channel that's broken.

### Accompanying metrics (document alongside, not instead of)

| Metric | Formula | Why it matters |
|---|---|---|
| Time-to-aha (median) | Median minutes/hours from start trigger to aha-moment event | Reveals onboarding friction even when activation rate looks healthy |
| Aha-moment completion funnel | Step-by-step conversion from signup to aha event | Pinpoints where to intervene |
| Activation-to-retention correlation | Day-N retention rate split by activated vs. not | The validation that the metric is predictive |

---

## Step 4 — Dashboard spec

Specify the minimum viable activation dashboard so an analytics engineer can build it without guessing:

```
## Activation Dashboard Spec

### Tiles (required)
1. Activation Rate (Day-[N]) — time series, trailing 30 days, with 7-day rolling avg
2. Activation Rate by Acquisition Channel — bar chart, current period vs. prior period
3. Time-to-Aha Distribution — histogram, p25 / p50 / p75 labeled
4. Step-by-Step Onboarding Funnel — funnel chart from [start trigger] to aha-moment event

### Filters
- Date range (default: last 30 days)
- Acquisition channel
- Plan / tier
- ICP segment (if scored)

### Alert thresholds
- Activation Rate drops >15% WoW → notify growth lead [verify threshold with team]
- Time-to-Aha p50 increases >25% WoW → flag for onboarding review

### Data sources
- Events table: [specify — GA4, Amplitude, Mixpanel, custom] [map to schema]
- Cohort start table: [signups / trial_started / first_login]

### Refresh cadence: daily
```

If the user's stack is known (from brand-brain or stated), adjust the spec to that platform's conventions.

---

## Step 5 — Activation brief

Compile a one-page brief and save it to `./reports/activation-metric-[brand-slug]-[YYYY-MM-DD].md`. Include:

- **Aha moment statement** (one sentence, behavioral, ICP-anchored)
- **Activation metric formula** (filled-in template from Step 3)
- **Confidence level** (Path A = data-validated | Path B = hypothesis; mark clearly)
- **Discovery method** (correlation analysis or structured inference)
- **Data sources used** and any gaps
- **Segmentation cuts** to track
- **Dashboard spec** (from Step 4)
- **Recommended next experiment** (pointer to `a-b-multivariate-test-designer`)
- **Open questions / `[verify]` items**

Do not save to the skill folder. Do not write to `brand.md` — activation metrics are product artifacts, not brand artifacts.

---

## Principles

- **Correlation over conviction.** Gut feel about the aha moment is a starting hypothesis, not a finding. Always document the confidence level and the path to validation.
- **Behavior, not belief.** The aha moment is an observable event in the data, not "when users realize the value." If you can't specify the event name, you haven't defined the metric.
- **The window matters as much as the event.** An activation metric without a time window is not a metric — it's a funnel step with no urgency signal.
- **Segment from day one.** An aggregate activation rate is a lagging average. Channel-level and ICP-level splits are where you find the levers.
- **Brand context over benchmarks.** "Users who do X in 7 days" is not one-size-fits-all. The window, the event, and the target rate must fit the product's natural usage cycle and ICP.
- **Truth discipline.** If data is absent, say so and mark outputs `[verify with data]`. Do not hallucinate retention lift numbers.

---

## What not to do

- Do not name a setup step (email confirmed, profile completed) as the aha moment — setup is prerequisite, not value delivery.
- Do not define the aha moment from product description alone without explicitly labeling it a hypothesis.
- Do not produce a dashboard spec for a metric that hasn't been defined (steps are sequential).
- Do not write to `brand.md` — activation metrics are saved to `./reports/`, not brand context.
- Do not use generic SaaS benchmarks ("30% activation rate is industry standard") as targets without verifying them for this product/ICP.
- Do not skip segmentation cuts — an unsegmented activation rate almost always obscures the real problem.

---

## Quality checklist

- `brand-brain` called and ICP + offer mechanics loaded before any analysis?
- Inputs classified and discovery path labeled (A = data-validated, B = hypothesis)?
- Aha moment stated as a specific, observable behavior — not a feeling or a setup step?
- Activation metric formula complete: event name, time window, start trigger, segmentation cuts?
- Accompanying metrics (time-to-aha, step funnel, activation-to-retention correlation) documented?
- Dashboard spec includes tile definitions, filters, alert thresholds, and data source mapping?
- Activation brief saved to `./reports/` (not brand.md, not skill folder)?
- All numbers from user data cited; all inferred numbers marked `[verify]`?
- Downstream skill pointers included (`a-b-multivariate-test-designer`, `usage-triggered-message-sequencer`)?
