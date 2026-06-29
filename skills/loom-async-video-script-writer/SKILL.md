---
name: loom-async-video-script-writer
description: >
  Turns a topic, audience, and key points into a tight, natural-sounding async-video script — Loom,
  Slack video, Notion recording, Zoom async clip, or any camera-to-camera message. Outputs a ready-to-record
  script with an attention-grabbing open (hook in the first 8 seconds), punchy body sections with
  natural verbal cues, and a clear single call to action. Uses SCOREcard — our working acronym (Setup,
  Conflict, Offer, Resolution, End-action) — to give every recording a narrative spine so the viewer
  stays engaged rather than dropping off at 30 seconds. Brand-voice and proof come from `brand-brain`
  so the script never sounds generic or off-brand. Optionally writes a companion thumbnail title and
  preview-text teaser. Use whenever the user says "write a Loom script," "async video script," "record
  a video for my team," "stakeholder update Loom," "product-demo Loom," "I need a script for a Loom,"
  "write a video update," "Loom for a customer," "script for a walking-through recording," or hands
  over talking points and asks for a camera-ready script.
---

# Loom & Async-Video Script Writer

Most async videos ramble because the sender didn't write before recording. This skill fixes that: topic and key points in, camera-ready script out. Every script follows SCOREcard, our working acronym — a narrative spine designed for the 2–5 minute async format where the viewer can mute, scrub, or close at any second.

Brand voice and proof come from `brand-brain`. A customer-facing Loom for a SaaS brand sounds nothing like an internal ops update, and neither should be written without knowing who's watching and why.

---

## Skills this calls

