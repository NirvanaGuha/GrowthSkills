---
name: canva-figma-workflow-accelerator
description: >
  Design-ops accelerator for growth and content marketers who live in Canva and Figma.
  Takes a campaign brief or existing design file and outputs the full production package
  depending on the tool in play: Canva mode produces a template brief, Bulk Create CSV
  (for data-driven asset generation), and a Magic Resize spec covering every required
  format; Figma mode produces a component inventory audit, reusable component extraction
  brief, and a Dev Mode handoff pack with spacing tokens, typography scales, and
  annotated copy slots. When both tools are in play (Canva for content, Figma for product
  or landing pages) it coordinates outputs so brand tokens stay in sync. Stages production with the
  Double Diamond's four phase names (Design Council), adapted as a linear pipeline: Discover the real asset scope,
  Define the canonical template, Develop the production deliverables, Deliver the handoff. Brand voice and palette come from the
  shared brand-brain skill — not reimplemented here. Lean on brand-consistency-auditor to
  QA the final asset set.
  Trigger: "canva workflow," "figma handoff," "bulk create," "magic resize," "template brief,"
  "component library," "dev mode handoff," "resize for every format," "production-ready assets,"
  "design to copy handoff," "brand asset sprint," or whenever a campaign brief needs to become
  a full set of on-brand design deliverables without a dedicated designer.
---

# Canva & Figma Workflow Accelerator

Turns a campaign brief or design file into production-ready design deliverables. Canva gets a template brief, Bulk Create CSV, and Magic Resize spec. Figma gets a component inventory, extraction brief, and Dev Mode handoff pack. Either way, the marketer ships the right asset in every format without starting from a blank canvas or hunting for tokens.

The skill covers creative ops mechanics, not creative strategy. It does not invent the campaign concept — it builds the system that scales the concept across every surface. For concept generation call `campaign-concept-developer` first; for final copy QA call `brand-consistency-auditor` on the output.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's palette, typography, voice, and banned words. No design output before this returns.
- **`campaign-brief-builder`** (recommended upstream) — if the user has a rough idea not yet a brief, this structures it before the accelerator runs.
- **`infographic-data-viz-spec-writer`** (compose) — when the asset set includes a data visual, delegate spec-writing for charts and infographics.
- **`brand-consistency-auditor`** (downstream QA) — once assets exist, call to audit color, font, logo, and tone violations across the set.
- **`creative-brief-image-prompt-crafter`** (compose) — when AI-generated imagery is needed inside Canva or Figma frames, call for the precise prompt.
- **`design-qa-handoff-pack-builder`** (downstream) — for complex Figma projects, call after this skill's component inventory step to produce the full QA report and dev annotation.
- **`alt-text-accessibility-copy-batch-writer`** (compose) — once the asset manifest is finalized, batch the alt-text strings for every visual.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain; never proceed without it
Step 1  Scope the job         ──► Brief mode, File mode, or Full sprint?
Step 2  Discover              ──► enumerate surfaces, formats, asset count
Step 3  Define                ──► canonical template + master frame
Step 4  Develop               ──► Canva OR Figma deliverables (or both)
Step 5  Deliver               ──► handoff pack + asset manifest + downstream calls
```

### Step 0 — Brand context (always first)

Invoke the `brand-brain` skill to load the active brand. Use the returned palette (hex values), typography scale, voice adjectives, banned words, and real proof. If the brand is new, `brand-brain` bootstraps it before this skill proceeds.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none, ask the user for: primary + accent hex colors, headline / body font pair, and 3 voice adjectives + banned words, then proceed.

### Step 1 — Scope the job

Three entry points:

| Entry | What the user provides | Output mode |
|---|---|---|
| **Brief mode** | Campaign brief (copy, angles, channels) | Template brief + format matrix + Bulk Create CSV |
| **File mode** | Canva share link or Figma file URL/JSON | Inventory + gap audit + handoff pack |
| **Full sprint** | Both brief AND existing file | End-to-end: audit existing, template new, build full matrix |

Ask for any missing entry point before generating. One clarifying question at most; don't stall on secondary details.

---

## Discover → Define → Develop → Deliver (how this skill thinks)

**Discover → Define → Develop → Deliver.** We borrow the four phase names from the Design Council's Double Diamond as a linear production checklist — note we use them to stage asset production, not to run the divergent/convergent exploration the original model is built around. Marketers skip Discover and wonder why every campaign spawns ten ad-hoc files. Force the discipline here.

- **Discover:** enumerate every surface the campaign touches (social formats, email header, blog hero, display ad sizes, push thumbnail, landing-page OG image). Real surfaces only — list what the brief calls for, not every format that could theoretically exist.
- **Define:** identify the *master frame* — the one design at maximum canvas size from which all other sizes derive. Everything else is a resize variant or a data-driven Bulk Create row.
- **Develop:** build the canonical template brief (Canva) or component extraction brief (Figma). Generate the Bulk Create CSV or Dev Mode handoff. This is the meatiest step.
- **Deliver:** produce the asset manifest, name the downstream calls needed (alt-text, QA, etc.), and hand off.

---

## Canva mode

### Template brief

One concise brief per template needed. Format:

```
Template: [descriptive name]
Canvas size: [W × H px] — primary format
Layers to lock:  [logo zone, brand color backgrounds, icon placement]
Layers to free:  [headline slot (font, size, color), subhead, CTA text, main image frame]
Brand tokens to enforce:
  Primary color:    [hex from brand-brain]
  Accent color:     [hex from brand-brain]
  Headline font:    [family, weight, size range]
  Body font:        [family, weight, size range]
