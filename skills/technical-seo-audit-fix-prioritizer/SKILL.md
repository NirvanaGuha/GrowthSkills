---
name: technical-seo-audit-fix-prioritizer
description: >
  Takes a site URL or crawl export and returns a fully prioritized technical SEO fix backlog —
  crawlability blockers, Core Web Vitals failures, redirect chains, schema gaps, indexation
  leaks, and canonical/hreflang issues — each scored by effort × impact so any junior can
  act like a senior SEO engineer. Two modes: Audit (diagnose and score everything) and
  Fix Brief (produce the exact implementation spec — JSON-LD, robots rules, .htaccess
  redirects, canonical tags, meta robots — ready to hand to a dev with zero guesswork).
  Does NOT rewrite page copy or research new keywords; sibling skills handle those.
  Calls `brand-brain` for brand context (domain, staging vs prod distinction, tech stack),
  and calls `schema-markup-generator` for JSON-LD output rather than re-implementing it.
  Use when the user says "technical SEO audit," "fix my crawl issues," "why aren't my pages
  indexed," "Core Web Vitals are failing," "prioritize my SEO issues," "technical SEO
  checklist," or hands over a Screaming Frog / Ahrefs / Sitebulb export and asks what to
  fix first.
---

# Technical SEO Audit & Fix Prioritizer

Technical SEO issues get triaged by feelings. This skill ends that. It applies the
**Effort × Impact Priority Matrix** — a named, repeatable framework — to every crawl
finding so the fix list is always in the right order, the highest-ROI issues surface
first, and a developer brief is ready to copy-paste without a back-and-forth.

This is an auditor and a fix-brief generator. It does not write body copy, research
keywords, or rebuild the site. When it finds issues solvable by sibling skills (schema
markup, page-speed fixes, on-page SEO), it delegates.

---

## Skills this calls

- **`brand-brain`** (required) — loads domain(s), tech stack, staging/prod distinction,
  crawl budget context, and any known indexation history. Do not guess the canonical
  domain or CMS; get it here.
- **`schema-markup-generator`** (when schema gaps are found) — generates valid JSON-LD
  for missing or broken structured data rather than re-implementing it here.
- **`page-speed-core-web-vitals-fixer`** (when CWV failures dominate) — hands off the
  LCP/CLS/INP fix list with the full PageSpeed Insights context.
- **`on-page-seo-optimizer`** (when thin/missing title/meta is found at scale) — produces
  the corrected meta in bulk; this skill flags and counts, the sibling writes.
- **`gsc-monitoring-suite`** (optional, when GSC export is present) — cross-references
  crawl findings against actual impression/click data so high-traffic pages with issues
  get elevated priority.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (domain, stack, GSC property)
