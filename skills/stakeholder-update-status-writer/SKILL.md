---
name: stakeholder-update-status-writer
description: >
  Turns raw project status data into a polished, stakeholder-ready update — for Slack, email, or a
  board-deck slide. Accepts any combination of: pasted progress notes, closed Jira/Linear/Asana
  tickets, sprint retrospective bullets, raw metrics, incident summaries, or a brain dump. Outputs a
  structured context → progress → risks/blockers → decisions → next-steps narrative sized to the
  audience and channel. Uses the SBAR (Situation, Background, Assessment, Recommendation) framework
  as its spine, with a RAG (Red/Amber/Green) health signal so readers know what needs attention at a
  glance. Composes with growth-diagnostic-deep-dive and channel-roi-scorecard for numbers-heavy updates,
  and with okr-suite for OKR progress commentary. Light-touch brand voice for shareable output — loads
  brand-brain to honor the user's preferred tone and banned words without forcing product positioning
  into a standup. Use whenever the user says "write a status update," "draft my stakeholder update,"
  "turn these tickets into a summary," "board update," "leadership summary," "project recap," "weekly
  update email," "progress report," or hands over a task list / ticket export and asks for a narrative.
---

# Stakeholder Update & Status Writer

Raw project data in, polished stakeholder update out. Whether the input is a paste of closed tickets, a brain dump of what shipped, or a full incident timeline, this skill structures it into a clear narrative — calibrated to the audience (team Slack, VP email, board deck) and anchored to a RAG status signal so readers know instantly what needs attention and what doesn't.

It does not generate the underlying metrics. If a numbers story is central to the update, it composes with `growth-diagnostic-deep-dive` or `channel-roi-scorecard` rather than reproducing that analysis here.

---

## Skills this calls

- **`brand-brain`** (always, Step 0) — loads voice adjectives and banned words for any shareable output. Light-touch: honors the user's preferred tone without forcing product positioning into a status update.
- **`growth-diagnostic-deep-dive`** — call when the update requires a growth/traffic/funnel narrative to back a claim. Do not re-derive GA4 data inline.
- **`channel-roi-scorecard`** — call when the update requires spend/ROAS/CPA channel performance commentary.
- **`okr-suite`** — call when the update covers OKR progress, key result health, or a quarterly review signal.
- **`pre-mortem-post-mortem-generator`** — call when the update is an incident or post-launch retro writeup.
- **`experiment-results-analyzer`** — call when the update includes experiment win/loss/inconclusive results.

---

## How a run works

```
Step 0  Brand voice      ──► call brand-brain (voice + banned words; skip positioning injection)
Step 1  Intake           ──► classify the input type; detect audience + channel + urgency
Step 2  RAG triage       ──► assign overall RAG; flag any workstream that is Red
Step 3  Apply SBAR       ──► structure the content; compose with sibling skills for numbers if needed
Step 4  Format           ──► size and format to the channel (Slack / email / deck slide)
Step 5  Self-review      ──► quality checklist before presenting
```

---

## Step 0 — Brand voice (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing any named brand or context. Use the returned voice adjectives and banned-words list as overrides on tone and word choice. Do NOT inject product positioning or marketing messages into a status update — this is an internal or executive communication, not a sales document.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, ask for 2–3 voice adjectives and any words to avoid, then proceed.

---

## Step 1 — Intake and audience detection

Classify what was handed in (tickets, brain dump, metrics paste, incident log, OKR data, sprint notes) and infer three things before writing:

| Signal | Options | Implication |
|---|---|---|
| **Audience** | Team / IC, Manager / Director, VP / C-suite, Board | Detail level, assumed context, tone formality |
| **Channel** | Slack post, email, deck slide (one per slide) | Length, formatting (bullets vs prose), headers |
| **Cadence / urgency** | Weekly standup, sprint review, incident update, board meeting | Emphasis (blockers vs wins vs strategic signal) |

Ask if ambiguous. Default to "Director-level email" if no signal.

---

## Step 2 — RAG triage (the signal every update must carry)

Before writing a word of narrative, assign a RAG status to the overall project and to each major workstream:

- **Green** — on track; no action needed from reader.
- **Amber** — at risk; awareness needed; action may be required soon.
- **Red** — blocked or behind; decision or escalation needed now.

State the RAG explicitly at the top of the update. Flag any Red workstream with a one-line owner + blocker statement. This is non-negotiable — an update that buries bad news is worse than no update.

---

## Step 3 — SBAR structure

Apply SBAR as the skeleton. Compress or expand each section to audience need — board decks collapse S+B into one sentence; team Slack often skips the formal Recommendation section.

### S — Situation (what is happening right now)
One or two sentences. The reader should understand the overall project state and RAG in under 10 seconds. Include the time period this covers.

