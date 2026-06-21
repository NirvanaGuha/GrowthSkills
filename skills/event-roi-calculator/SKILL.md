---
name: event-roi-calculator
description: >
  Turns raw event data — costs, registrant counts, pipeline, and closed revenue — into a
  rigorous ROI summary with cost-per-lead (CPL), cost-per-opportunity (CPO), cost-per-acquisition
  (CPA), pipeline multiple, and benchmark comparisons against industry averages (virtual events,
  field marketing, conferences). Supports webinars, virtual summits, trade shows, field events,
  and hybrid. Two modes: Quick (one event, immediate outputs) and Batch (compare 2–6 events
  side-by-side, rank by ROI efficiency). Uses the Grubb/ITSMA B2B Event ROI Framework as the
  calculation spine — it's the closest thing to a standard practitioners actually reference.
  Surfaces the "so what" for finance and the CMO, not just the raw numbers. Calls `brand-brain`
  for voice and any brand-specific benchmarks; calls `roi-business-case-calculator` when the
  output needs a full champion-ready executive memo; calls `cfo-ready-budget-summary-slide-builder`
  when the user asks for a board-ready slide pack. Saves the model to ./events/[slug]-roi.md by
  default so the next event's run can diff against actuals. Use when the user says "calculate event
  ROI," "what did the webinar cost per lead," "event ROI report," "prove the event's value," "CPL
  for the conference," "did the event pay off," "compare our events," or hands you an event budget
  plus lead/pipeline numbers and asks you to make sense of them.
---

# Event ROI Calculator

Every event should pay its way — but "it felt great" doesn't survive a budget review. This skill
converts event inputs (costs, registrants, leads, pipeline, revenue) into a structured ROI summary
with the ratios finance asks for, benchmark context so the numbers mean something, and a plain-English
verdict the CMO can defend in a board meeting.

It calculates. It does not run events, write event copy, or design the program. If the event data is
absent or incomplete, it asks only for the gaps — it does not invent them.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's context: company ARR tier, ACV, sales cycle
  length, known event benchmarks, and voice. Use returned data for context-specific benchmark framing
  and the written verdict's tone. Obey voice + banned words throughout.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written
  brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists,
  ask the user for [event type, average deal ACV, typical sales cycle length, and voice adjectives].

- **`roi-business-case-calculator`** (optional) — call when the user wants a champion-ready executive
  ROI memo or needs to build the case for next year's event budget. Pass the computed model as input;
  don't rebuild the math.

- **`cfo-ready-budget-summary-slide-builder`** (optional) — call when the user needs a board-ready
  slide pack from the ROI summary. Pass the verdict + metrics table.

- **`budget-variance-p-l-reconciliation-analyst`** (optional) — call when the user wants a
  full actuals-vs-budget reconciliation (not just ROI). Pass the cost breakdown.

- **`analytics-report-reviewer`** (reviewer pass) — call after the model is built if the user
  asks for a quality review of assumptions, or before presenting to finance. It reviews; it does
  not rebuild the model.

---

## How a run works

