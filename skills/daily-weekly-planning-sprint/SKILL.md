---
name: daily-weekly-planning-sprint
description: >
  Incomplete tasks + calendar + priorities + energy preferences → energy-matched time-blocked daily
  plan and GTD-style weekly review with next-week commitments. Runs in two modes: Daily Sprint (today's
  schedule, energy-matched to task type, time-blocked, with a declared Most Important Task) and Weekly
  Review (GTD-style capture, clarify, organize, reflect, engage cycle plus next-week commitment list).
  Pulls the active brand context lightly to honor voice preferences on any shareable output (standups,
  status updates, async planning notes). Composes with eisenhower-matrix-weekly-priorities-sorter to
  sort the backlog, okr-suite to anchor week priorities to OKRs, and meeting-prep-follow-up-pack to
  handle high-leverage meetings. Use when the user says "plan my day," "weekly review," "help me
  prioritize this week," "what should I focus on today," "time-block my schedule," "I have too much
  on my plate," "GTD review," "brain dump," or pastes a messy task list and asks where to start.
---

# Daily & Weekly Planning Sprint

A messy task list is not a plan. This skill turns incomplete tasks, calendar constraints, and energy levels into an actual time-blocked schedule — and runs a GTD-style weekly review that clears the backlog and commits to what matters next week.

Two modes, one discipline. Neither produces a wish list; both produce a committed schedule with a declared Most Important Task and explicit decisions about what does not get done today.

---

## Skills this calls

- **`brand-brain`** (light-touch, always first) — loads the active brand's voice + format preferences so any shareable output (async standup, planning note, team status) honors the user's established voice. Does not impose brand positioning into internal planning artifacts; keep it light.
- **`eisenhower-matrix-weekly-priorities-sorter`** — when the user has a large undifferentiated backlog (5+ items with no clear stack rank), delegate sorting to this skill before time-blocking.
- **`okr-suite`** — when the user wants week-level commitments anchored to OKRs or quarterly goals, compose with this skill for the "Reflect" and "Engage" phases.
- **`meeting-prep-follow-up-pack`** — when a high-leverage meeting appears on the calendar, delegate prep/follow-up work to this skill rather than treating the meeting as a planning black box.
- **`async-standup-writer`** — when the user wants to publish the day's plan or weekly commitments as a team standup/status, delegate formatting to this skill.
- **`pre-mortem-post-mortem-generator`** — during the weekly review "Reflect" phase, when an initiative missed or overran significantly, offer to run a lightweight post-mortem.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (light-touch; voice + format only)
Step 1  Detect mode          ──► Daily Sprint | Weekly Review
Step 2  Collect inputs        ──► tasks, calendar, energy, OKRs/goals (ask for missing pieces)
Step 3  Sort + prioritize     ──► compose eisenhower-matrix-weekly-priorities-sorter if backlog is large
Step 4  Build the plan        ──► time-block (daily) or GTD cycle (weekly)
Step 5  Self-review + output  ──► MIT declared, conflicts resolved, deferral decisions explicit
Step 6  Save the artifact     ──► ./planning/[date]-daily.md or ./planning/[YYYY-Www]-weekly.md
```

---

## Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). For planning artifacts, use only the voice adjectives and format preferences from the returned digest — do not force brand positioning into a personal schedule. If `brand-brain` is absent, read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, proceed with a neutral professional voice.

---

## Mode A — Daily Sprint

Triggered by: "plan my day," "what should I focus on today," "time-block my schedule," or any date-specific planning request.

### Framework: MIT + Time-Blocking + Energy Matching (Cal Newport / GTD hybrid)

**1. Capture the inputs (ask for any missing)**

| Input | What to ask if absent |
|---|---|
| Task list (incomplete / today's backlog) | "Paste your open tasks for today." |
| Calendar (meetings, fixed blocks) | "Any meetings or fixed commitments today?" |
| Energy arc | "When is your best focus window — morning, midday, or afternoon?" |
| Most Important Task (MIT) candidate | "If you only shipped one thing today, what would move the needle most?" |
| Carry-overs / deadlines | "Anything overdue or due today?" |

If the backlog has 5+ unsorted items, call `eisenhower-matrix-weekly-priorities-sorter` before continuing.

**2. Declare the MIT**

Before time-blocking, force one decision: the single Most Important Task. The MIT is the one item that, if completed, makes today a success regardless of everything else. It must be:
- Concrete (completable in one session, not "work on X")
- High-leverage (tied to a goal or OKR, or a hard deadline)
- Protected (gets the first uninterrupted focus block)

If the user can't identify the MIT, ask: "What would you feel best about having done by end of day?"

**3. Match tasks to energy, not clock**

Three energy tiers:
- **Deep work** (writing, analysis, complex decisions, creative) → goes in the peak focus window
- **Shallow work** (email, Slack, admin, scheduling, quick reviews) → goes in low-energy slots
- **Meetings** → schedule around energy peaks, not in them when avoidable

Explicitly protect the peak window. Do not fill it with shallow tasks because they are easier.

**4. Time-block the schedule**

Build a contiguous schedule with named blocks (not a task list):

```
## Today — [Day, Date]
MIT: [one sentence]

