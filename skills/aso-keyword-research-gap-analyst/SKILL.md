---
name: aso-keyword-research-gap-analyst
description: >
  App Store Optimization keyword research, competitive gap analysis, and prioritization
  for iOS (App Store) and Android (Google Play). Takes an app category, seed terms, and
  up to five competitor app IDs or store URLs → produces a prioritized keyword list scored
  by estimated volume, difficulty, and gap opportunity, with metadata (relevance tier,
  placement mapping, localization flags) ready for immediate use in listing optimization
  or A/B metadata tests. Two modes: Discovery (net-new keyword universe) and Gap (owned
  keywords vs. competitor footprint). Knows that ASO keyword data lives in proprietary
  tools (AppFollow, AppTweak, Sensor Tower, data.ai, MobileAction) — it drives structured
  analysis around whatever data the user can paste or export, and flags any number it
  cannot verify. Does NOT rewrite listing copy — that is handled by the sibling skill
  `app-store-listing-aso-optimizer-ios-android`, which consumes this skill's output.
  Use when the user says "ASO keywords," "App Store keyword research," "find gaps in
  competitor keywords," "keyword list for Google Play," "which keywords should I target,"
  "improve my app ranking," "what are my competitors ranking for," or pastes raw app
  keyword data and asks for analysis.
---

# ASO Keyword Research & Gap Analyst

Give it your app category, seed terms, and a few competitor IDs. Get back a prioritized, scored, placement-mapped keyword list with a clear gap matrix — ready to hand directly to listing optimization or metadata A/B tests.

This skill does research and triage. It does not write store listing copy; that is `app-store-listing-aso-optimizer-ios-android`'s job, which reads this skill's output file. Keep the two passes separate.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active app's brand context: voice, ICP, positioning, core value prop, current category, and any platform-specific constraints. ASO keyword strategy without ICP context produces a generic list; brand context ensures relevance tier scoring is anchored to actual target users.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [app name, primary category (iOS/Android), core ICP (who they are + their job-to-be-done), and 3–5 seed keywords the team already uses].
- **`icp-persona-builder`** *(optional)* — if personas aren't in `brand.md`, call to ground search-intent scoring in actual user language. Synthesize inline when absent.
- **`keyword-research-clustering-suite`** *(optional)* — for large raw exports (200+ terms), delegate clustering to this sibling; fold the cluster map back into the scored output.
- **`competitor-review-gap-spotter`** *(optional)* — call to surface pain vocabulary in competitor reviews that maps to uncontested long-tail keywords. Use when the user provides competitor app names.
- **`app-store-listing-aso-optimizer-ios-android`** — the downstream consumer. This skill saves `./aso/[slug]-keywords.md` which that skill reads. Do not conflate the two.

---

## How a run works

