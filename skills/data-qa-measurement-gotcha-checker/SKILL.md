---
name: data-qa-measurement-gotcha-checker
description: >
  GA4 report, data export, or analytics screenshot → systematic checklist of the most
  damaging measurement pitfalls marketers routinely miss: self-referral inflation, spam
  hostname pollution, broken/duplicate events, session vs. user metric confusion,
  attribution-model blending, sampling/threshold suppression, conversion-window
  mismatches, and direct/none dark traffic. For each problem found, provides a
  one-paragraph remediation with the exact GA4 filter, setting, or BigQuery fix.
  Flags items that corrupt downstream decisions (CAC, ROAS, LTV, funnel rates) so you
  know which numbers to quarantine before sharing. Use whenever someone says "check my
  GA4 data," "is this traffic real," "why is direct so high," "our numbers look off,"
  "QA the analytics," "data quality audit," "trust these numbers," "measurement review,"
  or pastes a GA4 export or screenshot.
---

# Data QA & Measurement Gotcha Checker

Analytics lies more often than analysts admit. GA4 is powerful and privacy-forward — and ships with a dozen silent failure modes that turn your reports into fiction. This skill runs a structured audit against the most damaging gotchas, names the specific problem, quantifies its blast radius, and hands you the fix before the bad number reaches a board deck or a budget decision.

Input: a GA4 report (pasted table, screenshot, CSV, or property ID + date range), a plain description of what looks odd, or a specific claim you want to pressure-test.

---

## Skills this calls

