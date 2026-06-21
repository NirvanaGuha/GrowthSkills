---
name: pre-mortem-post-mortem-generator
description: >
  Turns a planned or completed initiative — campaign, product launch, experiment, system change,
  or any operational incident — into a structured pre-mortem or post-mortem document. Pre-mortem
  mode uses Gary Klein's prospective hindsight technique: it imagines the initiative has already
  failed and systematically surfaces failure modes, their probability and blast radius, and the
  mitigations you still have time to act on. Post-mortem mode reconstructs what actually happened
  — timeline, root causes (via 5 Whys), contributing factors, impact, and blameless action items
  with owners and due dates. Both modes read the active brand's context from brand-brain to stay
  grounded in real positioning, real proof, and real ICP stakes — so a pre-mortem on a product
  launch brief surfaces audience-specific failure modes, not generic ones. Saves reusable artifacts
  to ./reports/. Use when the user says "pre-mortem," "post-mortem," "what could go wrong," "learn
  from what happened," "failure analysis," "incident review," "retrospective on this launch,"
  "blameless post-mortem," "kill this plan before it starts," or hands over a brief or incident
  summary and asks you to stress-test or debrief it.
---

# Pre-Mortem / Post-Mortem Generator

Imagine the initiative already failed — then work backwards. That's the whole insight. Klein's prospective hindsight technique reliably surfaces failure modes that brainstorming misses because it bypasses optimism bias by treating failure as a given, not a hypothesis.

This skill does that rigorously for both directions: **pre-mortem** (before launch, while you can still act) and **post-mortem** (after an incident or completed initiative, while you still remember what happened). Both are blameless by default. Both produce a reusable artifact you can share with a team, link in a brief, or drop into a retro.

---

## Skills this calls

- **`brand-brain`** (required) — resolves brand context so failure modes and impact statements reference real ICP stakes, offer mechanics, and proof (not generic placeholders).
- **`prioritization-framework-suite`** *(optional)* — if the pre-mortem produces a long mitigation backlog, call this skill to score and rank mitigations by effort vs. risk reduction before presenting.
- **`experiment-results-analyzer`** *(optional)* — when post-morteming a completed experiment, call this skill for the significance verdict and uplift before attributing the outcome to a root cause.
- **`a-b-multivariate-test-designer`** *(optional)* — when a pre-mortem uncovers a testable assumption, suggest a test brief via this skill rather than resolving the assumption by fiat.
- **`growth-diagnostic-deep-dive`** *(optional)* — when the post-mortem touches a traffic or revenue drop, defer the quantitative diagnosis to this skill and fold the output into the root-cause section.
- **`data-qa-measurement-gotcha-checker`** *(optional)* — before accepting reported numbers as ground truth in a post-mortem, pass them through this skill to rule out tracking artifacts.
- **`campaign-brief-builder`** *(optional)* — when a pre-mortem is run on a campaign brief that doesn't exist yet, call this skill first so there's a structured brief to stress-test.

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain (brand voice, ICP, offer, real proof)
Step 1  Detect the mode     ──► Pre-Mortem (future initiative) | Post-Mortem (past event)
Step 2  Gather the input    ──► brief, ticket, notes, or incident summary
Step 3  Run the framework   ──► prospective hindsight (pre) or 5-Whys timeline (post)
Step 4  Output the artifact ──► structured doc; save to ./reports/
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice, banned words, offer mechanics, real proof, ICP + awareness tendency, and positioning. Use these to make every failure mode and impact statement brand-specific — the ICP names are real, the offer stakes are real, the proof you can lose is real.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (what it is, ICP, offer/destination, voice), then proceed. Always prefer the call.

### Step 1 — Detect the mode

| Signal | Mode |
|---|---|
| Future tense — "about to launch," "planning to," "should we," brief/spec handed over | **Pre-Mortem** |
| Past tense — "what went wrong," "incident," "post-mortem this," results/timeline handed over | **Post-Mortem** |

When ambiguous, ask one question: "Is this initiative still ahead of you, or has it completed?"

---

## Pre-Mortem mode — prospective hindsight

**The frame:** "It's [date 6 months out]. The initiative failed. Revenue/retention/activation targets were missed. Leadership is asking what happened. Walk back through why."

### Phase 1 — Inject the brief

