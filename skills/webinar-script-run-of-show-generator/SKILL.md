---
name: webinar-script-run-of-show-generator
description: >
  Session topic, speakers, duration, and flow → a timestamped script outline with host cues,
  live-poll questions, Q&A prompts, and a minute-by-minute run-of-show. Produces two tightly
  interlocked artifacts: (1) a Speaker Script — complete with host introductions, segue cues,
  slide-change callouts, and speaker talking-point bullets at the right cadence; and (2) a
  Producer Run-of-Show — the master timing sheet with owner columns, tech-check gates,
  backup contingencies, and post-event actions. Uses the brand's real voice and real proof so
  the webinar sounds like the company, not a generic vendor deck. Supports solo host, panel,
  fireside chat, and product-demo formats. Saves to ./webinars/ for re-use across the
  webinar promotion suite. Use whenever the user says "write a webinar script," "run of show,"
  "webinar outline," "speaker guide," "host cues," "webinar agenda," "minute-by-minute
  timing," "script for my webinar," or hands over a webinar brief and asks what to say.
---

# Webinar Script & Run-of-Show Generator

Give it a topic, speakers, and duration — get two documents that let any host run a tight webinar without winging it: a scripted flow they can actually read from (or scan during live delivery), and a producer timing sheet that keeps every moving part on track. Brand voice and real proof carry through from `brand-brain`, so the script sounds like the company.

This skill writes the live experience. The promotion side (registration page copy, reminder emails, post-event follow-up) lives in the sibling skills listed below — call them from here if you need the full kit, or run them independently.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, ICP, offer, proof, and banned words. Script does not implement brand resolution or scan.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for: brand name, ICP + pain point, positioning line, 3 voice adjectives, and any banned words.
- **`headline-hook-generator`** — generates the webinar title and opening hook line if the user hasn't locked one. Call when the session title is still vague.
- **`cta-variant-generator`** — produces the end-of-webinar offer CTA (demo/trial/download) with the right commitment level for a product-aware audience.
- **`proof-vault`** — surfaces the best 1–2 proof points (stat, case study, customer quote) to anchor the credibility segment; synthesize inline when absent.
- **`webinar-email-sequence-writer`** — builds the pre- and post-event email sequence once the script is approved.
- **`event-registration-on-demand-page-copywriter`** — writes the landing page that feeds registrants into this webinar.
- **`event-social-promotion-pack`** — social posts for the promotion window.
- **`loom-async-video-script-writer`** — if any segment is pre-recorded, adapt the live script to async format.

---

## How a run works

```
Step 0  Brand context     ──► call brand-brain; block until digest returns
Step 1  Intake            ──► extract or ask for the 7 required inputs
Step 2  Structure         ──► apply the AIDA-Arc framework to assign minutes
Step 3  Script            ──► write the Speaker Script segment by segment
Step 4  Run-of-show       ──► build the Producer Sheet from the same timing
Step 5  Self-review       ──► quality checklist; offer sibling-skill handoffs
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before writing a single line of script. It returns voice adjectives, banned words, offer mechanics, real proof points, ICP, and the active brand's positioning. Every host cue, opening hook, and closing CTA must obey the returned voice and banned-word list. Mark any proof not confirmed in the brand digest as `[verify]`.

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for: brand name, ICP + pain point, positioning line, 3 voice adjectives, and any banned words.

---

### Step 1 — Intake (7 required inputs)

Extract from the user's brief. Ask only for what's missing — never re-ask what was provided.

| # | Input | Why it matters |
|---|---|---|
| 1 | Session title / topic | Sets audience expectation and hook |
| 2 | Speakers (name, role, talking-point ownership) | Assigns script segments |
| 3 | Target duration | Drives the timing sheet |
| 4 | Format | Shapes segment ratio (solo vs. panel vs. demo vs. fireside) |
| 5 | ICP + awareness stage | Calibrates technical depth and CTA commitment |
| 6 | Primary CTA / offer | Anchors the close |
| 7 | Platform (Zoom, GoTo, Hopin, YouTube Live, etc.) | Informs tech-check items and polling mechanics |

If the session title is vague, call `headline-hook-generator` to sharpen it before continuing.

---

## The AIDA-Arc Framework

Every segment maps to the Attention → Interest → Desire → Action arc, adapted for live webinar pacing. Ratios below are for a 45-minute session — scale proportionally.

| Segment | AIDA stage | Default time | What happens |
|---|---|---|---|
| **Pre-roll & tech-check** | — | T−5 min | Waiting-room music/slide, chat prompt, housekeeping |
| **Cold open / hook** | Attention | 2 min | The one insight, stat, or question that makes the audience stay |
| **Credibility bridge** | Interest | 3 min | Speaker intro, company proof point, why this topic now |
| **Core content** | Interest → Desire | 25 min | Meaty teaching; 3–4 key points; demos, case studies, live polls woven in |
| **Proof / case study spotlight** | Desire | 5 min | Real customer story or data; the "it works" moment |
| **Offer / CTA reveal** | Action | 3 min | Single, clear next step; low-commitment framing for live audiences |
| **Q&A** | Action | 7 min | Pre-seeded questions + live queue; ends with a re-CTA |
| **Wrap & handoff** | — | 1 min | Thank-yous, replay info, next-step reminder |

For 30-minute sessions: compress core content to 15 min and Q&A to 5 min. For 60-minute sessions: expand core content to 35 min and add a second proof spotlight.

---

## Artifact 1 — Speaker Script

Write segment by segment. Each segment block contains:

```
## [SEGMENT NAME] — [START TIME]–[END TIME]

