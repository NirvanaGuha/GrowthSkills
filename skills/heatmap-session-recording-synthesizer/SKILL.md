---
name: heatmap-session-recording-synthesizer
description: >
  Takes raw heatmap images (click, scroll, move, attention) and/or session-recording notes,
  observations, or transcripts from any tool (Hotjar, Microsoft Clarity, FullStory, Heap, PostHog,
  Lucky Orange, etc.) and synthesizes them into a structured behavioral insight report: dominant
  friction patterns ranked by severity, signal triangulation against the page's stated conversion
  goal, and a stack-ranked backlog of experiment hypotheses in the IF/THEN/BECAUSE format ready
  to hand to a-b-multivariate-test-designer or experiment-pipeline-backlog-prioritizer.
  Applies the Jobs-to-Be-Done + COM-B behavioral model to distinguish "can't do" friction from
  "won't do" friction and "don't know" confusion — because the fix for each is different.
  Encodes the classic session-analysis gotchas: rage-click false positives, above-the-fold scroll
  bias, sample-size floors, device-split artifacts, and bot/internal-traffic contamination.
  Use when the user pastes a heatmap screenshot, shares Hotjar/Clarity session notes, says "tell
  me what's wrong with this page," "synthesize my session recordings," "what are users doing on
  the page," "heatmap analysis," "friction audit from recordings," or hands over a batch of UX
  observation notes and asks what to test next.
---

# Heatmap & Session Recording Synthesizer

Raw behavioral data is noisy, ambiguous, and easy to misread. This skill applies a repeatable
diagnostic framework — not ad-hoc observation — to turn heatmaps and session-recording notes
into ranked friction patterns, a triangulated severity call, and a hypothesis backlog the team can
act on. It does not redesign the page; it produces the evidence layer that drives the next test.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, offer mechanics, and page
  goal so friction is evaluated against the right conversion intent, not a generic UX standard.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for the page's conversion goal, ICP
  awareness stage, and the offer mechanics before proceeding.
- **`data-qa-measurement-gotcha-checker`** — gate on data quality before synthesizing. Flags
  bot/internal-traffic contamination, insufficient sample, device splits hiding mobile failures,
  and tool-instrumentation gaps. Call before the insight pass, not after.
- **`a-b-multivariate-test-designer`** — downstream consumer of this skill's hypothesis backlog.
  Pass the ranked IF/THEN/BECAUSE stack directly; it handles test design.
- **`experiment-pipeline-backlog-prioritizer`** — scores and sequences the hypothesis backlog
  against available dev/design capacity. Compose when the user needs a quarterly test roadmap.
- **`validity-threat-checker`** — reviews any specific experiment spec this skill surfaces for
  novelty effect, SRM risk, seasonality, and instrumentation bias before launch.
- **`cro-reporting-audit-packager`** — wraps this skill's output into a formatted stakeholder
  report or CRO standup digest.
- **`landing-page-heuristic-live-cro-auditor`** — complementary; covers expert-heuristic
  evaluation (copy, hierarchy, trust signals). Compose when the user wants both behavioral data
  AND heuristic analysis in one pass.

---

## How a run works

```
Step 0  Load the brand + page goal  ──► call brand-brain
Step 1  Data quality gate           ──► call data-qa-measurement-gotcha-checker
Step 2  Classify inputs             ──► heatmap type(s) and/or recording signal
Step 3  Pattern extraction          ──► annotated friction inventory
Step 4  COM-B triage                ──► "can't" vs "won't" vs "doesn't know"
Step 5  Triangulate + rank          ──► severity × conversion-impact matrix
Step 6  Hypothesis backlog          ──► IF/THEN/BECAUSE stack, top 3–5 prioritized
Step 7  Output + hand-off           ──► save to ./cro/[slug]-heatmap-synthesis.md
```

---

## Step 0 — Load the brand and page goal (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Pull the active brand's ICP, awareness
tendency, and offer mechanics. Then establish:
- **Page URL / section** under analysis
- **Primary conversion goal** (click, scroll-to-CTA, form submit, add-to-cart, etc.)
- **Traffic sources** feeding the page (affects awareness stage, device mix expectations)
- **Tool and time window** for the data (Hotjar, Clarity, FullStory; date range; session count)

Do not produce any insight until brand-brain returns and the conversion goal is confirmed.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the page's conversion goal, ICP awareness stage, and the offer mechanics before proceeding.

---

## Step 1 — Data quality gate

