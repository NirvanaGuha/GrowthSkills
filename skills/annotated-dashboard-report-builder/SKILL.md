---
name: annotated-dashboard-report-builder
description: >
  Turns a raw dashboard screenshot (GA4, HubSpot, Metabase, Looker Studio, or any BI surface)
  plus a metrics list into a fully annotated report asset — with metric labels, period-over-period
  deltas, color-coded callout boxes, anomaly flags, and plain-English insight captions that tell
  the reader exactly what changed, why it matters, and what to watch next. Produces a structured
  annotation spec (coordinates, label text, color codes, arrow targets) that can be applied via
  any design tool (Figma, Canva, Keynote, PowerPoint) or passed to the screenshot-annotator skill
  for render. Brand voice governs caption tone; only confirmed numbers are stated as facts — anything
  inferred or estimated is flagged with [verify]. Use when the user says "annotate this dashboard,"
  "make this report shareable," "add context to my metrics screenshot," "explain this GA4 screenshot,"
  "add arrows and callouts," "make this exec-ready," "dashboard screenshot report," or hands over
  a BI screenshot and asks for annotations, captions, or a narrative layer.
---

# Annotated Dashboard Report Builder

Raw dashboard screenshots tell you what the numbers are. This skill adds the layer that tells you what they mean — metric labels, period deltas, anomaly callouts, and plain-English captions that a stakeholder can read in 90 seconds without touching the source tool.

The output is a precise **annotation spec** — every callout positioned, labeled, colored, and captioned — plus a standalone **written report block** that narrativizes the same insights in prose. Together they cover both the visual asset (deck slide, async Loom frame, Slack post) and the written update (email, Notion, Confluence).

This skill annotates and narrates. It does not rebuild the dashboard, re-run queries, or reinterpret raw data it has not seen. If the user's numbers are internally inconsistent, it flags rather than smooths.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, color palette, and banned words so caption tone and callout colors are on-brand. Does not build or store brand data itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's primary/accent hex codes, voice adjectives (formal vs. casual; analytical vs. narrative), and any banned phrases before proceeding.
- **`data-qa-measurement-gotcha-checker`** (recommended) — before annotating, run the metrics through a data-quality gate to catch attribution gaps, sampling flags, or known GA4 gotchas (broken page_view, direct inflation, cross-domain bleed) that would make an annotation misleading. Call it when the user provides raw numbers; synthesize the check inline if the skill is absent.
- **`analytics-report-reviewer`** — optional second-pass reviewer for the written report block; flags unsupported claims, weak assumptions, missing context. Call when the user asks for a "reviewed" or "stakeholder-ready" version.
- **`screenshot-annotator-bug-reporter`** — sibling L22 skill that renders the annotation spec into an actual marked-up image. Hand off the annotation spec to it when the user wants a rendered output, not just the spec.
- **`weekly-wins-anomalies-slack-ping`** — optional: if the annotated report surfaces a clear weekly win or anomaly, pass the narrative block to this skill to produce a Slack-ready distillation.
- **`ga4-anomaly-detector`** — call when a metric movement exceeds ±20% WoW or ±15% MoM and the cause is not obvious from the screenshot; its output becomes the anomaly callout text.
- **`kpi-tree-builder`** — reference when annotating a dashboard that the user has not yet mapped to a KPI hierarchy; understanding which metrics are leading vs. lagging determines callout priority.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (voice + palette + banned words)
Step 1  Intake & parse          ──► screenshot + metrics list + comparison period + audience
Step 2  Data-quality gate       ──► call data-qa-measurement-gotcha-checker (or inline check)
Step 3  Apply FCF to each metric ──► Figure → Change → Framing per callout
Step 4  Build annotation spec   ──► callout map with coords, colors, labels, captions
Step 5  Write the report block  ──► Martini Glass narrative (context → focus → action)
Step 6  Self-review + deliver   ──► quality checklist, then output
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand digest: voice adjectives, banned words, palette hex codes, ICP, and the path to `brand.md`. Use the palette to assign callout colors (positive delta → brand primary or green-family; negative / anomaly → red-family; neutral context → brand neutral or grey). Obey voice adjectives in caption prose — a brand that is "direct, no-fluff" gets one-line captions; a brand that is "educational, warm" gets a sentence of context behind each number.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's primary/accent hex codes, voice adjectives (formal vs. casual; analytical vs. narrative), and any banned phrases before proceeding.

