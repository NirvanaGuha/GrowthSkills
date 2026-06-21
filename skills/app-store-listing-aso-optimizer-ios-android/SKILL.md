---
name: app-store-listing-aso-optimizer-ios-android
description: >
  Produces keyword-optimized App Store (iOS) and Google Play (Android) listing copy for any app —
  title, subtitle, keyword field (iOS), short description (Android), long description, and developer
  response notes — grounded in real search-volume data and competitive gap analysis. Applies the
  AppFollow / MobileAction ASO ranking framework: keyword discovery → competitive gap analysis →
  metadata architecture → copy-layer optimization → localization readiness. Calls brand-brain for
  voice and positioning, keyword-research-clustering-suite for keyword volume/difficulty data, and
  competitive-intelligence-dossier for competitor store listings. Outputs a ready-to-submit metadata
  sheet and a long-description HTML draft for Play Store. Use when the user says "optimize my App
  Store listing," "write ASO copy," "improve my Play Store description," "rank higher in app search,"
  "rewrite my app title/subtitle/keyword field," "app store optimization," or hands over an app name,
  category, and competitors and asks for store-ready copy.
---

# App Store Listing & ASO Optimizer (iOS + Android)

Turn an app name, category, and competitor list into store-ready metadata that ranks and converts. Every field — iOS title/subtitle/keyword field, Android title/short description/long description — is built on real keyword research, gap analysis against competitors, and brand-voice copy. Output is a submission-ready metadata sheet, not a list of vague suggestions.

Layer L33 · App Store & Mobile Marketing (ASO). This skill produces the copy artifact; it does not manage keyword trackers, review strategies, or paid Apple Search Ads (those are separate concerns).

---

## Skills this calls

- **`brand-brain`** (required) — resolves voice, ICP, offer, proof, banned words, and positioning. Never produce a single word of listing copy before this returns.
- **`keyword-research-clustering-suite`** (required) — supplies keyword volume, difficulty, and intent classification for ASO keyword pool construction. Pass the app category + seed terms; use returned data directly.
- **`competitive-intelligence-dossier`** (compose, do not re-derive) — pull competitor store-listing metadata (titles, subtitles, keyword choices) for gap analysis. Pass competitor app names/store URLs.
- **`on-page-seo-optimizer`** *(optional)* — when the user also wants web/SEO and ASO to share a keyword strategy; call it as a downstream handoff task, not inline.
- **`cta-variant-generator`** *(optional)* — for the long-description closing CTA and any creatives briefs attached to the metadata sheet.

---

## How a run works

```
Step 0  Load brand             ──► call brand-brain (voice, ICP, banned words, proof)
Step 1  Build keyword pool     ──► call keyword-research-clustering-suite + competitive-intelligence-dossier
Step 2  Rank & gap-score       ──► Relevance × Volume / Difficulty matrix; surface uncontested high-volume terms
Step 3  Architect metadata     ──► assign keywords to iOS fields + Android fields per platform rules
Step 4  Write copy layers      ──► title → subtitle/short desc → keyword field / long desc → developer notes
Step 5  Self-review & score    ──► checklist pass; deliver metadata sheet + long-description HTML
```

**Produce nothing in Steps 3-5 until Steps 0-2 are complete.** Keyword decisions drive every copy decision; skipping research produces metadata that feels on-brand but ranks nowhere.

---

## The ASO Framework: Relevance × Discoverability × Conversion

This skill builds on the three-axis model used by [AppFollow, AppTweak, MobileAction — verify which the user has access to]:

| Axis | What it means | How this skill handles it |
|---|---|---|
| **Discoverability** | Can the algorithm surface the app for relevant queries? | Keyword-pool construction + field-level keyword assignment |
| **Relevance** | Does the listing match what the searcher intends? | Intent clustering; never stuff high-volume off-topic terms |
| **Conversion** | Once found, does the listing earn the tap/install? | Value-prop copy, social proof, benefit-led description structure |

A listing optimized only for discoverability ranks but doesn't convert. A listing written only for conversion never surfaces. This skill builds both layers simultaneously.

---

## Step 1 — Keyword pool construction