Before synthesizing, call `data-qa-measurement-gotcha-checker` (or run the checks inline if not
installed). Flag and note in the output:

| Gotcha | Threshold / check |
|---|---|
| Sample floor | Click maps: < 1,000 clicks → low confidence; scroll maps: < 500 sessions → unreliable |
| Device split | If mobile > 40% of traffic, require a separate mobile heatmap — desktop-only maps mislead |
| Internal / bot traffic | Hotjar IP filter applied? Clarity bot detection? Tag self at the page? |
| Tool placement | Is the heatmap tag firing on the correct variant? SPA route-change tracked? |
| Recording bias | Session recordings oversample rage-clicks and errors — weight accordingly |
| Time window | < 7 days for low-traffic pages → seasonal noise; > 90 days → mixes pre/post changes |

If critical blockers exist (sample floor not met, no device split on mobile-heavy page), surface
them and recommend collection actions before proceeding. Partial synthesis is acceptable with
explicit caveats.

---

## Step 2 — Classify inputs

Identify what has been provided. Each type answers a different question:

| Input type | Primary question it answers |
|---|---|
| **Click heatmap** | Where are users clicking / where are they not clicking? |
| **Scroll heatmap** | How far do users get? Where does fold behavior create dead zones? |
| **Move / attention map** | Where does eye-path go? What's being ignored? |
| **Rage-click overlay** | Where is interaction frustrating or broken? |
| **Session recording notes** | Sequence of actions, hesitation points, abandonment triggers |
| **Form analytics** | Field-level drop, re-entry, hesitation time |

Triangulate across types when multiple are available — a click heatmap alone is ambiguous (is
the non-click zone unseen or ignored?); adding scroll data disambiguates.

---

## Step 3 — Pattern extraction (the annotated friction inventory)

For each friction signal identified, record:

```
Pattern: [name — e.g., "CTA buried below visible fold on mobile"]
Type: Navigation confusion / Ignored element / Rage-click / Scroll abandonment /
      Off-element click / Form hesitation / Dead-zone interaction
Evidence: [specific heatmap observation or recording note]
Device split: Desktop / Mobile / Both
Affected segment (if known): [new vs returning, source, etc.]
Confidence: High / Medium / Low  [based on sample + signal clarity]
```

Extract all distinct patterns before ranking. Don't collapse related patterns prematurely.

---

## Step 4 — COM-B triage (the fix-direction gate)

Apply the COM-B model (Capability, Opportunity, Motivation — Behaviour) to each pattern.
The model was originally a behavior-change framework (Michie et al.); in CRO it disambiguates
the intervention type required:

| COM-B category | User signal | Fix direction |
|---|---|---|
| **Capability — can't do** | Rage-clicks on non-links, form errors, broken interactions, navigation dead-ends | Fix the functional obstacle; no copy change will work |
| **Opportunity — doesn't know** | Scroll heatmap cuts off above the offer, users click the wrong element expecting it to be interactive, off-element clicks suggest missing affordance | Redesign visibility, hierarchy, or affordance signal |
| **Motivation — won't do** | Scrolls past CTA without clicking, high scroll depth but low conversion, hesitation time on price/commitment field | Address the persuasion gap: value, proof, risk-reducer, commitment match |

Label each pattern with its COM-B category before assigning a hypothesis. A Capability bug
treated as a Motivation problem gets the wrong fix and a failed test.

---

## Step 5 — Triangulate and rank by severity

Score each pattern on two axes:

**Conversion proximity** (how directly does this pattern sit between the user and the goal?)
- 3 = on the primary CTA or form submit path
- 2 = on a step that leads to the CTA
- 1 = peripheral / informational

**Reach** (what share of sessions does this affect?)
- 3 = > 50% of sessions / clicks
- 2 = 20–50%
- 1 = < 20% or unclear

**Severity score = Conversion proximity × Reach**

Rank by severity score descending. Flag ties. Surface the top 5 into the hypothesis backlog.

---

## Step 6 — Hypothesis backlog (the output artifact)

Write each hypothesis in IF/THEN/BECAUSE format. This is the hand-off contract for
`a-b-multivariate-test-designer` — it must be specific enough to derive a variant from.

```
Hypothesis #[n]  ·  Severity: [score]  ·  COM-B: [Capability / Opportunity / Motivation]
IF   [specific change to the page element / copy / layout]
THEN [predicted behavior change, measurable]
BECAUSE [the friction pattern it removes, referencing the evidence]
Metric to move: [primary conversion event + secondary leading indicator]
Suggested variant type: [copy swap / element reorder / component add / functional fix / A/B / MVT]
Data quality note: [any caveats from Step 1 that affect confidence]
```

