---
name: search-term-report-triage
description: >
  Takes a Google Ads search term report (CSV export, paste, or manual list) and produces a clean
  three-column triage: ADD AS KEYWORD / ADD AS NEGATIVE / IGNORE — with a rationale row for every
  decision. Applies the Search Term Triage Framework (intent classification → quality gate →
  structural action) so no wasted spend slips through and no high-intent query sits unmatched.
  Outputs a campaign-ready action table plus a negative keyword list ready to paste into Google Ads
  or upload via bulk sheet, and a summary of estimated wasted-spend exposure on the negatives.
  Calls brand-brain for brand context (offer, ICP, banned angles, voice) so triage decisions reflect
  what actually converts for THIS brand, not a generic keyword rubric. Composes data-qa-measurement-
  gotcha-checker before processing to catch sampling, attribution, and impression-share gotchas.
  Use when the user says "triage my search terms," "clean up search term report," "find negatives,"
  "what should I add to my keywords," "wasted spend on search terms," "STR review," or pastes /
  uploads a Google Ads search term CSV.
---

# Search Term Report Triage

Your Google Ads search term report is a leak-finder and a signal mine — if you read it right. This skill classifies every term in one pass: add it, negative it, or ignore it. Nothing manual, nothing gut-feel. The framework encodes intent classification, quality gating, structural match logic, and the classic GA4/Ads measurement gotchas that silently corrupt the numbers you're triaging against.

Output: a campaign-actionable table + a negative list you can paste straight into Google Ads Editor or upload as a bulk sheet. The brand context comes from `brand-brain` — so the rubric isn't generic; it's calibrated to your ICP, offer, and the angles that actually convert.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's ICP, offer, banned angles, and proof. Triage decisions depend on what the brand sells and who it sells to. Does not implement brand resolution itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for [product category, primary ICP, core offer/landing page URL, and any known irrelevant verticals or competitor names] before proceeding.
- **`data-qa-measurement-gotcha-checker`** (compose before triaging) — runs a pre-flight on the data for sampling flags, attribution window mismatches, impression-share blind spots, and `(not set)` exposure before any term gets a conversion-based verdict. Invoke at Step 1; its output gates the quality-score logic.
- **`keyword-list-builder-segmenter`** (optional) — if the ADD AS KEYWORD bucket is large, hand it off here to segment by match type, theme, and bid tier rather than doing it inline.
- **`analytics-report-reviewer`** (optional) — if the report has a conversion column and the numbers look wrong, invoke before triage to audit the metric quality independently.
- **`ad-to-landing-page-message-match-auditor`** (optional) — flag any ADD AS KEYWORD term whose intent implies a landing page the brand doesn't yet have; prevents adding keywords that will just raise CPL.

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain (ICP, offer, banned angles)
Step 1  Data quality pre-flight ──► data-qa-measurement-gotcha-checker
Step 2  Parse the report        ──► normalize columns, detect format
Step 3  Classify every term     ──► Search Term Triage Framework
Step 4  Produce outputs         ──► triage table + negative list + spend summary
Step 5  Optional downstream     ──► keyword-list-builder-segmenter if ADD bucket is large
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest — ICP job titles/personas, offer mechanics, primary destination URLs, known competitor names, banned angles — as the brand filter that runs over every triage decision. A term that's high-volume but maps to a competitor's product or an audience segment the brand doesn't serve goes NEGATIVE regardless of volume.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for [product category, primary ICP, core offer/landing page URL, and any known irrelevant verticals or competitor names] before proceeding.

### Step 1 — Data quality pre-flight

