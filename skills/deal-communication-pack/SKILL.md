---
name: deal-communication-pack
description: >
  Turns raw deal inputs — call transcript, stage, and a sentence of deal context — into three
  ready-to-send artifacts: (1) a Slack deal-status ping for the AE/revenue channel, (2) a
  Gong-style call debrief summary (MEDDIC-framed: Metrics, Economic Buyer, Decision Criteria,
  Decision Process, Identify Pain, Champion), and (3) a structured classification of inbound
  email replies with a drafted response for each. All three assets are brand-voice-compliant
  and stamped with the deal's current stage, next action, and close-date risk. Built for AEs,
  RevOps, and CS reps who spend 20–40 minutes per deal writing updates nobody reads. Use when
  the user says "deal update," "write my Gong notes," "call debrief," "Slack the deal," "reply
  to this prospect email," "deal recap," "summarize my call," "draft my deal summary," or hands
  over a transcript and asks what to send.
---

# Deal Communication Pack

Raw deal context in; three polished communication artifacts out. No more blank-cursor deal updates, no more informal Gong notes, no more "not sure how to reply to this" prospect emails.

The skill runs on the **MEDDIC framework** — the industry-standard deal-qualification methodology (Metrics, Economic Buyer, Decision Criteria, Decision Process, Identify Pain, Champion) — mapped to three output surfaces: async status ping, structured call debrief, and email-reply classification. Each artifact is sized for its audience: Slack ping is skim-readable in 10 seconds; Gong debrief is complete enough for a manager-review or a deal handoff; email drafts are ready to send with one read-through.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, product positioning, offer mechanics, and proof points so every artifact sounds like the company, not like a boilerplate template. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, one-line product positioning, and any terminology or phrases to avoid before proceeding.
- **`account-dossier-builder`** *(optional)* — if installed and a company name is present, pull the account dossier to pre-populate MEDDIC fields (firmographics, recent news, known stakeholders).
- **`stakeholder-update-status-writer`** *(optional)* — for executive-layer updates that need to accompany the Slack ping (e.g., board-visible deals).
- **`escalation-note-change-announcement-drafter`** *(optional)* — when a deal has gone sideways or stage has regressed, call this for the internal escalation note rather than drafting it here.
- **`data-qa-measurement-gotcha-checker`** *(optional, CRM/RevOps)* — when the user provides a CRM export or deal CSV, run this as a data-quality gate before extracting MEDDIC fields; flag missing close dates, orphaned contacts, and stage-mismatch records.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain
Step 1  Parse deal inputs   ──► extract MEDDIC scaffold from transcript / context
Step 2  Choose mode(s)      ──► Slack ping | Gong debrief | Email classification | All three
Step 3  Draft artifacts     ──► one pass per surface
Step 4  Risk flag           ──► surface close-date risk, missing MEDDIC fields, next action
Step 5  Present & save      ──► inline + optional file save
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, product positioning, offer mechanics, and real proof points. Obey voice and banned words as hard overrides; use only real proof from the returned digest (mark anything unconfirmed `[verify]`). Anchor all artifact language to the product's known value prop and ICP.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, one-line product positioning, and any terminology or phrases to avoid before proceeding.

---

## Step 1 — Parse inputs and scaffold MEDDIC

Extract what's available from the provided transcript, notes, or deal context. Never invent values; mark gaps `[unknown — ask or confirm]`.

| MEDDIC Field | What to extract |
|---|---|
| **Metrics** | Quantified business impact the prospect cited (%, $, time saved, units) |
| **Economic Buyer** | Name / title of the person with budget authority; noted or inferred |
| **Decision Criteria** | Stated eval must-haves: integration, price, feature set, security |
| **Decision Process** | Steps to a signed order: POC → security review → legal → sign |
| **Identify Pain** | The specific problem driving urgency — in the prospect's own words if possible |
| **Champion** | The internal advocate: name, credibility within the account, stated commitment |

Flag missing MEDDIC fields explicitly. A deal missing Economic Buyer or Champion is a risk item, not a data formatting issue — say so plainly.

---

## Mode A — Slack Deal-Status Ping

### Format

```
*[STAGE] [Account] — [One-line deal summary]*
AE: [name] | Close: [date] | ARR: [value or TBD]

*Pain:* [1 sentence from Identify Pain]
*Champion:* [name + role]
*Economic Buyer:* [name + role or ❓]
*Next action:* [concrete task + owner + due date]
*Risk:* [one flag or "None flagged"]
```

### Rules
- Fits in a Slack preview (no scroll on desktop). If it can't fit, compress — do not grow the format.
- Stage label is drawn from a standard pipeline (Prospecting / Discovery / Evaluation / Proposal / Negotiation / Closed-Won / Closed-Lost) or from what the user provides.
- "Next action" is a specific task with an owner and a date, never "follow up."
- ARR marked `TBD` when not stated — never estimated.
- Risk field: use plain English (e.g., "No Economic Buyer contact established," "Close date 14 days away, legal review not started," "Champion has no confirmed exec access"). One flag only — the most important one. "None flagged" when genuinely clean.

