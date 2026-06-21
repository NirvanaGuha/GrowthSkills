---
name: landing-page-builder-html-tailwind
description: >
  Takes a described offer and builds a complete, conversion-optimized, single-file HTML + Tailwind CSS
  landing page — fully coded, responsive, and ready to ship. It does NOT derive copy, positioning, or
  CTAs from scratch; those come from the sibling skills that already own that work: brand-brain (voice +
  proof), landing-product-page-copy-writer (headline/body), and cta-variant-generator (button labels +
  microcopy). This skill's job is to receive that structured content, apply a rigorous conversion
  architecture (above-the-fold hierarchy, trust ladders, friction reducers), and output a single file
  with Tailwind CDN, semantic HTML5, and inline <style> overrides only for anything Tailwind can't
  express. The result is a real, paste-and-deploy HTML file — not a wireframe, not a mood board. Use
  when the user says "build me a landing page," "code up this page," "make an HTML landing page,"
  "turn this brief into a page," "I need a Tailwind landing page," or hands over offer copy and asks
  for a coded page.
---

# Landing Page Builder (HTML + Tailwind)

Described offer in — working, conversion-optimized landing page out. The page ships as a single `.html` file with Tailwind CDN, no build step required. Every structural and styling decision is driven by CRO architecture, not aesthetic whim. Copy and CTAs come from the skills that own them; this skill's job is to wire them into a page that converts.

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
Step 3  Build               ──► write the single HTML file with Tailwind CDN
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

Every page is built on the **LIFT model** (Unbounce) cross-referenced with **Conversion Centered Design (CCD)** principles:

| LIFT lever | Page zone | Design decision |
|---|---|---|
| **Value clarity** | Hero (above fold) | Single dominant H1, benefit-led subhead, no competing headlines |
| **Relevance** | Hero + first feature block | Message-matches the traffic source (ad / email / organic); the headline mirrors the visitor's intent |
| **Incentive** | Offer/pricing section | Anchoring, risk-reducer microcopy, guarantee badge |
| **Urgency** | CTA + sticky bar | Honest scarcity only; no fake countdown timers |
| **Friction** | Form + CTA zone | Minimum fields; one primary CTA per viewport; secondary CTAs visually subordinate |
| **Distraction** | Nav + footer | Navigation stripped; footer is minimal (legal links only); zero outbound links that bleed the visitor |

**Standard section order** (adjust for offer type):

1. **Hero** — headline (H1), subhead, primary CTA, hero image or product shot (or a strong visual metaphor). Above the fold on a 1280×800 viewport.
2. **Social proof bar** — logo strip or stat callout; immediately below the fold.
3. **Problem/Benefit block** — 2–3 column feature cards with icon + headline + one-line description.
4. **Deeper proof** — testimonial pull-quote(s) with name/role/company; if real photos are unavailable use initials avatar with a comment noting `[add real photo]`.
5. **How it works** — numbered 3-step visual (works for SaaS trials, lead-gen, eCommerce checkout).
6. **Objection handler / FAQ** — 3–5 accordion items; use `<details>`/`<summary>` for zero-JS expand.
7. **Secondary CTA block** — repeat the primary CTA with a fresh proof anchor; higher conversion commitment is appropriate here (visitor is more convinced).
8. **Footer** — privacy policy, terms, company name; no nav.

Omit or merge sections only when the offer type genuinely doesn't need them (e.g., a super-short lead-gen squeeze page skips How It Works). Document the omission with a comment in the file.

---

## Step 3 — Build rules (Tailwind + HTML)

### Tailwind setup

Use the Play CDN for zero-build-step deployability:

```html
<script src="https://cdn.tailwindcss.com"></script>
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

### HTML rules

- **Semantic HTML5** — `<header>`, `<main>`, `<section aria-label="…">`, `<footer>`. Every image has meaningful `alt` text.
- **Single `<h1>`** per page, in the hero. Section headers are `<h2>` or `<h3>`.
- **Mobile-first responsive** — base classes are mobile; `md:` and `lg:` modifiers add the desktop layout. Test mentally at 375 px and 1280 px.
- **One primary CTA per section.** The hero gets the primary button. Every repeat CTA is the same action, not a competing one.
- **No JavaScript except Tailwind CDN** — accordion uses `<details>`, sticky bar is CSS `position: sticky`. If the offer requires a form, stub the `<form action="" method="post">` with a visible `[connect to your form backend]` comment.
- **Performance notes as HTML comments** — `<!-- TODO: replace with optimized image -->`, `<!-- TODO: swap CDN Tailwind for PostCSS build in production -->`.
- **Inline `<style>` block** only for things Tailwind cannot express (e.g., a specific gradient, custom animation keyframes). Keep it minimal and commented.

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
- **Compose, don't duplicate.** Copy comes from `landing-product-page-copy-writer`; CTAs from `cta-variant-generator`; proof from `proof-vault`. This skill wires them into a page; it does not rewrite the library.
- **LIFT every section.** Each section earns its place by moving one LIFT lever. Cut sections that don't.
- **Real output.** The file must open in a browser and look credible without any build step.
- **One job per CTA per section.** Two competing primary CTAs on a single page doubles the decision friction and halves the conversion.
- **Honest everywhere.** No invented proof, no fake social proof counters, no fabricated testimonials. Placeholders get `[verify]` or `[content needed]` comments, never made-up copy.
- **Mobile-first, not mobile-afterthought.** The base Tailwind classes render mobile; desktop is the modifier. Check the hero fold at 375 px.

---

## What Not to Do

- Don't start the build before `brand-brain` returns — colors and fonts are non-negotiable structural inputs.
- Don't write CTAs or body copy from scratch without calling `cta-variant-generator` or `landing-product-page-copy-writer` — those skills own that job; reinventing them here creates drift.
- Don't include site navigation that lets the visitor leave before converting.
- Don't use JavaScript for things HTML/CSS can do natively (`<details>`, `position: sticky`, CSS transitions).
- Don't fake testimonials, invent customer logos, or use placeholder numbers as if real.
- Don't output a wireframe, an outline, or pseudocode — deliver a `.html` file that opens in a browser.
- Don't use Tailwind arbitrary values for every color — map the brand palette to named config keys once.
- Don't include multiple competing primary CTAs; a secondary CTA must be visually subordinate and the same action.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest in hand (colors, font, voice, proof)?
- Every content slot filled or explicitly marked `[content needed]` — no empty sections shipped?
- `landing-product-page-copy-writer` called (or user copy used) for headline/body; `cta-variant-generator` called for button labels?
- Single `<h1>` in the hero; `<h2>` for section headers; semantic HTML5 structure throughout?
- Brand palette mapped to named Tailwind config keys, not scattered inline hex?
- Hero above the fold at 1280×800; mobile layout tested at 375 px (mentally or via viewport resize)?
- Navigation stripped; footer minimal; zero outbound links that bleed the visitor pre-conversion?
- No JavaScript except Tailwind CDN; accordion via `<details>`; form stubbed with backend comment?
- OG/SEO meta block present with benefit-led description and canonical placeholder?
- All unconfirmed proof marked `[verify]`; all placeholder content marked `[content needed]`?
- TODOs (form backend, real photos, production Tailwind build) documented in top-of-file comment block?
- File saved to `./pages/<brand-slug>-landing-<offer-slug>.html` and path confirmed to user?
