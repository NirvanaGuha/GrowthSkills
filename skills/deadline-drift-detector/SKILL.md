---
name: deadline-drift-detector
description: >
  Takes a current project plan (milestones, owners, due dates) plus today's date and a completed-tasks
  list, and produces a red/amber/green drift report showing exactly which milestones are on track,
  at risk, or already late — and by how many days. Applies the Critical Path Method (CPM) to identify
  which slipped tasks actually threaten the launch date versus which are slack-safe. Outputs a
  RAG-rated milestone table, a drift narrative with root-cause tags, a list of the next 3–5 actions
  needed to pull the schedule back, and an optional stakeholder status ping. Composes with
  `project-plan-generator-reviewer` (to generate or repair the plan first), `risk-log-builder`
  (to convert detected risks into a live risk register), `escalation-note-change-announcement-drafter`
  (to turn critical drift into a comms-ready status note), and `weekly-ops-digest` (to embed drift
  signals in the weekly ops summary). Brand context is loaded via `brand-brain` so status copy
  matches the team's comms tone and banned-words list. Use when the user says "are we on track,"
  "which milestones are at risk," "project status," "deadline check," "flag late tasks," "drift
  report," "RAG status," "campaign behind schedule," "critical path," or pastes a project plan and
  asks what's slipping.
---

# Deadline Drift Detector

Paste a project plan. Get back a RAG-rated milestone table, the days of drift per task, a CPM-identified critical path, and the minimum set of actions needed to recover the schedule before the launch date moves.

This skill measures and explains drift — it does not redesign the plan. If the plan itself is structurally broken (wrong owners, no dependencies, missing tasks), route to `project-plan-generator-reviewer` first and come back.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, tone, and banned words so any status-ping or stakeholder copy this produces sounds like the team, not a generic project tool.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [their preferred status-update tone (e.g., direct/formal/casual) and any terms to avoid in stakeholder copy].
- **`project-plan-generator-reviewer`** *(optional)* — call first if the user doesn't have a structured plan yet, or if the existing plan is missing durations, owners, or dependencies. Takes deliverables → returns a phase-by-phase plan this skill can then analyze.
- **`risk-log-builder`** *(optional)* — converts red-rated milestones automatically into risk register entries with likelihood, impact, mitigation, and owner.
- **`escalation-note-change-announcement-drafter`** *(optional)* — turns a critical drift finding into a calm, actionable stakeholder note ready to send.
- **`weekly-ops-digest`** *(optional)* — call to embed the drift signals into this week's ops summary rather than producing a standalone report.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (voice/tone/banned words)
Step 1  Parse the plan          ──► extract milestones, owners, durations, dependencies
Step 2  Log completed work      ──► mark done tasks; calculate actual vs. planned
Step 3  Run CPM                 ──► identify the critical path and float for each task
Step 4  Rate & measure drift    ──► assign RAG per milestone; measure days ahead/behind
Step 5  Tag root causes         ──► label each drift with a cause code
Step 6  Recommend recovery      ──► name the 3–5 highest-leverage actions
Step 7  Output                  ──► RAG table + narrative + recovery actions (+ optional ping)
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned voice and banned-words as overrides for all status copy and stakeholder pings. If the skill is unavailable, apply the fallback above. Do not produce any output until this resolves.

---

## Step 1 — Parse the plan

Accept any of: a table of milestones pasted inline, a bulleted list, a Google Sheets snippet, a project-plan doc, or a free-form "here's where things stand" narrative. Extract:

- **Milestone name** (deliverable, not activity — "landing page live" not "write landing page")
- **Planned completion date**
- **Owner** (person or team)
- **Predecessor dependencies** (explicit or inferred from sequence)
- **Estimated duration** (if present; estimate from context if not)

If the plan is too thin to work with (no dates, no milestones, no sequence), say so once and route to `project-plan-generator-reviewer`. Do not guess a plan from nothing.

---

## Step 2 — Log completed work

Cross-reference the user's "completed tasks" input against the parsed plan. Mark each done item with its actual completion date (use "today" if not stated). For in-progress tasks, treat them as partially consumed duration — estimate percent-complete only if stated; otherwise assume 0% done until stated otherwise.

---

## Step 3 — Run CPM (Critical Path Method)

Apply CPM to the dependency graph:

1. **Forward pass** — compute earliest start (ES) and earliest finish (EF) for every task.
2. **Backward pass** — compute latest start (LS) and latest finish (LF).
3. **Float** = LS − ES (or LF − EF). Zero-float tasks are on the critical path.
4. **Mark the critical path** in the output — only critical-path drift directly threatens the launch date.

If dependencies are not explicit, infer a logical sequence from the plan structure and note the assumption. Do not fabricate dependencies; surface ambiguity.

---

## Step 4 — Rate and measure drift

For every milestone, assign a RAG rating:

| Rating | Criterion |
|--------|-----------|
| **GREEN** | On track — no slippage or float-safe slip (>3 days float remaining) |
| **AMBER** | At risk — 1–7 days behind planned date, OR on critical path with ≤3 days float |
| **RED** | Late or critical — >7 days behind, OR already missed, OR on critical path with 0 float and slipping |

