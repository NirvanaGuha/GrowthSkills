---
name: page-speed-core-web-vitals-fixer
description: >
  Turns a PageSpeed Insights JSON report (or a live WordPress/Shopify URL) into a
  prioritized, actionable fix list with concrete implementation steps — not advice. For each
  flagged issue it outputs the exact change required, its estimated LCP/CLS/INP impact bucket
  (High / Medium / Low), and the specific plugin, code snippet, or server config to apply.
  Covers every Core Web Vitals signal: LCP root-cause triage (TTFB, render-blocking resources,
  slow server, unoptimized images), CLS shift-source identification (ad slots, font-swap,
  late-loading embeds), INP interactivity bottlenecks (long tasks, third-party scripts), FID
  fallback for older data. Also produces a redirect-chain audit map and a caching/plugin-bloat
  assessment for WordPress. Composes `technical-seo-audit-fix-prioritizer` for crawl/schema
  issues and `tracking-plan-taxonomy-builder-auditor` to validate that tag managers aren't
  inflating page weight. Use when someone pastes a PageSpeed URL or JSON, says "my site is
  slow," "fix my Core Web Vitals," "improve LCP/CLS/INP," "failing CWV," "page speed audit,"
  "WordPress too slow," or "Lighthouse score is bad."
---

# Page-Speed & Core Web Vitals Fixer

Give it a PageSpeed Insights URL or raw JSON, get a ranked fix list with real implementation steps — not a generic "compress your images" lecture. Every fix is scoped to the actual offenders the report names, bucketed by CWV impact, and paired with the specific code, config, or plugin change required. The goal is a passing set of Core Web Vitals within the smallest number of changes, prioritized by bang-for-buck.

This skill diagnoses and prescribes. It does not rebuild pages, pick a hosting stack from scratch, or substitute for a full performance engineering engagement on a complex SPA. For crawlability and schema issues surfaced during the audit, it composes `technical-seo-audit-fix-prioritizer`; for tag-manager bloat, `tracking-plan-taxonomy-builder-auditor`.

---

## Skills this calls

- **`brand-brain`** (Step 0, required) — loads the active brand's context so output references the correct site, stack (WP / Shopify / custom), and tech constraints. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the site URL, CMS/stack, and hosting provider before proceeding.
- **`technical-seo-audit-fix-prioritizer`** (compose, optional) — if the PageSpeed report surfaces crawlability, redirect, or schema issues, delegate those to this skill rather than duplicating its logic.
- **`tracking-plan-taxonomy-builder-auditor`** (compose, optional) — if GTM or third-party tag weight is a named offender in the report, invoke this skill to audit and trim the tag payload without breaking measurement.
- **`data-qa-measurement-gotcha-checker`** — gate: before accepting field data (CrUX) as the truth, run a quick data-quality pass. Lab data (Lighthouse) and field data (CrUX) can diverge; flag the discrepancy before prescribing fixes calibrated to the wrong baseline.

---

## How a run works

