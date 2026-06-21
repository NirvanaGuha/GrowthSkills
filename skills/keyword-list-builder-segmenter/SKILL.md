---
name: keyword-list-builder-segmenter
description: >
  Takes seed terms plus intent notes and produces a match-type-segmented keyword list grouped by
  theme with suggested bid tiers and negative seed candidates. Two modes: Quick (one theme,
  fast structure) and Full (multi-theme, complete match-type matrix, negative seed library, bid
  guidance). Calls brand-brain to anchor ICP, offer mechanics, and banned vocabulary before a
  single keyword is selected. Designed for paid-search operators who need a ready-to-upload
  keyword structure — not a raw dump from a keyword tool. Downstream of keyword-research-clustering-suite
  (use that for organic research) and upstream of audience-targeting-spec-writer,
  ad-copy-variant-generator, and ad-to-landing-page-message-match-auditor. Use whenever the
  user says "build me a keyword list," "segment these keywords," "what match types should I use,"
  "organize my PPC keywords," "suggest bids by intent," "give me negatives," "keyword groups,"
  "structure my ad groups," or hands over a seed list and asks for paid-search structure.
---

# Keyword List Builder & Segmenter

Seed terms go in; a match-type-segmented, bid-tiered, theme-grouped keyword list comes out — ready to upload to Google Ads or Microsoft Advertising, not just ready to think about. The skill builds structure, not just vocabulary. It applies the SKAGs-vs-STAGs tradeoff consciously, enforces match-type hygiene, and plants a negative seed library so spend doesn't bleed from day one.

Brand context comes from `brand-brain` first. The ICP, offer mechanics, and banned/off-brand terms shape which intent signals belong in the list and which become negatives.

---

## Skills this calls

