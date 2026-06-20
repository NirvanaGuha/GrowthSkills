---
name: a-b-multivariate-test-designer
description: >
  Hypothesis, page section, goal metric → complete test brief (variants, success metric, MVT
  combinations, feature-flag spec, platform setup checklist). Two modes: A/B (two-variant,
  single change) and Multivariate / MVT (multiple elements, factorial or fractional factorial).
  Loads brand context via brand-brain so variants are on-voice and ICP-aligned. Calls
  cta-variant-generator for CTA element tests and funnel-drop-off-analyzer when the
  hypothesis needs data grounding. Flags sample-ratio mismatch risk, minimum detectable
  effect sizing, and the classic measurement gotchas before you run a single impression.
  Produces a structured test brief ready for handoff to an engineer or a no-code experiment
  platform. Use when the user says "design a test," "A/B this page," "multivariate test,"
  "write a test hypothesis," "experiment brief," "what should I test," "how many visitors do
  I need," or hands over a CRO audit and asks what to run next.
---

# A/B & Multivariate Test Designer

Turn a conversion instinct into a rigorous experiment brief. Give it a hypothesis (or a page section and a goal), get back a complete test spec: variant descriptions, primary success metric, sample-size calculation, MVT combination matrix if needed, feature-flag spec, and a platform-setup checklist. Every variant is written in the brand's real voice, against the brand's real ICP — because brand context comes from `brand-brain`, not from guessing here.

This skill designs and documents experiments. It does not run statistical analysis on results — that is `experiment-results-analyzer`. It does not audit the live page — that is `landing-page-heuristic-live-cro-auditor`.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, ICP, offer, proof, and banned words before writing any variant copy.
- **`cta-variant-generator`** (when the test element is a CTA) — generates the on-brand CTA battery; this skill wraps it in the test structure.
- **`funnel-drop-off-analyzer`** (when the user has GA4 funnel data) — surfaces the highest-leverage drop-off to ground the hypothesis in data rather than instinct.
- **`landing-page-heuristic-live-cro-auditor`** (when no hypothesis is supplied) — fetches the live page, scores heuristics, returns the top-priority issue; this skill converts it into a test brief.
- **`experiment-results-analyzer`** (downstream) — hands off the completed test brief; user runs it there once results are in.
- **`prioritization-framework-suite`** (when building a test backlog) — ICE/RICE scores the queue; this skill provides the structured hypotheses.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (voice, ICP, offer, proof)
Step 1  Pick mode       ──► A/B (single change) | MVT (multiple elements)
Step 2  Write or refine the hypothesis (LIFT framework)
Step 3  Size the test   ──► MDE, baseline rate, power/α, required n
Step 4  Specify variants + implementation (copy, flag, platform checklist)
Step 5  Flag gotchas    ──► SRM risk, attribution, segment pollution
Step 6  Output the test brief; offer to save
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, ICP + awareness tendency, offer mechanics, real proof, and positioning. Do not write a single variant until it returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (product · ICP · offer mechanics · voice adjectives + banned words), then proceed.

---

## Step 1 — Pick the mode

| Mode | When to use | Max elements | Combinations |
|---|---|---|---|
| **A/B** | One change, cleanest signal | 1 | 2 (control + variant) |
| **A/B/n** | One element, more than one treatment | 1 | 3–5 (control + n variants) |
| **MVT — full factorial** | 2–3 elements, ≥10k visitors/week | 2–3 | 4–8 |
| **MVT — fractional factorial** | Same but traffic-constrained | 2–4 | Partial Taguchi array |

**Default to A/B.** Upgrade to MVT only when: (a) the user explicitly asks, (b) funnel data points to two interacting elements, or (c) the page has a big enough traffic budget to avoid 6-month runtimes. Call out runtime risk proactively.

---

## Step 2 — The LIFT Hypothesis

Every test needs a hypothesis written in this form, encoding the **LIFT model** (Value Proposition, Relevance, Clarity, Urgency, Anxiety, Distraction):

> **If** we [change X on page/step Y]  
> **then** [primary metric] will [increase/decrease] by [MDE %]  
> **because** [the LIFT element this addresses: value prop / relevance / clarity / urgency / anxiety / distraction]  
> **for** [ICP segment or all visitors].

