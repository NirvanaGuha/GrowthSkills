---
name: pricing-page-optimizer
description: >
  Audits any pricing page — screenshot, URL, or pasted copy — and returns a
  prioritized improvement plan covering the four levers that move pricing-page
  conversion: tier anchoring (decoy/champion/contrast), value communication
  (feature-to-benefit translation, value metric clarity), objection handling
  (trust signals, FAQ gaps, risk reducers), and CTA placement + copy. Uses
  classic behavioral-economics anchoring (decoy effect, good-better-best
  contrast) for tier structure combined with the
  Fogg Behavior Model (Motivation × Ability × Prompt) to score each finding by
  conversion impact. Outputs a structured audit table, ranked quick-wins, a
  diff-ready copy block for every weak CTA or headline, and a ready-to-run A/B
  test brief for the highest-impact single change. Calls brand-brain for
  voice/ICP/proof, offer-pricing-brain for tier canon, cta-variant-generator
  for CTA rewrites, and objection-library-builder for objection coverage.
  Invoke when the user says "audit my pricing page," "why isn't my pricing page
  converting," "pricing page feedback," "improve pricing page," "pricing page
  CRO," "fix my pricing tiers," "pricing page A/B test," or pastes/shares a
  pricing page and asks what to change.
---

# Pricing Page Optimizer

A pricing page is not a feature list. It is a decision architecture. This skill audits that architecture — anchoring logic, value communication, objection coverage, and CTA mechanics — and returns a ranked action plan with copy ready to drop in.

It does not redesign the page from scratch. It surgically identifies the highest-leverage changes and tells you why, in priority order.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads active brand voice, ICP, offer mechanics, proof, and banned words. All audit findings and rewrites must honor the returned digest. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, current tier names + prices, and 3 voice adjectives before proceeding.
- **`offer-pricing-brain`** (required when installed) — loads canonical tier structure, value metrics, and limits so the audit checks the real tiers, not a stale screenshot. If absent, ask the user to paste or confirm tier names, prices, and included features before scoring anchoring.
- **`objection-library-builder`** (compose) — supplies the brand's known objections and their current coverage status; the audit checks which objections the page still leaves unanswered.
- **`cta-variant-generator`** (compose) — rewrites every weak primary and secondary CTA found in the audit using the brand's awareness stage and offer.
- **`proof-vault`** (compose when installed) — surfaces real proof points and social-proof assets to slot into trust-signal gaps flagged by the audit.
- **`a-b-multivariate-test-designer`** (compose) — after the audit, generates a single prioritized A/B brief for the top-impact finding.
- **`pricing-page-copywriter-reviewer`** (optional reviewer pass) — after rewrites are drafted, route through this skill to catch voice and compliance issues before presenting.
- **`competitor-price-benchmarking-analyst`** (optional) — when competitive anchoring context is needed, call this to compare against top competitors.
- **`ad-to-landing-page-message-match-auditor`** (optional) — when the user mentions a paid channel is driving pricing-page traffic, call this to confirm ad-to-page message match is not breaking conversion.

---

## How a run works

```
Step 0  Load brand + offer context
Step 1  Ingest the page
Step 2  Score all four levers
Step 3  Rank findings, draft rewrites
Step 4  Generate A/B brief for #1 finding
Step 5  Self-review + present
```

### Step 0 — Load brand and offer context (always first)

Invoke **`brand-brain`** (Skill tool, `skill: brand-brain`). Do not produce any audit output until it returns. Extract: ICP awareness stage, offer mechanics (free trial / freemium / card required / guarantee), primary CTA destinations, real proof points, voice + banned words.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, current tier names + prices, and 3 voice adjectives before proceeding.

Then invoke **`offer-pricing-brain`** to confirm canonical tier data. If unavailable, prompt: *"Paste your current tiers (name / price / headline feature) so the audit checks the real tiers."*

### Step 1 — Ingest the page