| Time | Block | Task(s) |
|------|-------|---------|
| 09:00–10:30 | Deep work (protected) | [MIT] |
| 10:30–11:00 | Shallow batch | [email / Slack / quick reviews] |
| 11:00–12:00 | Meeting | [meeting name] |
| ...         | ...    | ... |

Deferred (not today, not forgotten):
- [item] → [date / person / condition]

Dropped (won't do):
- [item] → [reason in one line]
```

**5. Declare deferral and drop decisions**

Every item on the original list must be assigned to a block, deferred explicitly, or dropped. "I'll get to it later" is not a decision — assign a day or a person.

---

## Mode B — Weekly Review

Triggered by: "weekly review," "help me plan next week," "GTD review," "I want to close out this week," or end-of-week planning requests.

### Framework: GTD Five Phases (Allen) + OKR Pulse Check

**Phase 1 — Capture (get it all out)**

Sweep every collection point: open browser tabs, notebook pages, Slack bookmarks, email flags, calendar scratchpad, voice memos, mental open loops. Paste or dictate everything. The capture is not the plan — the goal is an empty head.

**Phase 2 — Clarify (what is each item, actually?)**

For every captured item, decide:
- Is it actionable? No → trash / reference / someday-maybe list.
- Does it take < 2 minutes? Do it immediately (do not schedule it).
- Does it belong to someone else? Assign and move to "waiting for."
- What is the single next physical action? Rewrite vague items as concrete next actions.

**Phase 3 — Organize (bucket the actions)**

| Bucket | Criteria |
|---|---|
| This week's commitments | Must happen this week; tied to OKR or hard deadline |
| Next-week candidates | Important but not urgent; scheduled tentatively |
| Someday/maybe | Good ideas; no near-term commitment |
| Waiting for | Delegated; needs follow-up date |
| Reference | No action; just needs filing |

If OKRs are available (or the user wants to pull them), call `okr-suite` to score next-week candidates against current OKRs and surface misalignment.

**Phase 4 — Reflect (what did last week actually produce?)**

Answer four questions:
1. Did I complete my MIT each day? (Yes/No/Partial — be honest)
2. Where did time go that wasn't planned? (calendar vs. actuality gap)
3. What is one thing I would do differently next week?
4. What project or commitment is drifting and needs a decision?

If a significant initiative missed or overran, offer to run `pre-mortem-post-mortem-generator`.

**Phase 5 — Engage (commit to next week)**

Produce a concrete next-week commitment list:
- 1 Most Important Outcome for the week (not a task — an outcome)
- 3–5 committed tasks mapped to days (not a wish list)
- Standing meetings accounted for
- One item deliberately deferred with a date
- One item dropped with a one-line reason

```
## Week [Www YYYY] Review + [Next week] Plan

### Last week — honest ledger
MIT completion: [X/5 days]
Biggest win: 
Biggest miss:
Drift flag: 

### Next week commitments
Most Important Outcome: 

| Day | Task | OKR / Goal |
|-----|------|------------|
| Mon | ...  | ...        |
| ...

Deferred: [item] → [date]
Dropped: [item] → [reason]
Waiting for: [item, person, due date]
```

---

## Principles (Non-Negotiable)

- **MIT first.** Declare the Most Important Task before time-blocking. A schedule with no MIT is a schedule that drifts.
- **Explicit deferral and dropping are decisions.** "I'll do it later" is not on the plan. Assign a day, assign a person, or drop it with a reason.
- **Energy matching is load-bearing.** Deep work in peak windows is not a preference — it is the architecture. Shallow work filling the peak window is a planning bug.
- **Weekly review is a commit ceremony.** It ends with a written commitment, not just a cleaned-up list. "I will do X on Wednesday" is a commitment. "I want to get to Y" is not.
- **Honesty over comfort.** The reflect phase requires an honest ledger of what actually happened vs. what was planned. Without it, the same overruns repeat.
- **Brand-brain is light-touch here.** Load voice and format preferences; do not force positioning language into a personal planning doc.

---

## What Not to Do

- Do not produce a task list and call it a schedule. Time-blocking requires actual clock blocks.
- Do not skip the MIT declaration and go straight to filling the calendar.
- Do not carry forward overdue items silently — surface them explicitly, then assign or drop.
- Do not let the weekly review end with "next week's list" without day-level commitments on the top items.
- Do not put deep creative or analytical work in low-energy slots to accommodate meetings that could be rescheduled.
- Do not call eisenhower-matrix-weekly-priorities-sorter and then re-sort the result here — trust the output.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice/format preferences honored on any shareable artifact?
- MIT declared (one item, concrete, completable, tied to a goal or deadline)?
- Every input task assigned to a time block, explicitly deferred, or explicitly dropped?
- Energy arc honored — deep work in peak focus window?
- Weekly review: all five GTD phases completed; "Reflect" section has honest ledger, not aspirational recap?
- Next-week output shows an Outcome (not just tasks) + day-level commitments for top items?
- Artifact saved to `./planning/[date]-daily.md` or `./planning/[YYYY-Www]-weekly.md`?
