---
name: weekly-ops-digest
description: >
  Turns the week's operational noise — scattered notes, meeting transcripts, Slack threads,
  task logs, and carry-over items — into a single owned weekly plan: a status-and-decision
  recap for last week, a prioritized initiative list with owner and due-date for this week,
  a risks-and-blockers register, and a Slack-ready status ping for stakeholders. Uses the
  Weekly Planning Triangle framework (Review → Decide → Commit) to prevent the most common
  ops failure modes: half-finished last week bleeding into this week unacknowledged, initiatives
  running without a named owner, and blockers sitting invisible until they become crises.
  Calls brand-brain for voice on stakeholder-facing outputs; calls meeting-agenda-action-item-builder
  to process any raw transcripts embedded in the inputs; calls eisenhower-matrix-weekly-priorities-sorter
  when the initiative list needs triage before commitment. Does NOT write campaign briefs, project
  plans, or OKR docs — those are scope-creep; it hands off and names the right skill. Use whenever
  the user says "weekly ops digest," "week in review," "plan my week," "ops update," "weekly
  planning," "what are we doing this week," "weekly status," "pull together the week," "action items
  from last week," or dumps a pile of notes, threads, and transcripts asking for a coherent weekly plan.
---

# Weekly Ops Digest

One job: turn last week's noise and this week's intentions into a single, owned, actionable plan — and a status ping your stakeholders can actually read. Growth teams drown in information and starve for clarity; this skill trades one for the other.

The output is not a diary of activity. It is a decision record for what happened, a commitment register for what's next, and a blocker surface that makes invisible friction visible before it burns a week.

It does not generate campaign briefs, OKR docs, or project roadmaps. It names the right sibling skill and hands off cleanly.

---

## Skills this calls

- **`brand-brain`** (required, light-touch on stakeholder outputs) — loads active brand voice for the Slack ping and any shareable status narrative. Internal action tables do not carry brand positioning; voice applies to tone only. Fallback: read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if absent, default to direct/confident/low-friction prose.
- **`meeting-agenda-action-item-builder`** — call when the user pastes raw meeting notes or a transcript. It extracts decisions and attributed action items (DACI structure); this skill receives that structured output and folds it into the weekly plan.
- **`eisenhower-matrix-weekly-priorities-sorter`** — call when the initiative list has more items than the week can absorb. It applies the urgency/importance matrix; this skill receives the sorted output and maps it to the weekly commitment register.
- **`async-standup-writer`** — call when the user needs a daily-granularity update instead of a weekly one, or wants the weekly plan converted into daily standup format after it is built.
- **`stakeholder-update-status-writer`** — call when the stakeholder audience is VP+, cross-functional leadership, or a board context requiring a full narrative report rather than a Slack ping.
- **`okr-suite`** *(if installed)* — call when this week's initiatives need to be logged against or reconciled with active OKR targets.

---

## How a run works

```
Step 0  Load brand voice (light)   ──► call brand-brain
Step 1  Intake and classify input  ──► notes / transcripts / task logs / explicit goals
Step 2  Run Review pass            ──► close last week (outcomes, decisions, carry-over)
Step 3  Run Decide pass            ──► triage this week's initiative list (call eisenhower if needed)
Step 4  Run Commit pass            ──► assign owners + due dates to surviving initiatives
Step 5  Build the Slack ping       ──► stakeholder-ready, brand-voiced
Step 6  Self-review                ──► quality checklist, then deliver
```

---

## Step 0 — Load brand voice (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before writing a single line. Receive voice adjectives, format conventions, and any standing stakeholder communication preferences. Apply to tone only on the Slack ping and shareable recap; do not inject offer messaging or ICP copy into an internal ops document.

If `brand-brain` is not installed: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If none exists, proceed with direct/confident/action-forward prose and note the assumption inline.

---

## Step 1 — Intake and classify input

Accept any of the following, alone or mixed:

