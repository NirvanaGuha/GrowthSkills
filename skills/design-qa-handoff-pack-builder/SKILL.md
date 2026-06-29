---
name: design-qa-handoff-pack-builder
description: >
  Takes approved designs, a launch checklist, and a copy doc — then produces a pass/fail QA report,
  mismatch flags, and a complete handoff pack with asset manifest and dev annotations, so nothing
  ships with wrong copy, off-brand color, or missing specs. Two modes: QA-only (review without a
  handoff pack) and Full Handoff (QA report + annotated asset manifest + dev-ready spec sheet).
  Calls brand-brain to load voice, color, font, and banned-word constraints before a single check
  runs, and calls brand-consistency-auditor when a full visual sweep is needed. Use whenever the
  user says "QA this design," "design review before dev," "handoff pack," "check the designs against
  copy," "pre-launch design check," "dev annotations," or "asset manifest." This skill reviews and
  annotates — it does not produce new designs or rewrite copy from scratch.
---

# Design QA & Handoff Pack Builder

Designs approved in Figma aren't necessarily ready for dev. The gap between "looks good" and "ships correctly" is where wrong CTAs, off-brand colors, mismatched copy, and missing hover-state specs live. This skill closes that gap: it runs a structured QA pass against the brand and the brief, flags every mismatch, and hands off a pack the developer can build from without guessing.

Two modes:
- **QA-only** (default) — pass/fail report, mismatch list, and a recommended fix list. Fast.
- **Full Handoff** — QA report plus an annotated asset manifest (every file, its status, its export spec) and a dev annotation sheet (spacing, states, breakpoints, copy). Triggered by "handoff pack," "ready for dev," "asset manifest," or "full handoff."

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, color tokens, typography, banned words, and real proof. No QA check runs before it returns.
- **`brand-consistency-auditor`** (call when doing Full Handoff or when the user provides screenshots/Figma exports) — deep visual sweep for off-brand color, font, logo, and tone violations. Fold its flagged report into the QA report rather than duplicating the check.
- **`alt-text-accessibility-copy-batch-writer`** (call when the asset manifest needs WCAG-compliant alt-text for every image/icon). Synthesize inline when absent.
- **`cta-variant-generator`** (optional) — if a CTA fails QA, call it to generate a replacement on-brand option rather than leaving the designer with a void.

---

## How a run works

```
Step 0  Load the brand  ──► brand-brain (always first)
Step 1  Ingest inputs   ──► designs + copy doc + checklist
Step 2  Pick the mode   ──► QA-only | Full Handoff
Step 3  Run the QA framework (5 gates)
Step 4  Build outputs   ──► report (both modes) + pack (Full Handoff)
Step 5  Self-review, then present
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before touching any design. It returns: color tokens (hex), typography stack, voice adjectives, banned words, offer mechanics, positioning line, and the path to `brand.md`. Use the color tokens as the QA pass/fail reference. If brand-brain is absent, read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install brand-brain or confirm the hex palette, font stack, and banned words inline before proceeding.

### Step 1 — Ingest inputs

Accept any combination of:
- Design files: Figma share link, exported PNGs/JPGs, PDF, or a written description of the screens
- Copy doc: Google Doc, Notion page, pasted text, or a prior skill output (e.g., `landing-product-page-copy-writer` output)
- Launch checklist: from `campaign-qa-launch-checklist-generator` or a user-written list
- Dev spec: spacing system, breakpoints, state requirements (if provided)

If no copy doc is provided, QA copy against the brand's offer and messaging from `brand.md` and flag anything unconfirmed as `[verify against copy doc]`.

### Step 2 — Pick the mode

- **QA-only (default):** triggered by "review," "check," "QA," or no explicit handoff ask.
- **Full Handoff:** triggered by "handoff," "asset manifest," "dev-ready," "dev annotations," or "full pack."

When ambiguous, default to QA-only and offer Full Handoff at the end.

---

## The QA Framework — 5 Gates (our purpose-built QA rubric)

Run every gate in order. Each item is pass / fail / needs-clarification. Fail = block; needs-clarification = hold with a specific question.

### Gate 1 — Copy Fidelity
Verify every text element in the design against the approved copy doc:
- Headline matches exactly (no paraphrasing by a designer who "improved" it)
- CTA label matches the approved CTA (check against `cta-variant-generator` output if present)
- Body copy, disclaimers, and micro-copy are verbatim
- Banned words from `brand.md` are absent
- Real proof claims match `brand.md` proof; unconfirmed claims are flagged `[verify]`
- Character limits respected per placement (hero headline, push, email button, etc.)

### Gate 2 — Visual Brand Compliance
Reference the brand's hex tokens, typography stack, and logo usage rules from `brand-brain`. Delegate to `brand-consistency-auditor` when visual files are available:
- Primary/secondary/accent colors within ±2% of brand hex (or exact)
- Font family and weight matches brand typography
- Logo size, clear space, and usage rules followed
- Icon style consistent with design system (outline vs filled, stroke weight)
- No gradient or shadow effects not sanctioned by the brand

### Gate 3 — Copy–Design Message Match
The visual hierarchy must reinforce the copy's persuasion intent:
- Headline is visually dominant; not competing with a subhead or image caption
- CTA button is the most visually prominent action; no competing calls to action at same visual weight
- Social proof / trust signals are placed at the right funnel position (not buried below the fold on a high-intent page)
- Awareness stage of the page matches the commitment level of the CTA (Schwartz awareness ladder)
- Any urgency or scarcity element is accurate (not manufactured) and time-boxed `[verify]`

### Gate 4 — Completeness & States
Flag missing deliverables before dev starts:
- Mobile breakpoint provided (at minimum 375px and 768px)
- Interactive states specified: hover, focus, active, disabled, loading, error
- Empty states and zero-data states included for dashboards or dynamic content
- Form validation error messages written (not placeholder "[error text here]")
- Dark mode variant (if the brand ships one)
- All assets exported at correct resolution: 1×, 2× (Retina), and any platform-specific formats (SVG for icons, WebP for hero images, etc.)

### Gate 5 — Accessibility Baseline (WCAG 2.1 AA)
Check against published contrast ratios [verify exact values per WCAG 2.1 spec]:
- Text on background: ≥4.5:1 for body, ≥3:1 for large text (18pt+ or 14pt bold)
- Interactive elements (buttons, links): ≥3:1 against adjacent colors
- Focus indicator visible and at sufficient contrast
- Alt-text spec present for every non-decorative image (call `alt-text-accessibility-copy-batch-writer` to generate if absent)
- No meaning conveyed by color alone

---

## Output formats

### QA-only output

```
## Design QA Report — [project / screen name]
Brand: [slug, via brand-brain]
Date: [today]
Mode: QA-only