```
Step 0  Brand context  ──► call brand-brain
Step 1  Pick the mode  ──► Discovery (default) | Gap (on request or when competitor IDs given)
Step 2  Build the raw keyword universe
Step 3  Score and tier every keyword
Step 4  Gap matrix (Gap mode) or volume-ranked table (Discovery mode)
Step 5  Output + save
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill. From the returned digest, extract: app/product name, primary category slug, ICP (role, JTBD, awareness stage), positioning line, platform(s) (iOS / Android / both), and any store-policy banned terms. Do not produce a single keyword until brand-brain returns or the fallback path completes.

---

## Mode selection

| Mode | Trigger | Output |
|---|---|---|
| **Discovery** (default) | No competitor IDs given, or user says "find keywords for me" | Full scored keyword universe from seed expansion |
| **Gap** | Competitor app IDs / URLs given, or user says "gap analysis," "what are they ranking for" | Discovery output + gap matrix showing contested vs. uncontested terms |

When unsure, run Discovery and offer Gap if the user can provide competitor IDs.

---

## Step 2 — Build the raw keyword universe

### 2a. Seed expansion (always)

From the brand context + user-provided seeds, generate keyword variants across four expansion vectors:

1. **Feature/function** — core capability terms (e.g., "push notification scheduling," "subscriber analytics")
2. **Job-to-be-done** — outcome language ("increase app retention," "re-engage lapsed users")
3. **Comparison / alternative** — "[category] alternative," "[competitor] vs [your app]" patterns
4. **Long-tail modifiers** — platform, integration, use-case, persona qualifiers

Cap expansion at ~80 terms before scoring unless the user provides a raw export.

### 2b. Ingest user data (when provided)

Accept any of: keyword export CSV (AppFollow, AppTweak, Sensor Tower format), pasted keyword tables, competitor listing screenshots, or raw review excerpts. Parse to extract: term, volume estimate, difficulty score, ranking position, and current/competitor ownership flags. Anything not provided is flagged `[data needed]` — never invent volume numbers.

### 2c. Platform-specific considerations

**iOS App Store:**
- Title field: 30 chars, highest weight [verify current Apple algorithm weighting]
- Subtitle: 30 chars, second highest weight
- Keyword field: 100 chars, no spaces, no repetition of title/subtitle terms
- No keyword stuffing in description (description does not index) [verify]

**Google Play:**
- Short description: 80 chars, indexes for ranking [verify]
- Long description: 4,000 chars, semantic indexing; keyword density ~1–3% [verify]
- Title: 30 chars [verify current character limit]
- No duplicate terms needed across fields — Play indexes all fields independently [verify]

Mark every platform rule with `[verify]` if you cannot confirm the current spec from user-provided data; store policies change.

---

## Step 3 — Score and tier every keyword

Apply the **RVDI scoring matrix** (Relevance × Volume × Difficulty inverse × Intent fit):

| Dimension | 1 (low) | 3 (mid) | 5 (high) | Notes |
|---|---|---|---|---|
| **Relevance** | Off-ICP or tangential | Related feature or adjacent use case | Exact JTBD match for primary ICP | From brand context; hard-override anything brand-brain marks off-brand |
| **Volume** | Very low / niche | Moderate — category mid-tail | High — category head term | From tool data; `[data needed]` if absent |
| **Difficulty** | Very hard, dominated by large apps | Contested | Achievable, thin competition | Invert for scoring: easy = 5 |
| **Intent fit** | Browse / generic | Feature research | High-intent install / conversion | Map to awareness stage from brand digest |

**RVDI composite = (R × 0.35) + (V × 0.25) + (D_inv × 0.25) + (I × 0.15)**

Weight rationale: Relevance is the highest single weight because an off-ICP #1 ranking drives unqualified installs and inflates churn; volume matters but is often unavailable without paid tools; difficulty is partially controllable; intent fit tracks conversion quality.

Tier every keyword:

| Tier | RVDI score | Action |
|---|---|---|
| **T1 — Priority** | ≥3.8 | Place in title/subtitle/short-desc immediately |
| **T2 — Test** | 3.0–3.7 | Rotate into keyword field or long-desc; A/B test |
| **T3 — Monitor** | 2.0–2.9 | Watch; revisit if category shifts |
| **T4 — Drop** | <2.0 | Remove from current listing if present |

---

## Step 4 — Gap matrix (Gap mode only)

For each competitor app ID provided:

1. Map the competitor's indexed terms (from user-pasted data, screenshot, or `competitor-review-gap-spotter` output) against the scored keyword universe.
2. Classify every T1/T2 keyword:
   - **Contested** — competitor ranks and you rank; who ranks higher?
   - **Competitor-only** — competitor ranks, you don't → gap opportunity
   - **Your advantage** — you rank, competitor doesn't → defend
   - **Uncontested** — neither ranks → virgin territory if volume/relevance justify it

Output the gap matrix sorted by opportunity score (RVDI × gap_size, where gap_size = 1 if uncontested, 0.5–0.9 if you're behind):

```
## Gap Matrix — [your app] vs [competitor(s)]
| Keyword | RVDI | You | Competitor | Gap Type | Opportunity |
```

---

## Step 5 — Output format and save

### Inline output

Present in three sections:

**1. T1 Priority Keywords** (≤15 terms) — the terms to place in title/subtitle/high-weight fields now. Include: term, RVDI score, platform(s), recommended placement, one-line rationale anchored to ICP + brand.

**2. T2 Test Keywords** — rotation candidates. Brief table: term, RVDI, suggested field, test hypothesis.

**3. T3/T4 summary** — one-line count + a note on what to remove from current listings if T4 terms are present.

**4. Gap matrix** (Gap mode only).

**5. Data confidence note** — which numbers are from real tool data vs. estimated vs. `[data needed]`. Never conflate sources.

### Save

Save full scored output to `./aso/[slug]-keywords.md` when the user confirms or asks to save. This is the file `app-store-listing-aso-optimizer-ios-android` reads. Include: run date, app slug, platform, RVDI table, gap matrix, competitor IDs used, and data source notes.

---

## The RVDI framework — why it exists

Most ASO keyword lists are ranked by volume alone. Volume-only ranking optimizes for traffic that doesn't convert: a productivity app ranking #1 for "free games" is an example of volume without relevance. RVDI forces the analyst to price in ICP fit (so rankings drive qualified installs), difficulty (so effort is spent where wins are achievable), and intent (so the traffic tier matches the awareness stage the store listing is built for). It is not a proprietary formula with verified external validation — it is a structured decision aid. Apply judgment; override tiers when product context changes the calculus.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No keywords before the brand context loads; relevance scoring depends on ICP.
- **Never invent volume numbers.** ASO volume estimates are tool-proprietary. If the user doesn't provide data, mark `[data needed]` and score what you can from the other three dimensions.
- **Platform rules change.** All character limits, indexing behaviors, and algorithmic weights must be confirmed by the user or marked `[verify]`. Do not present platform mechanics as settled facts.
- **Separate research from listing.** This skill scores keywords. Writing optimized title/subtitle/description copy is `app-store-listing-aso-optimizer-ios-android`'s job.
- **Relevance is the load-bearing dimension.** A T1 keyword that is off-ICP is actually T4. Override the score if the brand digest says so.
- **Gap ≠ opportunity.** A competitor-only term you're missing may be missing because it doesn't convert for your product. Validate gap picks against ICP fit before promoting them to T1.

---

## What Not to Do

- Don't write listing copy, screenshot text, or metadata — delegate to `app-store-listing-aso-optimizer-ios-android`.
- Don't invent volume, difficulty, or ranking data — flag it and ask the user to paste from their tool.
- Don't repeat the same keyword across iOS title + subtitle + keyword field — Apple deduplicates and penalizes repetition.
- Don't claim a platform indexing rule as settled without verification — ASO platform mechanics shift quarterly.
- Don't cluster 200 keywords manually — call `keyword-research-clustering-suite` and fold the map back.
- Don't skip the RVDI relevance score because volume data is missing — score the available dimensions and note the gap.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; ICP + platform + category confirmed before any scoring?
- Every volume/difficulty number either sourced from user-provided tool data or marked `[data needed]`?
- All platform character limits and indexing rules marked `[verify]` where not confirmed by user input?
- T1 keywords (≤15) genuinely represent the highest-RVDI set — not just high-volume head terms?
- Gap matrix (if Gap mode) distinguishes contested / competitor-only / your advantage / uncontested?
- Data confidence note present and honest about what was estimated vs. measured?
- Output saved to `./aso/[slug]-keywords.md` when confirmed, in a format `app-store-listing-aso-optimizer-ios-android` can read?