- **`brand-brain`** (required) — loads ICP, offer, banned words, and positioning so keyword selection reflects the real conversion intent, not a generic category. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for ICP description, primary offer/product, and any terms to exclude before proceeding.
- **`keyword-research-clustering-suite`** (optional) — if the user wants organic research to inform the paid structure, or if seed coverage is thin, call this first and feed its clusters as seeds here.
- **`ad-copy-variant-generator`** (optional) — once the ad groups are structured, hand the theme + ICP + offer to this skill to generate headline/description variants per group. Call explicitly; don't inline.
- **`ad-to-landing-page-message-match-auditor`** (optional) — after ad copy is written, check that the landing page destination matches the keyword intent cluster. Call explicitly.
- **`audience-targeting-spec-writer`** (optional) — to layer audience signals (RLSA, Customer Match, in-market segments) on top of the keyword structure.
- **`data-qa-measurement-gotcha-checker`** (call before finalizing bid recommendations) — surfaces GA4/conversion-tracking gotchas (attribution window mismatches, self-referral, (not set)) that would corrupt the bid data the recommendations depend on.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain; absorb ICP + offer + banned terms
Step 1  Pick mode           ──► Quick (one theme, fast) | Full (multi-theme matrix)
Step 2  Classify seeds      ──► sort into intent tiers (BOFU / MOFU / TOFU / Brand / Competitor)
Step 3  Build match types   ──► apply the STAGs framework per theme
Step 4  Tier bids           ──► intent-signal heuristic (no live data required)
Step 5  Plant negatives     ──► cross-contamination negatives + ICP exclusions
Step 6  Self-review         ──► quality checklist; flag data-quality risks
```

### Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Absorb the active brand's ICP, offer mechanics, primary destination URLs, and banned words. Use the offer mechanics to anchor conversion-intent terms (e.g., a free-trial offer should have BOFU terms like "free trial" and "sign up" in exact; a demo-request offer should anchor on "demo," "pricing," "comparison"). Use banned words as negative seeds automatically.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for ICP description, primary offer/product, and any terms to exclude before proceeding.

Do not produce a single keyword grouping until brand context is in hand.

---

## Quick mode (default)

Triggered when the user provides a tight seed set (≤15 terms) or asks for fast structure on one theme. Returns a single ad-group block, not a full campaign.

1. Classify each seed by intent tier (see *Intent Tier Map* below).
2. Assign match types: exact for high-intent conversion terms; phrase for qualifier variants; broad match modifier / phrase for discovery (use sparingly; flag the bleed risk).
3. Produce one tightly-scoped ad group with:
   - Exact match core (≤10 terms — SKAGs discipline; if the theme is broad, split it)
   - Phrase match expansion (≤10 terms)
   - Suggested negatives for this group (cross-contamination + ICP exclusions)
   - Bid tier label (Tier 1 / 2 / 3) with one-line rationale
4. Note which siblings to call next (ad copy, message-match audit).

Quick output is inline; no file saved unless asked.

---

## Full mode (on request)

Triggered by "full list," "all themes," "campaign structure," "multi-theme," or an explicit count of themes / a large seed set (>15 terms).

### 1 — Seed classification

Sort every input seed (plus obvious morphological variants) into the Intent Tier Map:

| Tier | Intent signal | Match type default | Bid tier |
|---|---|---|---|
| BOFU — Conversion | "buy," "pricing," "trial," brand + product, "[brand] vs [competitor]" | Exact | Tier 1 (highest) |
| BOFU — Comparison | "best [category]," "top [category] software," "[competitor] alternative" | Exact + Phrase | Tier 1–2 |
| MOFU — Evaluation | "how does [product] work," "reviews," "[feature] for [ICP]" | Phrase | Tier 2 |
| TOFU — Awareness | problem/symptom terms, "what is [category]," broad job-to-be-done phrases | Phrase (capped) | Tier 3 (lowest) |
| Brand | exact brand name, brand + feature, brand + [role] | Exact | Tier 1 (defend) |
| Competitor | competitor brand names, competitor + "alternative/vs" | Exact + Phrase | Tier 1–2 |
| Negative seeds | off-ICP roles, irrelevant verticals, banned words, DIY/free/open-source signals (if your offer is paid) | — | Exclude |

Flag any seed that spans tiers — split it or flag the ambiguity.

### 2 — Theme grouping (STAGs framework)

Apply **Single-Theme Ad Groups (STAGs)**: one tightly-defined conversion intent per ad group. Do not use one-keyword-per-ad-group (SKAGs) as the default — STAGs give Google enough signal while keeping theme coherence and Quality Score defensible. Each STAG:

- 3–12 exact-match terms clustered on a single intent signal
- 5–15 phrase-match expansions on the same intent
- 1–3 broad match terms *only* if the account is new and discovery is the goal — flag the bleed risk explicitly
- A negative list scoped to this group (terms that would trigger this ad but belong to a *different* STAG)

Name each group: `[Brand Slug] | [Intent Tier] | [Theme]` — e.g., `AcmeSaaS | BOFU | Free Trial`.

### 3 — Bid tier heuristic (no live data required)

Without a live account, bid recommendations are directional. Use this signal stack:

- **Tier 1** (bid highest): BOFU exact, Brand, Competitor-alternative terms. Highest commercial intent; direct path to conversion event.
- **Tier 2** (bid mid): BOFU comparison, MOFU evaluation. Intent is present but the decision isn't made; optimize for engagement/lead, not direct sale.
- **Tier 3** (bid lowest / set a hard daily cap): TOFU awareness. Informational; treat as brand-building, not conversion. If budget is limited, exclude Tier 3 entirely and state why.

Add a `[verify]` flag to any specific CPC estimate — live auction prices require the Keyword Planner or a live account pull. Do not invent CPCs.

### 4 — Negative seed library

Three categories, each output as a flat keyword list (ready to paste as campaign-level negatives):

1. **Cross-contamination negatives** — terms that would trigger an ad from the wrong STAG within this campaign (e.g., if "free trial" has its own STAG, add it as a negative to the Pricing STAG so Google doesn't route the wrong ad).
2. **ICP exclusions** — off-audience signals from the brand's ICP (wrong roles, wrong company size, wrong vertical). Source from the brand-brain ICP digest.
3. **Offer exclusions** — if the offer is paid, add: free, open source, crack, torrent, DIY, and equivalents. If it's B2B SaaS, add: jobs, career, salary, resume, intern (block job-seeker traffic).

### 5 — Output format (Full mode)

```
## Keyword Structure — [Brand Slug] | [Campaign Name]
Brand: [slug, via brand-brain]
ICP anchor: [one-line ICP from brand-brain]
Offer anchor: [offer mechanic from brand-brain]
Generated: [date]

