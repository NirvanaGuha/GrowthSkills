---
name: project-plan-generator-reviewer
description: >
  Turns a campaign or project deliverables list into a phase-by-phase project plan — with tasks,
  owners, durations, critical path, dependencies, and milestones — then runs a structured quality-gate
  review of the plan's timelines and assumptions before you commit to them. Two modes in one: Generate
  builds the plan from scratch; Review audits an existing plan for Planning Fallacy bias, missing
  dependencies, under-resourced critical-path steps, and gate criteria gaps. Brand context loads via
  brand-brain so voice, channel mix, and launch constraints stay consistent with every other marketing
  asset. Use when someone says "build a project plan," "create a campaign plan," "map out the
  deliverables," "timeline this," "review our project plan," "spot the critical path," "what's
  missing from this plan," "do a plan QA," or hands over a deliverables list and asks what order to
  do it in. Generates the plan AND reviews it — it does not execute tasks or manage the live project.
---

# Project Plan Generator & Reviewer

Turn a deliverables list into a sequenced, owned, gated project plan — then immediately stress-test it before the team commits. Two jobs; one skill.

**Generate** takes a campaign brief or deliverables list and produces a phase-by-phase plan with tasks, owners, durations, dependencies, critical path, and milestone gates. **Review** takes any existing plan — yours or another team's — and audits it for Planning Fallacy padding, broken dependency chains, missing gate criteria, and timeline risk. You can run Generate then Review in a single pass (the default) or invoke either mode on its own.

This skill plans and critiques. It does not write the campaign copy, run the analytics, or manage the live project once underway — for those, call the siblings listed below.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's launch context: channel mix, known constraints (blackout dates, approval cycles, compliance requirements), voice, and audience. This skill does not re-derive brand context.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for the brand name, key launch constraints (blackout dates, approvals required), and channel mix, then proceed.
- **`campaign-brief-builder`** (optional) — if no brief exists yet, call this first to define goals, audience, and deliverables before planning begins.
- **`risk-log-builder`** (optional) — hand off the completed plan to generate a full risk register; synthesize a top-3 inline when absent.
- **`deadline-drift-detector`** (optional, Review pass) — if live task completion data is provided alongside the plan, call this to produce a red/amber/green drift report.
- **`go-no-go-gate-evaluator`** (optional, Review pass) — evaluate the plan's gate criteria against pass/fail thresholds before launch.
- **`campaign-qa-launch-checklist-generator`** (optional, Review pass) — cross-check the plan's final milestone against launch readiness; synthesize a checklist inline when absent.
- **`stakeholder-update-status-writer`** (optional) — after planning is approved, this writes the kick-off status update; call on request.
- **`marketing-roadmap-builder`** (optional) — if the plan is one initiative among many, hand it off here to slot it into the broader quarterly roadmap.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain
Step 1  Clarify scope        ──► fill any blocking gaps; proceed on reasonable assumptions
Step 2  Generate the plan    ──► phase-by-phase, CPM critical path, milestones
Step 3  Review the plan      ──► Planning Fallacy audit + dependency/gate QA
Step 4  Surface the output   ──► plan artifact + review verdict + risk flags
```

---

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill. It returns the active brand's digest including known calendar constraints (blackout periods, fiscal deadlines, approval cycles), channel sequencing conventions, compliance requirements, and ICP-relevant timing. Use these as hard constraints in the plan — do not invent or override them. Apply the fallback if brand-brain is unavailable.

---

### Step 1 — Clarify scope

If the input is a full brief (goals, deliverables, owners, deadline), proceed directly. If inputs are thin, ask only the blocking questions in one batch — never re-ask what the brief or brand context already answers:

| Blocking gap | Question |
|---|---|
| Missing deadline | "What is the hard launch date?" |
| No owner map | "Who owns each major deliverable, or should I note TBD?" |
| Ambiguous scope | "Is [X deliverable] in or out of scope?" |
| No phasing guidance | "Is there a preferred phase structure, or should I recommend one?" |

Use the Planning Fallacy audit (Step 3) to pressure-test whatever inputs the user provides — do not ask the user to pre-validate their own estimates.

---

### Step 2 — Generate the plan

Apply the **Five-Phase Campaign Planning Model** (PMI-aligned, adapted for marketing):

| Phase | Typical tasks |
|---|---|
| **1. Foundation** | Brief lock, stakeholder alignment, brand/legal/compliance sign-off |
| **2. Strategy** | Audience targeting, messaging hierarchy, channel mix, A/B test design |
| **3. Production** | Asset creation (copy, design, landing page, email, push, video), QA passes |
| **4. Pre-launch** | Tracking setup, list/segment prep, approval gates, rehearsal/dry run |
| **5. Launch & Measure** | Go-live, monitoring cadence, post-launch reporting window |

For each task, record:

```
Task ID | Phase | Task name | Owner | Duration | Depends on | Milestone gate?
```

**Critical Path Method (CPM).** After listing all tasks and dependencies, trace the longest path from start to end — tasks on this path have zero float; any slip delays launch. Mark critical-path tasks explicitly. Flag any critical-path task whose owner is not named or whose dependency is circular.

**Milestones and gate criteria.** Place a named milestone at the end of each phase. Each milestone needs a pass/fail gate criterion (not just a date): what must be true for this phase to be considered complete? Vague criteria ("assets ready") are a red flag — name them specifically ("all copy approved by legal, final design files in shared drive").

**Duration estimates.** State the basis for each estimate where possible (brand context, stated deadline, industry convention). Mark any estimate without a stated basis as `[assumed]`.

**Output format — plan artifact:**

```markdown
## Project Plan: [Campaign Name]
Brand: [slug] | Launch date: [date] | Plan generated: [date]

