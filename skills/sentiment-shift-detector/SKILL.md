---
name: sentiment-shift-detector
description: >
  Ingests two time-period exports of reviews, support tickets, NPS verbatims, survey open-ends,
  or social comments and produces a side-by-side topic-sentiment delta report — surfacing which
  topics got meaningfully better or worse, by how much, what customers are saying in their own
  words, and what the brand should do about each shift. Built on the Topic-Sentiment Delta (TSD)
  framework: cluster → score → delta → triage. Works with any text-based VoC source: G2, Capterra,
  App Store, Trustpilot, Intercom/Zendesk export, Typeform/Delighted NPS batch, subreddit dump.
  Calls brand-brain to load the active brand's ICP, positioning, and known pain vocabulary before
  running the analysis, so the clustering and triage are anchored to what actually matters for
  this product and audience — not generic sentiment bins. Saves a structured delta report to
  ./sentiment/[slug]-delta-[date].md for downstream use by copywriters, PMs, and CSMs.
  Use when the user says "what's changed in our reviews," "compare sentiment before and after
  [launch/change/period]," "are customer complaints shifting," "what topics are trending up or
  down in feedback," "VoC delta," "did our NPS verbatims change," or pastes two batches of
  reviews/tickets and asks what's different.
---

# Sentiment Shift Detector

Two batches of customer text in → a topic-sentiment delta report out. Not "overall sentiment went up 0.3 points" — that's noise. This skill finds which *specific topics* moved, in which direction, by how much, with the actual customer language proving it. Every finding is grounded in the brand's ICP and product positioning so the output is triage-ready, not just analytical.

The job is signal extraction, not sentiment scoring for its own sake. A brand already knows its reviews improved. This skill tells the PM which features drove it, the CSM team which support topics escalated, and the content team which objections just got louder.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand's ICP, positioning, known pain vocabulary, and banned words so topic clustering is anchored to real product context. Not re-implemented here.
- *(compose when input is raw/unstructured)* **`voice-of-customer-mining-pipeline`** — if the user has unprocessed feedback (URLs, raw CSVs) rather than two ready-to-compare exports, delegate extraction there first, then re-enter this skill with the two structured batches.
- *(compose for competitive framing)* **`competitor-review-gap-spotter`** — if the shift analysis reveals a topic where a competitor is gaining ground, hand off to that skill to run the comparative gap audit.
- *(compose for follow-up research)* **`jtbd-customer-interview-suite`** — when a shift is large but unexplained, this skill flags it; `jtbd-customer-interview-suite` builds the targeted discussion guide to investigate it.
- *(compose for survey design)* **`survey-designer-analyzer`** — when a delta report surfaces a hypothesis that needs quantitative validation, delegate survey design there.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain; get ICP + pain vocabulary + positioning
Step 1  Ingest & label batches ──► Period A (baseline) and Period B (comparison)
Step 2  Topic clustering       ──► TSD cluster map anchored to brand context
Step 3  Score per topic        ──► polarity ratio + representative quotes per period
Step 4  Compute delta          ──► direction, magnitude, velocity
Step 5  Triage                 ──► Action flags: FIX / AMPLIFY / MONITOR / INVESTIGATE
Step 6  Write & save report    ──► ./sentiment/[slug]-delta-[date].md
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's ICP profile, known product pain points, positioning line, and any relevant banned words or competitor framing to avoid. Use the returned ICP to seed the topic taxonomy in Step 2 — this prevents the clustering from landing on generic categories ("price," "support") instead of product-relevant ones ("onboarding drop-off," "push delivery latency," "pricing tier confusion").

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for the product category, ICP, and 3–5 known pain topics before proceeding. Always prefer the call.

---

## Step 1 — Ingest & label the batches

Accept any combination:
- **Structured exports** (CSV/TSV with date, rating, text columns)
- **Pasted text** (raw review block, ticket dump, NPS verbatims)
- **URLs** to a public review source the user pastes content from
- **Summarized batches** (user already hand-extracted key quotes per period)

