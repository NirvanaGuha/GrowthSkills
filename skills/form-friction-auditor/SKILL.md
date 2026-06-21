---
name: form-friction-auditor
description: >
  Audits any lead-capture, checkout, signup, or gating form for conversion-killing friction —
  field by field, in one structured pass. Input a form screenshot, a pasted field list, or field
  names paired with completion-rate or drop-off data; output a prioritized friction score per
  field, a removal/reorder action list, a progressive-disclosure blueprint for long forms, and
  inline microcopy fixes. Uses the Fogg Behavior Model (motivation × ability × prompt) as the
  scoring spine: every field is rated on cognitive load (label clarity, expected vs. required
  input), trust tax (sensitive data asked too early), necessity (dead fields that should never
  have existed), and sequence logic (value-first vs. ask-first ordering). A data mode activates
  when drop-off rates or completion CSVs are available — fields are ranked by friction ×
  abandonment-weight, not just gut feel. Integrates with brand-brain for voice-compliant
  microcopy and placeholder rewrites; calls landing-page-heuristic-live-cro-auditor and
  a-b-multivariate-test-designer to hand off high-impact changes into a live experiment pipeline.
  Use whenever the user says "audit my form," "why is my form converting badly," "reduce form
  fields," "form UX review," "form drop-off," "signup friction," "progressive disclosure for my
  form," "checkout form audit," "gating form," or pastes a field list and asks what to cut.
---

# Form Friction Auditor

Forms are where intent dies. A prospect who clicked your CTA, read your copy, and reached the form is motivated — the form is the only thing standing between that motivation and a conversion. This skill audits that gap field by field, scores the friction, and hands you a ranked action list: what to remove, what to reorder, what to hide behind progressive disclosure, and what microcopy to fix.

Framework: **Fogg Behavior Model** applied per-field — a form converts when motivation × ability × prompt are in alignment at the moment of fill. Every field that taxes ability (complexity, cognitive load, trust cost) without returning perceived value is a lever.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, ICP, offer mechanics, banned words, and proof for on-brand microcopy and placeholder rewrites. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP awareness level, and offer type before proceeding.
- **`landing-page-heuristic-live-cro-auditor`** (optional, Step 4) — when the form lives on a landing page, hand off the audit summary to get the surrounding-page context scored in the same pass.
- **`a-b-multivariate-test-designer`** (optional, Step 4) — convert the top-3 field-removal or reorder recommendations into a structured A/B test brief.
- **`data-qa-measurement-gotcha-checker`** (data mode) — when the user supplies completion-rate or drop-off data, run a quality gate before weighting the friction scores by abandonment numbers. GA4 form data especially needs SRM and (not set) checks before being trusted.
- **`cta-variant-generator`** (optional) — rewrite the form's submit button label in Quick mode once field audit is complete.

---

## How a run works

```
Step 0  Load brand context    ──► brand-brain (always first)
Step 1  Ingest the form       ──► screenshot / field list / completion CSV
Step 2  Score each field      ──► Fogg friction matrix
Step 3  Produce the audit     ──► field table + action list + progressive-disclosure blueprint
Step 4  Handoffs (optional)   ──► CRO auditor / A/B designer / CTA rewrite
```

---

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before producing any output. Use the returned digest for: voice-compliant placeholder and label rewrites, ICP awareness level to calibrate how much friction is acceptable (bottom-of-funnel/most-aware audiences tolerate more fields than top-of-funnel/unaware ones), offer mechanics to verify that gating requirements (e.g. "company size required for a trial") are justified by the actual use of that data.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP awareness stage, and offer type before proceeding.

---

### Step 1 — Ingest the form

Accept any of:

- **Screenshot** — identify every visible field (label, placeholder, required marker, helper text, CTA button).
- **Field list** — plain text or table of field names, types, and required/optional status.
- **Completion data** — field list + per-field drop-off rate or completion %; activates **data mode**.

If the user provides no data, run the **heuristic mode** (friction scoring only). If completion/drop-off rates are present, activate **data mode** and gate the numbers through `data-qa-measurement-gotcha-checker` before weighting. Note any GA4-sourced completion data: check for SRM, (not set) dimension, and form_submit vs. form_start event mismatches before treating the numbers as reliable.

Clarify before proceeding if: the form's funnel stage is ambiguous (awareness vs. product-aware vs. most-aware) or the offer type is unclear (gated content, trial, checkout, event registration, consultation request).

---

## The Fogg Friction Matrix (the audit engine)

Score each field on four dimensions, each 1–3:

| Dimension | Score 1 (low friction) | Score 2 (medium) | Score 3 (high friction) |
|---|---|---|---|
| **Cognitive load** | Clear label, familiar format, one input | Ambiguous label or multi-part input | Ambiguous + unfamiliar format (e.g. VAT ID early in a free trial) |
| **Trust tax** | Non-sensitive, expected at this stage | Mildly sensitive or unexpected (phone on a content gate) | Sensitive data asked too early (SSN, revenue, company headcount on a top-of-funnel gate) |
| **Necessity** | Demonstrably used downstream (e.g. email for delivery) | Nice-to-have segmentation enrichment | Dead field — data collected but never acted on [verify with ops team] |
| **Sequence logic** | Value-exchange is clear before the ask | Partially clear | Ask-first without stated reason ("Why do you need my phone number?") |