- **`brand-brain`** (required for external-facing scripts) — loads active brand voice, banned words, offer mechanics, and real proof. Optional but still invoked for internal scripts to ensure tone consistency across contexts.
- **`sales-asset-reviewer`** (optional) — if the Loom is a sales or demo video, run the output through the reviewer for messaging sharpness before handing to a rep.
- **`deck-presentation-writer`** (optional) — if the user needs a companion slide deck or screen-share outline to record alongside the script.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (always; returns digest or bootstraps)
Step 1  Classify the video  ──► type × audience × length
Step 2  Apply SCOREcard  ──► generate the structured script
Step 3  Self-review  ──► brand voice, hook strength, CTA clarity, pacing
Step 4  Deliver  ──► script + optional thumbnail title + teaser line
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, offer mechanics, and real proof points. Use the voice as a hard constraint on tone and vocabulary. Use only returned proof (mark unconfirmed numbers `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none exists, ask three questions: (1) Who is watching and what's their relationship to the sender? (2) What is the one thing they should do or understand after watching? (3) Three words that describe the right tone. Proceed only after answers are in hand.

---

## Classify the video (Step 1)

Before scripting, resolve these four attributes — infer from context, confirm only if ambiguous:

| Attribute | Options |
|---|---|
| **Type** | Internal update · Customer / prospect · Product demo · Sales follow-up · Onboarding · Tutorial · Stakeholder report |
| **Audience relationship** | Peer · Direct report / team · Manager / exec · Customer (warm) · Prospect (cold) · Partner |
| **Target runtime** | Short ≤ 90s · Standard 2–4 min · Long 5–8 min |
| **Screen share?** | Camera-only · Screen + camera · Screen-only |

Target runtime governs density: Short = hook + one point + CTA; Standard = hook + 2–3 points + CTA; Long = hook + 4–5 structured beats + CTA. Never pad to fill time.

---

## SCOREcard (our working model)

SCOREcard is a house acronym we use here, not an established external framework. Every script gets a narrative spine; following the beats helps a script stay tight and engaging regardless of who writes it.

| Beat | What it does | Target length |
|---|---|---|
| **S — Setup** | Who you are (if external), why you're recording instead of typing, what the viewer will get | 8–15 s |
| **C — Conflict** | The problem, gap, decision, or question this video resolves — creates the reason to watch | 10–20 s |
| **O — Offer** | Your core message, demo, walkthrough, or recommendation — the substantive payload | 60–80% of runtime |
| **R — Resolution** | What the viewer now knows, has, or can do that they couldn't before | 10–15 s |
| **E — End-action** | One specific CTA — reply, click, decide, schedule, approve | 8–12 s |

**The hook is the Setup + opening line of Conflict.** It must land in the first 8 seconds or the drop-off rate spikes. Start with the payoff or the problem — never with "Hi, so, today I wanted to…"

---

## Script format

Deliver a clean, ready-to-record document. Use this structure:

```
## [VIDEO TITLE — internal label only]
Type: [type] · Audience: [relationship] · Target runtime: [N min] · Screen share: [yes/no]
Brand: [slug, via brand-brain]

---

### THUMBNAIL TITLE
[Punchy, benefit-led title for the video link preview — ≤ 60 chars]

### PREVIEW TEASER
[One sentence the recipient reads before clicking — raises curiosity or urgency, ≤ 120 chars]

---

### SCRIPT

[SETUP — ~10s]
[Spoken text here. Write in natural speech — contractions, conversational rhythm, no corporate-ese.]

[CONFLICT — ~15s]
[...]

[OFFER — X min]
[Break into labeled sub-sections if 2+ points. Each ends with a micro-transition to the next.]

[RESOLUTION — ~10s]
[...]

[END-ACTION — ~10s]
[Single CTA. Be specific: "Reply to this Loom with your approval" beats "Let me know what you think."]

---

### PACING NOTES
[Optional: flag any segment likely to run long; suggest where to cut if over-budget.]
```

Save to `./async-scripts/[slug]-[date].md` if the user wants persistence; default is inline delivery.

---

## Script craft principles

**Hook in 8 seconds.** The viewer's decision to watch is made by second 8. Open with what they'll get, the problem you're solving, or a surprising fact — never with pleasantries or slow context-setting.

**Write for the ear, not the eye.** Contractions always ("you're," not "you are"). Short sentences. Trailing thoughts cut. Read it aloud and time it — if you stumble, rewrite.

**One main point per recording.** If the topic requires more than five beats, it needs a meeting or a doc, not a Loom. Flag this and offer to split.

**Micro-transitions maintain momentum.** "Here's why that matters…" / "Let me show you what I mean…" / "So the question is…" keep the viewer from scrubbing. Place one at each beat boundary in the Offer section.

**The CTA is one action.** "Reply, click, and share" is three CTAs — it's zero CTAs. End with a single, specific ask with a clear verb and deadline if relevant ("Reply by Friday with a thumbs up or a question").

**Proof goes in the Offer, not the Setup.** Don't front-load credentials or data in the intro. Earn attention first, then substantiate.

**Screen-share scripts need visual anchors.** For recordings with screen, mark `[SHOW: ...]` cues in brackets so the recorder knows exactly what to have on screen during each spoken beat.

---

## Type-specific conventions

**Internal team update / stakeholder report**
- Lead with the decision or status, not background. Execs scrub to the outcome.
- Resolution = what you need from them (approve, unblock, acknowledge).
- Tone: direct, no corporate filler.

**Customer / warm prospect Loom**
- Use the brand's voice fully — this is external comms.
- Offer beat = value demo or relevant case (brand proof from `brand-brain` only; mark unconfirmed `[verify]`).
- CTA = one low-friction next step (reply, book a call, view the linked doc).

**Sales follow-up / post-demo Loom**
- Start with a callback to the specific conversation ("You mentioned your biggest pain was X…").
- Offer = the one thing from the demo that maps to their stated problem.
- Optionally route through `sales-asset-reviewer` before sending.

**Tutorial / walkthrough**
- Script the narration AND the screen actions in parallel (`[SHOW: ...]` for every visual beat).
- Offer section = numbered steps, each with the spoken instruction + on-screen action.
- End with the "now you can" moment — Resolution is critical here.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No external-facing script before `brand-brain` returns. Voice + banned words override everything.
- **SCOREcard always.** Every script has a Setup, Conflict, Offer, Resolution, and End-action — regardless of length.
- **Hook by second 8.** If the first sentence doesn't earn the watch, rewrite it.
- **One CTA.** Never end with multiple asks or an open-ended "thoughts?"
- **Ear-first writing.** Contractions, short sentences, natural rhythm. If you wouldn't say it aloud, cut it.
- **Real proof only.** Unconfirmed numbers or outcomes are `[verify]`.
- **Right length.** Over-long scripts get cut down before delivery, not padded to meet a target.

---

## What not to do

- Don't start the script with "Hi [Name], so today I just wanted to…" — rewrite every slow opener.
- Don't write five CTAs or end ambiguously ("let me know what you think").
- Don't use banned words from `brand-brain` even if the script is internal — tone consistency matters.
- Don't invent proof, stats, or customer outcomes — use only what `brand-brain` confirms or mark `[verify]`.
- Don't stack all the proof in the Setup — earn the viewer's attention first.
- Don't skip the SCOREcard for "short" recordings — even a 60-second script needs a hook and a CTA.
- Don't write a script that requires a 10-minute Loom — flag and offer to split or suggest a doc instead.

---

## Quality checklist (self-review before delivering)

- `brand-brain` called and brand loaded; voice + banned words applied as hard overrides?
- Video type, audience relationship, target runtime, and screen-share status confirmed?
- Hook lands by second 8 — no slow opener?
- SCOREcard complete: Setup → Conflict → Offer → Resolution → End-action?
- Offer section labeled and broken into beats with micro-transitions?
- Screen-share scripts have `[SHOW: ...]` cues at every visual beat?
- Single, specific CTA with a clear verb?
- Written for the ear: contractions, short sentences, no corporate-ese?
- Thumbnail title (≤ 60 chars) and preview teaser (≤ 120 chars) included?
- All proof from `brand-brain`; unconfirmed items marked `[verify]`?
- Runtime estimated; pacing notes added if any beat risks over-running?