Label and bracket:
- **Period A** = baseline (older, pre-event, or "before")
- **Period B** = comparison (recent, post-event, or "after")

If dates are missing, ask. If volume is lopsided (e.g., 50 reviews vs. 500), note it — large imbalances can bias polarity ratios and should be flagged in the report caveats. If only one period is provided, say so and offer to run a single-period VoC extraction instead (route to `voice-of-customer-mining-pipeline`).

---

## Step 2 — Topic clustering (TSD Cluster Map)

This is the core intellectual work. Do NOT use flat sentiment bins ("positive / negative / neutral"). Cluster by **topic** — a specific product or experience dimension — then score sentiment within each topic.

**Seed the taxonomy from the brand context (Step 0)** then expand from the data. A well-formed topic cluster is:
- Specific: "push notification delivery speed" not "notifications"
- Bounded: one thing a PM could act on
- Present in both periods (or notable for *appearing* in one and not the other)

**Build the TSD Cluster Map:**

| Topic Cluster | A count | B count | A polarity | B polarity | Delta | Signal |
|---|---|---|---|---|---|---|
| [topic] | n | n | +/− % | +/− % | ▲/▼ pts | flag |

**Polarity ratio** = (positive mentions − negative mentions) / total mentions for that topic. Range −1.0 to +1.0. Neutral mentions are counted in the denominator but not the numerator. This is intentionally simple — it degrades gracefully on small samples and is auditable.

**Minimum cluster size:** report only clusters with ≥3 mentions in at least one period, or ≥1 mention in both. Smaller signals are noted in an "Emerging signals" appendix section, not in the main table.

---

## Step 3 — Score per topic (with representative quotes)

For each cluster in the main table, extract:
- **1–2 representative positive quotes** from each period (or the period where positive is present)
- **1–2 representative negative quotes** from each period (or the period where negative is present)

Quote selection criteria: the most *specific* quote wins over the most *emphatic* one. "The push editor takes 4 clicks to set up a drip" is better evidence than "this product is terrible." Truncate to ≤40 words; ellipsis is fine. Never paraphrase — use the customer's own words. If no quote exists for a cell, write `—`.

---

## Step 4 — Compute delta and velocity

**Delta** = Period B polarity − Period A polarity (signed, e.g., +0.4 is an improvement, −0.3 is a deterioration).

**Magnitude thresholds:**
- |Δ| ≥ 0.30 → **Large shift** — lead the report
- |Δ| 0.15–0.29 → **Moderate shift** — main table
- |Δ| < 0.15 → **Stable** — suppressed from the main table unless it's a strategically important topic per the brand context

**Velocity flag:** if a topic *newly appears* in Period B (zero mentions in A, ≥3 in B), label it **NEW TOPIC ↑** or **NEW TOPIC ↓**. If a previously prominent topic disappears in Period B (≥5 mentions in A, 0 in B), label it **RESOLVED?** and note in caveats that it may be recency bias or sample gap rather than a genuine fix.

---

## Step 5 — Triage (Action Flags)

Every topic in the main table gets exactly one action flag. This is the output that makes the report actionable rather than analytical:

| Flag | Condition | Owner |
|---|---|---|
| **FIX** | Large negative shift (Δ ≤ −0.30) or newly emerging negative topic | Product / Support |
| **AMPLIFY** | Large positive shift (Δ ≥ +0.30) or newly emerging positive topic | Marketing / CS |
| **MONITOR** | Moderate shift in either direction (0.15–0.29) | Any |
| **INVESTIGATE** | Large shift but sample < 10, or conflicting signal within the cluster | Research |
| **STABLE** | |Δ| < 0.15 — suppressed from main table | — |

**AMPLIFY** topics are the most under-used output of sentiment analysis. A topic that shifted strongly positive is a proof point, a case study angle, a CTA, and an objection-handler. Flag it as such; don't just note it neutrally.

---