Step 1  Classify the input   ──► URL only | crawl export | GSC export | mixed
Step 2  Audit (surface all issues, group by category)
Step 3  Score each issue     ──► Effort × Impact Matrix → Priority tier (P0–P3)
Step 4  Build the fix backlog (sorted; with implementation specs for P0/P1)
Step 5  Delegate deep work   ──► schema → schema-markup-generator; CWV → page-speed fixer
Step 6  Save artefact        ──► ./seo/[slug]-tech-audit-[YYYY-MM-DD].md
```

---

## Step 0 — Load brand context (always first)

**Invoke `brand-brain`** before touching any URL or crawl file. It returns:

- **Canonical domain** (www vs bare, HTTPS enforced?) and any known subdomains.
- **Tech stack** — WordPress, Shopify, Webflow, custom? Determines which fixes are
  plugin-side vs code-side vs infra-side.
- **Staging / prod split** — critical for robots.txt and noindex audits.
- **GSC property** slug if registered in the brain.
- **Known issues** flagged in any prior `brand.md` notes.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and
that brand's `brand.md`; if absent, ask the user: canonical domain, CMS/stack, and
whether a GSC export or crawl CSV is available. Always prefer the call.

---

## Step 1 — Classify the input

| Input type | What this skill does |
|---|---|
| URL only | Live crawl simulation: fetch robots.txt, sitemap(s), canonical, hreflang; PageSpeed API if key available; GSC manual check note |
| Screaming Frog / Sitebulb CSV | Full column-mapping + issue extraction across all categories |
| Ahrefs / Semrush site audit export | Re-score by Effort × Impact (their severity ≠ yours) |
| GSC Coverage / Performance export | Index coverage + query-gap triage |
| Mixed (crawl + GSC) | Cross-reference: surface pages that crawl clean but have zero impressions (indexation black holes) |

State clearly which mode is active and what's missing. If only a URL is supplied, note
what a full crawl export would add and offer to produce a crawl brief.

---

## Step 2 — Audit categories

Work through all eight categories systematically. Skip none; mark N/A when genuinely
not applicable (explain why).

### 1. Crawlability & Robots

- `robots.txt` — present, reachable, not blocking `Googlebot` or `Googlebot-Image` on
  canonical pages; no accidental `Disallow: /` on production.
- `XML sitemap(s)` — present, submitted to GSC, no 4xx/redirected URLs, no noindexed
  URLs, `<lastmod>` accurate.
- Crawl budget — duplicate parameter URLs, infinite-scroll pagination, faceted navigation
  generating URL explosions.
- `meta robots` and `X-Robots-Tag` — noindex/nofollow usage is intentional; staging env
  fully blocked.

### 2. Indexation & Canonicalization

- Canonical tag present, self-referential on canonical pages, pointing to the right URL
  on pagination / filter / parameter variants.
- Duplicate content clusters — www vs non-www, HTTP vs HTTPS, trailing-slash vs no,
  parameter variants all resolving to one canonical.
- Noindex pages receiving internal links (PageRank waste).
- Thin / near-duplicate pages that should be canonicalized or consolidated.

### 3. Redirects & HTTP Status

- Redirect chains > 1 hop — collapse to direct.
- Redirect loops.
- Broken internal links (4xx) — every one costs crawl budget and bleeds link equity.
- 5xx pages in the crawl.
- HTTPS enforced everywhere; no mixed-content warnings.

### 4. Core Web Vitals & Page Speed

- LCP > 2.5 s, FID/INP > 200 ms, CLS > 0.1 — flag per page or per template.
- Root causes: unoptimized hero images, render-blocking JS/CSS, missing lazy-load,
  no CDN, font-swap not set.
- If CWV failures are the dominant finding, call **`page-speed-core-web-vitals-fixer`**
  with the PageSpeed JSON; do not reproduce its logic here.

### 5. Structured Data / Schema

- Present on appropriate page types (Product, Article, FAQ, BreadcrumbList,
  Organization, LocalBusiness, etc.).
- Valid JSON-LD (no syntax errors, required fields present, no deprecated properties).
- Rich-result eligibility — flag opportunities.
- For every schema gap or error found, call **`schema-markup-generator`** with the
  page URL + page type; embed its output in the fix brief.

### 6. Internal Linking & Architecture

- Orphan pages — crawlable but no internal links.
- Link depth > 3 clicks from homepage for important pages.
- Over-optimized / keyword-stuffed anchor text on internal links (Penguin risk).
- Navigation items pointing to noindexed or redirected URLs.

### 7. Hreflang & Internationalisation (if applicable)

- `hreflang` tags present, bidirectional (each locale points to all others), using
  valid ISO language-region codes, not pointing to noindexed/redirect URLs.
- Skip category if single-language site (mark N/A explicitly).

### 8. Mobile & Technical Compliance

- Mobile-first indexing: mobile page renders the same content as desktop; no blocked
  resources in mobile user-agent.
- Core page templates: `<title>` present + unique, `<meta description>` present (no
  hard duplicates at scale), exactly one `<h1>` per page.
- Security headers: HTTPS, HSTS present [verify if relevant to the user's stack].

---

## Step 3 — Effort × Impact Priority Matrix (the framework)

Every finding gets two scores (1–3), producing a Priority tier.

```
Impact score
  3 = High   — ranking / indexation / traffic directly affected at scale
  2 = Medium — affects a template or section of the site
  1 = Low    — isolated, cosmetic, or very unlikely to affect rankings

Effort score (to fix)
  1 = Quick  — <1 hour, one person, no deploy needed (robots.txt edit, meta tag fix)
  2 = Medium — hours to a day; dev + QA cycle needed (redirect map, canonical tag changes)
  3 = Heavy  — multi-day; infra, plugin, or framework change (server-side rendering,
               CDN setup, site-wide JS refactor)

Priority tier = Impact − (Effort − 1)   →   clamp to range [1, 3], then bucket:

  P0  Impact 3, Effort 1–2    Quick wins with big ranking payoff — fix this sprint
  P1  Impact 3, Effort 3      Strategic must-do — plan into next cycle
  P1  Impact 2, Effort 1      Easy medium-wins — batch into this sprint
  P2  Impact 2, Effort 2–3    Scheduled work — roadmap in next quarter
  P3  Impact 1, any effort    Backlog / ignore — only if spare capacity