If the user provides a vague brief ("make the homepage convert better"), do not guess — ask which LIFT element the evidence points to, or call `landing-page-heuristic-live-cro-auditor` to surface it first.

---

## Step 3 — Size the test (non-negotiable gate)

Undersized tests are broken before they start. Always calculate before writing variants.

```
Inputs needed:
  baseline_rate   (current CVR / CTR for the primary metric)
  MDE             (minimum detectable effect — typically 10–20% relative lift)
  α               (significance threshold — default 0.05 / 95% CI)
  power           (default 0.80; use 0.90 for high-stakes or irreversible tests)
  tails           (2-tailed default; 1-tailed only if the direction is guaranteed)
  weekly_visitors (to the tested URL/step)

Sample-size formula (two-proportion z-test approximation):
  n_per_variant = 2 × (z_α/2 + z_β)² × p(1−p) / (MDE_abs)²
  where p = baseline_rate, MDE_abs = baseline_rate × MDE_rel

Runtime estimate = (n_per_variant × num_variants) / weekly_visitors  [weeks]
```

| Signal | Action |
|---|---|
| Runtime > 8 weeks | Flag risk; consider raising MDE or collapsing variants |
| Runtime > 16 weeks | Hard stop — recommend a different lever or page with more traffic |
| baseline_rate < 1% | Micro-conversion proxy required (add-to-cart, scroll depth, click) |
| baseline_rate unknown | Mark as `[verify]` and use a conservative 2–3% default with the caveat |

Always surface the runtime alongside the sample size — a statistically valid test that runs for 6 months is not operationally valid.

---

## Step 4 — Variant specification

### A/B mode

Write both control (documented, not invented — what is live today) and variant. For each:

- **Element changed:** exact field (headline H1, hero CTA, subhead, form label, image, pricing display)
- **Control copy / state**
- **Variant copy / state** (on-brand, on-voice, using real proof; mark anything unconfirmed `[verify]`)
- **Implementation note:** CSS class, flag name, or component prop
- **Why this angle:** which LIFT element it addresses + ICP hook

For CTA element tests, invoke **`cta-variant-generator`** (Battery mode) and pull the recommended primary + Variant B into this brief.

### MVT mode

Build the combination matrix. For 2 elements (A: 2 levels, B: 2 levels):

| Combination | Element A | Element B | Label |
|---|---|---|---|
| C | Control headline | Control CTA | Control |
| 1 | Variant headline | Control CTA | A only |
| 2 | Control headline | Variant CTA | B only |
| 3 | Variant headline | Variant CTA | A+B |

For 3+ elements or traffic-constrained pages, apply a **Taguchi L8 or L9 fractional factorial array** — list the array explicitly in the brief. Never run an undocumented fractional design; every combination must be named.

---

## Step 5 — Measurement + gotchas

Encode the standard measurement contract in every brief:

| Field | Spec |
|---|---|
| **Primary metric** | One metric only. Conversion events must be GA4-fired or platform-tracked — not inferred. |
| **Secondary guardrail metrics** | Bounce rate, avg. session duration, downstream funnel step (max 2; for safety, not optimization) |
| **Attribution window** | Match the buyer's typical decision lag; flag if the window is shorter than the sales cycle |
| **Minimum runtime** | At least 2 full business cycles (typically 2 weeks) regardless of whether n is reached sooner |
| **Segment purity** | New vs. returning visitors must be isolated if offer or awareness differs |

**Classic gotchas — flag all that apply:**

- **Sample-ratio mismatch (SRM):** if observed traffic split deviates > 1% from expected, the test is invalidated. Check on day 2 and weekly. SRM checklist: flicker, bot traffic, redirect timing, crawler exclusion.
- **Novelty effect:** new variant sees a temporary lift from curiosity; minimum 2-week runtime mitigates.
- **Peeking / early stopping:** stopping when significance is first reached inflates false-positive rate. Commit to the pre-calculated n before looking at results.
- **GA4 event integrity:** if the primary metric is a GA4 event, verify it fires exactly once per conversion, is not inflated by self-referral or bot traffic, and is not the same broken event documented in `data-qa-measurement-gotcha-checker`.
- **Platform blending:** if the test platform uses Bayesian inference (Optimizely Stats Engine, VWO, etc.), the 95% CI interpretation differs from frequentist — note it.
- **Segment pollution:** personalization layers, cached CDN pages, or feature flags targeting a different dimension can contaminate the variant bucket.

