---
name: funnel-drop-off-analyzer
description: >
  GA4 funnel report, event list, or activation cohort data → ranked highest-leverage drop-off
  points with hypothesized causes and recommended tests. Accepts a GA4 exploration export, a
  pasted step table, a product-event list, or a plain description of the funnel and drop numbers.
  Ranks drop-offs by revenue impact, generates credible hypotheses using a 5-layer friction model
  (built on Fogg's B=MAT, extended with two house layers), and outputs a prioritized experiment
  backlog (ICE-scored) ready to hand to a test designer. Flags all classic GA4 measurement
  gotchas before any diagnosis. Does NOT run A/B tests or manage experiment records — it calls
  sibling skills for those. Use when the user says "why is my funnel leaking," "find my biggest
  drop-off," "funnel analysis," "where are users dropping," "step-by-step conversion analysis,"
  "activation funnel breakdown," or pastes a table of step-by-step conversion rates.
---

# Funnel Drop-Off Analyzer

Turn a leaky funnel into a ranked, hypothesized, experiment-ready fix list. This skill takes raw
funnel data — GA4 exploration output, a pasted step table, activation cohort numbers, or a plain
description — and returns the highest-leverage drop-off points with hypothesized causes and
ICE-scored tests to run next.

It does not guess. It checks for measurement problems first, then ranks by revenue impact, then
hypothesizes using a real friction model. If the data is too sparse to diagnose, it says so and
asks for what is needed.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand context: ICP, offer, positioning, proof.
  ICP awareness stage and conversion path are the lens for every hypothesis. Called in Step 0.
- **`data-qa-measurement-gotcha-checker`** — called on any GA4 data input before diagnosis;
  surfaces broken events, model blending, self-referral, and spam hostnames that corrupt funnel
  numbers. If absent, run the embedded GA4 Gotcha Gate (Step 2) inline.
- **`a-b-multivariate-test-designer`** — hand off the top experiment(s) from the backlog for a
  full test brief (variants, success metric, sample-size calculation, feature-flag spec).
- **`cta-variant-generator`** — when a drop-off hypothesis points to a weak CTA, call here for
  on-brand replacements before writing the test brief.
- **`lifecycle-journey-mapper`** — when the funnel covers post-signup activation, call here for
  the full lifecycle stage context.
- **`landing-page-heuristic-live-cro-auditor`** — when a top-of-funnel step is a landing page
  URL, call here for an evidence-backed heuristic audit before hypothesizing.

---

## How a run works

```
Step 0  Load the brand               ──► brand-brain (always first)
Step 1  Accept + structure the input ──► normalize funnel data into a step table
Step 2  GA4 Gotcha Gate              ──► measurement QA before any diagnosis
Step 3  Drop-off ranking             ──► revenue-impact sort (not raw % sort)
Step 4  Hypothesis generation        ──► 5-layer friction model per top drop-off
Step 5  ICE-scored experiment backlog──► ranked test list, ready for hand-off
Step 6  Offer to save + hand off     ──► ./experiments/[slug]-funnel-analysis.md
```

---

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool) before producing any output. Use the returned ICP — especially
awareness tendency and conversion path — as the interpretive frame for every hypothesis. A
"confusing pricing step" means something different for a most-aware buyer than for a
problem-aware one. Obey voice and banned words in all output.

Fallback if `brand-brain` is not installed: read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly; if absent, ask for brand slug + ICP + offer + primary funnel goal before
proceeding.

---

### Step 1 — Accept and structure the input

Normalize whatever the user provides into a **Funnel Step Table**:

| Step | Name | Users | Conv. Rate | Drop-Off % | Drop-Off N |
|---|---|---|---|---|---|
| 1 | Landing page view | — | — | — | — |
| 2 | Signup start | — | X% | Y% | N |
| … | … | … | … | … | … |

If the user gives a GA4 exploration screenshot or export, extract the step names and numbers
verbatim. If they describe it in plain text, construct the table and confirm it before proceeding.
If cohort or activation data is provided (e.g., Day 1 / Day 7 / Day 30 retention), add a Cohort
Drop-Off view alongside the step table.

