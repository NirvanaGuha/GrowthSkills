---
name: creative-feedback-translator
description: >
  Takes vague or emotionally-charged stakeholder feedback — "make it pop," "it feels flat,"
  "this doesn't look premium," "I don't like the vibe" — and translates it into specific,
  actionable design direction covering contrast, visual hierarchy, typography, color, spacing,
  copy adjustments, and layout rationale. Two modes: Translate (default) converts raw feedback
  into a brief a designer or AI image tool can act on immediately; Diagnose surfaces what the
  stakeholder actually means by mapping non-specific language onto real design principles,
  then pairs each diagnosis with concrete fixes. Works for static designs, landing pages, ads,
  social graphics, email templates, and decks. Loads active brand context via `brand-brain`
  so every direction stays on-visual-identity and voice. Use when a stakeholder says "it needs
  more energy," "make it feel modern," "something is off," "I want it to feel more premium,"
  "can you punch it up," "the design isn't landing," or hands you feedback notes from a client
  review. Does not redesign or produce final assets — it translates vague language into a
  precise direction brief that a designer, `ai-image-generator-on-brand-assets`, or
  `landing-page-builder-html-tailwind` can execute without a follow-up call.
---

# Creative Feedback Translator

Stakeholders feel things before they can name them. This skill bridges the gap — converting emotional, vague, or contradictory creative feedback into a precise design direction brief a designer or AI tool can execute without a back-and-forth loop.

The framework underneath is **Ogilvian Diagnosis**: every piece of vague feedback maps to one or more design-principle failures (hierarchy, contrast, whitespace, typographic rhythm, color temperature, visual tension, copy-design alignment). Name the principle; prescribe the fix; stay on brand.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, visual identity (colors, type, tone), ICP, and positioning. Visual identity is the anchor; all direction must stay within it unless the brief explicitly requests a departure.
- *(optional, compose when installed)* `ai-image-generator-on-brand-assets` — pass the translated brief directly for immediate asset generation. `landing-page-builder-html-tailwind` — pass hierarchy + spacing adjustments for coded implementation. `cta-variant-generator` — when copy adjustments are in scope and the CTA is one of the flagged elements.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain; extract visual identity + voice
Step 1  Pick the mode         ──► Translate (default) | Diagnose (on request)
Step 2  Map feedback → principles
Step 3  Write the direction brief
Step 4  Self-review; offer downstream handoff
```

### Step 0 — Load the brand (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`) before producing any direction. It returns the active brand's visual identity (colors, type stack, tone adjectives, banned words), ICP, and positioning. Every fix you prescribe must stay inside that identity — if the feedback asks for something outside it (e.g., "use a red CTA" and the brand forbids red), flag it as an **identity conflict** and offer an on-brand alternative instead.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and the brand's `brand.md`; if none exists, ask for: (1) primary and accent colors, (2) typeface stack, (3) 3 voice adjectives, (4) one "must-never-look-like" competitor. Proceed once you have these. Always prefer the call.

---

## Translate mode (default)

The everyday job. Feedback in → direction brief out. No open questions left for the designer.

1. **Collect the feedback.** Take everything the stakeholder said verbatim, even if contradictory.
2. **Sort by element.** Group comments by the design element they most likely refer to: Hierarchy, Contrast, Color & Temperature, Typography, Whitespace & Rhythm, Copy & Microcopy, Motion/Animation (if applicable). If a comment could map to multiple elements, note all of them.
3. **Apply the Translation Table** (see below). Map each phrase to the principle and the default remedy.
4. **Reconcile conflicts.** If two comments pull in opposite directions (e.g., "feels cluttered" + "feels empty"), surface the contradiction and propose a resolution with a rationale.
5. **Write the Direction Brief** (see format below). Every line is a prescription, not a wish.
6. **Flag identity conflicts.** Anything that contradicts the loaded brand identity gets a ⚠ tag and an on-brand alternative.

**Output format:**

```
## Direction Brief — [Design Asset Name / URL]
Brand: [slug, via brand-brain]
Feedback received: [verbatim, compressed]
Visual identity anchor: [colors + type + tone adjectives in one line]

### Hierarchy & Layout
- [specific fix, e.g. "Increase H1 size from ~36px to ≥52px; reduce body weight to 300 to widen the contrast gap"]

### Contrast & Color
- [specific fix, e.g. "CTA button: switch fill to [brand accent] #3B43FF on white, min 4.5:1 ratio; remove the current gray border"]

### Typography
- [specific fix, e.g. "Tighten leading on the subhead to 1.2 line-height; add 0.04em letter-spacing to the eyebrow tag"]

### Whitespace & Rhythm
- [specific fix]

### Copy & Microcopy
- [specific fix, e.g. "Replace 'Solutions for growing businesses' with a benefit-led line — suggest: 'Turn site visitors into subscribers in one click'"]

### Identity conflicts
- ⚠ [what was asked vs. why it breaks brand + on-brand alternative]

### Recommended next step
[One line: e.g., "Pass this brief to ai-image-generator-on-brand-assets with the social variant flag, or share with the designer as a single source of truth."]
```

