---
name: growth-diagnostic-deep-dive
description: >
  GA4 + GSC data, a pasted metrics dump, or a plain question → structured diagnosis of what is
  driving or dragging growth with a prioritized lever list. Applies the MECE growth tree (Acquire →
  Activate → Retain → Expand → Refer) to any dataset or freeform question, surfaces the highest-
  leverage breakdowns, and encodes the measurement gotchas that routinely produce false diagnoses
  (attribution window drift, GA4 session vs. user model shifts, GSC lag, spam hostname bleed,
  broken page_view events, sample-ratio mismatch). Delivers a crisp verdict — not a data dump —
  with a ranked lever list and the next question to answer at the top of each lever. Call this
  skill whenever someone says "why is growth down," "diagnose our funnel," "what's dragging CAC,"
  "where are we leaking," "growth deep dive," "break down the numbers," "what's the bottleneck,"
  "what should we fix first," or hands over a metrics table and asks what it means.
---

# Growth Diagnostic Deep-Dive

Most "growth is down" questions are answered wrong because the analyst describes the data instead of
diagnosing the system. This skill does the opposite: it applies a structured decomposition to find
the one or two levers that actually explain the movement, rules out measurement artifacts before
calling them real, and hands back a ranked list with next actions — not a summary of what you
already know.

---

## Skills this calls

- **`brand-brain`** (Step 0, required) — loads ICP, positioning, offer destinations, and proof so
  every recommendation is calibrated to this business, not a generic SaaS template.
- **`data-qa-measurement-gotcha-checker`** (Step 1, when installed) — runs the canonical
  measurement-artifact checklist on the input data before diagnosis begins; synthesize inline if
  absent.
- **`funnel-drop-off-analyzer`** — when the problem is activation or in-funnel conversion, call
  it with the relevant event/cohort slice rather than re-implementing the analysis.
- **`ltv-cac-payback-calculator`** — call when the lever list points to a CAC or payback problem.
- **`channel-roi-scorecard`** — call when the input includes cross-channel spend data.
- **`experiment-pipeline-backlog-prioritizer`** — when a lever is confirmed, route to this skill
  to scope and prioritize the experiments that test it.
- **`ga4-weekly-traffic-digest`** — call to pre-fetch a clean traffic summary if the user gives a
  GA4 property ID but no pre-pulled data.

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain (voice, ICP, offer, proof)
Step 1  Audit the data           ──► measurement-gotcha-checker (or inline checklist)
Step 2  Decompose the tree       ──► MECE growth tree, isolate the broken branch
Step 3  Diagnose the branch      ──► quantify, segment, compare; form the verdict
Step 4  Rank the levers          ──► Impact × Confidence × Effort (ICE) table
Step 5  Present + persist        ──► verdict, ranked levers, next question per lever
```

---

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned ICP and offer to
calibrate what "good" benchmarks look like for this business (self-serve SaaS ≠ enterprise ≠
eCommerce). Obey voice and banned words in the output narrative. **Fallback if `brand-brain` is absent or returns nothing:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for ICP, offer, and positioning, or proceed with clearly-labeled generic assumptions marked `[verify]`.

### Step 1 — Audit the data before diagnosing it

Run `data-qa-measurement-gotcha-checker` (or the inline checklist below) on every data input
**before** forming any hypothesis. A dirty number is not a lever — it is a measurement failure.
Flag and quarantine artifacts; only clean signals enter the tree.

**Inline gotcha checklist (use when the sibling skill is absent):**

| Gotcha | What to check |
|---|---|
| Spam / bot hostname bleed | Filter to verified hostnames; flag if `(not set)` or unknown hosts inflate sessions |
| Broken `page_view` event | Verify GA4 enhanced measurement is firing; missing `page_view` kills session counts silently |
| GA4 session vs. user model mismatch | Confirm whether report uses session-scoped or user-scoped metrics; mixing them inflates/deflates funnels |
| Attribution window drift | Check if the lookback window changed (default GA4 30-day click / 1-day view); a change shifts credit across channels |
| Model blending (data-driven vs. last-click) | Note the active attribution model; switching models changes channel split without changing reality |
| GSC 3-day lag | GSC data is ~3 days stale; exclude the trailing 3 days before WoW comparisons |
| GSC impressions ≠ GA4 organic sessions | These will never match; explain why in the output rather than papering over the gap |
| Sample-ratio mismatch (for experiment data) | If running A/B tests, verify variant allocation ratios before reading results |
| Cohort edge effects | Partial weeks/months at the boundary of a date range distort averages; use complete periods |

Flag every artifact found. If an artifact is likely causing the reported problem, say so and stop
the diagnosis until the user can provide cleaner data or confirms it can be excluded.

---

## Step 2 — Decompose the MECE growth tree

Slot every metric the user provides into the growth tree. Identify which branch deviates from
trend or benchmark. Do not analyze every branch — find and isolate the broken one(s).

```
Growth (Revenue / ARR / MRR)
├── Acquire            Sessions / Leads / Sign-ups / Pipeline
│   ├── Organic        Branded vs. Non-branded SEO · GSC clicks · DA/backlinks
│   ├── Paid           Impressions → Clicks → CPC → CPL
│   ├── Direct/Referral
│   └── Product-led    Viral / word-of-mouth / free-tier → paid signal
├── Activate           Free→Paid rate · Time-to-aha · Onboarding completion
├── Retain             Logo churn · NDR / GDR · Renewal rate · Cohort L28/L90
├── Expand             Upsell rate · Seat expansion · AOV growth · NRR > 100%
└── Refer              Referral rate · Advocate NPS · Review velocity
```

**Diagnosis rule:** the branch with the largest absolute deviation from expected drives 80 % of
the problem. Resist diagnosing multiple equal-priority problems in a single session — if the data
genuinely shows two breaks, note the smaller one and focus the lever list on the bigger.

---

## Step 3 — Diagnose the broken branch

Once the branch is isolated:

1. **Quantify the gap.** Express as absolute delta and % change vs. comparison period. Avoid
   describing direction without magnitude — "organic is down" is not a diagnosis.
2. **Segment to find the driver.** Cut the metric by channel, page/entry point, device, geography,
   cohort, or plan tier until one slice explains ≥ 50 % of the gap.
3. **Form the root-cause verdict.** One sentence. Example: *"Non-branded organic sessions fell
   −38 % MoM; the top-5 informational keywords each lost 3–6 positions, concentrated in the /blog/
   subdirectory — consistent with a content-quality update impact, not a technical issue."*
4. **Assign confidence.** High (data confirms), Medium (circumstantial), Low (hypothesis only).
   Mark anything Low-confidence as `[hypothesis — verify]`.

---

## Step 4 — Rank the levers (ICE)

Produce a lever table. Levers are actions — not symptoms. Each row = one thing the team can
actually do. Score 1–10; ICE = Impact × Confidence × Effort⁻¹.

```
### Growth Levers — [brand] — [date]
Diagnosis: [one-sentence verdict]

