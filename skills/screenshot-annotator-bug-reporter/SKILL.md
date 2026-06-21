---
name: screenshot-annotator-bug-reporter
description: >
  Takes raw screenshots and either an annotation brief or bug repro steps and produces two things:
  (1) a render-ready annotation spec — a structured list of every arrow, callout box, numbered badge,
  highlight region, and blur zone with coordinates, colors, and copy — that any image editor, Figma
  annotation plugin, or Claude in Chrome can execute immediately; (2) a structured bug report in the
  5-field engineering standard (Title / Steps to Reproduce / Expected / Actual / Environment) plus a
  severity classification and a recommended assignee category. Handles single screenshots or a
  multi-frame sequence (repro flows, before/after comparisons). Applies brand colors to annotation
  chrome when a brand brain is active. Composes with design-qa-handoff-pack-builder for launch-gate
  contexts and brand-consistency-auditor when the screenshot itself is a brand asset under review.
  Use whenever the user says "annotate this screenshot," "mark up this image," "write a bug report,"
  "document this issue," "show what's broken," "add callouts," "blur the PII," "create a repro," or
  hands over a screenshot with a description of a problem or highlight.
---

# Screenshot Annotator & Bug Reporter

Hand it a screenshot and a brief. Get back a render-ready annotation spec every editor can execute, plus a filing-ready structured bug report. Nothing generic — every callout has coordinates, every bug report has a reproducible step sequence and a severity verdict.

This skill annotates and documents. It does not redesign the UI, write copy for the page in the screenshot, or manage brand assets. When the screenshot is a brand asset being reviewed for consistency, it calls `brand-consistency-auditor` for that layer.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's color palette, voice, and any banned visual treatments, so annotation chrome (arrow color, callout border, badge fill) matches the brand. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their primary brand hex color and callout style preference (filled badge vs. outline) before proceeding.
- *(optional, when installed)* **`design-qa-handoff-pack-builder`** — when the annotation is part of a pre-launch QA pass, pass the annotation spec to it to generate a full handoff pack with asset manifest and dev fix list.
- *(optional, when installed)* **`brand-consistency-auditor`** — when the screenshot is a brand asset (ad, email render, landing page) being reviewed for off-brand treatments, compose it here rather than reimplementing the brand-check logic.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (palette, voice, style)
Step 1  Classify the job    ──► Annotate only | Bug report only | Both
Step 2  Parse the input     ──► screenshot(s) + brief/repro steps
Step 3  Apply DACS          ──► Describe → Annotate → Classify → Specify
Step 4  Output artifacts    ──► annotation spec + bug report (as applicable)
Step 5  Save to ./annotations/
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned palette for annotation chrome: primary brand color for callout borders and arrow strokes, a high-contrast fill (white or brand-light) for badge backgrounds, a semi-transparent brand-dark overlay for blur zones. If no brand is active, default to `#E5484D` (red) arrows/badges, `#FFFFFF` badge fill, `#000000` at 60% opacity for blur.

**Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their primary brand hex color and callout style preference (filled badge vs. outline) before proceeding.**

### Step 1 — Classify the job

| Input received | Mode |
|---|---|
| Screenshot + annotation brief (what to highlight/explain) | Annotation spec only |
| Screenshot + repro steps or "it broke when..." | Annotation spec + structured bug report |
| Repro steps, no screenshot | Bug report only (no annotation spec) |
| Before/after pair | Two-frame annotation spec with delta callouts |

When mode is ambiguous, produce both and let the user drop what they don't need.

---

## The DACS framework

**Describe → Annotate → Classify → Specify** — four passes that turn a raw screenshot and a brief into a complete, actionable annotation spec.

### D — Describe (understand the frame before marking it)