Ask for missing inputs (total users at each step, goal, attribution window) only once, batched.

---

### Step 2 — GA4 Gotcha Gate (measurement QA first)

Do not diagnose until measurement integrity is confirmed. If `data-qa-measurement-gotcha-checker`
is installed, invoke it now. Otherwise, run this inline checklist — flag any that apply before
proceeding:

| Gotcha | Check |
|---|---|
| **Broken page_view event** | Steps tracked via `page_view`? Confirm event fires on every virtual page; SPA navigation commonly misfires. |
| **Attribution-window bleed** | Funnel uses a default 30-day lookback? Cross-session steps (email → visit → convert) may undercount. |
| **Self-referral / spam hostnames** | Any non-production hostname (staging, localhost, partners) inflating top-of-funnel? |
| **GA4 model blending** | Report using `Data-driven` or `Last click` attribution? Blended models redistribute credit across steps and distort step-level rates. |
| **Sample-ratio mismatch (for A/B data)** | If data is from a live test, check that variant split is within ±5% of target ratio. Imbalance invalidates significance. |
| **Sessionization gap** | Steps spread over >30 minutes? GA4 sessions expire and restarts can split what is one user journey into two sessions. |
| **Cohort definition drift** | Activation cohort: is "Day 0" defined consistently (signup, first event, subscription)? Inconsistent anchoring inflates or deflates activation rates. |

Document which gotchas apply, which are clean, and which are unverifiable from the data provided.
Do not proceed with a fatally flawed dataset — ask the user to re-pull with gotcha fixes first.

---

### Step 3 — Revenue-impact ranking

Sort drop-offs by **revenue impact**, not raw percentage. A 3-point improvement at a high-value
step beats a 10-point fix at a zero-revenue one.

**Revenue-impact score formula (per step):**

```
Impact = Drop-Off N  ×  (Steps remaining to conversion)  ×  Avg. Revenue per Conversion
```

If ARR/ACV is unknown, use relative weighting: multiply drop-off N by downstream step count
(a proxy for "how much pipeline does this block"). Mark revenue inputs `[verify]` if not
confirmed.

Output a **Ranked Drop-Off Table**:

| Rank | Step Transition | Drop-Off % | Drop-Off N | Impact Score | Priority |
|---|---|---|---|---|---|
| 1 | Step 2 → 3 | X% | N | high | P1 |
| 2 | Step 4 → 5 | X% | N | medium | P2 |
| … | … | … | … | … | … |

Label top 1–3 as P1. Everything else is P2/P3. Focus the hypothesis and test work on P1.

---

### Step 4 — Hypothesis generation (Fogg B=MAT + 5-layer friction)

For each P1 drop-off, generate **3–5 hypotheses** using the five friction layers below. A
hypothesis is a falsifiable causal claim, not a vague observation.

**Foundation: Fogg Behavior Model (B = Motivation × Ability × Trigger).** Fogg's model has
exactly three factors. A user drops off because at least one of: motivation is insufficient,
ability (ease) is insufficient, or the trigger is poorly timed or absent. The two extra layers
below — **Trust** and **Confusion / Mismatch** — are a house extension, not part of Fogg's model;
we add them because they show up repeatedly in funnel diagnosis. Map each hypothesis to one or
more layers:

| Layer | What it tests | Typical evidence signal |
|---|---|---|
| **Motivation** | ICP desire, offer clarity, value-prop match | Heatmap engagement, session recording exits before CTA |
| **Ability / Friction** | Form length, cognitive load, number of steps, page speed | Time-on-step, form-abandon events, error events |
| **Trust** | Social proof, guarantee, logo credibility signals | Scroll depth, hover on trust elements |
| **Trigger** | CTA placement, visibility, timing, email reminder | Click rate on CTA, rage-click, scroll-depth vs. CTA position |
| **Confusion / Mismatch** | Message-match from upstream ad/email, unclear pricing | Bounce after specific referrer, exit on pricing/FAQ step |