| # | Lever | Branch | Impact | Confidence | Effort | ICE | Next question |
|---|---|---|---|---|---|---|---|
| 1 | … | … | 8 | 7 | 3 | 18.7 | … |
```

- **Impact:** magnitude of the opportunity if the lever moves.
- **Confidence:** how certain is the root cause? Penalise unverified data.
- **Effort:** inverse — high effort = low score.
- **Next question:** the single cheapest test or lookup that would raise confidence from Medium →
  High before committing to a fix. This is the most actionable column; never leave it blank.

Cap at 5–7 levers. If you have more, the diagnosis is not specific enough — keep decomposing.

---

## Principles

- **Verdict first, evidence second.** Lead with the one-sentence conclusion; show the work after.
  A diagnosis that buries the finding in caveats is not a diagnosis.
- **Measurement artifacts before root causes.** No hypothesis survives a data-quality failure.
  Always run Step 1 before Step 2.
- **One broken branch at a time.** Breadth is not depth. If everything looks broken, the
  measurement is broken — keep auditing.
- **Calibrate to the business model.** A 40 % organic drop is catastrophic for a content-led SaaS
  and irrelevant for a sales-led enterprise. Pull from `brand-brain` to set the right stakes.
- **Confidence levels are mandatory.** Every claim is High / Medium / Low. Low = `[hypothesis —
  verify]`. Never present a Low-confidence finding as an action item.
- **The lever is the action, not the symptom.** "Traffic is down" is a symptom. "Publish 8 updated
  informational posts targeting non-branded keywords" is a lever.

---

## What not to do

- Don't skip Step 1. A diagnosis built on dirty data sends the team after a ghost.
- Don't describe the data without a verdict. Restating what the user already sees is not value.
- Don't produce more than 7 levers — prioritization is the job.
- Don't use generic benchmarks ("industry average CTR is 3 %") without qualifying them to the
  brand's model, traffic mix, and stage.
- Don't diagnose across two equal root causes simultaneously — pick the higher-magnitude one and
  note the other.
- Don't call a `[hypothesis — verify]` a confirmed finding anywhere in the output.
- Don't reproduce the full data table the user pasted — reference it by name, not by copy.

---

## Quality checklist

- `brand-brain` called; ICP + offer used to frame stakes and benchmarks?
- Measurement-gotcha checklist run before any hypothesis formed; artifacts flagged?
- Growth tree decomposed; broken branch(es) named; only the dominant branch analysed in depth?
- Verdict is one sentence, states the lever, the magnitude, and the confidence level?
- Lever table present: ≤ 7 rows, ICE scored, "Next question" column populated for every row?
- No invented benchmarks; no Low-confidence claim presented as an action?
- Artifact-corrupted inputs quarantined or called out explicitly?
- Offer to save report to `./reports/growth-diagnostic-[brand]-[date].md` if the user wants it?
