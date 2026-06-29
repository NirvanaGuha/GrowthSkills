---
name: brand-consistency-auditor
description: >
  Audits any asset set — screenshots, copy docs, HTML, PDF, ad creatives, social posts, email
  templates, landing pages, decks — against the active brand guide and flags every off-brand
  violation: wrong colors, wrong fonts, unapproved logo treatments, banned words, tone drift,
  and visual hierarchy breaks. Returns a structured violation report scored by severity (Critical /
  Major / Minor), with per-asset correction notes and a pass/fail summary a designer, copywriter,
  or approver can act on in one sitting. Uses a consistent house checklist (our 5-Pillar Brand
  Consistency model) so a junior produces the same audit quality as a senior. Calls brand-brain to load the
  active brand's real style rules — never re-derives them inline. Optionally composes with
  editorial-style-guide, brand-voice-codifier, or proof-vault when companion files are present.
  Use when the user says "audit these assets," "brand check," "is this on-brand," "QA these
  creatives," "review for brand consistency," "check the visual identity," or sends a batch of
  screenshots/URLs/docs and asks whether they match the brand.
---

# Brand Consistency Auditor

Stamp out off-brand drift before it ships. This skill takes an asset set — any mix of screenshots, URLs, copy, HTML, decks, or social posts — loads the active brand's real rules from `brand-brain`, and returns a severity-ranked violation report with actionable correction notes. It runs on a 5-pillar brand consistency checklist (our working house model) so every audit is structured, repeatable, and defensible.

This skill audits. It does not redesign assets, rewrite full copy drafts, or rebuild brand rules from scratch — those live in the sibling skills it composes with.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, color, typography, logo rules, banned words, proof, and positioning. Never re-derived here.
- **`editorial-style-guide`** (if present) — pulls the house style companion file for grammar and terminology rules.
- **`brand-voice-codifier`** (if present) — pulls detailed lexicon and tone-of-voice parameters for deep copy audits.
- **`proof-vault`** (if present) — cross-checks any statistics or claims in the asset against approved proof points.
- **`post-quality-reviewer-voice-auditor`** — use instead (or in addition) for social-post-specific audits; this skill focuses on the cross-channel brand consistency layer.
- **`design-qa-handoff-pack-builder`** — use after this audit to package corrections into a dev handoff; that skill consumes this skill's output.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (+ companions if present)
Step 1  Intake the asset set  ──► identify asset types, scope
Step 2  Run the 5-Pillar audit  ──► per-asset, per-pillar scan
Step 3  Score and triage  ──► Critical / Major / Minor per violation
Step 4  Present the report  ──► summary + asset-by-asset detail
Step 5  (Optional) Save  ──► ./brand-audits/[slug]-audit-[date].md
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest: color palette (hex codes), approved fonts, logo usage rules, voice adjectives, banned words/phrases, positioning line, ICP, real proof points, and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it before returning. Do not begin auditing until it returns.

Also load any companion files the `brand-brain` result points to: `style-guide.md` (typography/grammar rules), `brand-voice-codifier` output (tone parameters), `proof.md` (approved claims). Cross-reference violations against these during the copy pillar audit.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` or supply a minimal brief (color hex codes, font names, logo rules, 3 voice adjectives, banned words, positioning). Always prefer the call.

### Step 1 — Intake the asset set

Accept: screenshots (described or attached), URLs, pasted HTML/copy, file paths, PDF descriptions, deck slide descriptions, ad creative descriptions, social post text.

For each asset, note: asset ID/name, asset type (email, landing page, social post, ad creative, deck slide, etc.), and the audit scope (full 5-pillar vs. copy-only vs. visual-only when the user requests a narrower scan).

If the asset set is large (10+ assets), ask whether the user wants a rapid triage pass (flag Critical only, skip Minor) or a full pass. Default to full.

---

## The 5-Pillar Brand Consistency Model

Every asset is evaluated against these five pillars, in order. Each pillar has sub-checks. A violation gets a severity score and a correction note.

### Pillar 1 — Color

Sub-checks:
- **Primary palette match:** does every color in the asset map to an approved hex value (or its approved tint/shade)? Flag any hex that does not appear in the brand palette with the closest approved substitute.
- **Hierarchy compliance:** is the primary brand color used as primary, accent as accent? Inverted hierarchy (accent used as background, neutral used as CTA) is a Major violation.
- **Gradient / shadow rules:** if the brand disallows or specifies gradients, is the rule honored?
- **Accessibility floor (WCAG AA):** text on background must meet 4.5:1 contrast minimum. Flag failures — this is always at least Major because it affects legal risk as well as brand quality. Use `[verify contrast ratio]` if exact values cannot be determined from the asset.

### Pillar 2 — Typography

Sub-checks:
- **Approved typefaces only:** any font family not on the approved list is a Critical violation. List the unapproved family and the approved substitute.
- **Type scale:** heading hierarchy (H1 > H2 > H3) must be consistent within the asset and match brand scale direction (e.g. "bold display, medium subhead, regular body"). Inconsistent sizing that undercuts hierarchy = Major.
- **Weight and style rules:** if the brand prohibits italic or uses only specific weights, flag violations.
- **Minimum size / line-height:** body copy below a legible floor (typically 14px web, 10pt print) = Major.

### Pillar 3 — Logo & Identity Marks

Sub-checks:
- **Version:** is the correct logo version used for the context (horizontal, stacked, icon-only, reversed)? Wrong version = Major.
- **Clear space:** is the exclusion zone around the logo respected? Logo touching other elements = Major.
- **Prohibited treatments:** stretched, rotated, recolored (outside approved alternates), dropshadowed, outlined, low-res logos = Critical.
- **Placement:** if brand rules specify placement (top-left, centered) and the asset violates it = Minor unless it conflicts with an approved template.

### Pillar 4 — Copy & Tone

Sub-checks:
- **Banned words / phrases:** any instance of a banned word or phrase from the `brand-brain` digest = Critical. List every instance and the approved alternative.
- **Voice adjective match:** score the copy on the brand's stated voice adjectives (e.g. "confident, plain-spoken, warm"). If the copy reads opposite to any adjective, flag it as Major with a brief rewrite suggestion (1 sentence max — full rewrites go to `brand-voice-codifier`).
- **Positioning alignment:** does the copy reinforce the brand's positioning line, or does it make claims that contradict or dilute it? Contradiction = Major.
- **Proof discipline:** any statistic or claim not present in `proof.md` (if available) or in `brand.md`'s proof section = flag as `[verify]`. Do not mark as a violation unless it is demonstrably false; mark as a verification gate.
- **Reading level / complexity:** flag if copy exceeds the brand's stated reading level target (if specified); otherwise skip.

### Pillar 5 — Layout & Visual Hierarchy

Sub-checks:
- **Spacing system:** if the brand uses a grid or spacing scale (e.g. 8px base unit), flag obvious breaks (crowded text, ragged margins). Minor unless it degrades legibility.
- **CTA prominence:** the primary CTA must be visually dominant — highest-contrast button, largest action element. Buried or visually subordinated CTA = Major.
- **Image style:** photography/illustration must match approved style direction (e.g. "real people, bright ambient light, no stock-photo poses"). Style drift = Major; one-off mismatch = Minor.
- **Template adherence:** if the asset type has an approved template, flag deviations beyond the approved customization range.

---

## Severity definitions

| Severity | Meaning | Turnaround expectation |
|---|---|---|
| **Critical** | Breaks brand identity, risks legal exposure, or publishes a banned claim. Must fix before any distribution. | Block publish |
| **Major** | Undermines brand consistency noticeably to an attentive viewer; creates a mixed signal. Fix before campaign launch. | Fix in next revision |
| **Minor** | Noticeable to a brand expert; unlikely to confuse an end user; fix in next template update. | Fix in backlog |

---

## Report format

```
## Brand Consistency Audit — [Brand slug] — [Asset set name]
Audited: [date] | Assets reviewed: N | Brand brain: [brand.md path]