Image style note:   [from brand.md style guidance or inferred from ICP]
Variants expected:  [N rows in Bulk Create CSV]
```

Build one brief for each distinct layout (e.g., square social card ≠ story ≠ email header). A campaign rarely needs more than 3–5 distinct layouts; flag scope creep if the count climbs.

### Bulk Create CSV

Canva's Bulk Create feature populates a template from a CSV: each row = one asset variant. Build the CSV spec:

1. List the column headers matching the template's text/image fields exactly (Canva is case-sensitive).
2. Populate sample rows (at minimum 3 representative variants) so the user can validate the mapping before running the full batch.
3. Note any column that maps to an image URL — Canva Bulk Create does NOT auto-resize imported images; flag this and recommend hosting dimensions.
4. Surface the row limit: Canva Bulk Create supports up to 500 rows per run [verify current limit before citing to client].

Output the CSV as a fenced code block (comma-delimited, headers row first) and also offer to save to `./design/[slug]-bulk-create.csv`.

### Magic Resize spec

Enumerate every output format the campaign requires, derived from the surface list in Discover. For each:

| Format | Canvas (W×H) | Key constraint | Resize from |
|---|---|---|---|
| Instagram feed (square) | 1080×1080 | Safe zone: 50px margins | master |
| Instagram story / TikTok | 1080×1920 | Text above midpoint for UI chrome | master |
| Facebook feed | 1200×628 | Text-overlay safe: ≤20% area [verify] | master |
| LinkedIn single image | 1200×628 | Same as FB feed | master |
| Twitter/X card | 1200×675 | Keep CTA above fold on mobile | master |
| Email header | 600×200 | No text in outer 20px (ESP clipping) | master |
| Google Display (leaderboard) | 728×90 | Text legibility at small size | master |
| Google Display (rectangle) | 300×250 | Brand mark must remain legible | master |
| OG / link preview | 1200×630 | Title visible without hover | master |
| Push thumbnail | 360×240 | Image-only safe; no text overlay | master |

Include only the formats the brief actually calls for. Add any nonstandard formats the user specifies. Flag formats where Magic Resize alone won't suffice (e.g., story vs. feed requires layout restructuring, not just scaling) — those need a second template, not a resize.

---

## Figma mode

### Component inventory

When a Figma file URL is provided, produce a structured audit:

```
Component inventory: [file name]
Last updated: [from file metadata if accessible]

Atoms (smallest reusable units)
  ✓ Color styles defined:  [Y/N — list names or flag gap]
  ✓ Text styles defined:   [Y/N — list names or flag gap]
  ✓ Effect styles:         [Y/N]

Molecules (composed components)
  [list each named component found: Button/CTA, Card, Badge, etc.]
  Flag: unnamed / detached instances count: [N]
  Flag: inconsistent instances (same visual, different structure): [list]

Organisms / pages
  [list page names and dominant component types]

Gaps
  [missing states: hover, focus, disabled, mobile breakpoint]
  [missing sizes: SM / MD / LG variants]
  [color or font values not linked to styles — hardcoded values: N found]
```

If only a brief is provided (no file), skip inventory and build the component extraction brief below as a spec for the Figma file to create.

### Component extraction brief

For each gap or new component needed:

```
Component: [name]
Type: [Atom / Molecule / Organism]
States needed: [default, hover, active, disabled, focus]
Sizes needed: [SM | MD | LG or breakpoint-driven]
Props / variants: [list Figma variant properties]
Copy slots: [field name → max chars, placeholder text]
Token bindings:
  Fill:   [color style name]
  Type:   [text style name]
  Radius: [value in px or token name]
  Shadow: [effect style name]