### Phase 1 — Foundation
| # | Task | Owner | Duration | Depends on | CPM? |
...
**Milestone:** [name] — Gate: [specific pass criterion]

### Phase 2 — Strategy
...

### Critical Path
[Task chain] → total duration: [X days] — Float remaining to launch: [Y days]

### Risk flags (top 3)
...
```

Save the plan artifact to `./plans/[brand-slug]-[campaign-slug]-plan.md` on request; inline otherwise.

---

### Step 3 — Review the plan (always runs after Generate; also runs standalone)

Apply the **Planning Fallacy Audit** (Kahneman 1979, 2011): humans systematically underestimate task duration and overestimate resource availability. The review pass corrects for this before commitment.

**Six review checks:**

1. **Duration realism.** Are production and approval tasks padded for revision cycles? A single round of copy review taking less than 2 business days is suspicious; legal/compliance review under 5 business days is a red flag unless pre-validated. Flag `[optimistic]` with a recommended buffer.

2. **Dependency integrity.** Does every dependency chain resolve without circularity? Does any task start before its predecessor can realistically finish? Find the first broken link in the critical path.

3. **Owner completeness.** Every critical-path task needs a named owner. Tasks marked TBD on the critical path are a blocker risk — flag immediately.

4. **Gate criteria specificity.** Vague milestones ("creative is done") are not gate criteria. For each milestone, verify a concrete, verifiable pass condition exists. Propose one if missing.

5. **Float adequacy.** With the current plan, how many days of float exist between the critical path's end and the hard launch date? Under 2 business days of float = high risk. Flag and recommend where to recover float (shorten non-critical-path tasks, parallelize sequential tasks).

6. **Resource concentration.** Does any single owner appear on more than 40% of critical-path tasks? Flag as a single-point-of-failure risk.

**Review output format:**

```markdown
## Plan Review — [Campaign Name]

### Verdict: [GO | GO WITH CONDITIONS | NO-GO]
[One sentence rationale]

### Check 1 — Duration realism
...

### Check 2 — Dependency integrity
...

[checks 3–6]

### Recommended changes
| Change | Priority | Impact |
| ... | High/Med/Low | [what it fixes] |
```

For a standalone review (no Generate pass), read the provided plan first, then output the same review format.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No plan before the brand's launch context and constraints are loaded. Calendar constraints and approval cycles override any user-provided duration estimate.
- **CPM over gut feel.** Always trace and label the critical path explicitly. Float must be stated, not implied.
- **Gate criteria must be verifiable.** "Done" is not a gate criterion. Name the specific condition and who verifies it.
- **Planning Fallacy correction is mandatory.** Review runs every time, even after a Generate pass in the same session. The author should not self-approve the plan.
- **Flag assumptions, don't hide them.** Every `[assumed]` duration is a debt to pay before launch. Surface them.
- **Separate reviewer from author.** Generate and Review are distinct passes — the skill does not skip Review because it just wrote the plan. Same separation a PMBOK-trained PM applies.

---

## What Not to Do

- Do not produce a plan before brand-brain returns (brand constraints and approval cycles live there).
- Do not let vague milestone criteria through — "assets ready" is not a gate; name the exact deliverable and verifier.
- Do not skip the critical path calculation. A task list without CPM is a to-do list, not a project plan.
- Do not mark a plan GO if float to launch is under 2 business days without flagging it.
- Do not reimplement brand context scanning or the risk-register — call `risk-log-builder`; synthesize a top-3 inline only when absent.
- Do not manage the live project or write campaign copy — hand off to the right sibling (see list above).
- Do not use `[assumed]` as a reason to skip questioning an estimate — flag it for human confirmation.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and launch constraints loaded (or fallback executed)?
- All five phases present with named milestones and verifiable gate criteria?
- Critical path traced, labeled, and float to launch stated?
- Every critical-path task has a named owner or an explicit TBD risk flag?
- Planning Fallacy audit run: duration-realism, dependency-integrity, owner-completeness, gate-criteria, float-adequacy, resource-concentration all checked?
- Review verdict is GO / GO WITH CONDITIONS / NO-GO with a one-sentence rationale?
- Recommended changes table included if any check failed?
- `[assumed]` durations surfaced for human sign-off?
- Plan artifact saved to `./plans/` if the user requested persistence?