```
Step 0  Load the brand     ──► brand-brain (stack, URL, hosting, constraints)
Step 1  Ingest the report  ──► PSI JSON / live fetch / user paste
Step 2  Data-quality gate  ──► lab vs. field delta; CrUX coverage check
Step 3  Root-cause triage  ──► LCP / CLS / INP / TTFB waterfall reads
Step 4  Build the fix list ──► ranked by impact bucket, concrete steps
Step 5  Compose siblings   ──► delegate crawl/tag issues out
Step 6  Save the artifact  ──► ./cwv/[slug]-cwv-fix-list.md
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest including the site URL, known CMS/stack (WordPress, Shopify, custom), hosting provider, and any existing perf context. Use the site URL to run a live PageSpeed Insights fetch if the user hasn't already provided a JSON report.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the site URL, CMS/stack, and hosting provider before proceeding.

---

## Step 1 — Ingest the report

Accept any of:
- **PageSpeed Insights URL** — fetch `https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=<URL>&strategy=mobile` (and desktop separately); parse the response.
- **Raw PSI JSON** pasted by the user — parse directly.
- **Lighthouse HTML/JSON export** — parse the `audits` object.
- **Plain URL** — fetch PSI live (mobile-first, then desktop).

Extract: overall Performance score, each CWV metric (LCP, CLS, INP, FCP, TTFB), field data origin (CrUX percentile), and the full `audits` object with `score`, `numericValue`, and `details.items` for each flagged audit.

---

## Step 2 — Data-quality gate (lab vs. field)

Before diagnosing, flag any measurement integrity issues:

- **Lab/field delta > 20%** on LCP or INP: note it — lab fixes alone may not move the needle on CrUX; field data (real users) is what Google uses for ranking signals.
- **CrUX "insufficient data"**: label the session as lab-only; all recommendations are still valid but ranking impact is unconfirmed until CrUX builds up.
- **HTTPS / redirect chain before the audit URL**: note it as a TTFB contributor.
- Invoke `data-qa-measurement-gotcha-checker` if the user's report or context suggests the data was pulled from a bot-heavy or CDN-cached test environment — gotchas here can waste a full sprint chasing phantom issues.

---

## Step 3 — Root-cause triage (the Google RAIL / CWV framework)

Use **Google's official CWV diagnostics framework** (RAIL model + Lighthouse audit taxonomy). Read each signal's top offenders from the report's `audits` object:

### LCP (Largest Contentful Paint) — target: ≤ 2.5 s
Work down the LCP waterfall in order. Fix the earliest bottleneck first; later ones may self-resolve.

| Stage | Audit IDs to check | Common fix |
|---|---|---|
| TTFB | `server-response-time` | Hosting upgrade, full-page cache (WP: WP Rocket / LiteSpeed Cache, Nginx FastCGI), CDN (Cloudflare) |
| Render-blocking resources | `render-blocking-resources` | Defer/async non-critical JS; inline critical CSS; remove unused CSS/JS (`unused-javascript`, `unused-css-rules`) |
| LCP resource load time | `lcp-lazy-loaded`, `uses-optimized-images`, `uses-webp-images`, `efficient-animated-content` | Remove `loading="lazy"` from the LCP image; serve WebP/AVIF; preload the LCP image with `<link rel="preload" as="image">` |
| LCP element delay | `largest-contentful-paint-element` | Identify the element; if a hero image, set explicit width/height, preconnect to CDN origin |

### CLS (Cumulative Layout Shift) — target: ≤ 0.1
| Cause | Audit ID | Fix |
|---|---|---|
| Images without dimensions | `unsized-images` | Add explicit `width` + `height` on every `<img>`; use `aspect-ratio` in CSS |
| Ad slots | `layout-shift-elements` | Reserve space with `min-height` before ad loads |
| Web fonts (FOUT/FOIT) | `font-display` | Add `font-display: swap` or `optional`; self-host critical fonts; use `<link rel="preload">` for the WOFF2 |
| Late-injected embeds | `layout-shift-elements` | Wrap iframes/embeds in aspect-ratio containers; load below the fold lazily |

### INP (Interaction to Next Paint) — target: ≤ 200 ms
| Cause | Audit ID | Fix |
|---|---|---|
| Long JS tasks | `long-tasks`, `bootup-time` | Code-split; defer non-critical third-party scripts via `type="module"` or Partytown for tag manager payloads |
| Third-party scripts | `third-party-summary` | Load analytics/chat/pixel scripts with `async` + `defer`; use Facade pattern for embeds (YouTube lite, chat widgets) |
| DOM size | `dom-size` | Break large React/Vue re-renders; reduce DOM to < 1,500 nodes |

### TTFB (Time to First Byte) — target: ≤ 800 ms
Check `server-response-time`. If > 800 ms, the entire waterfall shifts right. Fix order: full-page cache → object cache (Redis/Memcached) → CDN → database query optimization → hosting tier.

---

## Step 4 — Build the prioritized fix list

Output format — every item must have all five columns:

```
## Fix List — [Brand slug] — [URL] — [Date]
Strategy: mobile | Score: [X] | LCP: [X]s | CLS: [X] | INP: [X]ms