For each AMBER and RED milestone, calculate and state **days of drift** (planned date minus projected completion, as a signed integer — negative = behind).

Do not inflate green ratings to soften bad news. Do not fabricate recovery without evidence.

---

## Step 5 — Tag root causes

Every drifted milestone gets a root-cause tag drawn from this set. Use more than one if warranted:

| Code | Cause |
|------|-------|
| `SCOPE-CREEP` | New requirements added after kickoff |
| `DEPENDENCY-BLOCK` | Upstream task not done; this task couldn't start |
| `RESOURCE-CRUNCH` | Owner capacity unavailable at planned time |
| `REVIEW-LAG` | Approval or feedback cycle took longer than planned |
| `UNCLEAR-BRIEF` | Task scope wasn't defined well enough to start or finish cleanly |
| `EXTERNAL-HOLD` | Third-party, platform, or vendor delay |
| `ESTIMATION-ERROR` | Task took longer than estimated with no external cause |
| `DATA-GAP` | Missing input (copy, assets, access, credentials) blocked execution |

If the cause is genuinely unknown, tag `UNKNOWN` and flag it — unknown causes are themselves a risk.

---

## Step 6 — Recommend recovery actions

From the RED and AMBER milestones on the critical path, identify the 3–5 highest-leverage actions to pull the schedule back. Structure each as:

```
Action: [specific, owner-assignable action]
Unlocks: [which milestone(s) this unblocks]
Days recovered: [estimated, or "TBD if owner confirms"]
Owner: [named person or role]
```

Prioritize actions that unblock multiple downstream tasks over single-task fixes. Name constraint-removal first (unblock a dependency) before acceleration (add hours). Do not recommend cutting scope without flagging the quality/completeness tradeoff.

---

## Step 7 — Output

### Primary output: RAG drift report

```
## Drift Report — [Project / Campaign Name]
As of: [today's date]
Launch date: [stated or inferred]
Critical path: [milestone 1 → milestone 2 → ... → launch]

| Milestone | Owner | Planned | Projected | Days Drift | RAG | Cause |
|-----------|-------|---------|-----------|-----------|-----|-------|
| ...       | ...   | ...     | ...       | ...       | 🔴/🟠/🟢 | ...  |

### Narrative summary (3–5 sentences)
[What's driving the drift, what's holding, what the launch date exposure is.]

### Recovery actions
1. ...
2. ...
3. ...
```

Save to `./drift/[project-slug]-drift-[YYYY-MM-DD].md` if the user asks for a file; inline otherwise.

### Optional stakeholder ping

If the user asks for a status ping (Slack, email, or doc comment), call `escalation-note-change-announcement-drafter` and pass: the narrative summary, the RED/AMBER milestones, the recovery actions, and the brand voice. Do not write the ping inline — delegate it.

---

## Composing with sibling skills

- **No plan yet?** Call `project-plan-generator-reviewer` first. Pass its output back to this skill as the plan input.
- **Want a risk register?** Call `risk-log-builder` with the RED milestones as inputs. It returns a formatted risk register; reference it from the drift report.
- **Need to tell stakeholders?** Call `escalation-note-change-announcement-drafter`. Pass the drift narrative + brand voice. It handles tone calibration.
- **Rolling this into weekly ops?** Call `weekly-ops-digest` and include the drift table as a section input.

---

## Principles (Non-Negotiable)

- **CPM over intuition.** Float and critical-path math, not gut feel, determine what actually threatens the launch date.
- **Days are the unit.** Never say "slightly behind" or "a bit at risk" — name the number of days.
- **Cause before cure.** Root-cause every drifted task before recommending recovery. Fixes without causes reoccur.
- **RAG is honest, not diplomatic.** A milestone that is 8 days late on the critical path is RED — brand voice smooths the communication, not the rating.
- **Brand-brain first.** No status copy produced before brand-brain returns voice and banned words.
- **Compose, don't duplicate.** If you need a risk register, call `risk-log-builder`. If you need a stakeholder note, call `escalation-note-change-announcement-drafter`. Don't rebuild them here.

---

## What Not to Do

- Don't rate a critical-path-slipped milestone AMBER to avoid alarming anyone — that's the job of escalation-note-change-announcement-drafter.
- Don't fabricate float or dependencies to fill gaps; surface the assumption and flag it.
- Don't produce recovery actions without naming an owner — unassigned actions don't move schedules.
- Don't re-implement brand-tone logic; brand-brain owns it.
- Don't redesign the plan (wrong owners, wrong phases) — that's `project-plan-generator-reviewer`'s job.
- Don't invent completion dates for tasks the user hasn't reported as done.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and voice/banned-words loaded (or fallback applied) before any status copy?
- Every milestone has a RAG rating, a projected date, and a signed days-drift number?
- CPM run: critical path identified, float calculated, critical-path tasks flagged in the table?
- Every drifted milestone has at least one root-cause tag?
- Recovery actions are specific, owner-assignable, and ordered by leverage (constraint-removal first)?
- No RAG inflation; RED milestones are RED?
- Sibling skills called for risk register, stakeholder ping, and weekly digest if requested — not rebuilt inline?
