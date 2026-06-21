---
name: webinar-email-sequence-writer
description: >
  Registration details + agenda → pre-event reminder sequence, replay announcement, and
  post-event nurture drip across all attendance segments. Turns a webinar into a full
  email campaign: confirmation/welcome, 2–3 countdown reminders, day-of send, attended
  and no-show replay variants, and a segmented post-event nurture arc (attended · partial ·
  no-show) that keeps the conversation alive beyond the recording link. Every email is
  written on the active brand's voice using real proof and ICP language, with subject lines
  and preview text optimised per segment. Also calls subject-line-preview-text-optimizer
  (subject/preview scoring), cta-variant-generator (primary CTA per email), and
  de-slop-humanize-pass (final voice pass). Saves the finished sequence to ./events/.
  Use when someone says "write webinar emails," "remind people about the webinar,"
  "send the replay," "follow up with attendees/no-shows," "webinar drip," or hands over
  a registration page, agenda, or GoToWebinar / Zoom / Hopin export.
---

# Webinar Email Sequence Writer

A webinar is a campaign, not a one-off send. This skill produces the complete email arc — from
confirmation through post-event nurture — segmented by attendance behaviour so every reply
lands on the right reader at the right moment. Senior-copywriter output, brand-voice-locked,
with subject lines scored before delivery.

---

## Skills this calls

| Skill | When |
|---|---|
| **`brand-brain`** | Step 0 — always first; loads voice, ICP, banned words, offer, real proof |
| **`subject-line-preview-text-optimizer`** | Step 4 — scores and refines subject + preview text per email |
| **`cta-variant-generator`** | Step 4 — generates the primary CTA for each email |
| **`de-slop-humanize-pass`** | Step 5 — final voice pass before presenting output |

Optional: `proof-vault` for specific social proof pulls; `lifecycle-journey-mapper` when
the post-event nurture needs to slot into a wider product lifecycle.

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain (always first)
Step 1  Intake + segment map    ──► parse inputs, define attendance segments, map sequence spine
Step 2  Build the spine         ──► confirm which emails belong, assign triggers + delays
Step 3  Draft every email       ──► Pre-Event Ladder framework (below)
Step 4  Sharpen assets          ──► subject-line-preview-text-optimizer + cta-variant-generator
Step 5  Voice pass + save       ──► de-slop-humanize-pass → save to ./events/<slug>/
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before writing a single word. It returns voice adjectives,
banned words, offer mechanics, real proof, positioning, and ICP + awareness tendency. Obey
the returned voice and banned-words as hard overrides. Use only real proof; mark anything
unconfirmed `[verify]`.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md`. If none exists, ask the user for: what the product does · ICP job title/pain ·
offer/pricing + key proof · 3 voice adjectives + banned words. Proceed on those answers.

---

### Step 1 — Intake + segment map

Ask for (or extract from supplied material):

| Input | Why it matters |
|---|---|
| Webinar title, date/time, registration URL | Core copy anchors |
| Agenda + key takeaways (3–5 bullets) | The value promise driving every subject line |
| Speakers + credentials | Authority proof in reminders and replay emails |
| On-demand replay URL (or ETA) | Gates the replay email; mark `[TBD]` if not yet known |
| CRM / ESP segment labels (attended, partial, no-show) | Controls which emails fire per segment |
| Any hard offer or CTA post-event (demo, trial, discount) | Determines nurture arc aggression |

If registration details are handed over as a URL, read the page before asking.

**Attendance segments (always three):**

| Segment | Definition | Tone calibration |
|---|---|---|
| **Attended** | Stayed ≥ 70% of runtime | Warm, follow-through; they invested time |
| **Partial** | Joined but left early | Acknowledge the gap; lead with the replay |
| **No-Show** | Registered, never joined | Re-sell the value; no guilt, just benefit |

---

### Step 2 — Sequence spine (the Pre-Event Ladder + Post-Event Arc)

**The framework: Pre-Event Ladder + Post-Event Arc**

The Pre-Event Ladder builds anticipation and reduces no-shows. The Post-Event Arc converts
registrant momentum into a product action, segmented by how invested each person actually was.

```
PRE-EVENT LADDER
  T−0   Confirmation / welcome     (immediate on registration)
  T−7d  Countdown reminder #1      (the value promise; who should attend and why)
  T−1d  Countdown reminder #2      (agenda teaser + any live Q&A hook)
  T−2h  Day-of reminder            (link + frictionless join copy; short)

POST-EVENT ARC (segment forks from here)
  T+0h  Live replay send           (Attended + Partial: within 1–4 h of close)
         No-show replay send        (No-Show: same replay, re-sells the event)
  T+2d  Value-extract email        (All: 1–2 key takeaways turned into actionable insight)
  T+5d  Nurture email #1           (Attended: deepen → soft offer / trial / demo CTA)
                                    (Partial/No-Show: lower-commitment ask, e.g. read/watch)
  T+10d Nurture email #2           (Attended who clicked: harder offer / urgency anchor)
                                    (Non-converters: objection reframe + testimonial)