Save to `./creative/feedback-[asset-slug]-[date].md` if a project is active; otherwise present inline.

---

## Diagnose mode (on request)

Triggered by: "explain what they mean," "I don't understand the feedback," "help me decode this," "what's really wrong with it."

Go one layer deeper: before prescribing, name the underlying principle the stakeholder is reacting to and explain *why* the design is triggering that reaction. Then give the fix.

**Format per feedback item:**

```
Feedback: "[exact phrase]"
Principle: [the real design law being violated]
Why it triggers this: [1–2 sentences]
Fix: [specific, actionable]
```

Use Diagnose when the team needs to understand the root cause so they stop repeating the same mistake, not just fix this instance.

---

## Translation Table (core mapping)

| Vague phrase | Most likely principle | Default remedy |
|---|---|---|
| "Make it pop" | Contrast / focal hierarchy | Increase contrast ratio on the primary element; reduce visual weight on everything else |
| "It feels flat" | Depth / contrast / typographic rhythm | Add layering (shadow, gradient, stagger), increase type-weight contrast between levels |
| "Doesn't look premium" | Whitespace / typography / density | Add generous negative space; tighten typographic scale; reduce clutter by 30%+ |
| "Feels busy / cluttered" | Visual noise / hierarchy | Remove or de-emphasize 1–2 secondary elements; establish a clear Z or F scan path |
| "Feels empty / cold" | Warmth / density / image presence | Add a human image or warm brand color accent; tighten spacing between elements |
| "Not on brand" | Visual identity / voice misalignment | Audit every element against brand colors, type, and tone; replace OOB items |
| "Needs more energy" | Tension / motion / saturation | Increase color saturation on accent; tighten leading; use a diagonal or asymmetric layout |
| "The CTA isn't landing" | CTA contrast + copy weak | Boost CTA button contrast; rewrite label — lead with value, not a generic verb |
| "I don't like the vibe" | Tone / color temperature / image choice | Map to the ICP's emotional register; adjust image, copy tone, and color temperature together |
| "Looks cheap" | Typeface + color + spacing quality | Audit font weights for consistency; ensure colors use brand palette only; add breathing room |
| "Too corporate / stiff" | Voice / imagery / geometric rigidity | Soften corners, add organic shapes or candid imagery, loosen copy tone |
| "Feels outdated" | Layout conventions + icon style | Shift to modern grid (12-col fluid), update icon style to outlined or filled-not-stroked |

Add rows inline when new feedback phrases appear. Every novel phrase is reusable pattern capital.

---

## Principles

- **Brand-brain first.** No direction before the visual identity loads. Identity is the constraint, not the decoration.
- **Specificity is the deliverable.** Vague feedback in → specific brief out. "More whitespace" is not a fix; "increase padding above the H2 from 24px to 48px" is.
- **Name the principle, not just the fix.** Designers who understand *why* stop re-breaking the same thing.
- **Respect the contradiction.** Stakeholders often say contradictory things. Surface it; don't silently pick one side.
- **Copy is part of design.** If the feedback is about visual feel but the real problem is a misaligned headline, say so. Design-copy alignment is in scope.
- **One direction brief, zero open questions.** The output is a single-source-of-truth brief that can go straight to execution.

## What Not to Do

- Don't implement brand scanning, interviewing, or visual identity storage — that lives in `brand-brain`.
- Don't redesign or produce the final asset here — translate only; handoff to execution tools or the designer.
- Don't soften feedback into non-actionable suggestions: "consider more whitespace" is not a direction; a specific px value is.
- Don't silently ignore identity conflicts — flag them with ⚠ and offer an on-brand path.
- Don't guess at what an image looks like if you haven't seen it — ask for a screenshot or URL before prescribing specific fixes that depend on visual detail.
- Don't mark feedback as addressed if you couldn't map it to a principle — note it as "requires visual review" rather than inventing a fix.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and visual identity loaded (colors, type, tone, banned words)?
- Every feedback item mapped to at least one design principle?
- Every fix specific and actionable (not a wish — a measurable change)?
- Contradictions in the feedback surfaced and resolved with a rationale?
- Identity conflicts tagged ⚠ with on-brand alternative?
- Translate mode: single direction brief, zero open questions remaining for the designer?
- Diagnose mode: principle named + "why it triggers this" present for each item?
- Recommended next step (downstream handoff) included?