Call **`keyword-research-clustering-suite`** with:
- Seed terms: app category + core use-case verbs + brand name variants
- Ask for: volume, difficulty, and intent (transactional/navigational/informational)
- Flag any brand terms the competitor dossier shows in their titles (high-value anchors)

Simultaneously call **`competitive-intelligence-dossier`** for the top 3-5 direct competitors:
- Extract their iOS titles, subtitles, and any visible keyword field patterns
- Extract their Android titles and short descriptions
- Build a gap list: high-volume terms competitors rank for but the client's current listing does not contain

Minimum viable pool: 30–50 candidate keywords before filtering.

---

## Step 2 — Ranking & gap scoring

Score each candidate keyword on a 1–5 scale across three dimensions:

```
Relevance (R): Is this term genuinely what our ICP searches?        1 = weak, 5 = exact
Volume    (V): Search volume relative to category benchmark         1 = very low, 5 = high
Difficulty (D): Competitor dominance in this term                   1 = wide open, 5 = locked
```

**Priority score = (R × V) / D**

Sort descending. Tier the output:

| Tier | Criteria | Assignment |
|---|---|---|
| **Tier 1** | Score ≥ 10; exact brand relevance | Title and subtitle (iOS) / title (Android) — never waste title space on Tier 2 |
| **Tier 2** | Score 5–9; strong but competitive | Keyword field (iOS) / short description (Android) |
| **Tier 3** | Score 2–4; long-tail or niche | Long description, feature bullets |

Flag any competitor gap-terms that land in Tier 1 — these are the highest-leverage wins (high volume, competitor is ranking, you are not present).

---

## Step 3 — Metadata architecture (platform rules)

### iOS App Store — hard limits and indexing rules

| Field | Char limit | Indexed? | Assignment rule |
|---|---|---|---|
| **Title** | 30 chars | Yes | Brand name + 1 Tier-1 keyword. Format: `[Brand] — [Value Verb] [Category Noun]` |
| **Subtitle** | 30 chars | Yes | 1–2 Tier-1/Tier-2 keywords; lead with the benefit, not the brand |
| **Keyword field** | 100 chars | Yes | Comma-separated, no spaces after commas; single-word tokens preferred; never repeat title/subtitle terms (they are already indexed) |
| **Promotional text** | 170 chars | No (not indexed) | First impression copy; update for seasonal promos without triggering a full review |
| **Description** | 4,000 chars | No | Conversion copy; front-load benefits in first 255 chars (visible without "more" tap) |

**iOS indexing law:** A keyword in the title is indexed regardless of the keyword field. Never repeat a title word in the keyword field — that wastes tokens. The keyword field is for additive terms only.

### Google Play Store — hard limits and indexing rules

| Field | Char limit | Indexed? | Assignment rule |
|---|---|---|---|
| **Title** | 30 chars | Yes (high weight) | Same logic as iOS: brand + Tier-1 keyword |
| **Short description** | 80 chars | Yes | Benefit + 1–2 Tier-2 keywords; shown before the fold; written for humans, not bots |
| **Long description** | 4,000 chars | Yes (moderate) | Keyword-seeded but readable; repeat primary keywords 3–5× naturally; no lists of keywords |
| **Developer name** | — | Yes (low weight) | Not copy-writeable here, but note if it should match brand slug |

**Play Store indexing law:** Every word in the long description is crawled. Density matters but readability is gated — Google's algorithm demotes keyword-stuffed listings. Target each primary term 3–5× in 4,000 chars (~0.1–0.15% density).

---

## Step 4 — Copy layers

### Title (both platforms)

Formula: `[Brand Name] · [Power Verb] [Category Noun]`
- Power verb: what the user *does* (Track, Automate, Schedule, Grow, Ship, Earn)
- Category noun: the search term, not a clever metaphor
- 30-char constraint means 1 keyword only; pick the highest-scoring Tier-1 term
- Never sacrifice brand legibility for keyword density — the brand must be readable

### Subtitle (iOS) / Short description (Android)

- iOS subtitle: benefit-led, ≤30 chars, second Tier-1 or top Tier-2 keyword embedded naturally
- Android short description: ≤80 chars, 1-sentence value prop, include category keyword and one differentiator
- Write for a human skimming search results, not for an algorithm