### Summary
[X] Pass   [Y] Fail   [Z] Needs clarification

### Gate results
| Gate | Item | Status | Notes / Fix |
|------|------|--------|-------------|
| 1 – Copy Fidelity | Headline matches copy doc | FAIL | Design says "…"; approved copy says "…" |
| 2 – Visual Brand   | Hero background hex      | PASS | #191A35 ✓ |
...

### Blocked items (all FAILs)
1. [Gate] [Item] — [what's wrong] → [exact fix required]

### Clarifications needed
1. [Gate] [Item] — [specific question for designer/PM]

### Recommended next step
[One-sentence directive: fix the blocked items, re-submit for a re-check, or proceed if zero FAILs]
```

### Full Handoff output

Everything in QA-only, plus:

**Asset Manifest**
```
## Asset Manifest — [project]
| Asset name | Format | Size | Export spec | Status | Notes |
|------------|--------|------|-------------|--------|-------|
| hero-desktop | PNG / WebP | 1440×900 | 2× Retina | READY | |
| hero-mobile  | PNG / WebP | 375×667  | 2× Retina | MISSING | Needs mobile crop |
| logo-light   | SVG       | —        | vector    | READY | |
...
```

**Dev Annotation Sheet**
```
## Dev Annotations — [project]

### Spacing & layout
- Outer margin: 24px (mobile), 80px (desktop)
- Section gap: 64px
- Card padding: 24px internal

### Typography
- H1: [font family] [weight] [size desktop] / [size mobile], line-height [x]
- Body: [specs]
- [etc.]

### Breakpoints
- Mobile: 375px | Tablet: 768px | Desktop: 1280px | Wide: 1440px

### Interactive states (by component)
| Component | Hover | Focus | Active | Disabled |
|-----------|-------|-------|--------|---------|

### Copy strings (final, verbatim)
[Each text element from the design with its final approved string]

### Alt-text (from alt-text-accessibility-copy-batch-writer or inline)
[Every non-decorative image with its WCAG-compliant alt string]
```

Save Full Handoff output to `./design-handoff/[project-slug]-handoff.md`. QA-only output is inline.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No QA gate runs before the brand is loaded. Color and copy constraints are pulled from the brand, not assumed.
- **Fail means block.** A FAIL item does not ship. Offer a specific fix; never soften a failure into a "suggestion."
- **Copy verbatim.** A designer "cleaning up" headline copy is a copy mismatch. Flag it, even if the paraphrase is objectively better.
- **Compose, don't re-derive.** Call `brand-consistency-auditor` for visual checks; call `alt-text-accessibility-copy-batch-writer` for alt-text; call `cta-variant-generator` when a CTA needs replacing. Do not reimplement those skills here.
- **WCAG is a floor, not optional.** Contrast failures are blocks, not notes.
- **Flag first, then fix.** The QA report documents findings; replacements are offered as options, not silent substitutions.
- **Unconfirmed proof = `[verify]`.** If a design shows a stat not in `brand.md`, flag it.

## What Not to Do

- Don't load brand context without calling `brand-brain` (or falling back per the documented path).
- Don't run Full Handoff if inputs are descriptions only — note that visual files are needed for Gate 2 and ask.
- Don't mark contrast as passing without checking the actual hex against WCAG ratios.
- Don't silently rewrite copy in the handoff — flag mismatches and reference the correct string; let the human approve the swap.
- Don't generate a new design, Figma component, or full copy block — this skill reviews and annotates.
- Don't produce a QA report without a "Recommended next step" line — always close with a clear directive.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded before any gate ran?
- All 5 gates checked, each item explicitly pass / fail / needs-clarification?
- Every FAIL has a specific fix (not "fix the color" — "change background from #1A1A2E to #191A35")?
- `brand-consistency-auditor` called (or noted as unavailable) for visual file inputs?
- Full Handoff: asset manifest covers every file, dev annotation sheet covers spacing/type/states/copy/alt-text?
- Output saved to `./design-handoff/[project-slug]-handoff.md` (Full Handoff mode)?
- Recommended next step written at the end?
