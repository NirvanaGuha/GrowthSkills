---
name: landing-page-builder-html-tailwind
description: >
  Takes a described offer and builds a complete, conversion-optimized HTML + Tailwind CSS landing page —
  fully coded, responsive, accessible, and performance-aware. It does NOT derive copy, positioning, or
  CTAs from scratch; those come from the sibling skills that already own that work: brand-brain (voice +
  proof), landing-product-page-copy-writer (headline/body), and cta-variant-generator (button labels +
  microcopy). This skill's job is to receive that structured content, apply a rigorous conversion
  architecture (LIFT model: value prop, clarity, relevance, urgency, anxiety/risk-reversal, distraction),
  and output coded HTML5 with a clear preview build (Tailwind Play CDN) and a performance-aware ship
  target (purged CSS, LCP/CLS/WCAG-AA floor). The result is real code that opens in a browser — not a
  wireframe, not a mood board. Use
  when the user says "build me a landing page," "code up this page," "make an HTML landing page,"
  "turn this brief into a page," "I need a Tailwind landing page," or hands over offer copy and asks
  for a coded page.
---

# Landing Page Builder (HTML + Tailwind)

Described offer in — working, conversion-optimized landing page out. It ships in two honest grades: a single-file **preview build** (Tailwind Play CDN, opens instantly, no build step) for iteration and handoff, and a **performance-aware ship target** (purged CSS, with an LCP/CLS/contrast/focus/reduced-motion floor) for the version that actually faces paid or organic traffic. Every structural and styling decision is driven by CRO architecture, not aesthetic whim. Copy and CTAs come from the skills that own them; this skill's job is to wire them into a page that converts.

This is an engineering output skill. It produces real code, not outlines or mockups. If the copy is weak, it says so rather than paper-coating it with good design.

---

## Skills this calls

- **`brand-brain`** (required, first) — loads voice, colors, fonts, banned words, real proof, and positioning. Visual identity (hex values, font stack) drives the Tailwind config overrides.
- **`landing-product-page-copy-writer`** — generates the full copy scaffold (hero headline/subhead, benefit bullets, proof section, FAQ, final CTA). Call this before the build step if no copy exists yet.
- **`cta-variant-generator`** — produces the button label(s) + friction-reducer microcopy. Call when the user hasn't supplied CTAs.
- **`on-page-seo-optimizer`** *(optional)* — call after the page is built to harden `<title>`, meta description, OG tags, and H1/H2 hierarchy for the target keyword.
- **`landing-page-heuristic-live-cro-auditor`** *(optional)* — post-build self-audit pass. Call when the user wants a scored heuristic review of the completed page.
- **`proof-vault`** *(optional)* — surfaces real testimonials, statistics, and case-study pull-quotes to fill the social proof section.

---

## How a run works