Notes: [responsive behavior, accessibility requirements]
```

### Dev Mode handoff pack

Produce a handoff-ready spec document:

```
## Handoff: [feature / page / component set]
Brand: [slug from brand-brain]

### Design tokens (source of truth)
Primary:    [hex]  →  CSS var: --color-primary
Accent:     [hex]  →  CSS var: --color-accent
Background: [hex]  →  CSS var: --color-bg
Text/body:  [hex]  →  CSS var: --color-text

Typography scale
  Heading 1: [family, weight, size, line-height]
  Heading 2: ...
  Body:      ...
  Caption:   ...

Spacing scale (8px base grid recommended)
  4 / 8 / 12 / 16 / 24 / 32 / 48 / 64

### Component specs (one block per component)
[Component name]
  Props: [auto-layout direction, padding top/right/bottom/left, gap]
  States: [list with visual delta — e.g., hover: opacity 0.9, bg shifts to --color-accent-dark]
  Copy slots: [field → char limit → placeholder]
  Accessibility: [ARIA role, keyboard nav, contrast ratio — must meet WCAG AA minimum]

### Copy manifest
[List every copy slot, the approved final text, max-char constraint]

### Assets
[List image assets with exact filename, dimensions, format, and hosting path]
[Alt text: mark as TO-DO → call alt-text-accessibility-copy-batch-writer]
```

Save the handoff pack to `./design/[slug]-figma-handoff.md` by default.

---

## Asset manifest (all modes)

At the end of every run, output a concise asset manifest:

```
Asset manifest — [campaign slug] — [date]
Brand: [slug]

| # | Asset name | Tool | Template | Format | Size | Status |
|---|---|---|---|---|---|---|
| 1 | [name] | Canva / Figma | [template name] | JPEG/PNG/SVG | [WxH] | Brief ready / CSV ready / Handoff ready |
...

Next steps:
  □ Alt-text strings → call alt-text-accessibility-copy-batch-writer
  □ Brand QA → call brand-consistency-auditor
  □ Dev annotation → call design-qa-handoff-pack-builder (Figma only)
  □ AI-generated imagery → call creative-brief-image-prompt-crafter
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No template, no CSV, no handoff before the active brand's palette and typography are loaded.
- **One master frame.** Every resize derives from a single canonical design. Two source layouts means double the maintenance surface.
- **Compose, don't rebuild.** Infographic specs → `infographic-data-viz-spec-writer`. Image prompts → `creative-brief-image-prompt-crafter`. QA → `brand-consistency-auditor`. Call them; don't duplicate their logic.
- **Real format specs only.** Use real platform dimensions and constraints. Never invent canvas sizes or assert safe-zone percentages without flagging `[verify]` if uncertain.
- **Token-binding before values.** In Figma handoffs, name the design token; don't just paste a hex. One token change → every component updates.
- **Copy slots are contracts.** Every text field in a template or component needs a declared max-char limit. Overflows break layouts silently; surfacing them here prevents production scrambles.
- **Scope to the brief.** Don't generate formats the campaign doesn't need. An asset manifest that's 80% irrelevant is a noise machine.

## What Not to Do

- Don't invent palette values — use only what `brand-brain` returns (or what the user confirms).
- Don't assume Magic Resize handles story-to-feed layout changes — they require a second template brief.
- Don't generate a Bulk Create CSV without confirming the column headers match the template's layer names exactly.
- Don't produce a Figma handoff without a spacing system — unitless handoffs lead to inconsistent implementations.
- Don't list every possible ad format. Enumerate only the surfaces the brief calls for, then stop.
- Don't skip the asset manifest — it's the artifact that keeps the campaign production handoff from becoming a Slack conversation chain.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and palette / typography tokens loaded before any output?
- Every canvas size listed sourced from real platform specs (or flagged `[verify]`)?
- Bulk Create CSV headers exactly match template layer names?
- Magic Resize spec distinguishes true resizes from layout-restructuring needs (second template flagged)?
- Figma handoff includes design tokens, spacing scale, per-component copy slots with char limits, and WCAG AA contrast flag?
- Asset manifest produced with tool, format, size, and status per row?
- Downstream skill calls named: alt-text, brand-consistency-auditor, design-qa-handoff-pack-builder?
- No canvas sizes, color values, or platform constraints invented without `[verify]`?