| Input type | Handling |
|---|---|
| Last week's notes (bullet dump, Notion export) | Parse directly into Review pass |
| Meeting transcript or raw notes | Call `meeting-agenda-action-item-builder` first; receive structured action items + decisions |
| Task/project tool export (Asana, Linear, Notion table) | Parse task rows; map status to done/in-progress/carry-over |
| This week's goals or initiative list | Feed directly into Decide pass |
| Explicit "carry-over" flags | Surface in Review as unfinished work before committing forward |
| Nothing (blank slate) | Ask three targeted questions: (1) what shipped last week, (2) what did not, (3) what does this week need to close |

If the input contains a raw transcript, call `meeting-agenda-action-item-builder` **before** proceeding to Step 2. Do not attempt to parse transcripts directly — that skill has the decision and attribution logic.

---

## The Weekly Planning Triangle (the framework)

Every run produces exactly three passes in sequence. Each has a gate condition: you do not move to the next until the current one is complete.

```
       REVIEW
      (last week)
       /       \
    DECIDE ── COMMIT
  (what stays) (who/when)
```

**Why the sequence matters:** skipping Review means carry-over bleeds into this week as phantom capacity. Skipping Decide means the week is overcommitted before it starts. Skipping Commit means initiatives exist without owners — which is the same as not existing.

---

## Pass 1 — Review (close last week)

**Output: Last Week Closeout**

For each initiative that was in-flight last week:

| Initiative | Status | Outcome / why |
|---|---|---|
| [Name] | Done / Partial / Slipped | [One-line outcome or root cause] |

Rules:
- **Done** = the committed outcome was delivered, not just "worked on."
- **Partial** = meaningful progress but not closed; requires a carry-over decision.
- **Slipped** = did not start or materially progress; requires a root cause (one line; no blame, systemic framing preferred).

**Decisions made last week** (pulled from meeting notes / `meeting-agenda-action-item-builder` output):

| Decision | Owner (Approver) | What it unblocks |
|---|---|---|
| [What was decided] | [Name] | [Context] |

If no decisions were made and no meetings happened, say so. A week with no decisions is a signal, not a default.

**Carry-over list:** every Partial or Slipped item generates a carry-over candidate. Carry-overs are NOT automatically committed to this week — they go into the Decide pass first.

---

## Pass 2 — Decide (triage this week)

**Gate:** Pass 1 closeout must exist before this runs.

Collect all candidates for this week:
- Carry-over items from Pass 1
- New initiatives the user named
- Any action items from `meeting-agenda-action-item-builder`

If the list exceeds approximately 6 initiatives for a single person or 10 for a small team: call `eisenhower-matrix-weekly-priorities-sorter`. Pass it the candidate list and the user's stated goals for the week. Receive back a sorted output; use the top tier for this week's commitment register, move the rest to the parking lot.

If the list is manageable without triage, apply the three-question gate yourself:

1. **Does this move a needle that matters this week?** (Not eventually — this week.)
2. **Is there a named human who can own it to completion?**
3. **Is the "done" state specific enough to know when it's done?**

Any initiative that fails two or more gates goes to the Parking Lot, not the commitment register.

---

## Pass 3 — Commit (this week's plan)

**Gate:** Decide pass must exist; every surviving initiative must pass the three-question gate.

**Output: This Week's Commitment Register**

```
## This Week — [date range]

| # | Initiative | Owner | Due | Done-state |
|---|---|---|---|---|
| 1 | [name] | [person] | [day/date] | [specific outcome] |
| … | | | | |

**Blockers & risks**
| Blocker | Impact | Owner to resolve | Target resolution |
|---|---|---|---|
| [What's stuck] | [What it blocks] | [Name] | [Date or "urgent"] |

**Parking Lot (not this week)**
- [Initiative] — revisit: [when / what needs to change for this to be worth scheduling]
```

