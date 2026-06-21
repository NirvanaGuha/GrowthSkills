---
name: shopify-product-collection-page-seo-builder
description: >
  Turns a raw Shopify product or collection spec plus target keywords into a complete
  on-page SEO package — title tag, meta description, H1, URL slug, above-fold copy
  (hero sentence + benefit bullets), structured-data JSON-LD stub, and internal-linking
  anchors — that ranks AND converts. Applies the TRAF framework (Title · Relevance ·
  Authority signals · Friction removal) to every output. Handles both product detail
  pages (PDPs) and collection/category pages (PLPs) with mode-specific logic. Calls
  on-page-seo-optimizer for technical-signal scoring, keyword-research-clustering-suite
  for keyword fit confirmation, and cta-variant-generator for the primary CTA. Use when
  the user says "write SEO copy for my Shopify page," "optimize my product/collection
  page," "write a title tag and meta for this product," "Shopify on-page SEO," "rank my
  collection page," or hands over a product/collection brief and asks for SEO-ready copy.
---

# Shopify Product & Collection Page SEO Builder

Two page types. One ranking system. This skill turns a product spec or collection brief into
the complete on-page SEO package a Shopify merchant needs to rank and convert — title tag,
meta description, H1, slug, above-fold copy, structured-data stub, and internal-linking
anchors — without ever inventing a claim the brand can't back up.

The framework is **TRAF**: every element is evaluated against Title (keyword + brand fit),
Relevance (intent match for the page type), Authority signals (proof + trust triggers in
the copy), and Friction removal (clear value exchange, low commitment, zero jargon that
repels buyers). Senior operators use TRAF as the checklist; this skill enforces it automatically.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, ICP, proof, and positioning before
  any copy is written. Do not start without it.
- **`keyword-research-clustering-suite`** — confirms keyword fit and surfaces related terms,
  long-tail variants, and search intent signals for both PDP and PLP targets.
- **`on-page-seo-optimizer`** — scores the finished title/meta/H1 set against technical signals
  (char limits, keyword placement, duplication risk, cannibalization check).
- **`cta-variant-generator`** — produces the primary above-fold CTA (button label + microcopy)
  so the copy package converts, not just ranks.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain
Step 1  Confirm intent mode     ──► Product (PDP) or Collection (PLP)?
Step 2  Gather inputs           ──► spec + primary keyword + optional secondary keywords
Step 3  Keyword fit check       ──► call keyword-research-clustering-suite (if available)
Step 4  Write the SEO package   ──► TRAF framework pass
Step 5  Score + QA              ──► call on-page-seo-optimizer, self-review checklist
Step 6  CTA                     ──► call cta-variant-generator for the above-fold CTA block
Step 7  Deliver + persist       ──► present output; save to ./seo/ if the user asks
```

### Step 0 — Load the brand (always first)

Invoke **`brand-brain`** (Skill tool, `skill: brand-brain`). Wait for the active brand's digest:
voice adjectives, banned words, real proof, ICP, positioning, and offer mechanics.

**Fallback:** if `brand-brain` is not installed, read `~/.brandbrain/brands/.active` and that
brand's `brand.md` directly. If none exists, ask: product/store name · ICP (who buys and why) ·
3 voice adjectives + banned words · 1–2 real proof points (ratings, awards, specs — not invented).
Never write copy before this completes.

### Step 1 — Pick the mode

| Mode | Page type | Primary intent | Ranking target |
|---|---|---|---|
| **PDP** | Single product detail page | Transactional ("buy X") | Product keyword + variant modifiers |
| **PLP** | Collection / category page | Navigational + transactional | Category keyword + modifier clusters |

If ambiguous, ask one question: "Is this a single-product page or a category/collection listing?"

### Step 2 — Gather inputs

Minimum viable inputs per mode:

**PDP:** product name · primary target keyword · 2–3 key features/specs · any real proof (reviews,
rating, certifications) · price tier (budget/mid/premium) · main variant (color/size/material if
relevant).

**PLP:** collection name · primary category keyword · number of products in collection (if known) ·
brand USP for this category · top 2–3 sub-category keywords.

If the user pastes a raw spec dump, extract structured inputs before proceeding.

---

## The TRAF framework (applied to every element)

### T — Title tag
- Lead with the **primary keyword**, ideally in the first 3 words.
- Append brand name at the end, separated by `|` or `—` (Shopify default; keep it).
- Character window: **50–60 chars** (Google truncates at ~580px; 60 chars is the safe ceiling).
- Include the single strongest modifier that signals intent and differentiates: "free shipping,"
  a material, a use-case, a proof-stat — only if it fits; never stuff.
- PDP: `[Product Name] — [Key Feature/Modifier] | [Brand]`
- PLP: `[Category Keyword] — [Differentiator Modifier] | [Brand]`

### R — Relevance (meta description + H1)
**Meta description** (150–160 chars):
- Opens with the keyword or a restatement of the title's promise.
- One concrete benefit (the *why buy* answer).
- One friction-reducer (free shipping, returns, a guarantee) using brand's real proof only.
- Closes with a soft CTA phrase ("shop now," "explore the range," "get yours today") — not a
  formal CTA tag; that lives above the fold on-page.
- No duplicate of the title tag; adds new information.

**H1** (1 per page, on-page element — write it even if Shopify auto-generates from product name):
- Primary keyword in the H1, but write it for a human first. Never keyword-stuffed.
- PDP H1: can match the product name if the product name is already keyword-rich; otherwise
  rewrite to lead with the keyword: `[Keyword] — [Differentiating Detail]`.
- PLP H1: category keyword + optional qualifier: `[Category] for [ICP modifier]` or
  `[Category]: [Benefit/Selection Cue]`.

**URL slug** (the one Shopify lets you edit before publishing):
- PDP: `/products/[primary-keyword-hyphenated]` — drop stop words, keep brand only if it's the
  primary keyword, never include variant (color/size) in the canonical slug.
- PLP: `/collections/[category-keyword-hyphenated]` — shortest accurate phrase; no dates, no IDs.

### A — Authority signals (above-fold copy)
The **above-fold copy block** (hero sentence + 3–5 benefit bullets) is the copy a shopper
reads before scrolling. It carries the trust signals that make a ranking click into a conversion.

**Hero sentence** (1 sentence, ≤20 words): the brand's core value proposition for this product
or collection — written in brand voice, grounded in real proof. No unsupported superlatives.

**Benefit bullets** (3–5 items, each ≤12 words):
- Lead with a benefit, not a feature. Feature lives in the detail below the fold.
- At least one bullet is a proof-anchored claim (a rating, a spec, a certifications fact —
  all from the brand's real proof; mark `[verify]` if unconfirmed).
- At least one friction-reducer (returns, warranty, shipping — only if the brand has it).
- No duplicate of the meta description; this is the conversion layer, not the SERP snippet.

**Format** for above-fold copy output:
```
Hero: [one sentence]
• [Benefit bullet 1]
• [Benefit bullet 2]
• [Benefit bullet 3]
• [Benefit bullet 4 — proof anchor]
• [Benefit bullet 5 — friction reducer, if applicable]
```

### F — Friction removal (structured-data stub + internal links)
**JSON-LD structured-data stub** (PDP only):
Provide a minimal `Product` schema stub the developer can paste into the Shopify theme's
`<head>` (or a JSON-LD `<script>` block in the product template). Populate from the inputs;
mark dynamic fields with `{{ shopify_variable }}` notation so a dev knows what to hook up.

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "{{ product.title }}",
  "description": "{{ product.description | strip_html | truncate: 160 }}",
  "brand": { "@type": "Brand", "name": "[Brand name]" },
  "sku": "{{ variant.sku }}",
  "offers": {
    "@type": "Offer",
    "price": "{{ variant.price | divided_by: 100.0 }}",
    "priceCurrency": "USD",
    "availability": "https://schema.org/{% if variant.available %}InStock{% else %}OutOfStock{% endif %}",
    "url": "{{ shop.url }}{{ product.url }}"
  }
}
```
PLP pages: skip Product schema; note where `CollectionPage` + `BreadcrumbList` applies.