Accept: screenshot, URL, or pasted copy/HTML. Extract:
- Tier names, prices, value metric (per seat / per month / per feature)
- Champion (recommended) tier designation (explicit badge or implied default)
- All visible CTAs (label, color, placement, secondary links)
- Trust signals present (reviews, logos, guarantee copy, security badges)
- FAQ/objection content visible
- Hero/headline above the fold

If input is a URL, note that the analysis is based on the submitted state; live dynamic pricing or gated content may differ.

### Step 2 — Score the four levers

Run all four modules. Each finding gets an **impact score** (High / Medium / Low) and a **fix type** (copy-only / structural / A/B required).

---

## The Four Levers (audit framework)

### Lever 1 — Tier Anchoring (behavioral-economics anchoring: decoy effect, good-better-best)

Pricing pages convert by structuring choice, not by listing options. Score:

| Check | Pass/Fail | Notes |
|---|---|---|
| Champion tier is visually dominant (most-popular badge, contrast, size) | | |
| Three-tier good-better-best structure present (2 = no contrast; 4+ = paradox of choice) | | |
| Decoy tier present that makes the champion look rational by comparison | | |
| Highest-tier anchors the range (sets a ceiling the champion looks reasonable against) | | |
| Value metric (what the price scales with) is buyer-legible, not internal jargon | | |
| Annual/monthly toggle present; annual savings shown as a dollar amount, not only % | | |
| Free trial or freemium tier positioned to acquire, not to commoditize the paid tiers | | |

**Common failures:** No champion designation. Tiers differ only by limits, not by outcomes. Annual discount shown as "Save 20%" with no dollar anchor. Starter tier undercuts Champion.

### Lever 2 — Value Communication

Feature rows tell; benefit rows sell. Score:

| Check | Pass/Fail | Notes |
|---|---|---|
| Each tier headline names the buyer outcome or role, not a plan name ("Starter" → "For solo operators") | | |
| Feature labels are benefit-translated (not "API access" → "Connect your existing tools") | | |
| Champion tier's top 3 features have a proof tie-in (numbers, case study, quote) | | |
| Value metric is explained in the buyer's unit of value ("per 1,000 subscribers" vs "per send") | | |
| Upgrade unlock is legible: buyer understands exactly what they gain by moving up one tier | | |

**Common failures:** Feature rows are symmetric — same label, just a checkmark. No buyer outcome in any tier headline. Social proof is nowhere near the decision zone.

### Lever 3 — Objection Handling

A pricing page loses conversions to objections it never addresses. Score:

Compose **`objection-library-builder`** to get the brand's known objection map. Then check the page for coverage:

| Objection class | On-page signal | Gap? |
|---|---|---|
| "Is it worth the price?" | ROI stat, customer outcome quote | |
| "Can I trust this company?" | Logo bar, review aggregate (G2/Capterra), security badges | |
| "What if I need to cancel?" | Guarantee, no-card language, cancel-anytime | |
| "Is setup hard?" | Time-to-value stat, onboarding promise, integration count | |
| "Does it work for my use case?" | Use-case tags per tier, case study link | |
| "What happens when I hit a limit?" | Overage policy, upgrade prompt copy | |

Also check: FAQ existence and whether it addresses pricing-specific objections (not just product questions).

Compose **`proof-vault`** to surface proof assets available to fill identified gaps.

### Lever 4 — CTA Placement and Copy (Fogg Behavior Model)

A CTA fires when motivation + ability are high and the prompt arrives at the right moment. Score:

| Check | Pass/Fail | Notes |
|---|---|---|
| Primary CTA appears above the fold on each tier card | | |
| CTA label is action-specific, not generic ("Start free trial" not "Get started") | | |
| Secondary / low-commitment CTA present for unready buyers ("See a demo" / "Talk to sales") | | |
| CTA copy is first-person or value-framed (not "Submit") | | |
| CTA repetition: appears at top of tiers AND below the FAQ/objection section | | |
| No competing primary CTAs of equal visual weight on the same tier card | | |
| CTA color passes 4.5:1 contrast ratio against background [verify] | | |