Read the screenshot carefully. Note:
- **Subject:** what page / UI state / error is shown
- **Focal elements:** the components relevant to the brief or repro
- **PII / sensitive data present:** personal emails, names, API keys, financial figures, any identifying tokens — flag every instance; every flagged element becomes a mandatory blur zone
- **Frame dimensions:** estimate or accept user-provided pixel dimensions; used to anchor coordinates

Do not place any annotation until Describe is complete.

### A — Annotate (decide what goes where)

Map each annotation element to a focal item. Every element gets:

| Field | Values |
|---|---|
| `id` | Sequential integer (A1, A2 …) |
| `type` | `arrow` / `badge` / `callout-box` / `highlight-rect` / `blur-zone` / `crosshair` / `redline` |
| `region` | `x, y, w, h` in pixels from top-left (or `x%, y%` if dimensions unknown) |
| `label` | Short copy for the annotation (≤8 words); blank for blur/highlight |
| `color` | Hex (from brand palette or defaults) |
| `note` | Optional longer explanation for the spec reader — does not appear on the rendered image |

Rules:
- Arrows point TO the element, not away. Arrow tail is the label, arrowhead is the target.
- Callout boxes sit outside the focal region with a leader line; never occlude the element they describe.
- Badge numbers (1, 2, 3) are used for sequential steps in a repro flow; letter badges (A, B) for parallel callouts in a comparison.
- Blur zones cover PII fully with enough margin (+16px each side) to prevent context leakage. Use a filled rectangle at 100% opacity (not a gaussian-blur spec — tools vary; spec solid overlay for portability).
- Highlight rectangles use 30% opacity fill; callout boxes use 100% border + 0% fill.

### C — Classify (bug severity + annotation density)

**Annotation density check:** if the spec contains more than 7 annotation elements on a single frame, flag it. Dense annotation obscures rather than clarifies. Recommend splitting into two frames or reducing to the 5 most load-bearing callouts.

**Bug severity classification (when a bug report is being produced):**

| Severity | Definition | Examples |
|---|---|---|
| **P0 — Critical** | Data loss, security exposure, payment failure, complete flow block | API key exposed, checkout 500s, login loop |
| **P1 — High** | Core flow broken for a significant user segment; no workaround | CTA button non-functional on mobile, form submit silently drops data |
| **P2 — Medium** | Feature partially broken; workaround exists | Wrong price shown on plan comparison, tooltip misfire |
| **P3 — Low** | Visual / cosmetic; no functional impact | Misaligned label, wrong font weight, icon off by 2px |

Classify based on the repro steps and the user's described impact. State the rationale in one sentence.

### S — Specify (write the full output)

Produce the annotation spec table and the bug report in the formats below.

---

## Output formats

### Annotation spec

```
## Annotation spec — [screenshot filename or description]
Brand: [slug] | Dimensions: [w × h px or "estimated"]
Generated: [date]

| ID  | Type          | Region (x,y,w,h px) | Label                  | Color   | Note                              |
|-----|---------------|---------------------|------------------------|---------|-----------------------------------|
| A1  | badge         | 412,88,24,24        | 1                      | #E5484D | Step 1 in repro — click here      |
| A2  | arrow         | 460,92,80,0         | Points to submit btn   | #E5484D | Arrow tail at A1, head at button  |
| A3  | callout-box   | 20,140,220,60       | Missing required field | #E5484D | Leader line to field label        |
| A4  | blur-zone     | 0,320,540,40        | [PII — email address]  | #000000 | 100% opacity fill; +16px margin   |

### Render instructions
Tool: [Figma / Canva / Sketch / Preview / any raster editor]
Layer order (bottom → top): original screenshot → blur zones → highlights → callout boxes → arrows → badges
Export: PNG, 2× if retina source; JPEG acceptable for non-PII frames
```

### Structured bug report