### P1 — High LCP Impact (fix first)
| # | Issue | Metric | Estimated Impact | Exact Fix | Tool / File |
|---|---|---|---|---|---|
| 1 | LCP image not preloaded | LCP | −0.8–1.2 s | Add `<link rel="preload" as="image" href="[URL]">` in <head> | theme header.php / functions.php |

### P2 — High CLS Impact

### P3 — Medium Impact (INP / secondary LCP)

### P4 — Low Impact / Quick Wins

### Redirect Chain Audit
[List any redirect chains found: A→B→C with hop count and ms cost. Recommend 301 consolidation.]

### Caching & Plugin Bloat (WordPress / Shopify)
[List conflicting caching plugins, JS/CSS bloat sources, and render-blocking plugin scripts.]
```

**Estimated impact** uses Lighthouse's own opportunity savings when available (`numericValue` on opportunity audits); otherwise use Google's published benchmarks (e.g., "WebP images: typically −15–35% LCP on image-heavy pages" [verify exact % for the specific site]). Never invent savings numbers; mark uncertain ranges `[verify]`.

---

## Step 5 — Compose sibling skills

- If the PSI report surfaces 4xx/5xx URLs, broken canonicals, or schema errors → flag them and note: "Run `technical-seo-audit-fix-prioritizer` on this domain for a full crawl-issues pass."
- If `third-party-summary` shows Google Tag Manager contributing > 500 ms → flag it and note: "Run `tracking-plan-taxonomy-builder-auditor` to identify removable tags without breaking measurement coverage."

Do not duplicate those skills' output here — compose and route.

---

## Step 6 — Save the artifact

Save the full fix list to `./cwv/[brand-slug]-cwv-fix-list.md`. Confirm the path in one line. The file is overwritten on re-run (date-stamped at the top); tell the user to version-control it before a major sprint.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Don't audit a URL before knowing the stack — a WP fix and a Shopify fix for the same symptom are completely different.
- **Concrete output, not advice.** Every line in the fix list includes the exact file, tag, config value, or plugin setting to change — not "consider optimizing your images."
- **Waterfall order.** TTFB → render-blocking → LCP resource → LCP element. Skipping stages wastes sprints.
- **Lab ≠ field.** Always flag when the audit is lab-only; CrUX is what moves ranking signals.
- **Verified numbers only.** Use Lighthouse's own opportunity savings; mark anything else `[verify]`.
- **Compose, don't duplicate.** Crawl issues go to `technical-seo-audit-fix-prioritizer`; tag bloat goes to `tracking-plan-taxonomy-builder-auditor`. Don't reimplement their logic.

---

## What Not to Do

- Don't recommend generic perf advice ("use a CDN," "compress images") without naming the specific offending files/assets from the report.
- Don't prescribe WordPress plugin fixes for a Shopify or custom stack — check the brand's CMS first.
- Don't report only the lab score and ignore CrUX field data; note when they diverge significantly.
- Don't treat a 90+ Lighthouse score as "done" if CrUX field data still shows poor CWV — the ranking signal is field data.
- Don't conflate FID (legacy) with INP (current standard since March 2024 [verify date]); INP is the live CWV signal.
- Don't invent estimated ms/score savings; use Lighthouse opportunity values or mark `[verify]`.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; site URL, CMS/stack, and hosting confirmed before prescribing any fix?
- Lab vs. field delta flagged; CrUX coverage status noted?
- Every fix in the list has: metric affected, impact bucket, exact change, and file/tool/plugin?
- Redirect chain audit present (even if clean)?
- Caching/plugin-bloat section present for WP sites?
- Crawl/schema issues routed to `technical-seo-audit-fix-prioritizer`; tag bloat routed to `tracking-plan-taxonomy-builder-auditor`?
- Estimated savings cite Lighthouse opportunity values or are marked `[verify]`?
- Artifact saved to `./cwv/[slug]-cwv-fix-list.md`?
