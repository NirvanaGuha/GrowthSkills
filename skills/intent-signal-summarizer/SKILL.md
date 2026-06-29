---
name: intent-signal-summarizer
description: >
  Transforms raw third-party intent data — Bombora surges, G2 buyer-activity reports, 6sense score
  exports, Demandbase intent CSVs, or any structured intent feed — into a plain-language account
  digest that tells a rep or marketer exactly which accounts are surging, what topics they're
  researching, how hot each signal is, and what to do next. Uses our 6-Signal model (a house model
  that keeps 6sense's 6QA core — fit + intent + engagement — and extends it with timing, relationship,
  and authority for cross-platform prioritization) as the organizing lens. Outputs a ranked priority
  tier (Tier 1 / 2 / 3), per-account signal cards, an outreach-priority queue ready to hand off to
  SDR or ABM sequences, and a data-quality gate that flags stale, thin, or contradictory signals
  before they corrupt rep action. Composes `brand-brain` for ICP and messaging context, and
  optionally hands off to `cold-outreach-sequence-architect` or `sdr-daily-prioritization-engagement-digest`.
  Use when the user says "summarize my intent data," "which accounts are surging," "what does
  Bombora/G2/6sense say," "intent digest," "prioritize my ABM list," "who should we target this
  week," or pastes a raw intent export and asks what to do with it.
---

# Intent Signal Summarizer

Raw intent data answers one question: who is in-market right now? But Bombora CSVs, 6sense score
exports, and G2 buyer-activity tabs don't answer the follow-on question — *so what?* This skill
bridges the gap: it reads the signal feed, weights it through our 6-Signal model — 6sense's proven
6QA spine (fit + intent + engagement) extended with timing, relationship, and authority so the read
holds across vendors — and delivers a
plain-language account digest that tells reps and ABM managers exactly who to touch, what angle to
lead with, and how urgently to act. Data-quality gating runs before any prioritization, so junk
signals don't burn rep time.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, offer, and messaging so
  signal scoring is calibrated against *your* buyer, not a generic template. Fallback if brand-brain
  is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md`
  directly; if none exists, ask the user for their ICP firmographic filters (industry, headcount,
  revenue range), primary product category and top 3 value props, and the competitive context before
  proceeding.
- **`data-qa-measurement-gotcha-checker`** (required gate) — runs a data-quality pass on the intent
  feed before prioritization; surfaces stale scores, coverage gaps, duplicate domain rows, and
  suspicious baseline inflation before signals reach a rep.
- **`account-list-builder-icp-scorer`** (optional) — if the intent feed includes accounts not yet
  scored for ICP fit, call this skill to add fit scores before running the 6-Signal tier model.
- **`cold-outreach-sequence-architect`** (downstream, on request) — pass Tier 1 account cards to
  this skill to generate personalized outreach sequences anchored to the detected intent topics.
- **`sdr-daily-prioritization-engagement-digest`** (downstream, on request) — pass the ranked queue
  to this skill for a rep-ready daily action list.
- **`battlecard-objection-handler`** (downstream, on request) — if surge topics include competitor
  names, call this skill to add competitive-response angles to the account card.

---

## How a run works

```
Step 0  Brand context          ──► call brand-brain (ICP + messaging)
Step 1  Data quality gate      ──► call data-qa-measurement-gotcha-checker
Step 2  Signal ingestion       ──► parse intent feed; normalize fields
Step 3  6-Signal scoring       ──► score each account; assign tier
Step 4  Build account cards    ──► plain-language per-account digest
Step 5  Output the digest      ──► ranked queue + exec summary + handoff hooks
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's
ICP (firmographics, job titles, awareness tendency), competitive positioning, and proof points.
Use the ICP to weight intent topics: a surge on a topic that maps to your core ICP pain scores
higher than a surge on a peripheral topic. Use voice and banned-words for any generated copy in
account cards.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for their ICP firmographic filters,
product category + top 3 value props, and competitive context before proceeding.

---

## Step 1 — Data quality gate (required)

Before scoring a single account, call `data-qa-measurement-gotcha-checker` on the raw input. Flag
and surface any of these before proceeding:

