---
name: infographic-data-viz-spec-writer
description: >
  Turns raw data, stats, or a research dump into a production-ready infographic spec that a designer
  or Canva/Figma operator can execute without a briefing call. Applies the LATCH information
  architecture framework (Location, Alphabet, Time, Category, Hierarchy) to decide structure, then
  follows Cairo's Grammar of Graphics to select the right chart type for each data relationship — so
  every visual element earns its place and tells one honest story. Outputs a section-by-section spec
  with: narrative arc, section headers and one-line data points, chart-type recommendation per
  section with encoding rationale, layout direction (vertical scroll, horizontal panels, grid),
  color-encoding rules from the active brand palette, and a copy skeleton with real numbers in place.
  Does NOT design pixels or generate images; calls `creative-brief-image-prompt-crafter` when an
  AI-image placeholder is needed, and calls `brand-brain` for palette and voice. Use when the user
  says "make an infographic," "spec out a data viz," "turn this data into a visual," "what chart
  should I use," "design brief for stats," "infographic outline," or hands over raw numbers and asks
  for something a designer can build from.
---

# Infographic & Data-Viz Spec Writer

Raw data is not a story. This skill applies a named information-architecture framework to impose
structure, a named chart-selection grammar to pick the right visual encoding, and the active brand's
palette and voice to produce a brief a designer executes — not a vague mood board.

Output is a structured spec document, not a drawn asset. Every chart-type choice includes the
encoding rationale and the one thing the reader should take away from that panel. If the data
doesn't support a clean story, this skill says so instead of decorating noise.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand's palette, voice, and ICP. Do
  not begin the spec until brand-brain returns.
- **`creative-brief-image-prompt-crafter`** (optional) — when a section calls for an illustrative
  hero image or icon set rather than a data chart, generate the AI-image prompt here instead of
  improvising one.
- **`proof-vault`** (optional) — pull validated proof points and real stats when the user's raw
  input is thin; mark any unconfirmed number `[verify]`.
- **`canva-figma-workflow-accelerator`** (optional) — if the user wants a Canva template brief or
  Figma component inventory alongside the spec, route to this skill after the spec is done.

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain; get palette, voice, ICP, banned words
Step 1  Interrogate the data ──► apply LATCH to determine the right organizing structure
Step 2  Select chart types   ──► apply Cairo's grammar per data relationship
Step 3  Write the spec       ──► section-by-section: header, stat, chart type, encoding rationale,
                                  copy skeleton, layout + color direction
Step 4  Self-review          ──► truth discipline, brand voice, one-message-per-panel rule
Step 5  Present + save       ──► inline or to ./infographic/[slug]-spec.md
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's color
palette, voice adjectives, banned words, ICP, and positioning. Use the palette's primary, secondary,
and accent tokens for all color-encoding decisions in the spec. Apply the voice to copy skeleton
text. Mark any stat the user provided but cannot confirm as `[verify]`.

Fallback if `brand-brain` is not installed: read `~/.brandbrain/brands/.active` and that brand's
`brand.md`; if none exists, ask for palette (3 hex values minimum) and 3 voice adjectives before
proceeding.

---

## Framework 1 — LATCH (information architecture)

Before picking chart types, decide *how the whole piece is organized*. Apply Richard Saul Wurman's
LATCH to the data in hand and pick the one organizing axis that makes the story clearest:

| Axis | Use when | Example |
|---|---|---|
| **Location** | Geographic distribution is the insight | "Push opt-in rates by region" |
| **Alphabet** | No natural order; lookup is the use case | Feature comparison glossary |
| **Time** | Trend or before/after is the point | "How notification click rates changed 2022–2025" |
| **Category** | Discrete groups that don't rank | "5 notification types and when to use each" |
| **Hierarchy** | Magnitude or importance ranks | "Top 10 reasons subscribers churn, by volume" |

State the chosen axis in the spec header and explain in one sentence why it was selected over the
alternatives. If the user's data spans multiple axes, split into clearly-labeled sections — one LATCH
axis per section.

---

## Framework 2 — Cairo's Grammar of Graphics (chart selection)

For each data relationship in the spec, choose the encoding that matches the relationship type.
Always name the chart type AND write a one-line encoding rationale. Never choose a chart because
it looks interesting; choose it because it encodes the relationship honestly.