### Keyword field (iOS only)

- Build from Tier-2 + Tier-3 pool, minus any term already in title or subtitle
- Singular beats plural when both map to the same intent (saves tokens)
- Separate with commas, no spaces: `push,notify,alert,schedule,engage`
- Never include competitor brand names (policy violation risk)
- Fill to 100 chars; unused chars are wasted indexing capacity

### Long description (both platforms)

Structure:
```
[Hook — 1–2 sentences: the core job-to-be-done for the ICP]

[Feature block 1 — 3–5 bullets: primary use cases]
[Feature block 2 — 3–5 bullets: secondary capabilities or integrations]

[Social proof paragraph — real numbers + customer quote if available; else [verify]]

[Closing CTA — soft, benefit-forward, with the category keyword]
```

iOS description: not indexed — write purely for conversion. The first 255 chars are critical (above the fold on mobile). Lead with the transformation, not the feature list.

Android long description: indexed — weave primary keywords into the prose naturally. Repeat the top 3 keywords 3–5× across 4,000 chars. Use the brand voice; do not write like an SEO meta-description farm.

---

## Deliverable format

Save to `./aso/[brand-slug]-aso-metadata.md` when the user confirms.

```markdown
# ASO Metadata Sheet — [Brand] — [Date]

## iOS App Store
**Title (30):** [copy] ([char count])
**Subtitle (30):** [copy] ([char count])
**Keyword field (100):** [copy] ([char count])
**Promotional text (170):** [copy]
**Description (first 255):** [copy]
**Full description:** [copy]

## Google Play Store
**Title (30):** [copy] ([char count])
**Short description (80):** [copy] ([char count])
**Long description (4000):** [copy — plain text; HTML version in separate file]

## Keyword tier map
| Keyword | Tier | Score | Field assigned |
|---|---|---|---|

## Competitor gap wins
[Terms competitors rank for that this listing now captures, with rationale]

## What to A/B test next
[2–3 hypotheses for iterative improvement]
```

---

## Principles

- **Brand-brain first.** No metadata before the brand digest returns. Voice and banned words are hard overrides.
- **Platform rules are non-negotiable.** Every field obeys its char limit and indexing rule; never claim a keyword is indexed when it isn't.
- **Discoverability and conversion are both required.** A keyword-stuffed title that no human would tap is half a failure.
- **No wasted tokens.** Every character in the keyword field and every repeat of a term in the Play Store long description must earn its place.
- **Real proof only.** Social proof lines use verified numbers or are marked `[verify]`.
- **Never repeat title terms in the keyword field.** It wastes indexing capacity and is the single most common ASO mistake.
- **Competitor brand names are off-limits** in keyword fields — policy violation risk on both platforms.

## What Not to Do

- Don't produce any copy before `brand-brain` returns the active brand and `keyword-research-clustering-suite` returns the keyword pool.
- Don't repeat title/subtitle keywords in the iOS keyword field.
- Don't use the Play Store long description as a keyword list — it gets demoted.
- Don't invent download numbers, review counts, or ratings — `[verify]` or omit.
- Don't apply iOS metadata rules to Android or vice versa — the fields, limits, and indexing logic differ materially.
- Don't skip the gap-analysis step — without it, the keyword field is guesswork.
- Don't use competitor brand names in any indexed field.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest loaded before any copy written?
- `keyword-research-clustering-suite` returned volume/difficulty data; competitor gap analysis complete?
- iOS title ≤30 chars, subtitle ≤30 chars, keyword field ≤100 chars with no title/subtitle repeats?
- Android title ≤30 chars, short description ≤80 chars, long description ≤4,000 chars?
- Keyword field filled to ≥95 chars (no wasted indexing capacity)?
- Long description first 255 chars (iOS) lead with transformation, not features?
- Play Store long description has primary keywords 3–5× at natural density, not list-stuffed?
- All proof claims verified or marked `[verify]`?
- Competitor gap wins documented in the metadata sheet?
- At least 2–3 A/B test hypotheses noted for iterative improvement?