```
Step 0  Load the brand  ──► call `brand-brain` skill
Step 1  Pick the mode   ──► Quick (default) | Batch (2–6 events)
Step 2  Collect inputs  ──► costs + registrants + leads + pipeline + revenue
Step 3  Run the model   ──► Grubb/ITSMA spine + derived ratios
Step 4  Add benchmark context
Step 5  Write the verdict
Step 6  Save artifact   ──► ./events/[slug]-roi.md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Pull: company ACV, ARR tier,
sales cycle, any known CPL/CPO benchmarks, voice, banned words. If brand-brain has no event
benchmarks on record, use the library defaults in the Benchmark Reference section below —
and flag them as external rather than brand-specific.

### Step 1 — Pick the mode

- **Quick (default)** — one event, immediate output. Use when the user provides data for a single event.
- **Batch** — 2–6 events compared side-by-side, ranked by ROI efficiency. Trigger: "compare," "which
  event was better," a table/list of multiple events, or an explicit count.

When unsure, default to Quick and offer Batch at the end.

---

## The input set

Collect (or ask for) the following. Mark each `[verify]` if the user flags it as estimated.

**Cost inputs (total event spend)**

| Input | Notes |
|---|---|
| Direct costs | Venue/platform, A/V, catering, booth fees, sponsorship |
| Content + production | Speakers, content creation, recording, design |
| Promotion spend | Paid ads, email, PR, social to drive registrations |
| Staff time | Hours × fully-loaded rate (or accept a flat estimate) |
| Travel + logistics | Flights, hotel, shipping |
| Total Event Cost | Sum — accept a roll-up if line items are unavailable |

**Output inputs**

| Input | Notes |
|---|---|
| Registrants | Total sign-ups (pre-event) |
| Attendees | Actually attended (live + on-demand within the event window) |
| Leads generated | Net-new names captured (not existing contacts) |
| MQLs / SQLs | If tracked, split |
| Opportunities created | Pipeline sourced or influenced — be explicit which |
| Pipeline value | $ amount attributed (sourced preferred; influenced acceptable with flag) |
| Deals closed | Count |
| Closed revenue | $ amount |

If pipeline is "influenced" (not sourced), flag it throughout. Influenced pipeline inflates ROI;
sourced pipeline is the defensible number.

---

## The model — Grubb/ITSMA B2B Event ROI Framework

The calculation spine is the practitioner-standard Grubb/ITSMA approach: costs flow in, outcomes
flow out, and the model produces ratio-based metrics that map directly to what CMOs and CFOs ask for.
[verify the specific ITSMA publication year and page reference before citing it in a written report.]

### Core ratios

```
CPL  = Total Event Cost / Leads Generated
CPO  = Total Event Cost / Opportunities Created
CPA  = Total Event Cost / Deals Closed

Attendance Rate          = Attendees / Registrants × 100
Lead Conversion Rate     = Leads / Attendees × 100
Lead-to-Opp Rate         = Opportunities / Leads × 100
Opp-to-Close Rate        = Deals Closed / Opportunities × 100

Pipeline Multiple        = Pipeline Value / Total Event Cost
Revenue ROI              = (Closed Revenue − Total Event Cost) / Total Event Cost × 100
Revenue Multiple         = Closed Revenue / Total Event Cost
```

**Interpret Pipeline Multiple separately from Revenue ROI.** Pipeline multiple is forward-looking;
revenue ROI requires a closed-revenue input that may not exist for events with long sales cycles.
When revenue data is unavailable, present Pipeline Multiple as the primary metric and flag that
revenue ROI will be available in N months (estimated from the brand's sales cycle).

### Handling incomplete data

- Missing attendees → estimate from industry attendance rates (virtual: 40–55%; in-person: 85–95%);
  mark `[estimated]`.
- Missing pipeline → request it; if unavailable, produce CPL + CPO only and note the gap.
- Staff time unknown → use a flat-rate assumption (8–24 loaded hours per staff per day × rate); ask
  user to confirm rate. Mark `[estimated]`.
- Revenue not yet closed → produce pipeline-based metrics only; set a calendar reminder prompt for
  the user to re-run when revenue is known.

---

## Benchmark Reference

Use these external benchmarks when the brand has no internal baselines on file. Always cite the source
tier, not a specific dollar figure, since costs vary enormously by industry, company stage, and event
type. [All ranges below are directional — verify against the most recent industry survey before use.]

| Event Type | CPL Range | CPO Range | Pipeline Multiple (good) |
|---|---|---|---|
| Virtual webinar / summit | $25–$150 | $800–$3,000 | 3–8× |
| Field event / roadshow | $150–$600 | $3,000–$10,000 | 4–10× |
| Trade show / conference (sponsor/exhibitor) | $200–$1,200 | $4,000–$20,000 | 2–6× |
| Owned flagship conference | $300–$2,000 | $5,000–$25,000 | 5–15× |

Context rule: always compare against the **same event type**. A $400 CPL for a virtual webinar is
a red flag. A $400 CPL for a flagship conference is a reasonable outcome.

---

## Output format

### Quick mode

```markdown
## Event ROI Summary — [Event Name] ([Date])
**Brand:** [slug, via brand-brain]
**Event type:** [virtual / in-person / hybrid]
**Total cost:** $[X]   _(sourced / estimated — note which)_

