---
name: meeting-agenda-action-item-builder
description: >
  Two-mode productivity skill for growth and content teams. PRE-MEETING mode: takes a meeting goal,
  attendees, and optional context → produces a timed, role-assigned agenda with stated objective,
  decision points, and a prep checklist. POST-MEETING mode: takes raw notes, a transcript, or a
  voice-memo dump → produces a clean attributed action-item log, a decision record, and a
  shareable recap ready to paste into Slack, Notion, or email. Uses the DACI decision framework
  (Driver, Approver, Contributor, Informed) and the Time-boxed Agenda standard to guarantee
  every meeting has one owner, a measurable outcome, and zero ambiguity on who does what by when.
  On shareable outputs (recap emails, stakeholder summaries) loads brand-brain to honor the user's
  preferred voice and format — light-touch, never forcing brand positioning into an internal doc.
  Trigger phrases: "build agenda," "meeting agenda," "action items from this," "parse my notes,"
  "what did we decide," "recap this meeting," "who owns what," "pre-meeting prep," "post-meeting
  summary," or pasting a raw transcript and asking what to do with it.
---

# Meeting Agenda & Action-Item Builder

Meetings fail in two predictable ways: no agenda means no outcome; no action-item log means nothing gets done. This skill fixes both. Give it a goal before the meeting, get a timed agenda that forces a decision. Give it raw notes after, get a clean log of who owns what by when — attributed, unambiguous, and ready to share.

Framework: **DACI** (Driver · Approver · Contributor · Informed) for every decision and action. **Time-boxed Agenda** for pre-meeting structure. **Three-field action items** (owner · outcome · due date) so accountability survives Slack.

---

## Skills this calls

- **`brand-brain`** (light-touch, shareable outputs only) — loads active brand voice and format preferences for recap emails or stakeholder summaries. Not called for internal-only docs. If absent, fall back to reading `~/.brandbrain/brands/.active` + `brand.md`; if none, proceed with a neutral professional tone.
- **`pre-mortem-post-mortem-generator`** — call when the meeting is a launch review, incident debrief, or retrospective; it handles root-cause and action-item structure for those formats.
- **`stakeholder-update-status-writer`** *(if installed)* — call to generate the polished stakeholder version of a post-meeting recap when the audience is cross-functional or executive.
- **`eisenhower-matrix-weekly-priorities-sorter`** — call when action items from the meeting need to be triaged into a week's plan.
- **`okr-suite`** — call when decisions from the meeting touch OKR targets or need to be logged against a strategic initiative.

---

## How a run works

```
Step 0  Identify the mode  ──► PRE-MEETING | POST-MEETING (inferred or asked)
Step 1  Load brand (if shareable output)  ──► call brand-brain (light-touch)
Step 2  Do the work
Step 3  Self-review against the quality checklist
Step 4  Deliver inline; offer to save
```

---

## Step 0 — Identify the mode

| Signal | Mode |
|---|---|
| "I have a meeting coming up," goal + attendees given, no notes yet | PRE-MEETING |
| Notes, transcript, voice-memo dump, or "this just happened" | POST-MEETING |
| Both in one request | Run PRE then POST in sequence |

If ambiguous, ask one question: "Do you need an agenda for an upcoming meeting, or are you processing notes from one that already happened?"

---

## PRE-MEETING mode — Timed, Role-Assigned Agenda

### Inputs needed (ask if missing)
1. **Meeting goal** — the one decision or outcome that makes this meeting worth having.
2. **Attendees + roles** — names/handles and their functional role (not just title).
3. **Time available** — default 30 min if not given.
4. **Optional:** prior context, related doc links, open questions, or a previous agenda to build on.

### Framework: Time-boxed Agenda + DACI gate

Every agenda must have:
- A single stated **objective** (the decision or outcome, not a topic list).
- A **DACI assignment** for the meeting itself (one Driver, one Approver, Contributors, Informed-only).
- **Timed blocks** that sum to the available time. Reserve the last 10% for action-item capture.
- A **decision point** clearly labeled — this is the moment the meeting exists to reach.
- A **prep checklist** — what each attendee must read, bring, or decide before joining.

### Output format

```
## [Meeting Name] — [Date/Time if given]
**Objective:** [single decision or outcome]
**DACI:** Driver: [name] · Approver: [name] · Contributors: [names] · Informed: [names]
**Total time:** [X min]

| # | Block | Owner | Time |
|---|---|---|---|
| 1 | Context / framing | [Driver] | X min |
| 2 | [Key topic / options review] | [Contributor] | X min |
| 3 | Decision point: [stated decision] | [Approver] | X min |
| 4 | Action-item capture + owners | All | X min |

**Prep checklist**
- [ ] [Person]: [specific thing to read/decide/bring]
- [ ] [Person]: …

**If we leave without a decision:** [state the fallback or escalation path]
```