### B — Background (minimum context needed to understand the update)
Assume the reader's known context; don't re-narrate history they approved. Only include what changed since the last update or what is necessary to evaluate the assessment. For a first update on a project, a brief scope statement is appropriate.

### A — Assessment (the substance: progress, risks, decisions)

Break into sub-sections as needed:

**Progress** — what shipped, what moved, what closed. For ticket-based input: group by theme, not by ticket ID. Call out impact where known (metric moved, milestone hit, blocker cleared). Mark unconfirmed impact `[verify]`.

**Risks & blockers** — be specific. Name the risk, its probability/impact (High/Med/Low), the owner, and whether it is escalated. No vague "there may be delays." If it is Red, say what decision is needed and by when.

**Decisions made** — document any decisions taken since the last update, who made them, and what was changed. This is frequently omitted and frequently regretted.

### R — Recommendation / Next steps
What you need from the reader (if anything) and what the team is doing next. Separate "for awareness" from "need a decision" from "need unblocking." Use a numbered list for actions with owners and dates. If the reader needs to do nothing, say so in one line.

---

## Step 4 — Channel formatting

| Channel | Format rules |
|---|---|
| **Slack post** | Short prose intro (1–2 sentences) + tight bullets; RAG emoji (🟢🟡🔴) at the top; no markdown headers unless the workspace renders them; total ≤300 words for weekly; ≤150 for standup |
| **Email** | Subject line with RAG signal (e.g. "[AMBER] Sprint 14 Update — 2 blockers need attention"); sections with bold headers; bullets for lists; 400–600 words for a VP; shorter for a team lead |
| **Board deck slide** | Title = the one thing they need to know; 3–5 bullets max per slide; RAG badge in upper-right corner; speaker notes = the full SBAR narrative they read if they want depth |
| **General / unknown** | Default to email format |

Apply brand voice throughout. If the brand bans exclamation marks or jargon, honor it here too.

---

## Step 5 — Composing with sibling skills for numbers

When the update requires a data story:
- **Traffic / funnel / growth movement** → call `growth-diagnostic-deep-dive` with the available data; fold the returned diagnosis into the Assessment section. Do not re-derive it inline.
- **Paid channel performance** → call `channel-roi-scorecard`; use its ranked table as a subsection.
- **OKR health** → call `okr-suite`; use its RAG-progress narrative as the Assessment.
- **Experiment results** → call `experiment-results-analyzer`; summarize the verdict and learning in one paragraph.

---

## Persistence

Save completed updates to `./ops/status-updates/[YYYY-MM-DD]-[project-slug].md` when the user asks, or when the update is for a board or leadership audience (these are worth keeping). Inline delivery for team standup / Slack updates.

---

## Principles (Non-Negotiable)

- **RAG before narrative.** Signal first, story second. A stakeholder who has 30 seconds should still leave informed.
- **Bad news travels up immediately.** Red items go at the top of Assessment, not buried after wins.
- **Own the uncertainty.** Unconfirmed metrics are marked `[verify]`. Do not invent specificity.
- **SBAR is the skeleton, not the word count.** Compress aggressively for Slack; expand only for a board audience that needs the full context.
- **No ticket theater.** Listing 47 closed tickets is not a status update. Group by theme, impact, or milestone — readers don't care about PROJ-2847.
- **Decisions are as important as progress.** An update that omits decisions made is an incomplete record.
- **Brand voice, not brand positioning.** Honor the tone and banned words. Do not turn a project status email into a product pitch.

---

## What Not to Do

- Don't write the update before `brand-brain` returns (even for an internal update — tone matters).
- Don't hide Red status in neutral language. "We are monitoring the situation" is not acceptable when the truth is "we are blocked."
- Don't list raw ticket IDs without grouping them into a narrative.
- Don't fabricate metrics, timelines, or outcomes — mark anything unconfirmed `[verify]`.
- Don't reimplement growth analysis or OKR scoring inline — call the relevant sibling skill.
- Don't produce a 600-word email when the channel is Slack, or a 3-bullet Slack post when the audience is the board.
- Don't skip the Recommendation section because "they'll figure out next steps." Be explicit about what you need.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice + banned-words applied; no product positioning injected into an internal update?
- RAG status assigned and stated explicitly at the top — not buried?
- Any Red workstream has a named owner, a specific blocker, and a stated decision need?
- SBAR structure present and sized to the audience (not over-explained for a Slack standup, not under-explained for a board)?
- Progress is grouped by impact/theme, not by ticket ID?
- Decisions made are documented with owner and date?
- Recommendations separate "for awareness" from "need a decision" with owners and dates?
- Unconfirmed metrics marked `[verify]`; no invented specificity?
- Format matches the stated channel (Slack / email / deck)?
- Sibling skills called for any numbers story rather than re-derived inline?