---

## Mode B — Gong Call Debrief (MEDDIC-framed)

Output a structured debrief note suitable for logging in Gong, Salesforce, or any CRM notes field — and usable for deal review without re-listening to the call.

```markdown
## Call debrief — [Account] | [Date] | [Duration if known]
**Participants:** [AE] + [Prospect names + roles]
**Stage at time of call:** [stage]

### What we learned
**Metrics:** [quantified impact stated]
**Economic Buyer:** [name + confirmation status]
**Decision Criteria:** [ranked list if possible]
**Decision Process:** [steps + timeline if shared]
**Pain:** [primary pain, verbatim quote if available]
**Champion:** [name + trust signal — what did they commit to?]

### Key moments
- [Moment 1 — what was said / what it means]
- [Moment 2]
- [Moment 3 — max 4 bullets; quality over quantity]

### Gaps / risks
- [MEDDIC field that's still unknown or weak, with implication]

### Committed next steps
| Action | Owner | Due |
|--------|-------|-----|
| [action] | [AE/prospect] | [date] |

### One-line summary (for pipeline review)
[Single sentence: stage + where deal stands + the swing factor]
```

Length target: 150–250 words excluding the table. If the transcript is thin, say so — note which fields could not be extracted rather than padding.

---

## Mode C — Email Reply Classification

Given one or more inbound emails from the prospect, classify each and draft a response.

### Classification taxonomy (our working model)

| Class | Signal | Recommended move |
|---|---|---|
| **Forward motion** | "Sent to legal," "looped in my boss," "confirmed the date" | Confirm + lock next step; brevity wins |
| **Soft stall** | "We're still evaluating," "haven't had a chance to review" | Re-anchor to pain + deadline; don't push, pull |
| **Hard objection** | Price, competitor named, internal priority shift | Acknowledge → reframe → one question that reopens |
| **Ghost** | ≥5 business days, no reply to a confirmed-action email | Breakup-style bump; give permission to say no |
| **Champion escalation request** | "Can you send something for my VP?" | Provide an exec-ready one-pager or talking-points sheet — don't write a longer email |
| **Closed-lost signal** | "Going with another vendor," "pausing for the year" | Thank + learn + keep a door open in one short note |

### Draft format per email

```
**Email from:** [sender + date]
**Classification:** [class name]
**Why:** [one line — what in the email drove this classification]
**Draft reply:**
---
[Subject: Re: ...]

[Draft — short, specific, on-voice, with a concrete single ask or action]
---
**Char count:** [count]  |  **Reading time:** [~X sec]
```

Keep drafts short: ≤150 words for Forward motion / Ghost; ≤200 words for objections. Never write a wall of text to a prospect who just stalled.

---

## Risk flags (shared across all modes)

Automatically surface these at the end of any run:

- **Missing Economic Buyer contact** — deal cannot close without it
- **No confirmed Champion commitment** — "supportive" ≠ committed; note the difference
- **Close date within 30 days, legal/security review not started** — flag as timeline risk
- **Single-threaded deal** — only one contact; champion loss = deal loss
- **No quantified Metrics** — prospect hasn't said what success looks like in numbers; hard to write a business case
- **Stage regression signals in transcript** — if the call revealed a step backward, name it

---

## Artifact persistence

Save to `./deals/[account-slug]-[YYYY-MM-DD]-deal-pack.md` when the user asks to save or when producing all three artifacts in one run. Inline only by default. Never overwrite without asking if the file already exists.

---

## Principles (Non-Negotiable)

- **MEDDIC is a lens, not a form.** Extract what's there; flag what's missing; never fill gaps with guesses.
- **Brand-brain first.** No artifact before brand context loads; the brand's voice and positioning are non-negotiable overrides.
- **Gaps are information.** A missing Economic Buyer field is a risk insight, not a formatting problem — surface it.
- **Size to the surface.** Slack pings are skim-reads; Gong notes are manager-review depth; email drafts are human-speed readable. Don't over-write any of them.
- **Real proof only.** Brand-brain returns confirmed proof points; use them. Anything unconfirmed is `[verify]`.
- **One next action.** Every artifact that involves a next step names one concrete action with an owner and a date.

## What Not to Do

- Don't invent MEDDIC fields — a blank is better than a fabrication.
- Don't write a Slack ping that requires scrolling; compress the content, not the format.
- Don't write a reply to a soft stall that sounds like a push — pull instead.
- Don't estimate ARR or close probability; use what was stated or mark it unknown.
- Don't reimplement brand scanning or voice-building — call `brand-brain`.
- Don't produce all three modes by default if the user only asked for one — offer the others at the end.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before drafting?
- MEDDIC scaffold extracted; all unknown fields flagged (not invented)?
- Slack ping fits in one screen; has a concrete next action with owner + date?
- Gong debrief has a one-line summary usable in a pipeline-review meeting?
- Each email reply is classified with a one-line rationale and a draft within char target?
- Risk flags surfaced at the end, or "None flagged" stated explicitly?
- Persistence offered or executed per user instruction?