Do not produce any annotation spec or captions until brand context is resolved.

---

## Step 1 — Intake & parse

Collect the following before proceeding. Ask in one batch if anything is missing:

| Input | Why it's needed |
|---|---|
| Dashboard screenshot(s) | The visual surface being annotated |
| Metrics list | Which numbers to call out (if not obvious from the screenshot) |
| Comparison period | Current vs. prior period (WoW / MoM / QoQ / YoY) — no delta without this |
| Audience | Exec / analyst / channel team — sets callout density and caption depth |
| Platform hint | GA4 / HubSpot / Metabase / Looker / other — activates known gotcha checks |
| Desired output | Annotation spec only / rendered image / written report block / all three |

If the user provides a screenshot but no metrics list, infer visible KPI tiles and ask for confirmation before proceeding.

---

## Step 2 — Data-quality gate

Before annotating, run a lightweight quality pass. **Call `data-qa-measurement-gotcha-checker`** if available. Inline otherwise:

- **Platform-specific gotchas:** GA4 — broken `page_view` event inflation, (direct)/(none) attribution sink, cross-domain leakage, sampling in Explorations. HubSpot — deal-stage backdating, contact deduplication lag. Metabase — timezone mismatch between DB and dashboard filter.
- **Internal consistency:** Does the sum of segment rows equal the total row? Do period-over-period deltas match the stated comparison window?
- **Confirmed vs. estimated:** Any number inferred from partial data gets `[verify]` in its annotation caption. Never state an inferred delta as fact.

Flag quality issues in a pre-annotation note. Do not suppress them; do not refuse to annotate — flag and continue.

---

## Step 3 — FCF callout pattern (our working annotation model)

Every individual callout follows **Figure → Change → Framing** (FCF is a house mnemonic, not an external standard):

- **Figure** — the metric name and its current value, stated precisely. ("Sessions: 42,300")
- **Change** — the period delta, directional arrow, and percent. ("↑ 18% MoM")
- **Framing** — one clause of plain-English context that tells the reader what drove it or why it matters, using only what is confirmable from the screenshot or the user's stated context. ("Driven by paid campaign launch Jun 10 — organic held flat.")

Framing is the differentiator. Without it, the annotation is just a label. With it, the stakeholder can act. If the driver is unknown, say so explicitly: "Driver unclear — check channel breakdown." Never invent a cause.

**Callout priority tiers** — annotate in this order; high-density audiences (execs) get Tier 1 only; analyst/channel audiences get all three:

| Tier | Callout type | Example |
|---|---|---|
| 1 — Anomaly | Movement ≥ ±20% WoW or ≥ ±15% MoM, or a metric at all-time high/low | "Sessions ↓ 34% — investigate" |
| 2 — Primary KPI | North-star or campaign-objective metric (e.g., conversions, MRR, signups) | "Trials ↑ 9% MoM — above target" |
| 3 — Supporting | Metrics that explain or contextualize Tier 1–2 (bounce rate, source split, etc.) | "Organic share 61%, +4pp" |

Cap at 7 callouts per screenshot. Beyond that, context collapses into noise. If more than 7 metrics need annotation, produce a second annotated view or a supplemental table.

---

## Step 4 — Build the annotation spec

Produce a structured callout map the user (or `screenshot-annotator-bug-reporter`) can act on immediately:

```markdown
## Annotation spec — [Dashboard name / date range]
Brand: [slug] | Palette: primary [hex] / positive [hex] / negative [hex] / neutral [hex]
Audience: [exec / analyst / team] | Density: [Tier 1 only / Tiers 1–2 / All tiers]

### Callout 1 — [Metric name]
- Target element: [top-left tile / chart title / data point at approx. x% from left, y% from top]
- Box color: [hex] | Arrow: [direction + target description]
- Label: [Figure] [Delta arrow + %]
- Caption: [Framing sentence — max 12 words]
- Tier: [1 Anomaly / 2 Primary KPI / 3 Supporting]
- Data-quality flag: [none / [verify] reason]

### Callout 2 — ...
```

Repeat per callout. If a metric has a `[verify]` flag, include it in the caption verbatim — do not silently drop it.

---

## Step 5 — Write the report block (Martini Glass)

The written report block follows the **Martini Glass** narrative structure — standard in data journalism and executive reporting:

1. **Wide opening (context):** One sentence establishing the period, the brand/property, and the headline number. Sets the frame for a reader who has not seen the dashboard.
2. **Narrowing body (the story):** 2–4 bullets, each covering one annotated metric using FCF order. Only Tier 1 and Tier 2 callouts in the body; Tier 3 supporting detail in a sub-bullet or footnote.
3. **Pointed close (action):** One sentence stating what to watch, test, or decide next. Specific and actionable — not "continue to monitor."

```markdown
## [Brand] — [Dashboard name] | [Period]

[Context sentence: period, property, headline number.]

- **[Metric]:** [Current value] — [delta] [direction]. [Framing clause.]
- **[Metric]:** ...
- **[Metric]:** ...

**Watch:** [One specific next action or watch item.]
```

Tone obeys the brand voice from Step 0. Length: 80–150 words for exec audience; up to 250 words for analyst/team.

---

## Output format

Deliver in this order:
1. **Pre-annotation data-quality note** (if any flags exist — else omit)
2. **Annotation spec** (Step 4 structured callout map)
3. **Written report block** (Step 5 Martini Glass narrative)
4. **Handoff note** — one line: "Pass the annotation spec to `screenshot-annotator-bug-reporter` to render the marked-up image."

Save to `./reports/[brand-slug]-[dashboard-slug]-[YYYY-MM-DD].md` if the user asks for a saved artifact.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No annotations or captions before brand context is resolved. Color and voice are not optional.
- **FCF on every callout.** Figure + Change + Framing. A label without framing is noise.
- **Seven-callout ceiling.** Density kills comprehension. Tier and cut ruthlessly.
- **Martini Glass for prose.** Wide → narrow → point. Never a flat list of metric descriptions.
- **Confirmed or flagged.** Only state causation you can confirm from the data in view. Inferences get `[verify]`; unknown drivers are called out as unknown.
- **Data-quality gate is non-negotiable.** Platform gotchas exist; annotating corrupted data is worse than not annotating.

## What Not to Do

- Don't produce callouts or captions before `brand-brain` returns brand context.
- Don't state a cause you cannot confirm — "likely seasonal" is an opinion; flag it or ask.
- Don't annotate more than 7 items on one screenshot — route overflow to a second view or table.
- Don't drop `[verify]` flags from captions; they exist to protect the annotator and the reader.
- Don't reimplement brand scanning, data-quality logic, or anomaly detection — call the relevant siblings.
- Don't confuse annotation density with thoroughness. An exec-tier view has fewer callouts, not worse analysis.
- Don't present a negative delta as neutral to soften the message. Name it accurately; let the framing provide context.

## Quality Checklist (self-review before delivering)

- `brand-brain` called; palette hex codes and voice adjectives applied to callout colors and caption tone?
- Data-quality gate run; `[verify]` flags present on any inferred or estimated numbers?
- Every callout follows FCF: Figure + Change + Framing — no label-only callouts?
- Callout count ≤ 7; prioritized by tier (Anomaly → Primary KPI → Supporting)?
- Annotation spec includes: target element position, box color, arrow, label, caption, tier, quality flag?
- Written report block follows Martini Glass: wide context → narrowing body → pointed close?
- Report block length matches audience (≤150 words exec / ≤250 analyst)?
- Handoff note to `screenshot-annotator-bug-reporter` included?
- No invented causation; no suppressed quality flags; no brand-voice violations?
