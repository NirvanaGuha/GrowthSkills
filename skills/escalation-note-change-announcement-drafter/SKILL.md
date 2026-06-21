---
name: escalation-note-change-announcement-drafter
description: >
  Drafts two types of high-stakes internal messages that most marketers get wrong: (1) escalation notes
  — the upward-written, factual, request-for-action messages that move blocked decisions without
  triggering defensiveness; and (2) change-management announcements — the empathy-first, reason-before-
  rule messages that help teams absorb uncomfortable changes without revolt. Neither type should be
  alarming, vague, emotional, or passive-aggressive, yet most drafts are at least one of those. This
  skill applies the SBI (Situation-Behavior-Impact) scaffolding for escalations and the ADKAR
  (Awareness-Desire-Knowledge-Ability-Reinforcement) framing for change announcements, then dresses
  each in the brand's established communication tone. Saves a dated draft to ./internal-comms/ for
  audit trail. Use when someone says "escalate this," "draft a change announcement," "communicate the
  pivot," "write up the blocker," "announce the reorganization," "how do I tell the team about X,"
  "draft the Slack message about the policy change," "I need to flag this to leadership," or hands
  over a situation and asks what to send.
---

# Escalation Note & Change Announcement Drafter

Internal messages that should calm a situation almost always inflame it instead — because they're vague on facts, heavy on emotion, or structured around what the sender needs rather than what the reader needs to do or feel. This skill fixes that.