Hypothesis format (mandatory for each):

```
Layer:     [Motivation | Ability | Trust | Trigger | Confusion]
Claim:     Users drop at [step] because [specific, testable cause].
Evidence:  [What in the data (or brand context) supports this? Mark [verify] if inferring.]
Fix:       [One change that directly addresses the cause.]
```

Do not invent evidence. Mark inferences `[verify]`. If no credible hypothesis can be formed from
available data, say so and list what additional data would resolve it (session recordings, form
error events, heatmap, survey intercept, cohort segmentation by acquisition channel).

---

### Step 5 — ICE-scored experiment backlog

For each hypothesis that passed the evidence bar, build one test entry. Score ICE honestly:

| # | Hypothesis (short) | Fix / Variant | I | C | E | ICE | Layer | Skill hand-off |
|---|---|---|---|---|---|---|---|---|
| 1 | [claim] | [change] | 8 | 7 | 9 | 8.0 | Ability | a-b-multivariate-test-designer |
| 2 | … | … | … | … | … | … | … | … |

- **I (Impact):** revenue-impact score normalized 1–10.
- **C (Confidence):** 1–10 based on evidence quality; inferred hypotheses cap at 5.
- **E (Ease):** 1–10; no-code fix = 9, full-stack dev = 2.

Mark which tests need `a-b-multivariate-test-designer` for a full brief. Mark which copy
variants need `cta-variant-generator`. If a test requires a significant landing-page change,
flag `landing-page-heuristic-live-cro-auditor` as prerequisite.

---

## Principles

- **Measurement first, diagnosis second.** A corrupt event stream produces confident wrong
  answers. Never skip the Gotcha Gate.
- **Rank by revenue impact, not drop-off rate.** A 5% step that sits in front of a $5,000 ACV
  deal beats a 20% step that feeds a free tier.
- **Hypotheses must be falsifiable.** "Users are confused" is not a hypothesis. "Users exit the
  checkout step after viewing the shipping-cost reveal because the cost exceeds expected range
  (Motivation layer)" is.
- **Evidence or [verify].** Only use data actually present in the funnel table or brand context.
  Inferences are allowed but must be labeled.
- **Compose, don't duplicate.** When a hypothesis resolves to "fix the CTA," call
  `cta-variant-generator`. When the top-of-funnel step is a page, call
  `landing-page-heuristic-live-cro-auditor`. Don't reimplement their work here.
- **Brand lens on every hypothesis.** ICP awareness stage, offer mechanics, and conversion path
  from `brand-brain` are the frame for what "friction" means to this audience.

---

## What not to do

- Do not diagnose before running the Gotcha Gate — broken data produces confident wrong answers.
- Do not sort by drop-off percentage alone; a 40% drop at a zero-stakes step may not be P1.
- Do not invent supporting evidence; mark inferences `[verify]` or ask for the data.
- Do not produce a vague observation list ("users might be confused") — every hypothesis must
  name the layer, the specific cause, and the specific fix.
- Do not generate more than 5 hypotheses per drop-off step; more is noise, not depth.
- Do not attempt to run or track experiments here — hand to `a-b-multivariate-test-designer`.
- Do not re-implement brand resolution — call `brand-brain`.

---

## Quality checklist

- `brand-brain` called and ICP + conversion path loaded before any hypothesis?
- Funnel step table constructed and confirmed with the user?
- GA4 Gotcha Gate run; any fatal data issues called out before proceeding?
- Drop-offs ranked by revenue-impact score, not raw percentage?
- Each P1 hypothesis maps to one Fogg layer, names a specific testable cause, and cites
  evidence (or marks `[verify]`)?
- ICE table present; C-scores capped at 5 for inferred hypotheses?
- Sibling skill hand-offs noted where applicable?
- Output offered for save to `./experiments/[brand-slug]-funnel-analysis.md`?