| Issue | Gate action |
|---|---|
| Score date > staleness window (default 30 days — **tune per source**) | Flag as stale; exclude from Tier 1 unless user overrides |
| < 3 topics per account (default thinness floor — **tune per source**) | Flag as thin signal; downgrade one tier unless overridden |
| Duplicate domain rows | Deduplicate on domain; sum/average scores per platform guidance |
| Baseline inflation (score < platform's "surge" threshold) | Strip; not a signal |
| Account not in CRM / unknown owner | Flag for routing before outreach |
| Firmographic fields missing (size, industry) | Require for ICP fit scoring; flag if absent |

**The staleness window and thinness floor are defaults, not hard rules — tune them per source.**
Surge and decay windows differ materially across platforms, so apply a per-feed override the same way
you already deduplicate and set surge thresholds per platform:

| Source | Staleness window (default) | Why it differs |
|---|---|---|
| Bombora | ~7–14 days | Surge is reported on a rolling weekly composite; a 30-day-old surge is two cycles stale |
| 6sense | ~21–30 days | Buying-stage scores decay over a longer arc; recent stage *change* matters more than raw date |
| G2 | ~7–14 days | Buyer-activity (category/comparison/profile views) is high-recency; intent cools fast |
| Demandbase / other | Default 30 days | Confirm the feed's own freshness convention before applying |

Same logic for the thinness floor: 3 topics is a sane default for a topic-surge feed, but a 6sense
stage export or a single high-confidence G2 comparison view can be a real signal with fewer topics —
let the user override per source rather than auto-downgrading. When in doubt on any window, mark it
`[verify]` and confirm against the platform's own surge/decay definition.

Present the quality summary *before* the ranked digest. If > 30 % of the feed fails quality gates,
surface this prominently and ask whether to proceed on the clean subset or pause for a data refresh.

---

## The 6-Signal model (our scoring engine — a house working model)

**Provenance.** This is a house model, not a vendor framework. Its first three dimensions are the
established 6sense 6QA core: a *6sense Qualified Account* fires when **Fit + Intent + Engagement**
all cross threshold ([6sense, "6sense Qualified Account"](https://6sense.com/glossary/6sense-qualified-account/)).
We keep that proven spine and extend it with three dimensions of our own — **Timing, Relationship,
and Authority** — to make the lens work *across* platforms (Bombora + G2 + 6sense + Demandbase in one
digest) rather than inside a single vendor's score. The "6" here is the dimension count, not a vendor
name; this model is not 6QA and should never be labeled as such.

Every account is scored on six dimensions. No single strong signal creates a Tier 1 — the account
needs fit *and* intent *and* at least one accelerating dimension.

| Dimension | What it measures | Signal sources | Origin |
|---|---|---|---|
| **Fit** | ICP firmographic match (industry, headcount, tech stack, revenue) | CRM firmographics, enrichment, ICP from brand-brain | 6QA core |
| **Intent** | Topic-surge depth and ICP-relevance of surging topics | Bombora surges, G2 buyer activity, 6sense topic scores | 6QA core |
| **Engagement** | First-party signals (site visits, ad clicks, email opens, product trials) | CRM/MAP activity, GA4, ad platform data | 6QA core |
| **Timing** | Signal recency + acceleration (rising vs. plateauing) | Score date, week-over-week delta if available | House extension |
| **Relationship** | Existing contact depth (known champion, open opp, past customer) | CRM stage, contact count, last touch | House extension |
| **Authority** | Are the surging personas decision-makers for your category? | Job-title enrichment, seniority, buying-committee coverage | House extension |

**Tier assignment:**

| Tier | Criteria | Action |
|---|---|---|
| **Tier 1 — Strike Now** | High Fit + High Intent + at least 2 other dimensions strong | Immediate rep outreach; ABM personalized sequence; weekly review |
| **Tier 2 — Warm** | High Fit + Moderate Intent OR High Intent + Moderate Fit | Nurture sequence; monitor for Tier 1 escalation; 2-week cadence |
| **Tier 3 — Watch** | Moderate Fit + Low-Moderate Intent; or Timing is stale | Broad nurture; re-score in 30 days; no direct rep touch yet |

Mark every dimension score as **H / M / L**. Show the reasoning for tier placement in one line.

---

## Account card format

For every Tier 1 and Tier 2 account, produce a card:

```
### [Account Name] — [Domain] — Tier [1/2]
Industry: [x]  |  Size: [x]  |  CRM Stage: [x]  |  Owner: [x]
Signal date: [date]  |  Platform: [Bombora / G2 / 6sense / other]

Surging topics: [topic 1, topic 2, topic 3]  (ICP-relevance: High / Medium)
6-Signal scores: Fit [H/M/L] | Intent [H/M/L] | Engagement [H/M/L] | Timing [H/M/L] | Relationship [H/M/L] | Authority [H/M/L]
Tier rationale: [one sentence]

Recommended angle: [one sentence — what angle to lead with, grounded in surge topics + ICP pain]
Next action: [specific action, owner, timing — e.g. "SDR sends LinkedIn connection + email by EOD Fri"]
Watch-out: [any data gap, stale field, or competing signal that could change the read]
```

For Tier 3 accounts, produce a condensed row table — no full cards.

---

## Output structure

```
## Intent Signal Digest — [Brand slug] — [Date]
Data quality summary: [x accounts ingested | x passed QC | x flagged — see notes]

### Tier 1 — Strike Now ([n] accounts)
[Account cards]

### Tier 2 — Warm ([n] accounts)
[Account cards]

### Tier 3 — Watch ([n] accounts)
| Account | Domain | Top Topic | Fit | Intent | Next review |
|---|---|---|---|---|---|

### Data quality flags
[List issues surfaced in Step 1, with account names]

### Handoff actions
- Tier 1 ready for: cold-outreach-sequence-architect / sdr-daily-prioritization-engagement-digest
- Competitor surges detected: [list topics] → run battlecard-objection-handler
- Accounts missing ICP fit score: run account-list-builder-icp-scorer
```

Save digest to `./intent-digests/[brand-slug]-[YYYY-MM-DD].md` when the feed has ≥ 5 accounts or
when the user asks to save. Inline output for quick lookups.

---

## Principles (Non-Negotiable)

- **Data quality before prioritization.** Never score and rank before the QC gate runs. A bad signal
  handed to a rep is worse than no signal.
- **6-Signal, not just intent score.** An intent score alone is not a tier. The model keeps the
  6sense 6QA core (Fit + Intent + Engagement) and adds Timing, Relationship, and Authority so the read
  holds across platforms. Fit + Intent plus at least one accelerating dimension separate a Strike-Now
  from a Watch. Never label this house model "6QA" — that is a specific 6sense framework with three
  core dimensions, not six.
- **ICP-relevance weighting.** Topic surges that map directly to the brand's ICP pain (from
  brand-brain) score higher than off-topic surges. A software company surging on "push notification
  platforms" ranks above one surging on "payroll software" for a push-notification vendor — even at
  the same Bombora score.
- **One action per card.** Recommended next action is singular and owner-assigned. Do not produce a
  menu of options — commit to the right move.
- **Real data only.** Mark any enriched field that cannot be confirmed from the input as `[verify]`.
  Never invent firmographics or contact counts.
- **Freshness matters — per source.** Signal recency is a first-class input, but the staleness window
  is set per platform, not by one universal number. A Bombora surge cools in ~1–2 weeks; a 6sense
  buying-stage score holds longer. Apply the source's own decay window, default to 30 days only when
  the feed's convention is unknown, and flag the date either way.
- **Compose, don't duplicate.** Outreach copy belongs in `cold-outreach-sequence-architect`. Rep
  prioritization belongs in `sdr-daily-prioritization-engagement-digest`. This skill stops at
  the ranked digest and recommended angle — it does not draft emails.

---

## What Not to Do

- Don't hand accounts to a rep before the data-quality gate runs.
- Don't assign Tier 1 on intent score alone — fit and at least one accelerating dimension are
  required.
- Don't draft outreach sequences here — call `cold-outreach-sequence-architect` for that.
- Don't invent job titles, headcount, or contact names to fill 6-Signal gaps — flag them as missing.
- Don't carry stale signals past the source's freshness window into Tier 1 without an explicit user
  override — and don't treat 30 days as that window for every platform.
- Don't label the 6-Signal model "6QA." 6QA is 6sense's own framework (Fit + Intent + Engagement);
  this is a house model that extends it.
- Don't produce one massive undifferentiated account list — the entire value of this skill is in
  the tiered ranking and the per-account angle.
- Don't ask the user to re-paste data they already provided; work with what's in the thread.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and ICP loaded before any scoring?
- `data-qa-measurement-gotcha-checker` run; quality summary shown before digest?
- Staleness window and thinness floor applied *per source* (not a flat 30 days), with overrides noted?
- Every Tier 1 and Tier 2 account has all six 6-Signal dimensions scored?
- Tier assignment justified by Fit + Intent + at least one additional dimension?
- Surge topics rated for ICP-relevance against the brand-brain ICP (not generic)?
- Each Tier 1/2 card has a recommended angle grounded in surge topics + ICP pain?
- Each card has one next action with owner and timing?
- Data gaps and stale signals flagged with `[verify]` or explicit warning?
- Downstream handoff hooks listed (cold-outreach, SDR digest, battlecard)?
- Digest saved to `./intent-digests/` when warranted?