```

Every row in the fix backlog shows: **Issue · Category · Impact · Effort · Tier ·
Count (pages affected) · Fix Brief (for P0/P1)**.

---

## Step 4 — Fix backlog output format

```
## Technical SEO Fix Backlog — [domain] — [date]
Brand: [slug, via brand-brain]
Input: [crawl export / URL only / GSC export]
CMS: [from brand-brain]

### P0 — Fix this sprint

| # | Issue | Category | Impact | Effort | Pages | Fix |
|---|---|---|---|---|---|---|
| 1 | [issue] | [cat] | 3 | 1 | [n] | [exact spec] |

### P1 — Plan into next cycle
[same table]

### P2 — Roadmap for Q[n]
[same table, fix column is brief]

### P3 — Backlog / monitor
[bullet list only — no table overhead]

### Delegated to sibling skills
- Schema fixes → schema-markup-generator output embedded below
- CWV → page-speed-core-web-vitals-fixer called; output follows
```

---

## Fix Brief standard (P0 and P1)

For every P0 and P1 issue, the Fix column must contain a **ready-to-implement spec** —
not advice. Examples of what "ready to implement" means:

**Redirect chain collapse**
```
301 /old-url/ → /final-url/  (remove intermediate /mid-url/)
.htaccess: RedirectPermanent /old-url/ https://example.com/final-url/
```

**Canonical leak on filter pages**
```html
<!-- Add to <head> on all ?color=*, ?sort=* URLs -->
<link rel="canonical" href="https://example.com/[base-path]/" />
```

**robots.txt blocking JS assets**
```
# Remove this line (blocks rendering):
Disallow: /wp-includes/
# Replace with:
Allow: /wp-includes/
```

**Missing sitemap submission** — provide the exact GSC submission URL and the
sitemap endpoint to add.

For schema gaps: embed the JSON-LD block returned by `schema-markup-generator`, with
the target `<script>` placement noted.

---

## Persistence

Save the full audit to `./seo/[slug]-tech-audit-[YYYY-MM-DD].md` (relative to CWD, not
the skill folder). Confirm the save path to the user. Never overwrite `brand.md`.

If the user runs the skill on the same site 30+ days later, load the prior audit file,
diff the findings, and show resolved issues alongside new ones.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Never assume the canonical domain, CMS, or GSC property.
- **Effort × Impact, always scored.** Every finding gets a tier; no opinion-sorted lists.
- **Implementation specs, not advice.** P0/P1 fixes include copy-paste-ready code or
  config — not "add a canonical tag." Developers should be able to act without questions.
- **Delegate, don't duplicate.** Schema → `schema-markup-generator`; CWV →
  `page-speed-core-web-vitals-fixer`; meta-at-scale → `on-page-seo-optimizer`.
- **Real findings only.** Never invent issues. If a crawl export is absent, say what
  can't be verified and how to get it. Mark inferred findings `[verify]`.
- **Count pages affected.** An issue affecting 1 page and an issue affecting 10,000
  pages get different priorities even at the same severity.

---

## What not to do

- Don't start the audit before `brand-brain` returns the active brand.
- Don't re-implement JSON-LD schema generation — call `schema-markup-generator`.
- Don't re-implement CWV fix logic — call `page-speed-core-web-vitals-fixer`.
- Don't produce a generic checklist without input-specific findings.
- Don't sort by "severity" without scoring Effort — a medium-impact, zero-effort fix
  outranks a high-impact, months-long infrastructure project in this sprint.
- Don't flag N/A categories as issues; explain why they're N/A.
- Don't invent crawl data when only a URL is provided; state what the URL-only mode
  can and cannot check.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called first; canonical domain, CMS, and staging/prod confirmed?
- All eight audit categories worked through (or explicitly marked N/A)?
- Every finding has Impact + Effort + Tier; P-tier assignment follows the matrix formula?
- P0/P1 items include copy-paste-ready implementation specs, not narrative advice?
- Schema gaps delegated to `schema-markup-generator` with output embedded?
- CWV failures delegated to `page-speed-core-web-vitals-fixer` when dominant?
- Page-affected count included for every finding?
- Audit saved to `./seo/[slug]-tech-audit-[YYYY-MM-DD].md`?
- No invented data; any inferred finding marked `[verify]`?