**Internal-linking anchors** (3–5 suggestions):
For each page, suggest 3–5 other internal pages to link to (collections, blog posts,
complementary products) with exact anchor-text copy. Anchor text must be descriptive, on-topic,
and varied — never "click here" or exact-match keyword repetition that triggers over-optimization.

---

## Principles (Non-Negotiable)

- **brand-brain first.** No copy before the brand's voice, proof, and banned words are loaded.
- **One primary keyword per page.** Confirm it before writing; two competing primaries = dilution.
- **Intent match is non-negotiable.** Transactional pages get transactional copy; navigational
  collection pages get curated-range framing. Never mix.
- **Proof or [verify].** Every stat, rating, certification, or superlative is either from the
  brand's confirmed proof or flagged `[verify]`. Invent nothing.
- **Char limits are hard stops.** Title at 60, meta at 160. Over-limit = the SERP truncates your
  CTA; self-review catches this before delivery.
- **No cannibalization.** If two pages target the same keyword, flag the conflict and recommend
  which page should own it and which should pivot.
- **Write for the human, rank for the crawler.** The title, H1, and meta must make a scanner
  want to click before they're optimized for a bot to index.

## What Not to Do

- Don't write copy before `brand-brain` returns the active brand's context.
- Don't reimplement keyword research inline — call `keyword-research-clustering-suite`.
- Don't invent proof points, star ratings, award counts, or shipping policies.
- Don't produce PDPs and PLPs in the same output block without labeling each clearly.
- Don't keyword-stuff the H1 or meta; never repeat the exact title-tag string in the H1.
- Don't skip the slug recommendation — Shopify's auto-generated slugs are often
  long, product-ID-suffixed, or variant-polluted; the slug is a ranking lever.
- Don't provide a JSON-LD stub without the `{{ }}` variable annotations — a dev shouldn't
  have to guess what's static versus dynamic.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any output?
- Voice + banned-words honored; every unconfirmed claim marked `[verify]`?
- Title tag: primary keyword in first 3 words, 50–60 chars, brand appended, no stuffing?
- Meta: 150–160 chars, different from title, benefit + friction-reducer + soft CTA?
- H1: keyword present, human-readable, not a duplicate of title tag?
- Slug: shortest accurate hyphenated phrase, no stop words, no variant suffixes?
- Above-fold hero sentence ≤20 words; 3–5 bullets benefit-led, at least one proof anchor?
- JSON-LD stub (PDP): all dynamic fields annotated `{{ }}`, schema valid?
- 3–5 internal-linking anchors with descriptive, varied anchor text?
- `on-page-seo-optimizer` called (or noted as the next manual step if unavailable)?
- `cta-variant-generator` called for the primary above-fold CTA block?
- Keyword cannibalization conflict flagged if detected?
- Output saved to `./seo/[slug]-seo-package.md` if the user requested persistence?