### Key metrics
| Metric | Value | Benchmark (external) | Verdict |
|---|---|---|---|
| Cost per Lead (CPL) | $X | $Y–$Z (event type) | ✓ Below benchmark / ⚠ Above benchmark |
| Cost per Opportunity (CPO) | $X | $Y–$Z | … |
| Cost per Acquisition (CPA) | $X | — | … |
| Attendance Rate | X% | 40–55% (virtual) | … |
| Lead-to-Opp Rate | X% | — | … |
| Pipeline Multiple | Xx | 3–8× | … |
| Revenue ROI | X% | — | … |

### Verdict
[2–4 sentences in the brand's voice: did the event pay its way, what's the single strongest
number to defend, what's the one number to watch or improve next time]

### Caveats
[Flag any [verify] or [estimated] inputs; note if revenue data is pending]

### Recommended next action
[One concrete follow-up: re-run when pipeline closes, run `roi-business-case-calculator` for
next year's budget ask, or specific program change to improve the weakest ratio]
```

### Batch mode

Add a side-by-side comparison table (events as columns, metrics as rows) before the individual
summaries. Include a ranked ROI efficiency table (rank by Pipeline Multiple, then CPO).

---

## Artifact persistence

Save the full model to `./events/[event-slug]-roi.md` (project-relative CWD). Include all inputs,
computed metrics, and caveats — so future runs can diff against actuals when revenue closes. Never
overwrite a previously saved file without confirming with the user.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No output before `brand-brain` returns. Voice + banned words are hard overrides.
- **Sourced vs influenced is not optional.** Always flag which. Mixing them without disclosure is a
  material misrepresentation of event performance.
- **No invented data.** Missing inputs are either asked for, estimated with an explicit flag, or
  omitted from the metric rather than filled in silently.
- **Benchmarks add context, not cover.** A below-benchmark result is still a below-benchmark result;
  benchmarks explain the verdict, they don't overrule the math.
- **Revenue ROI requires revenue.** Don't present a revenue ROI calculation if closed revenue data
  isn't in. Use pipeline multiple as the forward-looking proxy and say so.
- **Save the model.** ROI is only useful over time; persist to ./events/ so the next run has a baseline.

## What Not to Do

- Don't invent pipeline or revenue numbers to complete a calculation — mark missing data as such.
- Don't treat influenced pipeline as sourced pipeline without explicit disclosure.
- Don't present benchmarks as the brand's own performance baselines unless `brand-brain` surfaced them.
- Don't build the executive ROI memo here — delegate to `roi-business-case-calculator`.
- Don't build the board slide here — delegate to `cfo-ready-budget-summary-slide-builder`.
- Don't run a full actuals-vs-budget reconciliation here — delegate to `budget-variance-p-l-reconciliation-analyst`.
- Don't skip the verdict. Metrics without interpretation are noise.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest applied (voice, ACV, any internal benchmarks)?
- All cost inputs accounted for (direct + staff time + promotion), or gap flagged?
- Sourced vs influenced pipeline labeled explicitly?
- All five core ratios computed (CPL, CPO, CPA, Pipeline Multiple, Revenue ROI or caveat for missing)?
- Each metric benchmarked against the correct event type (not a generic average)?
- Every estimated or unconfirmed input marked `[estimated]` or `[verify]`?
- Verdict is 2–4 sentences, on-voice, naming the strongest number and the one to improve?
- Artifact saved to ./events/[slug]-roi.md (or offered to the user)?
- Recommended next action is specific and actionable (not just "improve pipeline")?
