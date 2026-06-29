---
name: event-social-promotion-pack
description: >
  Turns an event brief (name, date, speakers, registration URL) into a complete, ready-to-publish
  social promotion set across LinkedIn, X, and Instagram — pre-event, day-of, and post-event phases —
  calibrated to each platform's format, algorithm preference, and audience mindset. Loads brand voice
  and ICP through brand-brain so every post sounds like the brand, not a generic event blast. Calls
  linkedin-post-writer for LinkedIn depth, x-thread-writer for X threads, cta-variant-generator for
  the registration CTA, and subject-line-preview-text-optimizer for the copy that surfaces in feed
  previews. Produces a dated content calendar table alongside the post copy so you can schedule
  immediately. Use when the user says "promote our webinar on social," "write event social posts,"
  "create a social promotion plan for our event," "I need LinkedIn and X posts for the webinar,"
  "event promotion content," or hands you an event brief and asks for channel copy.
---

# Event Social Promotion Pack

Most event social promotions fail not from lack of effort but from lack of sequencing. Three posts go out in the week before, all saying roughly the same thing, all sounding like a calendar reminder. Attendance stays low; no one clips the replay.

This skill builds a **promotion arc** on a **3-Phase Event Arc** (Pre-Event → Day-Of → Post-Event) — a sequenced, platform-native set of posts that builds anticipation before the event, drives last-mile registrations on the day, and extends shelf-life after. Each phase optimizes a different funnel transition, so no two posts compete for the same job. Every post is in the brand's voice, anchored to a real speaker or a real insight — never a generic "join us."

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, ICP, banned words, real proof, and positioning. No post is written until it returns.
- **`linkedin-post-writer`** — produces the narrative LinkedIn post(s) where depth earns dwell time.
- **`x-thread-writer`** — produces the X thread for high-reach pre-event or post-event amplification.
- **`cta-variant-generator`** — generates the registration CTA variants (button labels + friction-reducer microcopy) for each phase.
- **`subject-line-preview-text-optimizer`** — stress-tests feed preview text and post openers for click-magnetism.
- *(optional)* `content-repurposer-atomizer` — if post-event replay recording exists, atomizes it into additional assets. `social-content-calendar-builder` — if a standing content calendar already exists, slot the posts in rather than creating a standalone calendar.

---

## How a run works

```
Step 0   Load the brand          ──► call brand-brain
Step 1   Parse the event brief   ──► extract the 6 required inputs
Step 2   Build the arc           ──► assign posts to phases per the 3-Phase Event Arc (house model)
Step 3   Write platform-native   ──► LinkedIn, X, Instagram per phase (call siblings)
Step 4   Calendar table          ──► schedule with dates and optimal posting times
Step 5   Self-review             ──► quality checklist before presenting
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before writing a single word. It returns voice adjectives, banned words, offer mechanics, ICP + awareness tendency, and real proof. Obey these as hard overrides. Every post must sound like the brand; CTAs must point to real registered destination URLs.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If neither exists, ask the user: (1) 3 voice adjectives, (2) ICP + their awareness level of this event type, (3) any banned words. Proceed once you have this minimum.

### Step 1 — Parse the event brief

Extract or ask for:

| Input | Why it matters |
|---|---|
| Event name + topic | The hook; everything else anchors to it |
| Date + time (+ timezone) | Day-of posts and countdown copy depend on this |
| Speakers (name + title + org) | Credibility signal; LinkedIn tags; quote sources |
| Registration / replay URL | Every phase needs a real destination |
| Format (live webinar / virtual summit / in-person / hybrid) | Changes the copy promise ("attend" vs "watch live" vs "reserve a seat") |
| Key audience takeaway (1–3 bullets) | What post openers lead with — not the event title |

If any of these are missing, ask before proceeding. Do not invent speaker titles, credentials, or event statistics.

---

## The 3-Phase Event Arc (the opinionated core — house model)

A temporal sequencing model: **Pre-Event → Day-Of → Post-Event**, each with a different conversion job. (House model, named for the mechanic. It is not the PESO media-mix model — no Paid/Earned/Shared/Owned mapping here; this is purely a timeline.)

What separates an arc from a calendar blast is that each phase optimizes for a *different* funnel transition, so no two posts compete for the same job. At a glance:

| Phase | Window | Funnel job | Reader's head | The one thing each post does | Default CTA |
|---|---|---|---|---|---|
| Pre-Event | T−14 → T−1 | awareness → registration | "interesting, maybe later" | move them off the "I'll register later" default | Register / save your seat |
| Day-Of | T−4hr → end | registration → attendance | "I forgot it was today" | turn a registration into an actual show-up | Join now / here's the link |
| Post-Event | T+0 → T+7 | non-attendee → replay; attendee → next event | "missed it / what did I miss" | recover the missed-it audience and bank the source material | Watch the replay |

Three phases, each broken into countdown sub-phases below. The rule that makes it work: **one phase's posts never do another phase's job.** A save-the-date on day-of and a "watch the replay" before the event both miss.

### Phase 1 — Pre-Event (awareness → registration)

Goal: make the value case; overcome the "I'll register later" default.

**Sub-phases by countdown:**
- **T−14 to T−8 days:** Save-the-date anchor post. Lead with the *outcome* the attendee gains — not "we're hosting." Speaker credibility as a proof signal. One CTA: register.
- **T−7 days:** Speaker spotlight. One speaker per post. Quote or insight teaser sourced from their public writing or a real past talk. Tag them (LinkedIn/X) to activate their network.
- **T−3 days:** Urgency / value-reiterate. The specific, concrete thing they walk away knowing. Low-commitment CTA option ("save your seat, replay available") reduces fear of conflict.
- **T−1 day / morning-of-tomorrow:** Countdown reminder. Short. One image or visual cue. The friction-reducer: "30 min, zero fluff, replay for all registrants."

**Platform guidance (pre-event):**
- **LinkedIn:** narrative hook (a question the event answers, or a stat that creates tension) + full speaker + CTA block. 150–220 words. Call `linkedin-post-writer`.
- **X:** single punchy post for the save-the-date; thread for the speaker spotlight (hook → speaker bio → insight teaser → CTA). Call `x-thread-writer`.
- **Instagram:** visual-first; copy is the caption backup. Lead line in the first 125 chars (what truncates before "more"). Speaker headshot or event graphic implied. Emoji-use follows brand permission from `brand-brain`.

### Phase 2 — Day-Of (registration → attendance)

Goal: activate registered users, catch last-minute converters, build FOMO.

- **Morning post (2–4 hrs before):** "We're live today." Time, link, what they'll take away. Optimize the opener for preview text — call `subject-line-preview-text-optimizer` on the first line.
- **"Going live" post (15–30 min before):** The shortest post in the pack. Pure action: time, link, one-line outcome. No filler.
- **Highlight drop (during or immediately after, if team bandwidth allows):** Live quote or poll result from inside the event. Keeps the feed alive for those who couldn't attend and creates organic engagement.

**Platform guidance (day-of):**
- LinkedIn and X: brevity wins. The post is a door; don't over-dress it.
- Instagram Stories (if applicable): countdown sticker + link sticker. Caption is secondary. Note to user: confirm Stories are in-scope.

### Phase 3 — Post-Event (extend shelf-life → replay conversions)

Goal: convert missed-it prospects; surface insights; set up the next event.

- **Replay announcement (within 24 hrs):** "You asked for the replay." Lead with the top moment or result (a specific quote, a poll result, a surprising stat from the session). CTA: watch replay.
- **Key-insight pull (Day 2–5):** 3–5 posts over the following days, each surfacing one insight or quote. These extend organic reach without requiring more content production — the event already created the source material. Call `content-repurposer-atomizer` if a recording is available.
- **Thank-you / social proof post (Day 3–7):** Attendance/watch stat (real or `[verify]`) + a registrant quote if available + "next event" plant if scheduled.

**Platform guidance (post-event):**
- **LinkedIn:** the replay announcement is a full narrative post — it ranks in search and earns saves. Call `linkedin-post-writer`.
- **X:** the insight-pull posts are natively threaded. Call `x-thread-writer` for the replay thread.
- **Instagram:** insight carousels are high-engagement; note the asset requirement for the user.

---

## Calendar output format

After writing all posts, produce a dated content calendar table:

```
## Event Social Promotion Calendar — [Event Name]