Rules:
- Every initiative has exactly one owner. Not "the team," not "TBD" unless genuinely unresolved (flag those; they are risks, not commitments).
- Done-state is specific: not "work on X" but "deliver X to Y by Z."
- Blockers section is mandatory. "None" is valid. Omitting it means the check was never made.
- Parking Lot items get a revisit note — they are not discarded, they are scheduled.

---

## The Slack Ping (stakeholder output)

After the commitment register is built, generate a Slack-ready status message. This is the only output that uses brand voice from `brand-brain`.

**Format:**

```
:calendar: *Week of [date range]*

*Last week:* [2–3 bullet outcomes; what shipped/decided]
*This week:* [2–3 bullet commitments; owner in brackets where stakeholders care]
*Blockers needing input:* [1–3 items with suggested action; or "None"]

[Optional: one-line callout for anything a stakeholder should know or decide]
```

Length target: under 120 words. Stakeholders read it in a Slack thread; it must work at a scroll-past glance. If it requires context to understand, that context belongs in a linked doc, not the ping.

---

## Saving artifacts

Save the full digest (Passes 1–3) to `./ops/weekly/[YYYY-MM-DD]-weekly-digest.md` when the user says "save" or the output is substantial (more than one week's commitments and decisions). The Slack ping is inline-only unless asked.

Never overwrite a prior week's file — always use the date-stamped name.

---

## Principles

- **Review before you commit.** Carry-over that skips the Review pass is a lie your week tells itself; it reappears mid-week and wrecks the plan.
- **One owner per initiative.** Shared ownership means no ownership. Flag it; do not paper over it.
- **Specific done-states only.** "Work on" is not a commitment. If you cannot describe what done looks like, the initiative is not ready to be committed.
- **Blockers are mandatory.** A week plan with no blocker register is optimistic fiction. Name them or confirm there are none — but make the call explicit.
- **Brand voice on stakeholder outputs; neutral precision on internal tables.** The Slack ping carries brand voice. The commitment register carries clarity.
- **Parking Lot is not a trash bin.** Every parked item gets a revisit condition. Items without one are dropped on purpose, not by accident — say so.
- **Don't invent.** If the input is thin, ask the three baseline questions (what shipped, what did not, what does this week need to close) before producing a plan.

---

## What not to do

- Do not run Pass 2 or 3 before Pass 1 closes last week. Sequence is the discipline.
- Do not write "TBD" as a final owner without flagging it as a risk to the week's plan.
- Do not produce a commitment register with more initiatives than the team/person can actually close — the Decide pass exists to prevent this.
- Do not drop blockers because they feel awkward to raise. Surface them; that is the point.
- Do not build OKR docs, project plans, campaign briefs, or roadmaps — name the right skill and hand off.
- Do not parse raw meeting transcripts directly — call `meeting-agenda-action-item-builder`.
- Do not inject brand offer messaging or ICP positioning into the internal commitment register.

---

## Quality checklist (self-review before delivering)

- [ ] `brand-brain` called; voice applied to Slack ping only, not to the commitment tables?
- [ ] Raw transcripts routed through `meeting-agenda-action-item-builder` before being folded in?
- [ ] Pass 1 (Review) complete with done/partial/slipped status and a carry-over list?
- [ ] If decisions were made last week, are they in the decision record?
- [ ] Pass 2 (Decide): overloaded list sent to `eisenhower-matrix-weekly-priorities-sorter`, or three-question gate applied manually?
- [ ] Pass 3 (Commit): every initiative has a single owner, a specific done-state, and a due date?
- [ ] Blockers section present even if "None"?
- [ ] Parking Lot items have revisit conditions, not just names?
- [ ] Slack ping under 120 words and brand-voiced?
- [ ] Full digest offered for save to `./ops/weekly/[YYYY-MM-DD]-weekly-digest.md`?
- [ ] No invented tasks, fake outcomes, or plausible-sounding filler; thin input prompted the three baseline questions?