### [Group Name] | [Intent Tier] | Bid Tier [N]
**Exact match**
[keyword]
[keyword]
...
**Phrase match**
[keyword]
[keyword]
...
**Broad match** *(only if present; note bleed risk)*
[keyword]
**Group negatives** (prevent cross-contamination)
[keyword]

--- (repeat per STAG) ---

### Negative Library (Campaign-Level)
**Cross-contamination**   [list]
**ICP exclusions**        [list]
**Offer exclusions**      [list]

### Bid Tier Summary
| Group | Tier | Rationale |
...

### Data quality flag
[Call data-qa-measurement-gotcha-checker before relying on CPA/ROAS targets to set final bids.]
```

Save to `./paid-search/[brand-slug]-keyword-structure-[YYYY-MM-DD].md` when the user confirms.

---

## Framework note: STAGs vs SKAGs

SKAGs (one keyword per ad group) were a 2015–2018 tactic. Google's matching behavior and Smart Bidding degrade with hyper-fragmented groups. STAGs (one intent theme per ad group, 3–12 exact terms) give Smart Bidding enough conversion signal per group while preserving message-match control. Use STAGs by default; note if the account history is rich enough to experiment with tighter grouping.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No keyword is selected before the ICP, offer, and banned terms are loaded. Relevance is defined by the brand's conversion path, not the category.
- **Intent before volume.** A 50-search/mo exact-match conversion term beats a 10,000-search/mo awareness term every time for a performance campaign. Volume without intent is just spend.
- **STAGs discipline.** One intent theme per ad group. Resist the urge to over-consolidate (one giant ad group) or over-fragment (one keyword per ad group).
- **Negatives are half the list.** A keyword list without a negative library is incomplete. Spend leaks from day one without campaign-level negatives.
- **Mark every CPC estimate `[verify]`.** Auction prices require live data. Never invent specific bid amounts.
- **Cross-contamination negatives are mandatory in Full mode.** Every STAG must have negatives preventing other STAGs from triggering its ads.
- **Data quality gate before bid optimization.** Call `data-qa-measurement-gotcha-checker` before trusting any conversion data that informs bid recommendations.

---

## What Not to Do

- Don't produce keyword lists before `brand-brain` returns the active brand (or the explicit fallback is invoked).
- Don't conflate organic keyword research with paid keyword structure — the intent tier calibration, match types, and negative logic are paid-search-specific. Point to `keyword-research-clustering-suite` for organic work.
- Don't invent CPCs, search volumes, or Quality Scores — these require live tool access. Use `[verify]` and the tier heuristic instead.
- Don't over-fragment into SKAGs without flagging the Smart Bidding degradation risk.
- Don't skip cross-contamination negatives in Full mode; they prevent ad/landing-page mismatch and score degradation.
- Don't include competitor brand terms without flagging the legal/policy risk (trademark policies vary by platform and jurisdiction).
- Don't reimplement ICP derivation, brand scanning, or offer mapping — those live in `brand-brain`.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called (or explicit fallback invoked); ICP + offer + banned words loaded?
- Every seed classified into an intent tier; ambiguous seeds flagged or split?
- Quick: one STAG with exact + phrase + group negatives + bid tier label?
- Full: each STAG named `[Brand] | [Tier] | [Theme]`; 3–12 exact terms; phrase expansion; bid tier labeled?
- No specific CPC invented; all numeric estimates marked `[verify]`?
- Cross-contamination negatives present for each STAG (Full mode)?
- Negative library has all three categories (cross-contamination, ICP exclusions, offer exclusions)?
- Competitor terms flagged for trademark policy review?
- `data-qa-measurement-gotcha-checker` called or flagged before bid optimization?
- Output saved (or offered to save) to `./paid-search/[brand-slug]-keyword-structure-[YYYY-MM-DD].md`?