```
## Bug report — [short title, ≤10 words]

**Severity:** P[0–3] — [one-line rationale]
**Assignee category:** [Frontend / Backend / Design / DevOps / Data / PM decision]
**Reporter:** [user name or "unspecified"]
**Date filed:** [today]
**Status:** Open

### Steps to reproduce
1. [Precise action — include URL, user state, browser/device if relevant]
2. …
3. …

### Expected behavior
[One sentence: what should happen at the final step above]

### Actual behavior
[One sentence: what actually happens; reference the annotated screenshot by filename]

### Environment
- Browser / app version: [specify or "[verify]"]
- OS: [specify or "[verify]"]
- User state: [logged in / guest / specific plan tier / etc.]
- Reproducibility: [Always / Intermittent (X/Y attempts) / Once]

### Evidence
- Screenshot: [filename or inline]
- Annotation spec: see above / [path to saved spec]
- Console errors: [paste or "none observed"]
- Network request (if relevant): [endpoint + status code or "not captured"]

### Suggested fix (optional)
[One sentence hypothesis — only include if clearly evident from the screenshot; else omit]
```

---

## Multi-frame and before/after handling

For repro sequences (3+ frames showing the flow to the bug):
1. Number frames F1, F2, F3 … in sequence.
2. Annotation IDs restart per frame (F1-A1, F1-A2 … F2-A1 …).
3. Include a **transition note** between frames: "After F1 — user clicks button → F2."
4. The bug report's Steps to Reproduce references frame numbers: "Step 2 (see F2-A1)."

For before/after comparisons:
- Frame B = before, Frame A = after.
- Delta callouts mark what changed; use `highlight-rect` (green tint `#30A46C` at 30% for additions, red tint `#E5484D` at 30% for removals).

---

## Persistence

Save the full output (annotation spec + bug report) to:

```
./annotations/[brand-slug]-[short-descriptor]-[YYYYMMDD].md
```

Confirm the save path in one line after outputting. Never overwrite an existing file at the same path — append a `-v2` suffix instead. Never write brand data to brand.md.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Load brand context before choosing annotation colors or chrome style.
- **PII is a mandatory blur.** Any personal data visible in a screenshot gets a blur-zone entry — it is never optional. Flag it, spec it, note it.
- **Coordinates over vague descriptions.** "Arrow pointing roughly at the button" is not a spec. Give pixel coordinates; estimate if dimensions are unknown and flag the estimate.
- **Severity is a judgment call, not a formula.** State the rationale in one sentence; don't hide behind a rubric.
- **Density cap at 7.** More callouts than 7 per frame degrades comprehension. Split or prune.
- **Honest evidence only.** If console errors weren't captured, say so — don't invent them.
- **Spec, don't redesign.** Annotation documents the issue; it does not propose a new UI. If a fix is obvious, one sentence in "Suggested fix" — nothing more.

## What Not to Do

- Don't start annotating before `brand-brain` returns (or the fallback completes).
- Don't use annotation to redesign or critique the underlying UI — that's `design-qa-handoff-pack-builder` or `landing-page-heuristic-live-cro-auditor`.
- Don't invent coordinates from memory — estimate from proportional reasoning and flag as estimated if the user hasn't provided dimensions.
- Don't omit blur zones for PII even if the user doesn't mention it — you spotted it, you spec it.
- Don't produce a bug report without a severity classification and a reproducibility rating.
- Don't annotate beyond 7 elements without flagging density and offering to split.
- Don't write brand.md or modify any brand data file.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and palette loaded (or fallback completed)?
- Every annotation element has an ID, type, region coordinates, label, and color?
- PII detected → blur-zone entries present for every instance, with +16px margin?
- Annotation density ≤ 7 per frame, or split/prune recommendation included?
- Bug report has all 5 required fields (Title, Steps, Expected, Actual, Environment)?
- Severity classified with one-line rationale; reproducibility rated?
- Multi-frame: frame numbers consistent; transition notes between frames?
- Output saved to `./annotations/[brand-slug]-[descriptor]-[YYYYMMDD].md` and path confirmed?
- No brand data written to brand.md; no UI redesign embedded in the spec?