HOST CUE: [What the host says to open this segment — full sentence, in brand voice]

SPEAKER: [Name]

TALKING POINTS:
• [Bullet — the point, not a full sentence to read aloud]
• [Bullet]

SLIDE CUE: [Slide title or "no slide change"]
POLL CUE: [Poll text + answer options, or "none"]
TRANSITION: [Host line that bridges to the next segment]
```

Rules for script writing:
- **Host cues are fully scripted sentences**, not notes. The host should be able to read them verbatim under pressure.
- **Talking points are speaker bullets**, not a teleprompter. Speakers own the words; the script owns the arc.
- **Brand voice is non-negotiable.** Every scripted host line uses the voice adjectives returned by `brand-brain`; banned words never appear.
- **Cold open commits to a hook, not a welcome.** No "thanks for joining" as the first line — the hook comes first; housekeeping follows.
- **Proof points use only brand-brain-confirmed facts.** Unconfirmed stats are marked `[verify before live]`.
- **Polls are specific.** Each poll question is fully written with 3–4 answer options; note the platform mechanic (Zoom poll, Slido, chat-based).
- **Transitions are verbal bridges**, not "next slide." They name what just happened and set up what's next.

---

## Artifact 2 — Producer Run-of-Show

A separate table the producer (or a second tab in the deck) manages during the live event.

```
## Run-of-Show — [Session Title]
Platform: [X]   Total duration: [X min]   Host: [Name]   Producer: [Name]

| Clock | Elapsed | Segment | Owner | Action required | Tech cue | Status |
|---|---|---|---|---|---|---|
| T−15 | — | Pre-event setup | Producer | Load waiting-room slide; test audio | Screen share ON | ☐ |
| T−5 | — | Pre-roll | Host | Welcome message in chat; camera check | Music playing | ☐ |
| 0:00 | 0 | Cold open / hook | Host | Start recording; mute all attendees | Recording ON | ☐ |
...
```

Include:
- **Contingency rows** for common failure modes: speaker drops (backup slide + hold-music plan), poll platform fails (pivot to chat-based vote), demo crashes (fallback screenshot slide).
- **Post-event actions** at the bottom: stop recording, drop replay link in chat, trigger email sequence (link to `webinar-email-sequence-writer`), update CRM/tag registrant list.

---

## Output format

Present both artifacts inline in order. Then offer:

> "Want the full promotion kit? I can call:
> - `webinar-email-sequence-writer` for the reminder + follow-up series
> - `event-registration-on-demand-page-copywriter` for the registration page
> - `event-social-promotion-pack` for social posts during the promo window"

Save both artifacts to `./webinars/[session-slug]-script.md` and `./webinars/[session-slug]-run-of-show.md` if the user confirms. Do not save until confirmed.

---

## Format variants

| Format | Key adjustments |
|---|---|
| **Solo host** | No speaker handoffs; talking-point bullets own the full core-content block |
| **Panel (3+ speakers)** | Add a moderator-question bank (8–10 questions, 2 per speaker); reduce each speaker's segment to 6–8 min; add a panel intro segment |
| **Fireside chat** | Replace slide cues with "no slide" throughout core content; script the interview questions instead of talking points; keep it conversational |
| **Product demo** | Expand the core-content block with a Demo Script sub-section (step-by-step click path, what to say at each state, what to do if it breaks) |

---

## Principles

- **Brand-brain first.** No script before the brand digest returns. Its voice and banned words are hard overrides on every scripted line.
- **The hook is not a welcome.** Open with the insight or the tension — save logistics for minute 2.
- **Two documents, one clock.** The speaker script and the run-of-show share identical timestamps; they are views of the same plan, not separate documents.
- **Proof must be real.** Stats and case studies from the brand digest only; anything else gets `[verify before live]`.
- **Polls earn their time.** Each poll question is pre-written with answer options and tied to a key point — not dropped in as filler.
- **Plan for failure.** Every run-of-show has contingency rows. A producer who hasn't thought about what to do when the demo breaks will find out live.
- **Close with one action.** The end-of-webinar CTA is singular and concrete. Do not list multiple next steps.

---

## What not to do

- Don't open with housekeeping — the hook comes first, every time.
- Don't write talking-point bullets that are full paragraphs; speakers need cues, not a teleprompter.
- Don't script every speaker word — only the host cues are fully scripted; speaker bullets are signposts.
- Don't invent proof points or customer statistics; mark unconfirmed data `[verify before live]`.
- Don't produce a run-of-show without contingency rows — bare timing tables are incomplete.
- Don't use generic webinar language ("let's go ahead and," "as you can see here," "without further ado") unless the brand voice specifically permits that register.
- Don't bundle the promotion copy (emails, landing page) into this output — call the relevant sibling skills instead.

---

## Quality checklist

- [ ] `brand-brain` called and active brand loaded before writing any line?
- [ ] Voice adjectives and banned words honored throughout all host cues?
- [ ] All proof points confirmed in the brand digest, or marked `[verify before live]`?
- [ ] Cold open leads with the hook — not "welcome" or housekeeping?
- [ ] Every segment has: HOST CUE · TALKING POINTS · SLIDE CUE · POLL CUE · TRANSITION?
- [ ] AIDA-Arc segment ratios respected for the stated duration?
- [ ] Poll questions fully written with answer options and platform mechanic noted?
- [ ] Run-of-show has owner columns, tech cues, contingency rows, and post-event actions?
- [ ] Speaker Script and Run-of-Show share identical timestamps?
- [ ] End-of-webinar CTA is single, concrete, and commitment-level-appropriate for a live audience?
- [ ] Sibling-skill handoff offer presented after output?