**Total friction score per field: 4–12.** Fields scoring 9–12 are removal candidates. Fields scoring 7–8 are reorder or progressive-disclosure candidates. Fields scoring 4–6 are keepers (optimize microcopy only).

**Data mode weighting:** multiply the friction score by the field's abandonment weight (normalized drop-off rate 0–1). A field with friction score 8 and 60% abandonment contribution is a higher priority than a score-10 field that almost nobody actually quits on.

---

### Step 3 — Audit output

Produce three artifacts:

**Artifact 1 — Field-by-field friction table**

```
## Form Friction Audit — [Form name / URL / description]
Brand: [slug, via brand-brain]
Form stage: [awareness / consideration / decision / post-signup]
Offer: [what the user gets in exchange for filling]
Mode: [heuristic / data — if data, note source + QA status]

| # | Field | Type | Required | Cog Load | Trust Tax | Necessity | Sequence | Score | Action |
```

Action codes: **REMOVE** / **DEFER** (progressive disclosure) / **REORDER** / **REWRITE** (microcopy/label only) / **KEEP**.

**Artifact 2 — Prioritized action list**

Ranked by friction score (× abandonment weight in data mode). For each action:
- What to do (remove, defer to step 2, move after email, rewrite label)
- Why (one line, Fogg dimension)
- Estimated friction reduction (directional: High / Medium / Low) — never fabricate a conversion-lift percentage; mark any benchmark `[verify]`
- Any brand-voice note from brand-brain (placeholder language, banned words)

**Artifact 3 — Progressive disclosure blueprint** (for forms with ≥6 fields or a multi-step candidate)

Map the fields into a sequenced step structure:
- **Step 1:** minimum viable ask — the fewest fields needed to identify the lead and deliver the offer (almost always email only, or email + first name)
- **Step 2:** enrichment — fields the team genuinely uses for routing or personalization, shown only after Step 1 completes
- **Step 3 (if needed):** qualification — company size, use case, role — for sales-routed flows only

Include a one-line rationale for each step boundary. Note if a field belongs in a post-signup onboarding flow instead of the gate form.

Save audit output to `./cro/form-audit-[slug]-[date].md` when the user asks to save; otherwise deliver inline.

---

### Step 4 — Optional handoffs

- **Landing page context:** invoke `landing-page-heuristic-live-cro-auditor` with the form URL and the audit summary to check whether friction above the fold, social proof placement, or CTA placement amplifies the form abandonment.
- **Test pipeline:** invoke `a-b-multivariate-test-designer` with the top-3 REMOVE/REORDER actions as test hypotheses; it returns a properly structured A/B brief with sample-size and runtime estimates.
- **Submit button:** invoke `cta-variant-generator` with the current button label and the brand digest; receive on-voice alternatives.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit output before brand-brain returns. Voice and ICP override default label rewrites.
- **Fogg discipline.** Every recommendation traces to a specific friction dimension — not aesthetic preference, not gut feel.
- **Data mode honesty.** If drop-off data is present, gate it through `data-qa-measurement-gotcha-checker`. Never weight recommendations by numbers that haven't passed a basic quality check (SRM, event-mismatch, (not set) share).
- **No fabricated lift numbers.** Directional impact ratings (High/Medium/Low) only; any cited benchmark gets `[verify]`. The "remove every field" instinct is right directionally but wrong in context — a 2-field form for a $50K sales deal is under-qualified, not optimized.
- **Stage-aware.** The acceptable friction ceiling varies by funnel stage. A most-aware, decision-ready buyer filling out a demo request tolerates more fields than a first-touch content-gate visitor. Score in context.
- **Necessity is business logic, not opinion.** A field is a REMOVE only when data is provably unused or duplicated. Mark uncertainty `[verify with ops/CRM team]` rather than guess.

## What Not to Do

- Don't reimplement brand scanning or ICP resolution — call `brand-brain`.
- Don't produce microcopy or label rewrites before the brand digest returns.
- Don't recommend removing a field because it "feels like a lot" — trace every removal to a Fogg dimension.
- Don't fabricate conversion lift percentages or cite benchmarks without a `[verify]` tag.
- Don't run data-mode scoring on raw GA4 numbers without a quality gate — form_submit event parity is a known GA4 gotcha.
- Don't conflate progressive disclosure with multi-page forms — a two-step inline form and a multi-page wizard are architecturally different; name the right pattern.
- Don't recommend full form removal or a single-click social-auth unless the offer type actually supports it — that's a product decision, not a copy fix.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest loaded (or fallback executed) before any output?
- Every field scored on all four Fogg dimensions; score traceable to a specific dimension, not opinion?
- Data mode: if completion/drop-off data supplied, was `data-qa-measurement-gotcha-checker` invoked or explicitly noted as needed?
- Action codes assigned (REMOVE / DEFER / REORDER / REWRITE / KEEP); no field left unclassified?
- Progressive-disclosure blueprint present for forms with ≥6 fields or a multi-step candidate?
- No fabricated conversion-lift percentages; all cited benchmarks tagged `[verify]`?
- Funnel stage noted and recommendations calibrated to it (not one-size "remove everything")?
- Handoff offers made to `landing-page-heuristic-live-cro-auditor` and `a-b-multivariate-test-designer` where applicable?