Read the initiative: its goal, channels, audience segment (mapped against the brand's ICP), offer, timeline, team, budget, dependencies, and success metric. If a structured brief is missing, ask for the minimum viable inputs — goal, audience, offer, success metric — before continuing.

### Phase 2 — Failure mode catalogue (prospective hindsight)

Generate failure modes across six lenses. For each, state: the **failure mode** (one sentence), **leading indicator** (what would you see first?), **probability** (High / Medium / Low, with a one-line rationale), **blast radius** (who and what it affects), and **mitigation** (the concrete action you can take before launch, not a vague hedge).

| Lens | What to probe |
|---|---|
| **Strategy** | Goal–effort mismatch, wrong channel for the audience's awareness level, offer too weak for the ICP |
| **Measurement** | No defined success metric, lagging-only KRs, tracking gaps, model blending risk |
| **Execution** | Timeline compression, dependency on a single owner, asset readiness, approval delays |
| **Audience** | ICP fit for this specific offer, messaging mismatch, awareness stage mismatch |
| **Competitive / Market** | Timing risks, saturation, a competitor announcement that neutralises the campaign |
| **Ops / Compliance** | Data privacy, suppression list gaps, legal review, platform policy changes |

Aim for 6–10 failure modes total. Don't pad. If a lens has nothing real, say so.

### Phase 3 — Prioritise and act

Surface the **top 3 mitigations** the team should act on before launch. Rank by: (a) probability × blast radius and (b) how much lead time the mitigation needs. Use `prioritization-framework-suite` if the backlog is long.

For each unverified assumption underlying a high-probability failure mode, flag it as a **testable assumption** and optionally suggest a test brief via `a-b-multivariate-test-designer`.

### Pre-Mortem output format

```
## Pre-Mortem — [Initiative name]
Brand: [slug] · ICP: [from brand-brain]
Date: [today] · Launch target: [date]
Goal: [stated goal] · Success metric: [stated metric]

### Failure mode catalogue
| # | Failure mode | Leading indicator | Probability | Blast radius | Mitigation |
|---|---|---|---|---|---|

### Top 3 mitigations to act on before launch
1. [Mitigation] — Owner: [?] · Deadline: [before X]
2. …
3. …

### Testable assumptions
- [Assumption] → test via: [brief or note]
```

Save to `./reports/pre-mortem-[initiative-slug]-[YYYY-MM-DD].md`.

---

## Post-Mortem mode — blameless 5-Whys reconstruction

**The frame:** Assume good intent and competent people making reasonable decisions with the information they had. The goal is system improvement, not blame assignment.

### Phase 1 — Establish the facts

Reconstruct the **incident / outcome timeline** — a chronological list of what happened and when, sourced from what the user provides. Flag gaps explicitly. If numbers are provided, pass them through `data-qa-measurement-gotcha-checker` before accepting them as ground truth.

### Phase 2 — Root cause analysis (5 Whys + contributing factors)

Pick the central failure event. Ask "Why did this happen?" — and keep asking until you reach a systemic root cause (a process gap, a missing signal, a structural assumption) rather than a proximate human action.

**Distinguish:**
- **Root cause** — the systemic condition that enabled the failure. Fixing this prevents recurrence.
- **Contributing factors** — conditions that made it worse but are not the root cause alone.
- **Proximate cause** — what happened immediately before the failure. Usually a symptom, not a root.

Repeat for any secondary failure modes surfaced by the timeline.

### Phase 3 — Impact assessment

State the actual impact: metric delta, revenue/retention/activation effect (using real brand offer and ICP stakes from `brand-brain`), affected user segments, and time-to-detection. Mark unconfirmed numbers `[verify]`.

### Phase 4 — Blameless action items

Write action items that address root causes and contributing factors, not proximate events. Each item must have: a **what** (specific system/process change), an **owner** (role, not just "team"), and a **due date**. If an owner is unknown, write `[assign]`. No action item should be "be more careful."

### Post-Mortem output format

```
## Post-Mortem — [Initiative / Incident name]
Brand: [slug] · Severity: [P1 / P2 / P3 or High / Medium / Low]
Date of incident: [date] · Date of review: [today]
Author: [?] · Status: [Draft / Under Review / Final]

### Timeline
[Chronological list — timestamp · event · source]

### Impact
[Metric delta, user/revenue/activation effect, time-to-detect]

### Root cause analysis
**Root cause:** [One sentence]
**5 Whys chain:**
  1. Why …?  →  Because …
  2. Why …?  →  Because …
  [continue to systemic root]
**Contributing factors:** [Bulleted list]
**Proximate cause (not the root):** [One sentence]

### Action items
| # | What | Owner | Due date | Addresses |
|---|---|---|---|---|

### What went well
[Bulleted — honest, not performative]

### What we'd do differently
[Bulleted — systemic, not personal]
```

Save to `./reports/post-mortem-[initiative-slug]-[YYYY-MM-DD].md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No failure mode catalogue or impact statement before the active brand loads. Generic ICP/offer references are a smell — every material claim must come from the brand digest.
- **Prospective hindsight, not brainstorming.** "What could go wrong?" produces hedges. "It failed — why?" produces specific failure modes. Keep the pre-mortem in past tense.
- **Blameless post-mortems only.** Surface system conditions, not personal failures. If a name appears in a root-cause chain, reframe to the process or signal that was missing.
- **Root cause, not proximate cause.** Stopping at "the email went out with the wrong link" is not a post-mortem — it's a description. Keep asking why until you hit a systemic condition.
- **Testable assumptions, not false confidence.** When a pre-mortem assumption is unverifiable before launch, name it and flag it — don't paper over it with a mitigation that doesn't exist yet.
- **Real numbers or `[verify]`.** Never invent conversion impacts, revenue deltas, or probability estimates. Mark every unconfirmed number.

---

## What Not to Do

- Don't run a pre-mortem on a brief that doesn't exist — call `campaign-brief-builder` first if needed.
- Don't pad the failure mode table. Six real failure modes beat twelve padded ones.
- Don't attribute root causes to people or personal failures. Reframe to systems.
- Don't confuse contributing factors with root causes — the 5 Whys chain exists precisely to separate them.
- Don't produce "what went well" as a performative exercise. If nothing genuinely went well, say so.
- Don't skip the brand-brain step because the initiative "doesn't feel brand-specific" — ICP stakes and offer mechanics make impact statements real.
- Don't save artifacts to the skill folder. Use `./reports/`.

---

## Quality Checklist

- `brand-brain` called and active brand loaded before any failure mode or impact statement?
- Mode correctly detected (pre vs. post) — confirmed with user if ambiguous?
- Pre-mortem: six lenses covered, each failure mode has a leading indicator + probability + mitigation, top 3 mitigations surfaced, testable assumptions flagged?
- Post-mortem: timeline sourced from real inputs with gaps noted, 5 Whys chain reaches a systemic root cause (not a proximate event), action items have owners and due dates, blameless framing throughout?
- All unconfirmed numbers marked `[verify]`; real brand ICP/offer language used?
- Artifact saved to `./reports/[mode]-[slug]-[YYYY-MM-DD].md`?