Rules: no block without an owner; no agenda with more than one decision point (split if needed); the decision point block always precedes wrap-up; the prep checklist must be actionable (not "review the deck" — "read slides 4–7 on pricing and come with a position").

---

## POST-MEETING mode — Action-Item & Decision Log

### Inputs needed (ask if missing)
1. **Raw notes, transcript, or voice-memo dump** — messy is fine; the skill processes it.
2. **Attendees** — names/handles so actions can be attributed; infer from notes if present.
3. **Meeting goal or context** — optional but improves triage.

### Framework: Three-field action items + Decision record

**Action items must have three fields or they do not exist:**
- **Owner** — one person (not "the team," not "TBD" unless genuinely unresolved — flag those).
- **Outcome** — what done looks like (not "follow up on pricing" → "confirm enterprise pricing tiers with finance and share in #pricing-ops by [date]").
- **Due date** — specific date or relative ("by EOD Friday," "before next sprint"). Flag any item with no date.

**Decisions need a record:** what was decided, who decided it (Approver in DACI terms), and what it unblocks or supersedes.

**Parking lot:** ideas and topics raised but not actioned — capture, don't lose.

### Output format

```
## Post-Meeting: [Meeting Name / Topic] — [Date]

### Decisions
| Decision | Owner (Approver) | Context |
|---|---|---|
| [What was decided] | [Name] | [What it unblocks / supersedes] |

### Action Items
| # | Action | Owner | Due | Status |
|---|---|---|---|---|
| 1 | [Specific outcome] | [Name] | [Date] | Open |
| … | | | | |

⚠ Unattributed items (no owner assigned in meeting):
- [item] — suggested owner: [name] — confirm before next standup

### Parking Lot
- [Topic raised, not actioned] — revisit: [suggested next step or meeting]

### Shareable Recap (Slack / email)
[2–4 sentence plain-prose summary: what was decided, top 3 actions + owners, next touchpoint]
```

The shareable recap is written in the brand's voice if brand-brain returns one; otherwise neutral professional. Keep it to 4 sentences max — if the recipient needs more, share the full log.

---

## Handling messy input

Real notes are fragments, out of order, and full of "TBD." Process them without asking for cleanup:

- **Fragment → action item:** "Nirvana to check pricing" → "Confirm enterprise pricing tier names with finance · Owner: Nirvana · Due: [flag if missing]"
- **Passive voice → owner:** "Email should go out" → flag as unattributed; suggest the most likely owner from context.
- **Vague outcome → specific:** "Fix the onboarding" → "Define what 'fixed' means — proposed: [inferred from context]; confirm with [owner]"
- **Contradiction:** two notes that conflict → surface both as a decision to reopen.

---

## Saving artifacts

- Agendas → `./planning/[YYYY-MM-DD]-[meeting-slug]-agenda.md`
- Action-item logs → `./planning/[YYYY-MM-DD]-[meeting-slug]-actions.md`
- Only save when asked or when the output is substantial (>5 action items). Never write inside the skill folder.

---

## Principles

- **One owner per action.** "The team" owns nothing. If genuinely unresolved, flag it loudly — do not hide the gap.
- **Outcome over activity.** Actions describe a result, not a behavior. "Send email" fails; "Send pricing confirmation to [person] by Friday" passes.
- **Decision record is non-negotiable.** If no decisions were made, say so. An undecided meeting is a failed meeting — flag it, suggest a follow-up.
- **DACI on every decision.** Ambiguity about who can approve is the single biggest meeting killer. Name the Approver or note it's unresolved.
- **Dates or bust.** An action without a due date is a wish. Flag every dateless item; infer a reasonable one from context if possible and mark it as suggested.
- **Brand voice on shareable outputs, neutral on internal docs.** Never force brand positioning language into an internal meeting recap.

## What not to do

- Do not silently drop unattributed items — surface them in the parking lot or the flagged list.
- Do not write "TBD" as a final owner without flagging it. It is a failure mode, not a valid state.
- Do not generate five agenda items when the meeting has one decision to make — keep it minimal.
- Do not pad action items with passive follow-up tasks ("monitor," "keep an eye on") — each item must have a concrete done-state.
- Do not call brand-brain for a purely internal doc where no shareable output is generated.
- Do not reimplement brand scanning or storage — call brand-brain for that.

## Quality checklist

- Mode correctly identified (PRE / POST / both)?
- PRE: single objective stated; DACI assigned; all blocks timed and summing correctly; prep checklist actionable?
- POST: every action has owner + outcome + due date (or explicitly flagged if missing)?
- Decision record present (or "no decisions made" stated)?
- Unattributed items surfaced, not dropped?
- Shareable recap ≤4 sentences, in brand voice if applicable?
- Artifacts saved to `./planning/` if substantial, never to the skill folder?