| Relationship | Preferred encoding | Avoid |
|---|---|---|
| Ranking / comparison (few items) | Horizontal bar chart | Pie / donut with >4 slices |
| Part-to-whole (≤5 parts) | Donut or stacked bar | 3D pie |
| Trend over time | Line chart (multiple series → multi-line) | Bar chart for continuous time |
| Distribution | Histogram or box plot | Area chart for count data |
| Correlation | Scatter plot | Dual-axis bar (misleading scale) |
| Geographic distribution | Choropleth or bubble map | Data table |
| Flow / process | Sankey or linear flow diagram | Pie chart |
| Single big number | Stat callout (large numeral + context line) | Gauge chart |

Apply Cairo's principle: **every ink element encodes data or aids navigation; everything else is
chartjunk and gets cut.**

---

## The spec format (section by section)

Produce this structure for every infographic spec:

```markdown
## Infographic Spec — [Title]

**Brand:** [slug, from brand-brain]
**Organizing axis (LATCH):** [axis] — [one-sentence rationale]
**Layout direction:** [vertical scroll | horizontal panels | grid] — [rationale]
**Intended output format:** [social static 1080×1080 | Pinterest 1000×1500 | blog embed 800px wide |
  presentation slide 1920×1080 | etc.]
**Target reader action:** [what should the reader do or believe after seeing this?]

---

### Section [N]: [Section Header]

**Data point(s):**
- [Real stat or `[verify]` placeholder — source if known]

**Chart type:** [name]
**Encoding rationale:** [one sentence — why this chart for this relationship]
**Color encoding:** [which brand palette token maps to which variable/category]
**One takeaway (the caption job):** [the single sentence a reader must walk away with]
**Copy skeleton:**
- Headline: [draft]
- Subhead / annotation: [draft]
- Source line: [publication + year, or `[verify]`]

---
[repeat per section]

---

## Layout & hierarchy notes

[2–4 bullets on reading order, section weight, whitespace, and any responsive considerations]

## Typography direction

[brand font(s) from brand-brain; role assignments: stat numerals → [weight], section headers →
[weight], body annotations → [weight]; cap at 3 type styles total]

## What to hand the designer

[Checklist: spec file path, brand palette hex values, any icon/image briefs, preferred tool
(Canva / Figma / Illustrator), output format + dimensions, deadline if known]
```

Save to `./infographic/[brand-slug]-[topic-slug]-spec.md` when the user asks to save or when the
spec exceeds a single screen of output.

---

## Principles

- **Brand-brain first.** No palette, no spec. Color decisions are made from the brand's real tokens,
  not invented.
- **One takeaway per panel.** Each section answers one question. If a section answers two, split it.
- **Name the chart; justify the choice.** "Bar chart because we're ranking" is better than a
  beautiful wheel chart that encodes nothing clearly.
- **Real numbers or `[verify]`.** Never invent a statistic to make a panel look substantiated. Flag
  thin data and suggest what to collect.
- **LATCH before pixels.** Structure determines comprehension. Wrong organization makes good charts
  unreadable.
- **Chartjunk is waste.** Every decorative element that doesn't encode data slows the reader.
  Call it out in the spec so the designer knows what to cut.
- **Honest scales.** Bar charts start at zero. Dual-axis charts are flagged as misleading by default
  unless the user confirms intentional scale mismatch.

---

## What not to do

- Don't invent stats to fill a panel — mark gaps `[verify]` and suggest a source type.
- Don't pick a chart type for visual novelty; justify every choice against the data relationship.
- Don't write the spec in brand colors you invent — wait for brand-brain to return the palette.
- Don't produce more than 7 sections in a single infographic spec without asking whether the piece
  should be split into a series.
- Don't ignore the output format — a 1080×1080 square has a completely different layout logic than
  a 1000×1500 Pinterest pin.
- Don't skip the "one takeaway" line — it is the brief the copywriter uses; without it, captions
  drift off-message.
- Don't use dual-axis charts without an explicit warning about scale distortion.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and palette + voice loaded before any color or copy decision?
- LATCH axis named and justified in the spec header?
- Every chart type named with a one-line Cairo encoding rationale?
- All stats real and sourced, or marked `[verify]`?
- Each section has exactly one takeaway line (not two, not zero)?
- Color encoding uses brand palette tokens, not arbitrary hex values?
- Layout direction stated and matched to the declared output format/dimensions?
- Designer handoff checklist complete (palette, file path, tool, dimensions)?
- Spec saved to `./infographic/` if output is long or user requested save?