Invoke `data-qa-measurement-gotcha-checker` on the report before touching triage. Flag and surface to the user:
- **Sampling / impression-share floor.** Terms with <10 impressions in the window have noise-level signals; mark them LOW-SIGNAL in the triage table (don't auto-add or auto-negative from thin data).
- **Attribution window.** If the report date range is shorter than the account's conversion window (often 30–90 days), conversion counts on recent terms are understated — label those cells `[window]`.
- **`(not set)` or zero-conversion columns.** If conversion tracking is broken or the column is absent, flag: triage proceeds on intent + spend signals only, not conversion data.
- **Self-referral / brand queries mixed with non-brand.** If the report isn't segmented, note it: brand terms need a separate negative list for non-brand campaigns.

Do not soft-pedal gotchas. Surface them in a one-row preface above the triage table.

### Step 2 — Parse the report

Accept any of: CSV file path, pasted rows, or a manually typed list of terms. Normalize to: `search_term | impressions | clicks | cost | conversions | conv_value | campaign | ad_group`. Missing columns are fine — note which signals are absent and adjust confidence accordingly.

### Step 3 — Classify every term: the Search Term Triage Framework

The framework has four dimensions applied in sequence. The first dimension that triggers a hard gate wins; don't override gates with soft signals.

#### Dimension 1 — Brand filter (hard gate)
Apply the brand context from Step 0.
- Term maps to a **competitor's brand/product name** → ADD AS NEGATIVE (competitor campaign), unless the account explicitly runs conquesting.
- Term maps to an **irrelevant vertical** the brand doesn't serve (e.g., "push notifications app for food delivery" when brand only serves eCommerce) → ADD AS NEGATIVE.
- Term contains a **banned angle** from brand.md → ADD AS NEGATIVE.

#### Dimension 2 — Intent classification (Mikael Yang / standard SEM ladder)
Classify intent: **Transactional** (buy, pricing, trial, best X for Y) · **Commercial Investigation** (compare, reviews, alternatives, vs) · **Informational** (what is, how to, guide) · **Navigational** (brand + destination) · **Junk** (random, misspelled, unrelated).

| Intent | Default verdict | Override condition |
|---|---|---|
| Transactional | ADD AS KEYWORD | Only if ICP-aligned and landing page exists |
| Commercial Investigation | ADD AS KEYWORD | Review for brand-fit; if clearly a competitor's branded review query → NEGATIVE |
| Informational | IGNORE (typically) | ADD if volume is high AND content/blog campaign in scope |
| Navigational (own brand) | IGNORE if already exact-match branded; ADD if new variant | |
| Navigational (competitor brand) | ADD AS NEGATIVE | |
| Junk / irrelevant | ADD AS NEGATIVE if high cost; IGNORE if zero spend | |

#### Dimension 3 — Quality gate (performance signals, only if data is clean per Step 1)
- `CTR ≥ account average AND conversions > 0` → strong ADD signal.
- `High cost, zero conversions, sufficient impressions` (low-signal flag NOT set) → ADD AS NEGATIVE.
- `High impressions, zero clicks` → check if the query is already covered by an exact-match keyword; if so, IGNORE or tighten match type upstream.
- `[window]` tagged cells → defer conversion verdict; classify on intent alone.

#### Dimension 4 — Structural action
For ADD AS KEYWORD terms, note the recommended match type:
- Exact match if the term is a known high-value query.
- Phrase match if variations are likely valuable.
- Broad match modifier → avoid unless the account has smart bidding with sufficient conversion data (≥50 conv/mo per campaign) [verify current Google policy].

For ADD AS NEGATIVE, specify level: **ad-group negative** (query irrelevant only for this ad group) vs **campaign negative** (irrelevant across the campaign) vs **account-level shared list** (irrelevant across all campaigns).

---

## Step 4 — Outputs

### Output A — Triage table

```
| Search Term | Impressions | Cost | Conv | Verdict | Rationale | Match Type / Negative Level |
|---|---|---|---|---|---|---|
| push notification pricing | 340 | $42 | 4 | ADD AS KEYWORD | Transactional, ICP-aligned, strong ROAS | Exact |
| pushover app | 88 | $18 | 0 | ADD AS NEGATIVE | Competitor app name, zero conv | Campaign negative |
| web push notification tutorial | 210 | $8 | 0 | IGNORE | Informational, no content campaign in scope | — |
```

Sort: ADD AS NEGATIVE (highest cost first) → ADD AS KEYWORD (highest conv value first) → IGNORE.

### Output B — Negative keyword paste list

```
## Negative keywords — campaign: [campaign name]
[term 1]
[term 2]
...

## Negative keywords — account shared list
[term 1]
...
```

Format: one term per line, lowercase, no match-type brackets (user pastes into Google Ads Editor; match type is set in the UI). Flag any term that should be phrase or exact negative (e.g., "[brand name]" to avoid over-blocking).

### Output C — Spend exposure summary

```
Estimated wasted spend on NEGATIVE terms this period: $[sum]
Top 3 drain terms: [term · $X] | [term · $X] | [term · $X]
Data quality note: [output from Step 1 pre-flight, condensed to 1–2 lines]
```

Save the full triage table to `./paid/str-triage-[YYYY-MM-DD].md` if the user asks or if the report has >50 terms. Inline for small reports.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No triage decision before the brand filter is loaded. A term that's high-performing for a different ICP is still a drain.
- **Data quality gates conversion verdicts.** If Step 1 flags sampling or broken tracking, never issue a performance-based ADD or NEGATIVE — classify on intent alone and say so.
- **One verdict per term.** The framework is sequential; the first hard-gate dimension that fires wins. Don't average across signals to produce a "maybe."
- **Negative specificity matters.** Ad-group negatives protect one unit; campaign negatives protect the whole campaign; account-level negatives protect everything. Over-blocking with account-level negatives kills valid queries in adjacent campaigns.
- **Intent-only triage when data is thin.** <10 impressions = LOW-SIGNAL. Never auto-add or auto-negative from noise; label and defer.
- **Wasted spend is the headline.** Always surface the estimated spend on NEGATIVE terms first — that's why the operator ran this.

---

## What Not to Do

- Don't triage before brand-brain returns brand context; you'll miss irrelevant verticals and competitor terms.
- Don't skip the data quality pre-flight; a broken conversion column silently turns a good keyword into a "zero-conv drain" false negative.
- Don't add broad-match keywords from a search term report without confirming smart-bidding conversion volume thresholds are met [verify].
- Don't issue account-level negatives for terms that are only irrelevant in one campaign — that's an over-block.
- Don't flatten the verdict to IGNORE for every informational term; if the account runs content/TOFU campaigns, some are worth adding.
- Don't invent conversion data. If tracking is absent, say so and classify on intent.
- Don't output the negative list as a single paragraph — operators need one-term-per-line for paste.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand context loaded (or fallback executed) before any classification?
- `data-qa-measurement-gotcha-checker` run; gotchas surfaced in the preface row?
- Every term has exactly one verdict — no blank cells, no "uncertain"?
- ADD AS NEGATIVE terms include a negative level (ad-group / campaign / account)?
- ADD AS KEYWORD terms include a recommended match type?
- Spend exposure summary present with total and top-3 drain terms?
- Data-quality caveats on `[window]` and LOW-SIGNAL rows explicit in the table?
- Negative paste list is one term per line, ready to copy into Google Ads Editor?
- Output saved to `./paid/str-triage-[YYYY-MM-DD].md` if report >50 terms or user requested?
