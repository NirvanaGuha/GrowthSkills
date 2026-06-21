---
name: eisenhower-matrix-weekly-priorities-sorter
description: >
  Task dump or board export → 2×2 Urgent/Important grid with Do/Schedule/Delegate/Delete
  assignments and a ranked top-five for the week. Applies the Eisenhower Decision Matrix
  with strict urgency/importance definitions so a growth marketer walks away with a clear,
  sequenced week — not a colour-coded parking lot. Brand context is loaded via brand-brain
  so strategic priorities align with the active brand's growth goals, ICP, and OKRs. Calls
  experiment-results-analyzer and growth-diagnostic-deep-dive when task lists reference
  live tests or open metrics problems, and cross-checks against any OKRs stored by the
  okr-suite. Designed for the solo operator overwhelmed at the start of the week as well as
  the team lead grooming a sprint. Use when the user says "sort my to-do list," "help me
  prioritise this week," "Eisenhower matrix," "what should I focus on," "I have too much on
  my plate," "task triage," "weekly planning," or pastes a brain dump, a Jira/Linear
  export, a sticky-note list, or a Notion table.
---

# Eisenhower Matrix & Weekly Priorities Sorter

Paste your task dump. Get a ruthlessly sorted week.

This skill does one thing: it turns a chaotic list of tasks into a 2×2 Eisenhower grid with Do/Schedule/Delegate/Delete assignments, then distils that into a ranked top-five the operator can act on immediately. The framework is the _Eisenhower Decision Matrix_ (popularised by Covey's _7 Habits_ Q2 model) — but applied with the strict definitions that make it useful rather than merely decorative: urgency is time-bound consequence, not noise volume; importance is proximity to strategic outcomes, not personal attachment.

The output is a working plan, not a management slide.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's strategic context (growth goals, ICP, OKRs, offer stage) so the importance axis is anchored to real business priorities, not gut feel.
- **`okr-suite`** *(optional, call if OKRs are stored)* — cross-references task list against active OKRs so Q2 tasks are ranked by KR proximity.
- **`experiment-results-analyzer`** *(call when tasks reference a live or recently concluded test)* — resolves whether a test task is genuinely urgent (significant result, decision imminent) or merely feels urgent.
- **`growth-diagnostic-deep-dive`** *(call when tasks reference a metrics problem)* — diagnoses lever priority so analytical tasks land in the right quadrant.
- **`a-b-multivariate-test-designer`** *(recommend, do not auto-invoke)* — if a Q2 task is "design next experiment," hand off there.
- **`daily-weekly-planning-sprint`** *(recommend, do not auto-invoke)* — for energy-matched time-blocking of the confirmed top-five.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (brand goals anchor the importance axis)
Step 1  Ingest the task list ──► parse, normalise, deduplicate; ask for missing context
Step 2  Classify             ──► score each task on U (urgency) and I (importance) axes
Step 3  Assign quadrants     ──► Do / Schedule / Delegate / Delete
Step 4  Rank within Q1+Q2    ──► apply tiebreaker stack; produce top-five
Step 5  Surface the output   ──► matrix table + top-five + delegations + deletions + nudges
Step 6  Save if requested    ──► ./plans/weekly-priorities-[YYYY-MM-DD].md
```

---

## Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** before scoring a single task. It returns the brand's growth goals, OKR stage, offer mechanics, and ICP, which define what "important" means this week. A task that grows ARR, reduces churn, or unblocks an ICP activation milestone scores high on importance; internal busywork scores low regardless of how urgent it feels.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If neither exists, ask the user: "What are your top 1–2 business goals this week?" and proceed with those as the importance anchor.

---

## Step 1 — Ingest and normalise

Accept any input format: bullet list, numbered list, Jira/Linear export, Notion table, or free-form brain dump. For each item:

- Extract the **task name** and any stated deadline, owner, or dependency.
- Normalise vague entries ("email follow-ups") to a concrete, action-led task ("Reply to inbound demo requests from this week").
- Flag duplicates or sub-tasks and collapse them with a note.
- If fewer than five tasks are provided, proceed. If a deadline or dependency is ambiguous and changes the quadrant, ask a single clarifying question before scoring.

---

## Step 2 — Classify on U and I axes

Apply strict definitions — the most common failure mode is confusing noise with urgency.

### Urgency (U): time-bound consequence
A task is **urgent** if and only if missing a near-term deadline (within ~2 business days) produces a material, irreversible negative consequence: a deal goes cold, a campaign misfires, an SLA breaches, a partner window closes. Notification volume, feeling of pressure, or someone else's impatience does not make a task urgent.

| Urgent | Not urgent |
|---|---|
| Fix broken UTM on live paid campaign | Redesign UTM naming convention |
| Respond to prospect who asked for proposal by EOD | Draft a partner outreach template |
| Fix site-down bug blocking conversions | Refactor tracking implementation |

### Importance (I): proximity to strategic outcomes
A task is **important** if completing it directly moves a goal from brand-brain (revenue, activation, retention, SEO authority, experiment velocity). Proximity matters: "write blog post" is important if content is a primary acquisition lever; "fix homepage typo" is not important unless that page is the conversion bottleneck.

Leverage the brand's OKRs if available (call `okr-suite`). Score importance high when the task is on the critical path to a Key Result.

---

## Step 3 — Assign quadrants (the 2×2)

| | **Urgent** | **Not Urgent** |
|---|---|---|
| **Important** | **Q1 — Do** (crises, deadline-driven high-value work) | **Q2 — Schedule** (strategic, proactive, growth-compounding) |
| **Not Important** | **Q3 — Delegate** (interruptions, other people's urgencies) | **Q4 — Delete** (busywork, low-value habits, vanity tasks) |

**Covey's Q2 principle:** the operator's highest-leverage shift is growing Q2 (strategic non-urgent work) at the expense of Q3 (interruptions). Surface this explicitly if the list is overweight Q3. A healthy week for a growth operator is ~60% Q2, ~20% Q1, ~10% Q3 (delegated), ~10% Q4 (removed).

**Red flags to name aloud:**
- More than 4 tasks in Q1: the operator is in constant-crisis mode; diagnose the upstream cause.
- Zero tasks in Q2: strategic growth work has been crowded out; flag this as a compounding risk.
- Q3 tasks with no named delegate: they will drift to Q1; name a delegate or convert to Q4.

---

## Step 4 — Rank within Q1 + Q2 (the top-five)

Q1 tasks come first by default. Within each quadrant, break ties with this stack:

1. **Deadline proximity** — earliest hard deadline first.
2. **Irreversibility** — can this be undone if delayed? (Lower reversibility = higher rank.)
3. **KR proximity** — does this task unblock a Key Result this sprint?
4. **Effort-to-impact** — favour quick wins when two tasks have equal importance and no deadline difference (ICE-style: impact ÷ effort, estimated roughly).

Produce **exactly five items** in the top-five. If Q1+Q2 has fewer than five tasks, promote the highest-scoring Q3 items (if delegatable options exist) or note that this is a light week.

---

## Output format

```markdown
## Eisenhower Grid — [Brand] | Week of [date]

### Q1 — Do (Important + Urgent)
| Task | Why urgent | Why important | Deadline |
|---|---|---|---|

### Q2 — Schedule (Important + Not Urgent)
| Task | Suggested time block | Why important | OKR link |
|---|---|---|---|

### Q3 — Delegate (Not Important + Urgent)
| Task | Suggested delegate or defer | Note |
|---|---|---|

### Q4 — Delete (Not Important + Not Urgent)
| Task | Reason to remove |
|---|---|

---

## Top 5 for the week
1. [Task] — [one-line rationale]
2. …
5. …

---

## Observations
- [Q2 deficit / Q3 overload / crisis-mode flag if applicable]
- [Delegate gap: tasks in Q3 with no named owner]
- [Suggested next step: e.g., hand off to daily-weekly-planning-sprint for time-blocking]
```

Save to `./plans/weekly-priorities-[YYYY-MM-DD].md` when the user asks for a persistent copy or when the list has more than ten items.

---

## Principles

- **Importance is anchored to the brand, not feelings.** Pull it from brand-brain's goals and OKRs. Never let the user's anxiety substitute for a strategic criterion.
- **Urgency is time-bound consequence, not volume.** Slack pings and inbound emails are rarely urgent in the matrix sense. Name this if the user's list is full of reactive work.
- **Q2 is the growth engine.** The matrix's real insight is that compounding work (content, experiments, relationships, systems) is always non-urgent until it's too late. Protect it visibly.
- **One owner per Q3 task.** A delegated task with no name is a dropped task. If there is no owner, it belongs in Q4.
- **Top-five is a commitment, not a wishlist.** Fewer than five wins is better than five tasks left undone. If the list is genuinely too long, say so and ask the user to cut before ranking.
- **Real proof, real priorities.** Do not invent strategic rationale. If the importance of a task is unclear, ask one question rather than assume.

---

## What not to do

- Don't score urgency based on emotional weight or message volume — use the time-bound-consequence test.
- Don't put everything in Q1 to avoid the discomfort of the Delegate/Delete assignment.
- Don't produce the matrix before `brand-brain` returns — importance scores without strategic context are arbitrary.
- Don't re-implement OKR storage or brand scanning here; call `okr-suite` and `brand-brain`.
- Don't time-block the week here; hand that off to `daily-weekly-planning-sprint`.
- Don't save to the skill folder; write plans to `./plans/`.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and strategic goals loaded (or fallback acknowledged)?
- Every task classified with an explicit urgency test (time-bound consequence) and importance test (proximity to brand goals)?
- Q1 items confirmed as genuinely deadline-driven, not merely loud?
- Q3 items each have a named delegate or a reason to promote to Q4?
- Top-five contains exactly five items ranked by the tiebreaker stack, with a one-line rationale per item?
- Q2 deficit or Q3 overload flagged if present?
- Output format rendered cleanly; `./plans/` path offered for saves on lists of >10 items?