Two modes, one framework spine. Escalations use **SBI** (Situation-Behavior-Impact) to build a calm, fact-first, action-requesting note that creates urgency without blame. Change announcements use **ADKAR** to sequence the message so the reader lands on acceptance rather than resistance. Both are written in the brand's established internal voice — because how you communicate internally teaches people how you speak externally.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's tone, voice adjectives, banned words, and any internal-comms style notes from `style-guide.md`. Every escalation and announcement must match how this org actually talks.
- **`loom-async-video-script-writer`** (optional) — when the announcement is high-stakes enough to warrant a recorded message from a leader rather than plain text, call this to turn the drafted announcement into a Loom-ready script.
- **`doc-note-summarizer`** (optional) — when the input is a long Slack thread, meeting notes, or email chain, call this first to extract the factual core before drafting.
- **`quick-tone-softener-diplomacy-pass`** (optional) — final pass when the user flags that the draft still reads too hard; call rather than re-draft from scratch.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; extract tone + banned words + comms style
Step 1  Classify the mode       ──► Escalation Note | Change Announcement | both
Step 2  Gather the facts        ──► situation intake (ask only what's missing)
Step 3  Apply the framework     ──► SBI (escalation) | ADKAR (change)
Step 4  Draft                   ──► tone-matched, channel-right, length-right
Step 5  Self-review + deliver   ──► checklist pass, then present with 1 alternate subject/opener
Step 6  Save artifact           ──► ./internal-comms/[slug]-[date].md
```

### Step 0 — Brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Extract: (a) internal-tone adjectives from `style-guide.md` if present, else voice adjectives from `brand.md`; (b) banned words; (c) any org-specific communication norms (directness level, honorifics, emoji policy). These override everything below.

**Fallback:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask for three tone adjectives + any banned words, then proceed.

---

## Step 1 — Classify the mode

| Signal in the request | Mode |
|---|---|
| "escalate," "blocked," "flag to leadership," "decision not made," "we need approval," "I need to escalate" | **Escalation Note** |
| "announce," "communicate the change," "tell the team," "policy update," "reorg," "pivot," "new process," "change management" | **Change Announcement** |
| Situation contains both a blocker AND a forthcoming policy/decision that needs announcing | **Both** — draft escalation note first, change announcement second |

When ambiguous, ask one question: *"Is this asking someone to act/decide (escalation), or informing people about a decision already made (announcement)?"*

---

## Mode A — Escalation Note (SBI Framework)

Most escalation notes fail because they either read as complaints (too emotional) or as vague status updates that don't create a clear action path. The fix is SBI: ground everything in observable facts, state the concrete impact, and make a single specific request.

### SBI applied to escalations

**S — Situation (1–2 sentences, purely factual)**
What is the objective context? Date ranges, systems, names of blockers, concrete state of the world. No editorializing. Write this so a reader who has zero background can understand the facts without asking follow-up questions.

**B — Behavior (1–2 sentences, behavior-level not character-level)**
What specific action or inaction created the blocker? Not "X doesn't care" — rather "The decision on [Y] has not been made as of [date], which was the agreed deadline from [meeting/doc]." Name the observable event, not an interpretation of motive.

**I — Impact (2–3 sentences, quantified where possible)**
What is at stake? Use numbers — revenue at risk, launch date slipping by X days, team idle hours, contract deadline, customer SLA. This is the engine of urgency; don't waste it on adjectives. If exact numbers aren't available, frame the range: "This delays the Q3 launch by at least 2–3 weeks" rather than "this is a big problem."

**Request (1 clear sentence)**
One ask. One owner. One deadline where appropriate. "I'm requesting a decision from [name/role] by [date]" or "I need [specific resource/approval] this week to hit [milestone]." If you need multiple asks, rank them and ask for the top one explicitly.

**Tone targets for escalations:**
- Calm authority, not urgency-panic
- Factual, not accusatory
- Forward-looking ("what we need") not backward-dwelling ("who failed")
- Concise: escalation notes above 250 words are usually padding

### Escalation output format

```
To: [role / name]
From: [sender role]
Date: [today]
Re: [one-line subject — factual, not dramatic]

[S paragraph]

[B paragraph]

[I paragraph]

[Request sentence]

Happy to discuss on a call — [optional].
```

Save to `./internal-comms/escalation-[slug]-[YYYY-MM-DD].md`.

---

## Mode B — Change Announcement (ADKAR Framework)

ADKAR (Prosci, [verify exact year of framework]) sequences a change message through the psychological stages people actually move through: they need to *know why* before they can *want to*, and they need to *know what to do* before they can *do it*. Most change announcements skip Awareness and Desire and land at Knowledge — "here's the new process" — and wonder why adoption is low.

### ADKAR applied to internal announcements

**A — Awareness (1 short paragraph)**
Why is this change necessary? Business context only. Not "leadership decided" — the actual driver: competitive shift, cost pressure, user feedback, compliance requirement, growth blocker. People accept change more readily when the reason is real and knowable, not a policy decree from above. Write as if the team is adults who can handle the truth.

**D — Desire (1–2 sentences)**
What's in it for them — or at minimum, what pain does this solve that they've already felt? If the change is net-negative for the audience (layoffs, pay cuts, reduced scope), do NOT manufacture false upside. Name what the org is trying to protect instead ("this protects the team's runway through Q4"). Honesty here builds more trust than optimism spin.

**K — Knowledge (bulleted list or short paragraphs)**
What exactly is changing? Effective date, new process, new tool, new reporting line — concrete and specific. If there are exceptions, name them. If the final details are still being determined, say that explicitly and give a date when they will be known.

**A — Ability (1 short paragraph)**
What support is available to make the transition? Training, documentation, a DRI to ask questions, an FAQ, a sandbox environment, office hours. If there is nothing yet, say when there will be. Never skip this — the perception of being left alone with a change is a primary driver of resistance.

**R — Reinforcement (closing 2–3 sentences)**
What signals that the change is real and lasting? A follow-up date, a milestone check-in, the name of the person accountable for the new state. Ends with a door-open line that invites questions without making it a committee decision.

**Tone targets for change announcements:**
- Empathy before information: acknowledge disruption before explaining rationale
- Confident, not defensive: state decisions as decisions, not as suggestions
- Direct about uncertainty: distinguish "decided" from "still in progress"
- No corporate fluff: "exciting opportunity" and "journey" are banned unless the brand uses them literally

### Change announcement output format

```
Subject / Slack header: [Factual, not alarmist]

[Opening line: acknowledge the significance before explaining it]

[A — Why this is happening]

[D — What this means for you / why it matters]

[K — What's changing, specifically]
  - [bullet]
  - [bullet]

[A — Where to get help / what support exists]

[R — Next touchpoint + who owns this + door-open close]

— [Sender name / role]
```

Save to `./internal-comms/announcement-[slug]-[YYYY-MM-DD].md`.

---

## Shared craft principles

**Channel calibration.** An escalation note to a C-level differs in length and register from one to a peer manager. A Slack announcement differs from an all-hands email. Ask or infer the channel and size accordingly: Slack posts are 150–250 words max; emails can go to 400 if the change is complex; notes to senior leadership compress to a single scroll.

**Naming people vs. naming roles.** In escalations: when a specific person is the blocker, name them (this is a fact, not a character attack). In announcements: name the role/DRI for decisions, not individuals (prevents the announcement becoming a blame document if things change).

**What to do when the situation is unclear.** Ask for four facts before drafting: (1) who is the audience, (2) what decision or behavior change is being requested/announced, (3) what is the concrete impact of the current situation or the change, (4) is this asking someone to act, or informing people of a decision already made. Do not draft on vibes.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No draft until brand tone is loaded. Internal voice is brand voice.
- **SBI for escalations, ADKAR for announcements.** Framework is not optional — it's why junior drafts become senior outputs.
- **Facts over feelings in the body.** Empathy belongs in the opener and the close; the body runs on observable, citable facts.
- **One ask per escalation.** Multiple asks create deniability; name the primary request.
- **No manufactured upside.** If a change is hard, say it's hard. Trust is the asset being spent.
- **Quantify impact.** Every impact statement should have a number, a date, or a concrete deliverable name.
- **Real proof or [verify].** Do not invent timelines, budget figures, or policy references.

---

## What not to do

- Don't draft before brand-brain loads the tone and banned words.
- Don't write an escalation that reads like a complaint — every sentence should have an observable referent.
- Don't skip the Desire section of ADKAR and jump to Knowledge; this is the most common failure mode.
- Don't soften the Impact section with adjectives when numbers are available.
- Don't use passive voice to obscure who owns the decision or who caused the blocker.
- Don't write more than 400 words for any internal message unless the audience explicitly requested a comprehensive briefing document.
- Don't use banned words; don't use "journey," "exciting," "synergy," or "circle back" unless the brand style guide says otherwise.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; tone, banned words, and comms style loaded?
- Mode correctly classified (Escalation / Announcement / Both)?
- Escalation: S paragraph is fact-only; B paragraph names behavior not character; I paragraph has a number or concrete timeline; Request is one ask with one owner?
- Announcement: Awareness explains the real driver (not "leadership decided"); Desire acknowledges impact honestly; Knowledge is specific and dated; Ability names concrete support; Reinforcement closes with a follow-up commitment?
- Channel length honored (Slack ≤ 250w, email ≤ 400w unless complex)?
- No invented numbers, no `[verify]`-worthy claims left unmarked?
- One alternate subject line or opening sentence offered so the user can A/B the tone?
- Artifact saved to `./internal-comms/[type]-[slug]-[YYYY-MM-DD].md`?
