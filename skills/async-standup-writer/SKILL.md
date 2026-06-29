---
name: async-standup-writer
description: >
  Turns a raw brain-dump — yesterday's tasks, today's plan, blockers, and
  anything worth flagging — into a ready-to-post async standup or weekly
  status digest. Outputs Slack-formatted prose, Notion bullet rows, or a
  short narrative digest depending on the channel and cadence the user names.
  Applies the active brand's preferred voice and any team-specific format
  conventions so shareable updates stay on-tone without extra editing.
  Handles three modes: Daily standup (3-section Yesterday / Today / Blockers),
  Weekly digest (accomplishments / next-week commitments / risks), and
  Sprint/iteration close (velocity summary + carry-over rationale). Calls
  brand-brain on every run for voice — but keeps it light for internal
  updates; doesn't inject marketing positioning into a team sync.
  Use whenever the user says "write my standup," "draft my status update,"
  "format my weekly update," "standup post," "async standup," "EOD summary,"
  "weekly status," "sprint recap," or pastes raw task notes and asks for
  help posting them.
---

# Async Standup Writer

Standups don't write themselves, and bad ones waste everyone's time. This
skill turns your raw notes into a clean, scannable async post that actually
gets read — formatted for the right channel, pitched at the right level of
detail, and written in the voice that fits how your team communicates.