### Summary
| Severity | Count |
|---|---|
| Critical | N |
| Major | N |
| Minor | N |
Overall verdict: PASS / CONDITIONAL PASS (fix Majors before launch) / FAIL (Criticals present)

### Asset-by-asset findings

#### [Asset 1 name / ID]
| Pillar | Violation | Severity | Correction |
|---|---|---|---|
| Color | CTA button uses #FF5733 (not in palette). Closest approved: #3B43FF | Critical | Replace button fill |
| Copy | Contains banned phrase "best-in-class" | Critical | Replace with [approved alternative or ask brand-brain] |
| Typography | Body set in Arial; approved stack is Myriad Pro / system sans | Critical | Update font |
| Layout | Primary CTA below the fold with no visual prominence | Major | Move above fold or increase contrast |
...

#### [Asset 2 name / ID]
...

### Cross-asset patterns
[If 3+ assets share the same violation, call it out here as a systemic issue — it likely points to a template or component that needs fixing at the source.]

### Recommended next steps
1. [Highest-impact fix, specific]
2. ...
```

Save to `./brand-audits/[slug]-audit-[YYYY-MM-DD].md` when the user asks or the asset set is large enough to warrant a file.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit begins until `brand-brain` returns the active brand. Its rules override any inline assumption.
- **Cite the rule, not just the violation.** Every finding references the brand guideline it breaks ("brand.md specifies #3B43FF as the only approved blue; this asset uses #007BFF").
- **Severity is calibrated.** Critical is not overused — reserved for identity-breaking or claim-false violations. Minor is not dismissed — it still gets a correction note.
- **Proof discipline.** Unverifiable claims get `[verify]`, not a violation flag unless they contradict approved proof.
- **Systemic patterns over one-off flags.** If the same violation appears in 3+ assets, flag the upstream source (template, component library, brand brief gap) — fixing there fixes everywhere.
- **Audit, don't redesign.** Correction notes are concise directives, not full redesigns. Flag the problem; the designer fixes it.

## What Not to Do

- Don't start auditing before `brand-brain` returns the active brand digest.
- Don't invent brand rules not present in `brand.md` or its companions — audit only what is documented.
- Don't reimplement brand scanning, voice analysis, or proof management — call the relevant siblings.
- Don't mark `[verify]` items as violations unless they are demonstrably false.
- Don't ignore Minor violations — they reflect real drift and compound over time.
- Don't rate every issue Critical to seem thorough — calibrated severity is the whole point.
- Don't produce a full rewrite of off-brand copy — give a one-sentence direction and point to `brand-voice-codifier`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest loaded (or bootstrapped) before any finding issued?
- All 5 pillars evaluated for each asset (or scope narrowed with user permission)?
- Every violation cites the specific brand rule it breaks?
- Severity calibrated correctly — no Critical inflation, no Minor dismissal?
- Cross-asset patterns identified and called out separately?
- Unverifiable claims marked `[verify]`, not flagged as violations?
- Report includes an overall verdict (PASS / CONDITIONAL / FAIL) and recommended next steps?
- Output saved to `./brand-audits/` when the set warrants a file?
