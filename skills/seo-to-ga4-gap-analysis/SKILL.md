---
name: seo-to-ga4-gap-analysis
description: >
  Cross-references an Ahrefs or Semrush ranking export against GA4 organic landing-page data
  to surface the ranking keywords that are not converting — or not even appearing — in GA4.
  Applies the canonical SEO-measurement gap framework (keyword→landing-page→session→conversion
  chain) to systematically diagnose where ranking signal breaks down: attribution failures,
  sampling artifacts, (not set) traps, GSC↔GA4 discrepancies, and genuine conversion
  voids. Output is a matched, prioritized gap list with per-gap root-cause labels and a
  remediation brief ranked by revenue-recovery potential. Also composes data-qa-measurement-
  gotcha-checker for data-quality gates and analytics-report-reviewer for output critique.
  Use when the user says "why aren't my rankings driving traffic," "GA4 doesn't match my
  keyword rankings," "which keywords are ranking but not converting," "SEO to GA4 gap,"
  "ranking but no clicks/sessions," "organic traffic not matching what Ahrefs shows," or
  hands over a ranking export and asks why the numbers don't add up.
---

# SEO-to-GA4 Gap Analysis

Rankings are a promise. GA4 is the receipt. This skill finds every place they don't match — and tells you whether the gap is a measurement problem, a UX problem, or a real conversion problem worth fixing.