It writes standups and status digests only. It does not run retros, generate
action items from transcripts (that's `meeting-prep-follow-up-pack`), or
build full project status reports for leadership (that's
`stakeholder-update-status-writer`). Know the scope, stay in it.

---

## Skills this calls

- **`brand-brain`** (always first, light-touch) — loads the active brand's
  voice adjectives and any format conventions logged there. For internal
  updates this means voice/tone only; brand positioning does not belong in a
  team standup. Fallback: read `~/.brandbrain/brands/.active` + that brand's
  `brand.md`; if absent, default to clear, direct, low-friction prose and
  note the assumption.
- **`meeting-prep-follow-up-pack`** — call when the user has meeting notes or
  a transcript and wants action items extracted before or after the standup.
- **`stakeholder-update-status-writer`** — call when the audience is a
  director, board, or external stakeholder and the format needs to shift from
  standup to a narrative status report with risks, decisions, and next steps.
- **`eisenhower-matrix-weekly-priorities-sorter`** — call when the user's
  "today's plan" is a chaotic task dump that needs triage before a clean
  standup can be written.

---

## How a run works

```
Step 0  Load brand voice      ──► call brand-brain (voice/tone, light)
Step 1  Classify the mode     ──► Daily | Weekly | Sprint close
Step 2  Parse the raw input   ──► extract done / doing / blockers / flags
Step 3  Apply the framework   ──► SCBF structure (see below)
Step 4  Fit the channel        ──► Slack prose | Notion rows | plain text
Step 5  Self-check            ──► quality checklist, then post
```

---

## Step 0 — Load brand voice (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before writing a
single word. Pass the user's request. Receive voice adjectives and any
standing format notes. Apply them to tone only — do not surface product
positioning, ICP messaging, or offer copy in a team standup.

**Why bother for an internal doc?** Because "direct and punchy" vs.
"warm and discursive" is a real choice that affects whether teammates
actually read the update. Honor the voice the user has established; don't
default to robotic bullet-point soup.

---

## Step 1 — Classify the mode

| Keyword or context | Mode |
|---|---|
| "standup," "daily," "EOD," "end of day," day-scoped tasks | **Daily** |
| "weekly," "week in review," "this week," "next week" | **Weekly** |
| "sprint," "iteration end," "sprint close," velocity | **Sprint close** |

When unclear, default to **Daily** and confirm at the end.

---

## SCBF structure (Status · Commitment · Blockers · Flags) — our working model

SCBF is a homegrown checklist we use here, not an established industry
framework. Every standup mode maps onto this structure. The labels change;
the logic doesn't.

| Slot | Daily | Weekly | Sprint close |
|---|---|---|---|
| **S — Status** | Yesterday: what shipped or moved | Accomplishments: outcomes, not activity | Velocity: tickets closed, goals met |
| **C — Commitment** | Today: what I'm taking on | Next week: what I'm committing to | Carry-over: what moves and why |
| **B — Blockers** | Blockers: what's in my way right now | Risks: what could slip | Impediments: systemic blockers |
| **F — Flags** | Optional: FYI, links, early warnings | Optional: decisions needed, callouts | Optional: process or tooling notes |

**Outputs over inputs.** Standups describe what moved, not how busy you
were. "Finished subscriber CSV export for Q2 cohort" beats "worked on the
export thing." If the raw input is activity-logged, translate it to outcomes
before writing.

**Size the update to the audience.** A 3-person startup standup needs four
lines. A cross-functional weekly digest needs a short paragraph per zone.
Scan any format conventions from `brand-brain` or ask the user once; then
apply consistently.

---

## Mode: Daily standup

**Input needed:** yesterday's done tasks, today's planned tasks, any
blockers. Flags optional.

**Output shape — Slack prose (default):**

```
:white_check_mark: *Yesterday*
[1–3 outcome sentences or tight bullets; shipped/moved/resolved framing]

:spiral_calendar_pad: *Today*
[1–3 commitments; specific and bounded, not "work on X"]

:no_entry_sign: *Blockers*
[Bullet each. If none: "None." Never omit the section.]

:pushpin: *FYI* (only if flags exist)
[One-liners; links; early warnings]
```

Keep it under ~120 words unless the user has specified a longer format.
Slack-format prose reads better than a wall of nested bullets.

**Notion row output** (when user says "Notion" or "table"): produce a clean
four-column row — Date | Done | Doing | Blockers — formatted as pipe-table
Markdown, ready to paste.

---

## Mode: Weekly digest

**Input needed:** tasks completed this week, next week's commitments, any
risks or decisions needed. 

**Output shape:**

```
## Week of [date range]

**Shipped / accomplished**
[2–4 outcome bullets; link artifacts where possible]

**Next week**
[2–4 commitments with clear owners/deadlines where known]

**Risks / decisions needed**
[Bullet each with a suggested resolution path. If none: "None."]

**Callouts** (optional)
[FYI context the team should have; one-liners]
```

Weekly digests are the record. Write them so a teammate who missed the week
can catch up in 45 seconds. Avoid insider shorthand unless the user's brand
voice says the team norm is casual.

---

## Mode: Sprint close

**Input needed:** sprint goal, completed tickets (with IDs if provided),
carry-over items and reasons, any process flags.

**Output shape:**

```
## Sprint [N] Close

**Goal:** [restate from input or ask]
**Outcome:** [Met / Partial / Missed + one sentence why]

**Shipped**
[Bullet with ticket ID + outcome; link to PR/deploy if provided]

**Carry-over**
[Ticket + reason for slip; no blame, root cause framing]

**Impediments / process notes**
[What slowed the team systemically; not individual callouts]
```

Carry-over rationale matters. "Scope grew when the API spec changed" is
useful. "Didn't finish" is not. If the user's input doesn't have a reason,
ask — don't invent one.

---

## Channel formatting rules

| Channel | Format | Length |
|---|---|---|
| Slack | Emoji section headers, prose or tight bullets, no raw Markdown heading syntax | ≤ 150 words daily, ≤ 300 words weekly |
| Notion | Pipe-table row or H3 sections; Markdown heading syntax fine | Match mode template |
| Email / doc | Plain heading sections (##); no Slack emojis unless user specifies | Match mode template |
| Linear / Jira comment | Plain text, ticket IDs as plain text refs | Tight bullets |

When the user doesn't specify, default to Slack and note the assumption.

---

## Persistence

Save weekly digests and sprint closes to `./ops/standups/[YYYY-MM-DD]-[mode].md`
when the user says "save" or "archive." Daily standups are ephemeral by
default — output inline and don't write to disk unless asked.

---

## Principles

- **Brand-brain first, light-touch.** Voice adjectives apply; offer
  positioning does not belong in a standup.
- **Outcomes, not activity.** Always translate task logs into what moved, not
  how long it took.
- **Blockers are mandatory.** Never silently drop the blockers section.
  "None" is a valid entry; omitting the section hides that the check was
  made.
- **Right size for the audience.** Three-person teams don't need six-bullet
  subsections. Distributed orgs across time zones need more context. Ask or
  infer from brand-brain's format conventions.
- **Specific commitments.** "Work on the dashboard" is not a commitment.
  "Ship the filters panel to staging by EOD" is.
- **Don't invent.** If the input is thin, ask one round of clarifying
  questions rather than padding the update with plausible-sounding tasks.

---

## What not to do

- Don't write the standup before `brand-brain` returns voice context.
- Don't pad a thin input with fake tasks or filler progress language.
- Don't omit the blockers section — even when it's "None."
- Don't inject brand positioning, offer messaging, or ICP copy into an
  internal team post.
- Don't produce a wall of nested bullets when prose reads more naturally
  (especially for Slack).
- Don't build a full retro, action-item log, or stakeholder deck — those
  are other skills. Flag the boundary and hand off.

---

## Quality checklist (self-review before posting)

- [ ] `brand-brain` called; voice applied to tone only, not positioning?
- [ ] Mode correctly classified (Daily / Weekly / Sprint close)?
- [ ] All SCBF slots present — Status, Commitment, Blockers, Flags?
- [ ] Blockers section present even if "None"?
- [ ] All items framed as outcomes, not activity?
- [ ] Commitments are specific and bounded (not "work on X")?
- [ ] Format matches the named channel (Slack / Notion / plain)?
- [ ] Length appropriate for the mode and audience?
- [ ] No invented tasks or proof; thin input prompted a clarifying question?
- [ ] Weekly/sprint outputs offered for save to `./ops/standups/`?
