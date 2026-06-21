---
name: alt-text-accessibility-copy-batch-writer
description: >
  Takes a batch of image assets — filenames with context, inline descriptions, live URLs,
  or Figma frame names — and returns a production-ready alt-text string for every asset:
  SEO-optimized, WCAG 2.2 AA-compliant, and written in the brand's real voice. Works in
  two modes: Single Asset (quick, default) and Batch Table (one pass for all assets, saved
  as a TSV handoff file). Applies the F.O.C.U.S. framework (Function → Object → Context →
  Use-case → Suppress decorative) to guarantee every string is actionable, not just
  descriptive. Calls brand-brain so ICP vocabulary and banned words carry through into alt
  copy. Flags decorative images as empty-alt candidates so developers get the correct
  attribute immediately. Triggers: "write alt text," "batch alt text," "alt text for
  images," "accessibility copy," "WCAG alt text," "SEO alt text," "alt text audit,"
  "image descriptions," "screen reader copy," or when a user shares a list of filenames,
  URLs, or Figma frames and asks what to write for each.
---

# Alt-Text & Accessibility Copy Batch Writer

Alt text is the smallest piece of copy that carries the most legal and SEO risk if skipped.
This skill turns a raw asset list — filenames, live URLs, pasted descriptions, or Figma
frame names — into a complete alt-text handoff table: one string per image, WCAG 2.2 AA
compliant, keyword-relevant, and written in the brand's actual voice.

It does not audit the page (use `brand-consistency-auditor` or a technical SEO audit for
that). It writes the strings and tells you exactly which attribute value to paste.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice adjectives, banned words, ICP vocabulary, and
  product/feature terminology. Alt copy must not contradict the product or use off-brand
  language. Do not write a single string before brand-brain returns.
- **`creative-brief-image-prompt-crafter`** (optional) — if images do not yet exist and the
  user only has a concept, call this first to clarify what the image will show; then write
  alt text against that spec.
- **`on-page-seo-optimizer`** (optional) — if the user wants keyword targets woven in, pull
  the target keyphrase list from there rather than guessing; alt text is a confirmed
  on-page signal for surrounding content relevance.
- **`design-qa-handoff-pack-builder`** (optional) — this skill can consume the TSV output
  from a batch run and include it in a dev handoff pack without duplication.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (or fallback)