Input: a keyword ranking export from Ahrefs or Semrush, plus GA4 organic landing-page data (sessions, key events, or revenue by landing page). Output: a matched gap list with root-cause labels, severity tiers, and a prioritized remediation brief.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's context, including known ICP, offer mechanics, and proof. Informs which gap categories are highest priority for this brand. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the site domain, primary conversion goal (purchase / lead / trial), and top-of-funnel ICP profile before proceeding.
- **`data-qa-measurement-gotcha-checker`** — run as a data-quality gate on both input files before the match. Catches sampling, hostname contamination, attribution-window mismatches, and self-referral loops that would corrupt the gap list.
- **`analytics-report-reviewer`** — optional post-production pass on the final gap report. Call when the user requests a structured review pass or when claims in the output are flagged `[verify]`.
- **`gsc-monitoring-suite`** — if GSC data is available, compose it to cross-check impressions vs. GA4 organic sessions. GSC is the ground truth for impression-to-click conversion; GA4 is the ground truth for session-to-conversion. Both are required for a complete picture.
- **`content-gap-finder`** — if the gap analysis surfaces ranking voids (keywords with zero GA4 landing-page match), delegate content strategy for those voids here.
- **`on-page-seo-optimizer`** — if the root cause is on-page or meta-tag mismatch (ranking keyword not present in the page's title/H1/meta), route remediation there.

---

## How a run works

```
Step 0  Load brand context      ──► call brand-brain
Step 1  Ingest + QA the inputs  ──► run data-qa-measurement-gotcha-checker on both files
Step 2  Build the match table   ──► keyword → canonical landing page → GA4 sessions → events
Step 3  Label each gap          ──► apply the five-gap taxonomy
Step 4  Score + tier            ──► severity × traffic opportunity × revenue proximity
Step 5  Remediation brief       ──► per-gap action, owner type, and estimated impact
Step 6  Persist artifact        ──► save to ./seo-ga4-gap/[brand-slug]-gap-[YYYY-MM-DD].md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned ICP, conversion goals, and offer mechanics to orient the severity scoring in Step 4 — a gap on a high-intent transactional keyword ranks higher than the same traffic gap on an informational term. Obey returned voice and banned-words if any output prose is drafted.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the site domain, primary conversion goal (purchase / lead / trial), and top-of-funnel ICP profile before proceeding.

### Step 1 — Ingest and QA the inputs

Accept either:
- **Ahrefs Organic Keywords export** (CSV: Keyword, Position, URL, Volume, KD, Traffic)
- **Semrush Position Tracking export** (CSV: Keyword, Position, URL, Search Volume, CPC)
- **GA4 Landing Page report** (CSV/export: Landing Page, Sessions, Key Events, Conversions, Revenue — filtered to `Session source / medium` contains `organic`)

Before matching, invoke **`data-qa-measurement-gotcha-checker`** on both files. The gate checks for:
- **GA4 sampling**: session counts < 500 at landing-page level are unreliable; flag rows.
- **Hostname contamination**: non-production hostnames (staging, preview) inflate or deflate session counts; exclude.
- **Attribution window mismatch**: confirm the GA4 date range spans at least 28 days (14-day lookback is too noisy for organic; 90 days is preferred).
- **(not set) mass**: if `(not set)` landing page rows account for >5% of organic sessions, the match rate will be artificially low — flag and document.
- **Self-referral loops**: internal links with missing `?` or misconfigured cross-domain setup that reassign sessions to `(direct)`.

If QA gates fail, report them before proceeding. Do not silently match contaminated data.

---

## The Five-Gap Taxonomy (the named framework)

Every gap between a ranking keyword and its GA4 signal falls into one of five root causes. Label each matched row accordingly — a single keyword can carry multiple labels.

### Gap 1 — Attribution Failure
The ranking keyword *is* sending clicks, but GA4 cannot attribute them correctly.

Signals: GA4 sessions exist on the landing page but source/medium is `(direct)`, `(none)`, or an incorrect referrer. Common causes: HTTPS→HTTP redirect stripping the referrer, UTM override from the same campaign, cross-domain tagging misconfiguration, or GA4 tag firing before the URL updates on SPAs.

Remediation: GTM tag audit (compose `gtm-tag-builder-server-side-conversion-setup` if available); referrer policy check; cross-domain tagging fix.

### Gap 2 — Sampling / (not set) Trap
The data exists in GA4 but is not surfaced in the landing-page dimension.

Signals: high `(not set)` share of organic sessions; landing-page report totals don't reconcile with the Sessions overview. In GA4 free tier, Explorations sample at >10M events; standard reports sample at lower thresholds for high-traffic properties.

Remediation: switch to BigQuery export for unsampled counts; break date range into smaller windows; use `analytics-report-reviewer` to flag unsupported aggregates.

### Gap 3 — Ranking-Page Mismatch
GA4 is tracking sessions but on a *different* URL than the one Ahrefs/Semrush reports as ranking.

Signals: keyword ranks for `/blog/old-url` but GA4 sessions land on `/blog/new-url` (post-redirect). Canonical mismatches, pagination variants (`?page=2`), and trailing-slash inconsistencies all produce this.

Remediation: reconcile ranking URL vs. GA4 landing page via GSC (`gsc-monitoring-suite`); ensure 301s pass PageRank and that GA4 is firing on the canonical destination, not the redirect source.

### Gap 4 — Click-to-Session Bleed
GSC shows impressions and clicks, but GA4 sessions on that landing page are materially lower.

Signals: compare GSC clicks for the URL to GA4 organic sessions for the same URL and date range. A ratio below 0.7 (i.e., fewer than 70% of GSC clicks appear as GA4 sessions) suggests tag-firing failure, ad blocker prevalence, or server-side redirect consuming the session before GA4 fires.

Remediation: GA4 Debugger on the landing URL; check tag firing on redirect chain; consider server-side tagging via `gtm-tag-builder-server-side-conversion-setup`.

### Gap 5 — Genuine Conversion Void
GA4 is correctly attributing organic sessions to this landing page, but those sessions are not converting.

Signals: sessions > threshold, key-event rate materially below the site average for equivalent intent. This is the only gap where the answer is not measurement — it's UX, offer, or content.

Sub-classify by intent:
- **Informational void**: keyword intent is TOFU; landing page is a product page. Mismatch. Route to `on-page-seo-optimizer`.
- **Commitment ceiling mismatch**: keyword is high-intent (buy / pricing / compare) but page has no transactional CTA. Route to `cta-variant-generator` and `landing-page-heuristic-live-cro-auditor`.
- **Offer-brand fit gap**: brand's ICP (from brand-brain) does not align with the audience segment the keyword attracts. Flag for `icp-persona-builder` review.

---

## Step 4 — Severity Scoring

Score every gap row on two axes, multiply for a priority score.

| Axis | Scale | Notes |
|---|---|---|
| **Traffic Opportunity** | 1–5 | Keyword monthly volume × estimated CTR at current position (use Ahrefs/Semrush traffic column if available; else apply standard CTR curves: pos 1≈28%, pos 2≈15%, pos 3≈11%, pos 4-7≈6-7%, pos 8-10≈4%) |
| **Revenue Proximity** | 1–5 | 5 = transactional/pricing/comparison keyword; 4 = product-feature keyword; 3 = use-case/how-to; 2 = comparison/best-of; 1 = informational/awareness |

Priority Score = Traffic Opportunity × Revenue Proximity (max 25). Tier thresholds:

- **Tier 1 (score ≥ 15):** fix this week — material revenue recovery potential.
- **Tier 2 (score 8–14):** fix this sprint — meaningful organic leverage.
- **Tier 3 (score < 8):** backlog or ignore — low ROI to fix.

---

## Step 5 — Remediation Brief

For each Tier-1 and Tier-2 gap, output one row:

```
Keyword | Position | Gap type | Root cause (1-2 sentences) | Fix | Owner | Est. lift
```

Keep fixes concrete and owner-typed (Dev / SEO / Content / Analytics). Mark any estimated lift `[verify]` unless drawn from brand's own historical data.

---

## GA4 + SEO Measurement Gotchas (encoded)

These are the classic failure modes that corrupt any SEO↔GA4 reconciliation. Check all of them before signing off:

1. **GSC vs. GA4 date alignment**: GSC data is available for ~16 months; GA4 free-tier data retention defaults to 2 months for user-scoped data. Match windows carefully.
2. **GA4 session deduplication**: GA4 counts sessions, not clicks. A user clicking three organic results in one session = 1 session, 3 GSC clicks. Never expect a 1:1 ratio.
3. **Keyword → landing page is many-to-many**: one landing page ranks for hundreds of keywords. Aggregate GA4 at the landing-page level; distribute sessions to keywords only when GSC provides per-URL per-keyword click data (Search Console → Performance → Pages → query drill-down).
4. **`(not provided)` is gone but `(not set)` is not**: GA4 doesn't block keyword data the way Universal Analytics did, but `(not set)` in the landing-page dimension is still common when the session doesn't resolve a page path (SPAs, late-firing tags, AMP pages).
5. **Organic social ≠ organic search**: ensure the GA4 filter is `session_source` contains `google|bing|duckduckgo` OR `session_medium` = `organic`, not just channel group = "Organic Search" (channel grouping can be misconfigured).

---

## Output Format

```markdown
## SEO-to-GA4 Gap Analysis — [brand slug] — [date range]

### Data quality gate (from data-qa-measurement-gotcha-checker)
[pass/fail per check; any blocking issues that limit match reliability]

### Summary
- Keywords in ranking export: [N]
- Matched to a GA4 landing page: [N] ([%])
- Gaps found: [N] ([Tier 1 / Tier 2 / Tier 3 split])
- Top gap type by count: [Gap type]
- Top gap type by traffic opportunity: [Gap type]

### Tier-1 Gaps (fix this week)
[table: Keyword | Position | Traffic Opp | Rev Proximity | Gap Type | Fix | Owner | Est. Lift]

### Tier-2 Gaps (fix this sprint)
[table]

### Tier-3 Gaps (backlog)
[list, no table — keep it scannable]

### Measurement notes
[any (not set) warnings, sampling flags, GSC↔GA4 session ratio anomalies]
```

Save to `./seo-ga4-gap/[brand-slug]-gap-[YYYY-MM-DD].md`.

---

## Principles

- **Measurement gaps first.** Before calling anything a "real" conversion problem, exhaust all five attribution/tracking explanations. Most SEO↔GA4 discrepancies are measurement artifacts, not genuine ranking failures.
- **Data QA is non-negotiable.** Run `data-qa-measurement-gotcha-checker` before matching. Contaminated inputs produce a contaminated gap list and misdirected fix effort.
- **Traffic Opportunity × Revenue Proximity, always.** Never prioritize a gap by volume alone; a high-volume informational keyword with no conversion intent is a Tier 3.
- **Compose, don't re-derive.** GSC data → `gsc-monitoring-suite`. Content voids → `content-gap-finder`. On-page fixes → `on-page-seo-optimizer`. CTA voids → `cta-variant-generator`. This skill diagnoses; siblings remediate.
- **Truth only.** No invented lift estimates. Mark every projection `[verify]` unless drawn from the brand's own historical data.

## What Not to Do

- Don't start the match before `data-qa-measurement-gotcha-checker` clears the inputs.
- Don't conflate a sampling artifact with a genuine conversion void — they have opposite fixes.
- Don't report a 1:1 GSC click→GA4 session expectation as a gap; the ratio is always < 1.
- Don't reimplement brand context; call `brand-brain` for ICP and conversion goal orientation.
- Don't save output inside the skill folder; write to the project CWD at `./seo-ga4-gap/`.
- Don't label every unmatched keyword as "Gap 5 — Conversion Void." Resolve measurement explanations first.

## Quality Checklist

- [ ] `brand-brain` called and brand context (ICP, conversion goal) loaded before scoring?
- [ ] `data-qa-measurement-gotcha-checker` run on both input files; any blocking issues documented?
- [ ] GA4 filter confirmed as organic-only (session_medium = organic or equivalent)?
- [ ] Every gap row labeled with one of the five taxonomy types?
- [ ] Severity scored on both Traffic Opportunity AND Revenue Proximity (not volume alone)?
- [ ] Tier-1 gaps each have a concrete fix, an owner type, and a `[verify]`-marked or evidence-backed lift estimate?
- [ ] Measurement gotchas 1–5 explicitly checked and noted in output?
- [ ] Output saved to `./seo-ga4-gap/[brand-slug]-gap-[YYYY-MM-DD].md`?
- [ ] No lift figures or proof points invented; anything unconfirmed marked `[verify]`?