- **`brand-brain`** — loads ICP and known channel mix to sense-check traffic composition against expectations.
- *(optional, when installed)* `ltv-cac-payback-calculator` — if inflated traffic or broken conversion tracking has corrupted CAC/LTV inputs, flag and offer to rerun clean numbers there. `funnel-drop-off-analyzer` — if a broken event is the cause of a funnel anomaly, hand off the clean event list. `growth-diagnostic-deep-dive` — escalate here after QA if the data passes but the growth story still doesn't add up. `experiment-results-analyzer` — if the QA surfaces a sample-ratio mismatch or event-count discrepancy in a running test.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (channel mix, known quirks, ICP)
Step 1  Triage the input    ──► identify report type, date range, dimensions, metrics
Step 2  Run the gotcha grid ──► 10-point checklist, surface every ★ RED finding first
Step 3  Score the damage    ──► flag which downstream metrics are corrupted
Step 4  Remediate           ──► one concrete fix per finding
Step 5  Verdict             ──► trust / quarantine / rerun call on the original claim
```

---

## Step 0 — Brand context (always first)

Invoke the `brand-brain` skill to load the active brand's known channel mix, ICP, and any documented measurement quirks. Many gotchas are only visible as anomalies against expectation (e.g., organic making up 2% for an SEO-led brand is the signal).

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for the dataset or GA4 export to QA (pasted table, CSV, or property ID), the analytics platform in use (GA4, Mixpanel, Adobe, etc.), the key conversion events or goals being measured, and the date range under review.

---

## The Gotcha Grid — 10-point checklist

Run every check. Severity: **★ RED** = corrupts aggregates / decisions; **◆ AMBER** = distorts a segment; **● GREY** = worth noting, low urgency.

| # | Gotcha | What it does | How to detect | Default severity |
|---|--------|--------------|---------------|-----------------|
| 1 | **Spam / bot hostname pollution** | Inflates sessions, users, engagement from non-human crawlers and referral spam | Filter Sessions by `hostname`; legitimate hostnames should account for ≥95% | ★ RED |
| 2 | **Self-referral loops** | Your own domain appears as a `referral` source, splitting sessions and mis-attributing revenue | Check `Session source / medium`; `referral` from your own domain is the tell | ★ RED |
| 3 | **Broken or missing `page_view` event** | Page-based metrics (bounce rate, engagement, pages/session) are meaningless; funnels won't fire | Pull `event_name = page_view` count; compare to sessions — should track closely | ★ RED |
| 4 | **Duplicate `purchase` / `conversion` events** | CAC, ROAS, and revenue metrics overstate by 2–10×; common with dual tag installs or SPA re-fires | Total conversion events vs. order-management-system revenue; >5% gap warrants investigation | ★ RED |
| 5 | **Attribution model blending without disclosure** | GA4 data-driven (DDA) default changes retroactively; comparing a DDA period to a last-click period is apples-to-oranges | Check the attribution model in Admin → Attribution settings; note the model in every report header | ◆ AMBER |
| 6 | **Dark traffic in `direct / (none)`** | Misconfigured UTMs, HTTPS→HTTP redirects, mobile app deep links, and email clients strip referrers; inflates direct, deflates every paid channel | `direct / (none)` >25–30% of paid-brand traffic is the warning threshold; examine landing pages receiving it | ◆ AMBER |
| 7 | **Conversion window mismatch** | A 30-day GA4 window vs. a 7-day platform window inflates GA4-reported conversions vs. platform | Cross-reference GA4 conversions against ad-platform conversions for the same campaign; state the window on every comparison | ◆ AMBER |
| 8 | **Sampling and data thresholds** | GA4 free tier samples at high traffic volumes; thresholds suppress rows below a minimum count, silently dropping long-tail data | Look for the sampling indicator icon in Explore reports; use date ranges ≤90 days, or export to BigQuery | ◆ AMBER |
| 9 | **Session vs. user vs. event metric confusion** | Mixing session-scoped and user-scoped metrics in the same row produces mathematically meaningless ratios | Verify metric definitions in the report column headers; session metrics and user metrics cannot be meaningfully divided | ● GREY |
| 10 | **Cross-domain / subdomain tracking gap** | Checkout on a subdomain or third-party processor resets session, creating a false `direct` spike at checkout and inflating self-referral | Check for session count spikes on the checkout domain; confirm `linker` config in GTM or cross-domain list in GA4 Admin | ★ RED if checkout-involved |

---

## Damage mapping — which downstream metrics break

When a gotcha fires, mark affected outputs quarantined until fixed.

| Gotcha # | Corrupts |
|---|---|
| 1 (spam hostnames) | Sessions, users, channel share, funnel rates |
| 2 (self-referral) | Channel attribution, revenue-by-source, ROAS |
| 3 (broken page_view) | Bounce rate, engagement rate, pages/session, session quality |
| 4 (duplicate conversions) | CAC, ROAS, revenue, conversion rate, LTV inputs |
| 5 (model blending) | Any period-over-period attribution comparison |
| 6 (dark direct) | Paid-channel CPA, organic share, email attribution |
| 7 (window mismatch) | Platform vs. GA4 reconciliation, blended ROAS |
| 8 (sampling) | Segment-level rates, long-tail keyword data, micro-conversion counts |
| 9 (metric confusion) | Any ratio metric mixing scopes |
| 10 (cross-domain gap) | Checkout funnel, revenue attribution, cart-abandonment rate |

---

## Remediation notes (one per gotcha)

**1 — Spam hostnames.** In GA4 Explore, create a filter `hostname contains yourdomain.com` and save it as a Comparison; use it on every report. For a permanent fix, add a GTM trigger condition that fires only on your canonical hostname(s). If using BigQuery export, add `WHERE hostname IN (...)` to every query.

**2 — Self-referral.** In GA4 Admin → Data Streams → More Tagging Settings → List Unwanted Referrals, add your own domain. For subdomains also configure cross-domain measurement. Reprocess history is not available; note the fix date and adjust trend comparisons.

**3 — Broken page_view.** In GA4 Admin confirm Enhanced Measurement → Page views is toggled on. If using GTM, verify the GA4 Configuration tag fires on all pages including SPAs (History Change trigger). Spot-check the DebugView in real time.

**4 — Duplicate conversions.** Audit active tags in GTM or GA4's Tag Diagnostics. One `purchase` event per transaction — fire it on the order-confirmation page only, with `transaction_id` deduplication enabled (GA4 deduplicates by `transaction_id` within a 24h window).

**5 — Attribution model blending.** Document the model change date. Never compare periods straddling a model change without a note. Standardize on one model per report set; DDA requires sufficient conversion volume (≥[verify] conversions/month) to be stable.

**6 — Dark direct.** Enforce UTM tagging on all paid and email links (use the `utm-parameter-bulk-builder` skill). Fix HTTP→HTTPS redirect chains. For iOS deep links, use Universal Links + UTMs. Audit your top 20 direct landing pages — if they are not your homepage, something is stripping a referrer.

**7 — Conversion window.** Always state the attribution window in report headers. When reconciling GA4 vs. platform, align windows first (pull both on the same 7-day or 30-day lookback). Expect a structural gap of 5–20% — document it, don't hide it.

**8 — Sampling.** Use GA4 Explorations with date ranges under 90 days, or switch to BigQuery export for unsampled data. Report any sampled report with the sample rate disclosed. For property-level reports in the standard interface, sampling is not applied — only Explorations sample.

**9 — Metric scope.** In GA4, session-scoped metrics (sessions, engaged sessions) cannot be meaningfully divided by user-scoped metrics. Build reports in a single scope; if you need cross-scope insight, use BigQuery.

**10 — Cross-domain gap.** In GA4 Admin → Data Streams → Configure Tag Settings → Configure your domains, list every domain involved in the user journey. In GTM, enable the Linker parameter. Test with DebugView by traversing the checkout flow.

---

## Principles

- **Quarantine before sharing.** A ★ RED finding means the metric is unreliable until fixed. Say so explicitly rather than caveating in footnotes.
- **Numbers need provenance.** Every metric in a stakeholder report should have a stated model, window, and filter set. If it doesn't, flag it as undocumented.
- **Gaps are structural, not failures.** GA4 vs. platform discrepancies of 10–20% are normal and expected — the problem is presenting them as equivalent without explanation.
- **Fix the plumbing first.** Optimization advice built on broken tracking produces optimization theater. Pause decision-making on affected metrics until the fix is verified in DebugView.
- **Mark `[verify]`** any threshold or benchmark cited here that may have changed — GA4 behavior evolves with Chrome privacy changes and GA4 product updates.

## What Not to Do

- Don't dismiss a RED finding as an edge case without checking the data.
- Don't average a clean period with a polluted period to "smooth" the impact.
- Don't present a DDA-to-last-click comparison without disclosing the model change.
- Don't fix tracking in production without a staging test first — a broken GA4 fix creates a new data gap.
- Don't call the data "clean" unless all 10 checks pass or are explicitly ruled out for this property type.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; channel expectations loaded to calibrate anomaly thresholds?
- All 10 gotchas evaluated, not just the ones the user asked about?
- Every ★ RED finding surfaced first, with the exact metric it corrupts named?
- Downstream damage table completed so the user knows which numbers to quarantine?
- Each finding paired with a specific, actionable remediation (not "check your settings")?
- Verdict delivered — trust / quarantine / rerun — on the original claim that prompted the audit?
- If CAC/LTV inputs are corrupted, offer to hand off to `ltv-cac-payback-calculator` with clean numbers?