| Date | Platform | Phase | Post type | Copy (truncated) | CTA destination |
|------|----------|-------|-----------|------------------|-----------------|
| MMM DD | LinkedIn | Pre / T−14 | Anchor | [first 15 words…] | [URL] |
…
```

Save the full post copy and calendar to `./events/[event-slug]-social-pack.md`. The calendar table also prints inline for immediate use.

---

## Principles (Non-Negotiable)

- **Brand-brain loads first.** No copy before it returns. Voice and banned words are hard overrides.
- **One post, one job.** Every post has a single conversion goal. Don't stack the CTA and the speaker bio and the agenda into the same post.
- **Lead with the outcome, not the event.** "Learn how to cut cart abandonment 28%" beats "Join us for our webinar."
- **Real proof only.** Speaker titles, attendance stats, and quotes are confirmed or marked `[verify]`. Never invent them.
- **Platform-native, not platform-agnostic.** LinkedIn rewards narrative depth. X rewards brevity and threads. Instagram rewards the first 125 chars. Writing one post for all three is always the wrong call.
- **Tag speakers.** Tagging activates the speaker's network at zero cost. Note when you can't confirm handles.
- **Phase logic is not optional.** Each post does its own phase's job and no other — a day-of "join now" published in the pre-event window and a save-the-date published day-of both miss the funnel transition they land on. Date discipline is the whole game.

---

## What Not to Do

- Don't write posts before `brand-brain` returns the active brand.
- Don't use generic openers: "We're excited to announce," "Join us for," "Don't miss." These are the first words the algorithm learns to bury.
- Don't publish the same copy across platforms — adapt format and length to each.
- Don't invent speaker quotes, credentials, or attendance numbers. Mark anything unconfirmed `[verify]`.
- Don't skip the post-event phase. For most webinars the replay audience rivals or exceeds the live audience [verify against the brand's own replay-vs-live numbers] — skipping it leaves the larger conversion window on the table.
- Don't produce more posts than a lean team can actually schedule — 8–12 total (across all platforms and phases) is the realistic ceiling for a single event without a dedicated social team. Offer to expand if bandwidth exists.
- Don't use emojis or exclamation marks unless `brand-brain` confirms they're on-brand.

---

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called; voice + banned words honored; only real proof used (rest `[verify]`)?
- [ ] All 6 event brief inputs confirmed (name, date, speakers, URL, format, takeaway)?
- [ ] Posts cover all three phases of the 3-Phase Event Arc (Pre-Event, Day-Of, Post-Event), and each post does only its own phase's job?
- [ ] Each platform's format respected (LinkedIn narrative, X brevity/thread, Instagram caption-first)?
- [ ] Every CTA points to a real destination URL from the brief?
- [ ] Speaker names, titles, orgs confirmed — not invented?
- [ ] `linkedin-post-writer` called for LinkedIn posts; `x-thread-writer` called for X threads?
- [ ] Calendar table produced with dates, platforms, and copy?
- [ ] Output saved to `./events/[event-slug]-social-pack.md`?
- [ ] No opener begins with "We're excited to," "Join us," or "Don't miss"?