## Step 6 — Output format and save

### Report structure

```
# Sentiment Shift Report — [Brand] — [Period A] vs [Period B]
Generated: [date]  |  Source: [review platform / ticket system / NPS]
Sample: Period A = N reviews  |  Period B = N reviews

## Executive summary (3 bullets max)
- Biggest negative shift: [topic], Δ −X.X — [one sentence why it matters]
- Biggest positive shift: [topic], Δ +X.X — [one sentence amplification opportunity]
- Key new signal: [topic or "none"]

## TSD Cluster Map
[table from Step 2]

## Per-topic deep-dives (large shifts only)
### [Topic] — FIX | Δ −X.X
**Period A (baseline):** polarity X.X | N mentions
Representative quotes: "…" / "…"
**Period B (comparison):** polarity X.X | N mentions
Representative quotes: "…" / "…"
**Triage note:** [one specific, actionable recommendation — not "address this issue"]

### [Topic] — AMPLIFY | Δ +X.X
...same structure...

## Moderate shifts (MONITOR)
[condensed table — topic, delta, 1 quote each period, owner]

## Emerging signals (NEW TOPIC / RESOLVED?)
[brief list with quote and caveat]

## Caveats & data quality
[sample imbalance, date gaps, source bias, anything that could distort the delta]

## Recommended next steps
[max 4 bullets — prioritized by flag severity and strategic fit to brand ICP]
```

**Save** the report to `./sentiment/[slug]-delta-[YYYY-MM-DD].md` (create the `./sentiment/` directory if it doesn't exist). Confirm the path in one line. Do not overwrite a file with the same date — append `-v2`, `-v3` if needed.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No clustering before the brand's ICP and pain vocabulary loads. Generic topics produce useless deltas.
- **Topics, not scores.** Overall sentiment scores are suppressed; every finding is anchored to a specific, actionable topic cluster.
- **Customer language, not paraphrase.** Quotes are verbatim, truncated, never summarized.
- **AMPLIFY is equal to FIX.** Positive shifts are marketing opportunities, not just reassurance. Give them the same depth.
- **Sample honesty.** Flag imbalances, gaps, and low-count clusters. A delta from 2 reviews is noise; say so.
- **Truth discipline.** No invented quotes, no extrapolated percentages. If the data doesn't support a claim, say "insufficient data" or flag as INVESTIGATE.
- **Caveats are first-class.** A misleading insight is worse than no insight. Data quality notes are mandatory, not optional.

---

## What Not to Do

- Don't report an overall sentiment score without topic breakdown — it tells the user nothing actionable.
- Don't paraphrase quotes — use the customer's exact words, even awkward ones.
- Don't suppress negative shifts to make the report look better; surface them; that's the value.
- Don't cluster at a level where a PM can't act ("UX" is too broad; "onboarding step 3 confusion" is right).
- Don't call a delta "significant" if |Δ| < 0.15 unless the brand context makes that topic strategically critical — explain why.
- Don't skip the AMPLIFY triage on positive shifts; this is consistently the most under-actioned finding.
- Don't produce the report without saving it — downstream skills (copywriters, CSMs, PMs) need the file.
- Don't re-implement VoC extraction if the input isn't structured; route to `voice-of-customer-mining-pipeline` first.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and ICP + pain vocabulary loaded before clustering?
- Period A and Period B clearly labeled with dates, source, and N count?
- Every cluster is specific enough for a PM to act on (not "support," "pricing" — a real dimension)?
- Polarity ratios computed per topic, not per review; methodology is auditable?
- Large-shift topics have verbatim quotes from both periods?
- AMPLIFY topics given equal depth to FIX topics?
- NEW TOPIC and RESOLVED? flags applied where appropriate?
- Sample imbalances, date gaps, and source bias documented in caveats?
- Report saved to `./sentiment/[slug]-delta-[date].md` and path confirmed?
- Recommended next steps reference the brand's ICP and strategic priorities, not generic advice?
