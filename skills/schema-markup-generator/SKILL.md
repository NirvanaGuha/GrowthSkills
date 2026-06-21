---
name: schema-markup-generator
description: >
  Takes a page URL (or pasted HTML) and a page type, then produces production-ready,
  validator-passing JSON-LD structured data markup — eliminating the structured-data gap
  that causes rich-result ineligibility. Supports every common page type: Article/BlogPosting,
  Product, FAQPage, HowTo, Organization, WebSite (SiteLinks search box), BreadcrumbList,
  LocalBusiness, Event, VideoObject, SoftwareApplication, and Review/AggregateRating.
  Outputs copy-paste <script type="application/ld+json"> blocks with inline annotation
  comments, a Google Rich Results Test URL, and a concise "why this schema / what it
  unlocks" note. Optionally writes the output to ./schema/<slug>.json for reuse.
  All brand-sourced values (organization name, logo URL, offer price, aggregate rating,
  address) come from the active brand.md to prevent stale or mismatched data.
  Use when the user says "add schema markup," "structured data," "rich results,"
  "JSON-LD," "schema.org," "FAQ schema," "product schema," "breadcrumbs not showing,"
  "rich snippets," "eligible for rich results," or pastes a URL and asks why it isn't
  showing stars/prices/FAQ dropdowns in Google.
---

# Schema Markup Generator

Give it a URL and a page type, get production-ready JSON-LD — the right types, the right nesting, every required field present, every recommended field populated where the brand data supports it. No guessing at property names, no omitting required fields and wondering why Google ignores the block.