Step 1  Pick the mode   ──► Single (default) | Batch Table (on request or ≥3 assets)
Step 2  Classify each image (informative / functional / decorative / complex)
Step 3  Apply F.O.C.U.S. per image
Step 4  Self-review (WCAG gates + brand gates)
Step 5  Deliver + offer to save batch output
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand digest:
voice adjectives, banned words, ICP vocabulary, product/feature names, and real proof points.
Do not write a single alt string before it returns.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md`. If neither exists, ask the user for: (a) 3 voice adjectives, (b) banned words,
(c) product name and core feature vocabulary. Prefer the call.

---

## The F.O.C.U.S. Framework

Every alt string is written by walking five questions in order:

| Letter | Question | Why it matters |
|--------|----------|----------------|
| **F** — Function | What job does this image do on the page? | Sets whether alt text is informative, functional, decorative, or complex |
| **O** — Object | What is the primary subject? (person, product, chart, icon) | The noun that opens the string |
| **C** — Context | What is the surrounding content? Headline, body copy, CTA? | Prevents redundancy; adds what the surrounding text doesn't say |
| **U** — Use-case | Is the image the sole carrier of information (e.g. a chart with data)? | Triggers long-description (`longdesc` or `aria-describedby`) flag |
| **S** — Suppress | Is the image purely decorative? | Outputs `alt=""` with a note to the developer — never write filler |

Apply F.O.C.U.S. to every asset before writing the string. Document the classification
(`informative` / `functional` / `decorative` / `complex-informative`) in the batch table.

---

## Image classification rules (WCAG 2.2, Images Tutorial)

| Type | Rule | Alt attribute |
|------|------|---------------|
| **Informative** | Adds visual info not in surrounding text | Descriptive string; convey the meaning, not just the content |
| **Functional** | Is a link or button (icon-only CTA, logo-link) | Describe the destination/action, not the image |
| **Decorative** | Adds no information; pure visual flair | `alt=""` — do NOT omit the attribute entirely |
| **Complex-informative** | Chart, diagram, map, infographic | Short alt + long description; flag the long-description need |
| **Text image** | Image of text | Reproduce the exact text in alt; flag as a WCAG 1.4.5 risk |

Flag "text image" occurrences as low-accessibility-risk items needing dev attention — do not
silently write an alt string and move on.

---

## Single mode (default — 1–2 images)

1. Ask (or infer from context): What page/placement? What surrounding copy says? Any target
   keyword?
2. Walk F.O.C.U.S. briefly (one sentence each, not shown to user unless asked).
3. Output: the `alt="…"` string, classification, char count, SEO keyword integration note (if
   any), and a one-line rationale. If decorative: output `alt=""` and a dev note.
4. Offer an alternate if a different keyword angle or shorter version is useful.

Keep single-mode output concise — the string, the rationale, done. No big tables.

---

## Batch mode (≥3 images, or on request)

### Input formats accepted

- **Filename list** (e.g. `hero-dashboard-screenshot.png`) — infer subject from filename;
  ask for missing context in one consolidated question before writing, not per image.
- **URL list** — fetch page context where possible; note when image is not publicly accessible.
- **Descriptions** — plain-English "image of X showing Y."
- **Figma frame names** — treat the frame name + any visible layer names as description inputs.

### Batch output format

Deliver as a Markdown table first (readable in chat), then offer to save as TSV to
`./alt-text/[brand-slug]-alt-text-batch.tsv` (importable to Figma, Notion, Google Sheets,
or a dev handoff doc):

```
| # | Filename / Asset ID | Classification | Alt Text String | Char Count | SEO Keyword | Dev Note |
|---|---------------------|----------------|-----------------|------------|-------------|----------|
| 1 | hero-screenshot.png | informative    | PushEngage dashboard showing a live push notification campaign with 94% delivery rate | 89 | push notification campaign | — |
| 2 | decorative-wave.svg | decorative     | (empty — alt="") | 0 | — | Set alt="" in HTML; do not omit attribute |
| 3 | open-rate-chart.png | complex-info   | Bar chart: email open rate 18% vs push notification open rate 32% across five industries | 94 | — | Add longdesc or aria-describedby with full data table |
```

**Quality gates run before delivery:**
- No string exceeds 125 characters (screen-reader optimal; flag over-runs)
- No string starts with "image of," "picture of," or "photo of" (redundant with the `img` role)
- No string duplicates the immediately adjacent visible caption or heading
- Decorative images output `alt=""` with a dev note, never filler text
- Complex images flag the long-description requirement
- All product/feature names match brand-brain vocabulary

Save only when the user confirms; never auto-overwrite a prior batch file.

---

## SEO integration (without keyword-stuffing)

Alt text is an on-page relevance signal. Rules:

1. **One target keyword per image, maximum.** If `on-page-seo-optimizer` has been called and
   returned a keyphrase list, pull from it. Otherwise use the page's `<h1>` or the user's
   stated target.
2. **Keyword must be truthful.** Only include a keyword if the image genuinely shows that
   concept. Forcing a keyword into a decorative image's alt (`alt=""`) is correct — empty;
   forcing it into an unrelated informative image is keyword-stuffing. Flag this situation
   instead of complying.
3. **Natural placement.** Keywords should read as description, not as tags. "Graph showing
   email vs push notification open rates" is fine. "push notification open rates push
   notification software" is not.
4. **Product-screenshot SEO pattern:** [product name] + [feature shown] + [key metric if
   visible]. E.g. `PushEngage Workflows builder showing a 3-step drip sequence`.

---

## Accessibility craft principles

- **Convey meaning, not appearance.** "Bar chart showing Q3 revenue up 42% year-over-year"
  beats "colorful bar chart."
- **Functional images describe the destination.** A logo that links to the homepage:
  `alt="[BrandName] — go to homepage"`.
- **Don't prefix.** Screen readers already announce "image." Starting with "image of" is
  double-announced.
- **Portraits and people.** Include person's name if publicly known and relevant; describe
  demographic detail only if it's the point of the image (e.g. a DEI campaign visual).
- **Localization.** If the brand serves multiple locales, note in the dev column that alt
  text requires translation — do not silently write English-only strings for multilingual
  pages.
- **SVG icons.** If the SVG is purely decorative: `aria-hidden="true"`, no `alt`. If
  functional: `aria-label` on the wrapping `<button>` or `<a>` — note this in the dev column.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No alt string before the active brand loads; voice + banned words
  are hard overrides.
- **F.O.C.U.S. every time.** Classification before copy; never skip decorative detection.
- **Decorative = empty, not filler.** `alt=""` is a deliberate, correct choice — not laziness.
- **Truth only.** Describe what is actually in the image; never invent content to hit a
  keyword.
- **Flag, don't hide.** Text images, missing long-descriptions, and keyword-stuffing requests
  get flagged, not silently mishandled.
- **125-character discipline.** Strings over 125 chars are flagged; the user chooses to trim
  or accept.

## What Not to Do

- Don't start strings with "image of," "picture of," "photo of," or "graphic of."
- Don't write `alt="decorative"` or `alt=" "` — the correct form is `alt=""`.
- Don't reimplement brand scanning; that lives in brand-brain.
- Don't silently stuff keywords into images that don't show the concept.
- Don't treat a complex chart as informative and write a one-liner — flag long-description.
- Don't omit the `alt` attribute entirely on decorative images (WCAG 1.1.1 failure).
- Don't duplicate the visible caption verbatim unless the image adds no additional meaning.

## Quality Checklist (self-review before presenting)

- [ ] brand-brain called and active brand loaded; voice + banned words applied?
- [ ] Every image classified (informative / functional / decorative / complex) before writing?
- [ ] No string starts with a redundant "image of" prefix?
- [ ] Decorative images output `alt=""` with a dev note (not filler)?
- [ ] Complex images flagged for long-description with `[needs longdesc]` in dev column?
- [ ] Text images flagged as WCAG 1.4.5 risk?
- [ ] No string exceeds 125 characters without an explicit flag?
- [ ] SEO keywords truthfully present in the image (not forced)?
- [ ] Batch output offered as TSV save to `./alt-text/[brand-slug]-alt-text-batch.tsv`?
- [ ] No prior batch file overwritten without confirmation?