---

## Output format — the test brief

```
## Test Brief — [Short name]

**Brand:** [slug, via brand-brain]
**Page / step:** [URL or funnel step]
**Mode:** A/B | A/B/n | MVT full factorial | MVT fractional (Taguchi L_)
**Status:** DRAFT

### Hypothesis (LIFT)
If we [change] then [metric] will [direction] by [MDE%]
because [LIFT element] for [segment].

### Sample size
| Input | Value |
| Baseline CVR | x% [or verify] |
| MDE (relative) | x% |
| α / Power | 0.05 / 0.80 |
| n per variant | x,xxx |
| Variants | x |
| Weekly visitors | x,xxx |
| **Estimated runtime** | **x weeks** |

### Variants
[Variant table per §Step 4]

### Combination matrix (MVT only)
[Matrix per §Step 4]

### Measurement contract
- Primary metric: [event name + platform]
- Secondary guardrails: [max 2]
- Attribution window: [x days]
- Minimum runtime: [x weeks]
- Segment filter: [new/all/returning]

### Feature-flag / platform spec
- Flag name: [snake_case]
- Targeting: [URL, cookie, segment rule]
- Platform checklist: [ ] Variant QA'd in staging  [ ] Primary metric verified firing  [ ] SRM check scheduled Day 2  [ ] Hold-out / rollback plan documented

### Gotchas flagged
[List applicable items from §Step 5]
```

Save test briefs to `./experiments/[slug]-test-brief.md` if the user wants to persist.

---

## Principles

- **Brand-brain first.** No variant copy before the brand is loaded. Its voice + banned-words override everything here.
- **One primary metric.** Multi-metric optimization is not optimization — it is noise. Pick one and hold it.
- **Size before you write.** A beautiful variant on an undersized test is theater. Calculate n first; if the page lacks traffic, say so and suggest a higher-traffic proxy.
- **Hypothesis is falsifiable.** If you cannot state what result would disprove the hypothesis, the hypothesis is not ready.
- **Minimum runtime over minimum sample.** Even if n is reached in 5 days, run at least 2 full business cycles.
- **Truth discipline.** Real proof in variants or `[verify]`; never invent a claim to make a variant sound stronger.
- **Compose, don't duplicate.** CTA variants come from `cta-variant-generator`; funnel drop-offs come from `funnel-drop-off-analyzer`; results analysis goes to `experiment-results-analyzer`.

## What Not to Do

- Don't design a test without sizing it first — an unsized test is not a test, it's a guess with extra steps.
- Don't design MVT for pages under ~5,000 visitors/week without flagging a 6+ month runtime risk.
- Don't write control copy from memory — document what is actually live today (ask if unknown).
- Don't use secondary metrics as stopping criteria.
- Don't allow peeking — the stopping rule must be pre-committed in the brief.
- Don't invent proof or differentiators in variant copy; mark anything unconfirmed `[verify]`.
- Don't reimplement brand scanning or results analysis — call the relevant sibling skills.

## Quality Checklist

- `brand-brain` called; voice + banned words obeyed; only real proof used in variants?
- Hypothesis written in LIFT form with a named LIFT element?
- Sample size calculated with explicit inputs; runtime surfaced; traffic-gate applied?
- Each variant has an element label, both control and variant states, an implementation note, and a LIFT rationale?
- MVT brief includes a fully enumerated combination matrix (no unnamed combinations)?
- Measurement contract: one primary metric, ≤2 guardrails, attribution window, minimum runtime, segment filter?
- Platform checklist includes SRM check on Day 2?
- All applicable gotchas (SRM, peeking, novelty, GA4 event integrity) flagged in the brief?
- CTA element tests delegated to `cta-variant-generator`; results analysis pointed to `experiment-results-analyzer`?