This skill produces markup. It does not modify page templates, audit crawlability, or diagnose Core Web Vitals — route those to `technical-seo-audit-fix-prioritizer`. It does not generate FAQ content or HowTo steps from scratch — route that upstream and bring the finished copy back here.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's Organization name, URL, logo, social profiles, offer/pricing, and aggregate proof. Schema values that must match the live page (price, name, URL, logo) come from `brand.md`, not invented here.
- *(optional)* **`technical-seo-audit-fix-prioritizer`** — if the page URL is passed and a crawl/audit is already running, reference its schema findings to avoid duplicate work.
- *(optional)* **`on-page-seo-optimizer`** — FAQPage and HowTo schemas must match the visible on-page content exactly (Google's Structured Data Quality guidelines); if FAQ copy hasn't been written yet, go there first.

---

## How a run works

```
Step 0  Load the brand  ──► call `brand-brain` (returns org data, logo, pricing, proof)
Step 1  Identify page type + collect inputs
Step 2  Select schema type(s) — primary + always-eligible companions
Step 3  Build JSON-LD — required fields first, then recommended
Step 4  Annotate + validate — inline comments + Rich Results Test URL
Step 5  Deliver + optionally persist to ./schema/<slug>.json
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before writing a single property. The returned digest provides: `organization.name`, `organization.url`, `organization.logo` (absolute URL), social profile URLs, offer pricing (for `Product` / `SoftwareApplication`), and aggregate rating data if confirmed in `proof.md`.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for: brand name, canonical domain, logo URL (absolute), and any pricing/rating data needed for the page type. Prefer the skill call.

### Step 1 — Identify page type + collect inputs

Ask (or infer from the URL/HTML) the minimum needed:

| Input | How to resolve |
|---|---|
| Page URL | From user; if absent, ask — it becomes `"url"` / `"@id"` |
| Page type | From user or inferred (product page → Product, blog post → Article, etc.) |
| Page-specific content | See per-type required fields below |

Infer as much as possible from the URL slug, title tag, and visible headings before asking. Never ask for data the brand brain already carries.

---

## Schema type selector (the PageType → Schema map)

| If the page is… | Primary type | Auto-add companions |
|---|---|---|
| Homepage | `WebSite` + `Organization` | `BreadcrumbList` (root) |
| Blog post / article | `Article` or `BlogPosting` | `BreadcrumbList`, `Person` (author) |
| Product page (eComm) | `Product` + `Offer` | `BreadcrumbList`, `AggregateRating` if proof exists |
| Pricing / SaaS product | `SoftwareApplication` | `Organization`, `AggregateRating` if proof exists |
| FAQ or knowledge base | `FAQPage` | `BreadcrumbList` |
| How-to / tutorial | `HowTo` | `BreadcrumbList` |
| Event / webinar | `Event` | `Organization` (organizer) |
| Video page | `VideoObject` | `BreadcrumbList` |
| Local service page | `LocalBusiness` | `BreadcrumbList` |
| Review landing page | `AggregateRating` within parent | Depends on parent type |

If the user specifies a type not listed, map it to the closest schema.org type and note the mapping.

**Always include `BreadcrumbList`** unless the page is a root/homepage with no parent path — it is nearly always eligible and low-effort.

---

## JSON-LD construction rules (the framework)

### 1. Required fields are non-negotiable

Every type has Google-required fields. Missing any = no rich result, no warning, just silence. These are the minimums per Google's Rich Results documentation:

| Type | Required by Google |
|---|---|
| `Article` | `headline`, `image`, `datePublished`, `author.name` |
| `BlogPosting` | same as Article |
| `Product` | `name` |
| `Offer` (within Product) | `price`, `priceCurrency`, `availability` |
| `FAQPage` | `mainEntity[]` each with `name` (question), `acceptedAnswer.text` |
| `HowTo` | `name`, `step[]` each with `text` |
| `SoftwareApplication` | `name`, `operatingSystem`, `applicationCategory` |
| `Event` | `name`, `startDate`, `location.name` |
| `VideoObject` | `name`, `description`, `thumbnailUrl`, `uploadDate` |
| `LocalBusiness` | `name`, `address.streetAddress`, `address.addressLocality`, `address.addressCountry` |

### 2. Recommended fields unlock features

Add where data is available (from `brand.md` or page content):

- `Product`: `description`, `brand`, `sku`, `image` → unlocks image in SERP
- `Article`: `image` (1200×630px min), `publisher` with logo → required for Top Stories
- `SoftwareApplication`: `aggregateRating`, `offers` → unlocks star rating + price
- `FAQPage`: No extras needed — the `@type` + `mainEntity` pair is sufficient
- `Organization`: `logo`, `sameAs` (social URLs) → populates Knowledge Panel

### 3. Always use absolute URLs

Relative URLs (`/images/logo.png`) are invalid in schema. Every `url`, `image`, `logo`, `contentUrl` must be absolute (`https://example.com/images/logo.png`).

### 4. Date format: ISO 8601

`datePublished`, `startDate`, `uploadDate` → `"2025-03-15"` or `"2025-03-15T09:00:00+00:00"`. Never `"March 15, 2025"`.

### 5. Nest types correctly

`Product` contains `Offer`, not a sibling. `FAQPage` contains `mainEntity` array. `HowTo` contains `step` array of `HowToStep`. Getting the nesting wrong is silent failure.

### 6. One `<script type="application/ld+json">` per type

Do not combine unrelated types in one block. Two `<script>` tags on one page is valid and preferred over cramming everything into one `@graph`.

---

## Output format

For each schema block, deliver:

```
### [PageType] Schema — [Page name / URL slug]

**What this unlocks:** [one sentence — e.g., "FAQPage dropdowns in the SERP for this keyword cluster"]
**Placement:** paste inside <head> or before </body> — either is valid

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",          // Required: the schema type
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is PushEngage?",          // Required: the question text verbatim
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "PushEngage is a web push notification platform..."  // Required: visible answer text
      }
    }
    // Add one object per Q&A pair visible on the page
  ]
}
</script>
```

**Validate:** https://search.google.com/test/rich-results?url=[URL]
**Schema.org reference:** https://schema.org/FAQPage
```

Inline comments use `//` notation (technically not valid JSON but instructive for reading — strip them before deploying to production, or provide a clean copy on request).

---

## Annotation and validation step (always)

After outputting the markup:

1. Provide the **Google Rich Results Test URL**: `https://search.google.com/test/rich-results?url=<encoded-url>`
2. Note which **rich result feature** the schema makes eligible (FAQ dropdowns, product price/rating, sitelinks search box, breadcrumb path, etc.)
3. Flag any fields that are `[verify]` — i.e., taken from brand context that should be confirmed against the live page (pricing, rating count, logo URL)
4. Note if the page content doesn't support a recommended field yet ("No aggregate rating found in brand.md — add `aggregateRating` once you have ≥1 confirmed review")

---

## Persistence (optional)

If the user asks to save, write clean JSON-LD (no `//` comments, valid JSON) to `./schema/<brand-slug>-<page-slug>.json`. Never overwrite `brand.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No property value invented here — brand data comes from `brand.md`; page-specific data comes from the page. If a value is unconfirmed, mark it `[verify]`.
- **Required fields before recommended.** A block with required + 2 recommended fields beats a 20-property block missing one required field. Silent validation failure is the enemy.
- **Absolute URLs always.** Relative paths fail silently. Catch them here so Google doesn't have to.
- **Match visible content exactly.** FAQPage and HowTo schemas must mirror what a user sees on the page — Google's guidelines reject schema that adds claims not in the visible content.
- **One type, one block.** Don't merge unrelated types into a single JSON-LD object without using `@graph`. Two clean `<script>` tags are better than one muddled one.
- **No invented proof.** AggregateRating must reflect real, confirmed review data. Never fabricate a rating or review count.

## What Not to Do

- Don't write schema before `brand-brain` returns (org name, URL, logo must be confirmed).
- Don't use relative URLs for any property — catch and correct them.
- Don't mark a page as `FAQPage` unless FAQ content is actually visible on the page — this triggers a manual action from Google.
- Don't put AggregateRating on a page that doesn't show reviews — also a manual-action trigger.
- Don't consolidate every type into one `@graph` unless there's a specific reason; separate blocks are easier to validate and debug.
- Don't omit the Rich Results Test URL — validation is part of the deliverable.
- Don't reimplement brand resolution here — call `brand-brain`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; org name, domain, and logo are from confirmed brand data?
- Every required field for this schema type is populated (check the table above)?
- All URLs are absolute, not relative?
- All dates use ISO 8601 format?
- `FAQPage` / `HowTo` content matches what's visible on the page (no ghost schema)?
- `AggregateRating` present only if real review data exists in `brand.md` / `proof.md`?
- Rich Results Test URL provided for every schema block?
- `[verify]` placed on every field sourced from brand.md that should be confirmed against the live page?
- Clean (no `//` comments) JSON-LD version offered or already persisted to `./schema/`?