```
Step 0  Brand context       ──► call brand-brain; extract colors, font, voice, proof
Step 1  Copy inventory      ──► collect or generate headline, body, CTAs, proof items
Step 2  Structure decision  ──► map copy to CRO page sections (framework below)
Step 3  Build               ──► write the HTML; CDN preview now, purged ship target for production
Step 4  Self-review         ──► quality checklist before presenting output
Step 5  Save artifact       ──► write to ./pages/<slug>.html
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (`skill: brand-brain`). From the returned digest, extract:

- **Hex palette** → map to Tailwind arbitrary-value classes (`bg-[#3B43FF]`, `text-[#191A35]`) or extend `tailwind.config` inside a `<script>` block.
- **Font stack** → set as `fontFamily.sans` in the config extension; fall back to the system stack if a web font is unavailable.
- **Voice adjectives + banned words** → apply as hard overrides to any copy this skill writes or edits.
- **Real proof** → social proof section populates only from `brand-brain`-confirmed items; unconfirmed items get `[verify]` and a placeholder comment in the HTML.

**Fallback:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If none, ask the user to supply at minimum: hex primary/secondary colors, font name, ICP one-liner, and offer headline.

---

## Step 1 — Copy inventory

Before writing a line of HTML, account for every content slot in the page skeleton. If a slot is empty:

- **Headline / subhead / body:** call `landing-product-page-copy-writer` (pass the offer description + brand digest).
- **CTA label + microcopy:** call `cta-variant-generator` (pass placement = hero, awareness stage, offer mechanics).
- **Proof items:** call `proof-vault`, or accept user-supplied copy. Never fabricate a testimonial or statistic.
- **FAQ:** derive from known objections in `brand.md` (`objections.md` companion if present), or from the user's supplied brief. If neither exists, generate plausible question stubs with `[content needed]` markers.

Populate a content map (internal working doc, not shown to the user) before touching markup.

---

## Step 2 — CRO Page Architecture (the named framework)

Every page is built on the **LIFT model** (WiderFunnel / Chris Goward). LIFT has exactly **six** factors: the Value Proposition is the central beam; Clarity, Relevance, and Urgency raise conversion; Anxiety and Distraction drag it down. Map each to a page zone and let it drive a concrete design decision. (Do not confuse this with Fogg's behavior model — "friction" and "incentive" are Fogg's vocabulary, not LIFT's. The LIFT analogue of friction is **Anxiety**, and it earns its own page zone.)

| LIFT factor | Role | Page zone | Design decision |
|---|---|---|---|
| **Value Proposition** | central beam | Hero + offer section | The strongest benefit, stated as outcome-for-the-visitor, dominates the fold. Everything else supports it. If the value prop is weak, no design saves the page — say so. |
| **Clarity** | conversion driver | Hero, feature blocks, CTA | One dominant H1, benefit-led subhead, no competing headlines; the CTA names the next action in plain language ("Start my free trial," not "Submit"). |
| **Relevance** | conversion driver | Hero + first feature block | Message-match the traffic source (ad / email / organic): the headline mirrors the visitor's intent and the ad/email promise. A scent break here kills the rest of the page. |
| **Urgency** | conversion driver | CTA + sticky bar | Honest scarcity only (real deadline, real stock, real cohort). No fake countdown timers — they erode trust and can be illegal in some jurisdictions. |
| **Anxiety** | conversion drag | Dedicated risk-reversal zone, near every CTA | The lever the audit must not drop. Reduce perceived risk: guarantee/refund badge, security and privacy signals (SSL, "we never share your email," payment-provider logos), social proof adjacent to the ask, transparent pricing, named human support. Minimize form fields and explain why each is needed — fewer asks = less anxiety. |
| **Distraction** | conversion drag | Nav + footer + body | Strip global navigation; footer carries legal links only; zero outbound links that bleed the visitor pre-conversion; one primary CTA per viewport with secondaries visually subordinate. |

**Anxiety gets its own zone.** Place an explicit risk-reversal block adjacent to the offer/pricing CTA (guarantee + security + a proof anchor), and repeat a compact trust signal beside the final CTA. This is where guarantees, refund policy, security badges, and "no credit card required" live — it is a page section, not a microcopy afterthought.

**Standard section order** (adjust for offer type):

1. **Hero** — headline (H1), subhead, primary CTA, hero image or product shot (or a strong visual metaphor). Above the fold on a 1280×800 viewport. Carries Value Proposition + Clarity + Relevance.
2. **Social proof bar** — logo strip or stat callout; immediately below the fold. First anxiety-reducer.
3. **Problem/Benefit block** — 2–3 column feature cards with icon + headline + one-line description.
4. **Deeper proof** — testimonial pull-quote(s) with name/role/company; if real photos are unavailable use initials avatar with a comment noting `[add real photo]`.
5. **How it works** — numbered 3-step visual (works for SaaS trials, lead-gen, eCommerce checkout).
6. **Risk-reversal / offer block** — the Anxiety zone: guarantee or refund terms, security/privacy signals, transparent pricing, "no credit card required" where true, placed adjacent to the offer CTA.
7. **Objection handler / FAQ** — 3–5 accordion items; use `<details>`/`<summary>` for zero-JS expand. Each answer should defuse one real anxiety.
8. **Final CTA block** — repeat the primary CTA (same action) with a fresh proof anchor and a compact trust signal beside it; the visitor is more convinced here.
9. **Footer** — privacy policy, terms, company name; no nav.

Omit or merge sections only when the offer type genuinely doesn't need them (e.g., a super-short lead-gen squeeze page folds risk-reversal into the hero and skips How It Works). Document the omission with a comment in the file. Never drop the Anxiety zone entirely — fold it into the hero or CTA, but the risk-reversal signal must appear.

---

## Step 3 — Build rules (Tailwind + HTML)

### Tailwind setup — two targets, stated honestly

The Play CDN (`cdn.tailwindcss.com`) compiles Tailwind in the browser at runtime. It is excellent for **preview and handoff** and terrible for a **production conversion page**: it ships the entire framework unpurged, blocks render, and causes a flash of unstyled content (FOUC) that tanks LCP and CLS — the exact metrics that gate paid-traffic Quality Score and SEO. So deliver one of two clearly-labeled targets, never a CDN page mislabeled "production":

- **Preview / handoff (default during iteration):** Play CDN, single file, opens instantly with no build. Top-of-file comment must read `<!-- PREVIEW BUILD: Tailwind Play CDN. Not production-ready (unpurged CSS + FOUC). Ship target below. -->`.
- **Ship target (what "ready to ship" means):** a performance-aware build. Either (a) run the Tailwind CLI to emit a purged, minified stylesheet and link it in place of the CDN script (`npx @tailwindcss/cli -i in.css -o dist.css --minify`), or (b) if you must hand back a single self-contained file, inline only the used utilities into a `<style>` block. Whichever you choose, the file must meet the **performance + a11y floor below** before it is called shippable.

Either way:

```html
<script src="https://cdn.tailwindcss.com"></script>  <!-- PREVIEW ONLY; swap for purged build before ship -->
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          brand:   '#HEXVAL',   // primary (from brand-brain)
          accent:  '#HEXVAL',   // secondary / CTA
          dark:    '#HEXVAL',   // text / background
        },
        fontFamily: {
          sans: ['"Font Name"', 'ui-sans-serif', 'system-ui', 'sans-serif'],
        },
      }
    }
  }
</script>
```

Use `bg-brand`, `text-accent`, etc. throughout; never scatter raw hex inline — it makes the brand uneditable. Arbitrary-value classes (`bg-[#xxx]`) are acceptable only when a shade is used exactly once.

### Performance + a11y floor (a page is not "ready to ship" until it clears this)

- **LCP:** the hero headline/image is the LCP element — it must be plain HTML/CSS, not injected by script. Hero image gets explicit `width`/`height` and `fetchpriority="high"`; everything below the fold gets `loading="lazy"`. No CDN-compiled CSS on the ship target (it pushes LCP past 2.5s on mid-tier mobile).
- **CLS:** reserve space for every image, embed, and the sticky bar (explicit dimensions or `aspect-ratio`). Self-host or `preload` the brand font and set `font-display: swap` so the swap doesn't shift layout. Target CLS < 0.1.
- **Color contrast (you pulled the brand hex — now check it):** body text ≥ 4.5:1 against its background, large text and UI/CTA components ≥ 3:1 (WCAG AA). The most common failure is a CTA using the brand accent with white text below 4.5:1, or yellow-on-white. If the brand pair fails, do not silently ship it — darken the text, add a stroke/overlay, or flag `<!-- contrast: accent #X on #Y = N:1, fails AA, needs darker text -->`.
- **Focus states:** every link, button, and form field has a visible focus indicator. Do not strip outlines without replacement — use `focus-visible:ring-2 focus-visible:ring-accent focus-visible:ring-offset-2`.
- **prefers-reduced-motion:** wrap any animation/transition/scroll effect so it is disabled for users who opt out — `@media (prefers-reduced-motion: reduce) { *, *::before, *::after { animation: none !important; transition: none !important; scroll-behavior: auto !important; } }` (or Tailwind's `motion-reduce:` variants on the specific elements).

### HTML rules

- **Semantic HTML5** — `<header>`, `<main>`, `<section aria-label="…">`, `<footer>`. Every image has meaningful `alt` text (decorative images get `alt=""`); icons that convey meaning get `aria-label`.
- **Single `<h1>`** per page, in the hero. Section headers are `<h2>` or `<h3>` in order — never skip levels for sizing (use Tailwind for size, not heading rank).
- **Mobile-first responsive** — base classes are mobile; `md:` and `lg:` modifiers add the desktop layout. Test mentally at 375 px and 1280 px.
- **One primary CTA per section.** The hero gets the primary button. Every repeat CTA is the same action, not a competing one.
- **A11y floor is mandatory, not optional** — every interactive element clears the contrast, focus-state, and reduced-motion bar in the Performance + a11y floor above. Form fields get associated `<label>`s.
- **No JavaScript except the (preview-only) Tailwind CDN** — accordion uses `<details>`, sticky bar is CSS `position: sticky`. If the offer requires a form, stub the `<form action="" method="post">` with a visible `[connect to your form backend]` comment.
- **Inline `<style>` block** only for things Tailwind cannot express (e.g., a specific gradient, custom keyframes, the `prefers-reduced-motion` guard). Keep it minimal and commented.

### OG / SEO meta block

Always include:

```html
<meta name="description" content="[benefit-led, 150–160 chars]">
<meta property="og:title" content="[headline]">
<meta property="og:description" content="[subhead or first benefit]">
<meta property="og:image" content="[placeholder or supplied URL]">
<link rel="canonical" href="[URL or placeholder]">
```

---

## Step 4 — Self-review before presenting

Run the quality checklist (below) silently. Fix any fails before presenting. Flag anything that requires a user decision (e.g., `[add real testimonial photo]`, `[connect form backend]`) as numbered TODOs in a block comment at the top of the file.

---

## Step 5 — Save the artifact

Write the completed file to `./pages/<brand-slug>-landing-<offer-slug>.html` (resolves to user CWD). Confirm the path in a one-line note after presenting the file. If the user is in a project folder with a defined pages directory, use that.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No markup before the brand digest is in hand — colors, fonts, and voice are structural inputs, not decorations.
- **Compose, don't duplicate.** Copy comes from `landing-product-page-copy-writer`; CTAs from `cta-variant-generator`; proof from `proof-vault`. This skill wires them into a page; it does not rewrite the library, and it does not invent proof, logos, or numbers — placeholders get `[verify]` / `[content needed]`, never made-up copy.
- **The value proposition carries the page.** If the offer's value prop is weak or fuzzy, design cannot rescue it — call it out and route back to copy/positioning rather than coding a beautiful page that won't convert.
- **Every section earns its place by moving one LIFT factor.** Clarity, Relevance, Urgency lift; Anxiety and Distraction drag. Cut sections that move nothing — but never cut the Anxiety/risk-reversal signal.
- **One job per CTA per section.** Two competing primary CTAs doubles decision friction and halves conversion; a secondary CTA must be visually subordinate and the same action.
- **Shippable means it clears the floor.** "Ready to ship" is the performance-aware target (purged CSS, LCP/CLS, WCAG-AA contrast, visible focus, reduced-motion), not the CDN preview. Label which grade you're handing back.
- **Mobile-first, not mobile-afterthought.** Base Tailwind classes render mobile; desktop is the modifier. Check the hero fold at 375 px.

---

## What Not to Do

- Don't ship the Play CDN build as "production" — it's unpurged and FOUCs; it is a preview/handoff grade only, labeled as such.
- Don't strip focus outlines, hardcode motion that ignores `prefers-reduced-motion`, or ship a CTA/text color pair that fails AA contrast — flag the failing ratio instead of silently shipping it.
- Don't include site navigation that lets the visitor leave before converting.
- Don't use JavaScript for things HTML/CSS do natively (`<details>`, `position: sticky`, CSS transitions).
- Don't use Tailwind arbitrary values for every color — map the brand palette to named config keys once.
- Don't import "incentive" or "friction" into the LIFT model — those are Fogg's terms; LIFT's six are Value Proposition, Clarity, Relevance, Urgency, Anxiety, Distraction.
- Don't output a wireframe, an outline, or pseudocode — deliver an `.html` file that opens in a browser.

---

## Quality Checklist (self-review before presenting)

**Inputs & composition**
- `brand-brain` called and brand digest in hand (colors, font, voice, proof)?
- `landing-product-page-copy-writer` (or user copy) used for headline/body; `cta-variant-generator` used for button labels?
- Every content slot filled or marked `[content needed]`; all unconfirmed proof marked `[verify]` — no fabricated proof, no empty sections?

**LIFT architecture**
- Value Proposition dominates the fold; Clarity (one H1, plain-language CTA) and Relevance (message-match to source) hold?
- Urgency is honest (real deadline/stock/cohort), not a fake timer?
- **Anxiety zone present** — guarantee/risk-reversal + security/privacy signal adjacent to the offer CTA, and a trust signal beside the final CTA?
- Distraction minimized — nav stripped, footer legal-only, one primary CTA per viewport, secondaries subordinate?

**Performance & a11y floor (ship target)**
- Hero/LCP element is static HTML/CSS (not script-injected); below-fold images `loading="lazy"`; hero image has dimensions + `fetchpriority="high"`?
- CLS guarded — dimensions/`aspect-ratio` on images, embeds, sticky bar; font `display: swap`?
- Color contrast checked against the actual brand hex — body ≥ 4.5:1, large text/CTA ≥ 3:1 (AA); any failure flagged, not shipped silently?
- Visible focus states on all interactive elements; `prefers-reduced-motion` honored?
- Ship target uses purged/minified CSS — the Play CDN is labeled preview-only, not handed back as production?

**Structure & output**
- Single `<h1>` in hero; `<h2>`/`<h3>` in order; semantic HTML5; form fields have `<label>`s; meaningful `alt` text?
- Brand palette mapped to named Tailwind config keys, not scattered inline hex?
- Mobile layout checked at 375 px and hero fold at 1280×800?
- OG/SEO meta block present with benefit-led description and canonical placeholder?
- TODOs (form backend, real photos, CDN→purged swap) in top-of-file comment block?
- File saved to `./pages/<brand-slug>-landing-<offer-slug>.html` and path confirmed to user?