Produce 3–5 hypotheses. Do not pad to 10. If the data only supports 2 clean hypotheses, say so.

---

## Output format

```markdown
## Heatmap & Session Recording Synthesis — [Page / URL]

**Brand:** [slug, via brand-brain]
**Tool / window:** [Hotjar/Clarity/etc., date range, session count]
**Conversion goal:** [primary CTA / event]
**Data quality:** [pass / flagged — list issues if flagged]

### Friction Pattern Inventory
[Table or numbered list from Step 3]

### COM-B Triage
[Pattern → category mapping]

### Severity Matrix
[Ranked table: Pattern | Proximity | Reach | Score | COM-B]

### Hypothesis Backlog (ranked)
[IF/THEN/BECAUSE blocks from Step 6]

### Recommended next step
[One sentence: e.g., "Run hypothesis #1 as an A/B test via a-b-multivariate-test-designer;
pass the backlog to experiment-pipeline-backlog-prioritizer to sequence against dev capacity."]
```

Save to `./cro/[brand-slug]-heatmap-synthesis.md`. Inline only if the user doesn't want a file.

---

## The classic gotchas (encode these, never skip them)

- **Rage-click ≠ broken element.** It often means a non-interactive element looks interactive.
  Check affordance before assuming a bug.
- **High scroll depth + low conversion is a Motivation problem, not a visibility problem.**
  Users saw the CTA; they didn't act. Don't bury it deeper — fix the persuasion.
- **Above-the-fold bias in scroll maps.** The top 10% of a page always has 100% scroll depth.
  The interesting signal is the cliff — where does depth drop sharply?
- **Device-split artifacts.** A desktop click heatmap showing dead-zone behavior above the CTA
  often means mobile users (merged view) are tapping the CTA that's above the fold on their device.
  Always split by device before concluding.
- **Sample size floors matter for click maps.** Sub-1,000 clicks make individual element
  conclusions noise. Cite confidence in every pattern.
- **Recording sessions are not random samples.** Most platforms oversample error and rage-click
  sessions. Normalize findings against aggregate quantitative data where possible.
- **Attribution window.** If the heatmap window overlaps a copy change, a promo, or a traffic
  source shift, the behavioral data is a blend. Flag it.

---

## Principles

- **Data quality before insight.** A synthesis built on contaminated or undersized data is
  worse than no synthesis — it misfires the test backlog.
- **COM-B before copy.** Capability obstacles require functional fixes. Applying persuasion
  to a broken interaction path wastes experiment budget.
- **Triangulate, don't cherry-pick.** One heatmap type rarely proves a pattern; look for
  convergent evidence across click, scroll, and recording signals.
- **Specificity over volume.** Three specific, testable hypotheses with clear metrics are worth
  more than ten vague ones.
- **Brand goal is the frame.** An element that looks "ignored" may be performing correctly if
  it's not on the conversion path. Evaluate friction relative to the goal, not generic UX norms.

## What Not to Do

- Don't produce insight before brand-brain returns the conversion goal and ICP.
- Don't skip the data quality gate — undersized or contaminated data corrupts the hypothesis.
- Don't conflate a scroll heatmap (reach) with a click heatmap (intent); they answer different questions.
- Don't redesign the page — surface patterns and hypotheses; let `a-b-multivariate-test-designer`
  or the team's design process own the variant.
- Don't generate hypotheses without a metric to move. "Improve engagement" is not a metric.
- Don't overindex on rage-click data — it is the most sampled and most misread signal in the toolkit.
- Don't merge device data before splitting; mobile and desktop behavioral models differ fundamentally.

## Quality Checklist

- brand-brain called; conversion goal and ICP confirmed before any insight produced?
- data-qa-measurement-gotcha-checker run; sample floor, device split, and bot-traffic flags addressed?
- Each friction pattern labeled with COM-B category before hypothesizing?
- Hypotheses in IF/THEN/BECAUSE format with a specific measurable metric?
- Rage-click, scroll-depth, device-split, and sample-size gotchas checked and noted?
- Top 3–5 hypotheses ranked by severity score (not intuition)?
- Output saved to `./cro/[brand-slug]-heatmap-synthesis.md` or confirmed inline?
- Next-step handoff to a-b-multivariate-test-designer or experiment-pipeline-backlog-prioritizer noted?