```

Adjust timing when the user specifies an ESP or provides their own send cadence. Drop emails
the user doesn't need (e.g. "we only want the pre-event series") without rebuilding the rest.

---

### Step 3 — Drafting each email

#### Anatomy of a webinar email

Every email in the sequence has a job. Match the body length to that job:

| Section | Rule |
|---|---|
| **Subject + preview text** | Drafted here; polished by `subject-line-preview-text-optimizer` in Step 4 |
| **Opening hook** | One sentence that pays off the subject line promise |
| **Body** | Pre-event: ~80–120 words. Post-event: 120–200 words max. Never longer. |
| **CTA block** | One primary CTA (generated by `cta-variant-generator`) + optional low-commitment secondary |
| **Signature / from-name** | Match the brand's send convention; use a human name where possible |

#### Craft rules by email type

**Confirmation email** — Prioritise clarity over persuasion. Calendar link, date/time in the
reader's local-time note, join link, and one sentence on what they'll leave with. Sets the
expectation that makes reminders welcome.

**Countdown reminders** — Each builds on a different angle: Reminder #1 = the value promise
(what they'll learn); Reminder #2 = social proof or speaker authority + agenda peek. Never
repeat the same hook. Subject lines should feel like a personal nudge, not a bulk blast.

**Day-of reminder** — Ruthlessly short. Date, time, one-click join link, zero friction.
If the brand voice allows it, a single line of warmth ("See you in two hours.").

**Replay email (Attended / Partial)** — Opens with genuine warmth ("Thanks for joining").
For Partial: acknowledge without guilt ("Had to leave early? Here's everything you missed.").
Lead with the replay link in the first 40 words — no burying.

**Replay email (No-Show)** — Re-sells the event content rather than leads with an apology.
Subject line must stand on its own as a content offer, not a reprimand.

**Value-extract email** — The highest-goodwill email in the arc. Take 1–2 of the most
actionable takeaways and deliver them as usable insight, not a summary of the webinar itself.
Earns the right to the nurture emails that follow.

**Nurture emails** — These convert. Use the ICP's actual language (from brand-brain's ICP
block), the brand's real proof, and a CTA calibrated to the segment's commitment level:
Attended → trial/demo/buy; Partial/No-Show → lower-commitment first.

---

### Step 4 — Asset sharpening

After drafting, call:

1. **`subject-line-preview-text-optimizer`** — pass every subject line + preview text pair.
   Revise based on returned scores and recommendations before presenting.
2. **`cta-variant-generator`** — request the primary CTA for each email's conversion goal
   (join, watch replay, start trial, book demo, etc.). Use Quick mode per email; no need for
   a full Battery unless the user asks for A/B options.

---

### Step 5 — Voice pass + save

Run **`de-slop-humanize-pass`** over the full sequence to strip hollow filler, passive
constructions, and any AI-register phrases that slipped through.

Save the polished sequence to `./events/<webinar-slug>/email-sequence.md`.
If a replay URL or other TBD item is still missing, mark clearly in the file as `[TBD: X]`
so the user can drop it in without re-running the skill.

---

## Principles

- **Brand-brain first.** No email until the brand context is loaded. Voice and banned-words
  are non-negotiable overrides for every word in the sequence.
- **Segment, don't blast.** Attended, Partial, and No-Show readers have different contexts
  and different trust balances. One email to all three degrades every segment.
- **Emails earn the next open.** The confirmation earns the reminder. The reminder earns
  the day-of. The value-extract earns the nurture. Every email must deliver before it asks.
- **Real proof or nothing.** No invented statistics, fabricated testimonials, or implied
  outcomes the brand cannot support. Unconfirmed proof is marked `[verify]`.
- **One CTA per email.** Multiple competing calls to action dilute all of them.
- **Timing is part of the copy.** Flag send delays clearly in the output so the ESP setup
  matches the creative intent.

---

## What Not to Do

- Don't write a replay email before confirming the replay URL exists (or marking `[TBD]`).
- Don't send guilt-based subject lines to no-shows ("You missed it" as a reproach). Re-sell.
- Don't collapse all three attendance segments into one email; the effort is the job.
- Don't exceed 200 words of body copy in any single webinar email.
- Don't reimplement brand scanning or voice derivation — call `brand-brain`.
- Don't use emojis, exclamation marks, or urgency language the brand bans.
- Don't produce subject lines before running them through `subject-line-preview-text-optimizer`.

---

## Quality Checklist

- [ ] `brand-brain` called; active brand loaded; voice + banned-words applied throughout?
- [ ] All three attendance segments have distinct replay and nurture emails?
- [ ] Pre-Event Ladder covers confirmation + at least 2 reminders + day-of send?
- [ ] Post-Event Arc has a value-extract email before any hard offer?
- [ ] Every subject/preview pair run through `subject-line-preview-text-optimizer`?
- [ ] Every primary CTA generated by `cta-variant-generator`?
- [ ] `de-slop-humanize-pass` completed on the full sequence?
- [ ] All TBD items (replay URL, speaker bio, offer link) clearly flagged in saved file?
- [ ] No invented proof; all claims traceable to brand-brain's returned proof block?
- [ ] Saved to `./events/<webinar-slug>/email-sequence.md`?