Compose **`cta-variant-generator`** for every CTA scoring Fail on label quality. Pass the brand digest, the tier it belongs to, the ICP awareness stage, and the placement. Request Quick mode per CTA.

---

## Step 3 — Rank findings and draft rewrites

After scoring all four levers, rank every finding:

```
## Audit findings — [brand / page URL / date]
Brand: [slug, via brand-brain]
ICP awareness stage: [from brand-brain]

### High impact
| # | Lever | Finding | Fix type | Recommended change |
|---|---|---|---|---|

### Medium impact
| # | Lever | Finding | Fix type | Recommended change |

### Low impact / polish
| # | Lever | Finding | Fix type | Recommended change |
```

For every finding with fix type = **copy-only**, provide a diff block:
```
BEFORE: [exact current copy]
AFTER:  [rewritten copy — brand voice, real proof, no invented claims]
```

Mark any proof-dependent claim you cannot confirm from the brand digest as `[verify]`.

### Step 4 — A/B brief for the top finding

After ranking, automatically compose **`a-b-multivariate-test-designer`** with:
- The #1 high-impact finding as the test hypothesis
- Current control copy
- The recommended variant
- Estimated conversion impact rationale

Request the brief returned as a 1-page test spec (hypothesis / control / variant / primary metric / sample size note / instrumentation requirement).

### Step 5 — Self-review before presenting

Run through the Quality Checklist below. Correct before presenting.

---

## Output format

```
## Pricing Page Audit — [brand] — [date]

### Context loaded
Brand (brand-brain): [slug]
Tiers confirmed (offer-pricing-brain / user): [tier list]
ICP: [one line from brand digest]
Page ingested: [URL or description]

### Audit summary
[3-sentence executive summary: biggest strength, top gap, headline recommendation]

### Findings by lever
[4-lever scoring tables as above]

### Ranked action plan
[High / Medium / Low tables with diff blocks for copy changes]

### A/B test brief — #1 priority
[test spec from a-b-multivariate-test-designer]

### What to leave alone
[1–3 elements the page does well — anchors reviewer confidence]
```

Save to `./cro/pricing-page-audit-[slug]-[YYYY-MM-DD].md` if the user asks; inline otherwise.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit output before `brand-brain` returns. Voice and banned-words override all rewrite suggestions.
- **Tier canon before anchoring critique.** Always confirm real tiers via `offer-pricing-brain` or explicit user input — never critique anchoring from a screenshot alone if tiers may be stale or region-gated.
- **Four levers, every run.** Skip none. A perfect CTA on a page with broken anchoring still loses.
- **Rank by impact, not by effort.** The operator needs to know what to fix first, not what is easiest.
- **Real proof only.** Compose `proof-vault` for social proof assets; never invent a stat or attribute a quote.
- **Show before → after.** Every copy finding gets a diff block. Vague feedback wastes operator time.
- **A/B brief is required output.** Every run ends with a testable, instrumentation-ready brief for the #1 finding.

## What Not to Do

- Don't produce audit findings before `brand-brain` and `offer-pricing-brain` (or confirmed tier data) load.
- Don't redesign or restructure the page architecture — flag structural issues, then scope them to a separate redesign brief.
- Don't reimplement objection logic — compose `objection-library-builder`.
- Don't rewrite CTAs without calling `cta-variant-generator` — it handles awareness-stage matching.
- Don't invent proof, review counts, or competitor pricing — use only confirmed data or `[verify]`.
- Don't give equal weight to Low and High findings — the ranked table exists for a reason; lead with High.
- Don't skip the "What to leave alone" section — partial praise anchors credibility and focuses the operator.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any output?
- `offer-pricing-brain` confirmed tiers (or user confirmed manually)?
- All four levers scored — anchoring, value comm, objection handling, CTA?
- Every copy-only finding has a BEFORE → AFTER diff block?
- All proof claims confirmed from brand digest or marked `[verify]`?
- Voice and banned-words from brand-brain honored across all rewrites?
- A/B brief generated for the #1 high-impact finding?
- "What to leave alone" section present?
- Findings ranked by conversion impact, not by lever order or ease?
