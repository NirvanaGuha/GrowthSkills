---
name: instagram-caption-reel-script-writer
description: >
  Turns a product brief, campaign idea, or image/video description into a publish-ready Instagram
  caption (with hashtag block) plus — when requested or when the input is video-native — a
  full 30–60-second Reel script with on-screen text cues, hook line, and B-roll direction.
  Uses the HOOK → VALUE → PROOF → CTA framework anchored to Instagram's real algorithm signals
  (save rate, share rate, watch time). Brand context always comes from the `brand-brain` skill —
  voice, proof, ICP, and banned words override everything. Calls `cta-variant-generator` for
  the caption CTA so you get tested, angle-varied options instead of a generic "link in bio."
  Calls `post-quality-reviewer-voice-auditor` at the end when the user wants a quality pass
  before publishing. Use when the user says "write an Instagram caption," "Instagram Reel
  script," "write a Reel," "caption for this post," "IG copy," "30-second Reel," "short-form
  Instagram video," or hands over an image, offer, or creative brief and asks for Instagram copy.
---

# Instagram Caption & Reel Script Writer

Product brief or image in → on-brand caption with hashtags out. Ask for a Reel and you get a
30–60-second script with hook, scene cues, on-screen text, and a closing CTA — written to
Instagram's actual ranking signals, not to generic "social media best practices."

Every caption and script is written in the active brand's voice, with the brand's real proof,
against the brand's real ICP. That context comes from `brand-brain`, once.

---

## Skills this calls

- **`brand-brain`** (required, always first) — active brand digest: voice adjectives, banned
  words, offer mechanics, proof, ICP, positioning. Does not get re-implemented here.
- **`cta-variant-generator`** (caption mode) — generates the caption CTA with angle variation
  and awareness-ceiling calibration. Synthesize inline when absent.
- **`post-quality-reviewer-voice-auditor`** (optional, on request or when user asks for a
  review pass) — structural review of hook strength, CTA, platform-fit, brand-voice
  consistency before the user publishes.

---

## How a run works

```
Step 0  Brand context    ──► call brand-brain; wait for the digest
Step 1  Mode             ──► Caption-only (default) | Reel script | Both
Step 2  Draft            ──► apply HOOK → VALUE → PROOF → CTA to the chosen format
Step 3  CTA              ──► call cta-variant-generator (caption) or embed hook-to-CTA arc (Reel)
Step 4  Hashtags         ──► build 3-tier block (niche / mid-range / broad)
Step 5  Self-review      ──► Instagram format rules + brand + quality checklist
Step 6  Present          ──► optional: call post-quality-reviewer-voice-auditor if requested
```

---

## Step 0 — Brand context (always first)

Invoke the `brand-brain` skill before writing a single word of copy. It returns the active
brand's digest — voice adjectives, banned words/phrases, offer mechanics + real proof, ICP,
positioning, awareness tendency. Obey voice and banned-words as hard overrides. Mark any
unconfirmed stat `[verify]`.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that
brand's `brand.md`; if absent, ask the user to install `brand-brain` (preferred) or answer a
4-question mini-setup (product + ICP + offer + 3 voice adjectives + banned words). Always
prefer the call.

---

## Step 1 — Mode selection

| Trigger | Mode |
|---|---|
| Image, screenshot, or product brief provided, no mention of video | **Caption-only** |
| User says "Reel," "video script," "30-second," "script" | **Reel script** |
| User says "both" or "caption + script" | **Caption + Reel** |

When the input is ambiguous, default to Caption-only and offer Reel at the end.

---

## Caption mode — HOOK → VALUE → PROOF → CTA

Instagram captions rank by **save rate** (utility signal) and **comment rate** (conversation
signal). Write for those two, not for likes.

### Line 1: The hook (above the fold)
The first line is the only line visible before "more." It must stop the scroll:
- **Pattern-interrupt or specificity:** "3 things your push notifications are doing wrong" beats
  "Want more conversions?"
- **Curiosity gap or identity signal:** the reader immediately knows this is for them.
- No branded filler ("We're excited to share…"). No period at the end — it invites the tap.
- 125 characters maximum; under 90 preferred.

### Lines 2–6: Value body
- Expand the hook promise: deliver the utility or story that justifies the save.
- Short paragraphs; 1–2 sentences each. Line breaks between every paragraph (Instagram renders
  blank lines, not `<br>`).
- One concrete insight, step, or proof point per paragraph.
- No walls of text. If a list works, use it; Instagram renders plain dashes or numbers.

### Proof line
- One real stat, customer outcome, or social proof from the brand's `proof.md` or the
  `brand-brain` digest.
- Unconfirmed claims get `[verify]`; never invent a number.

### CTA
- Call `cta-variant-generator` with the post context (brand slug, ICP, awareness stage,
  channel = Instagram caption, destination). Use its recommended CTA.
- Instagram CTAs must account for the no-link constraint: drive saves ("Save this for later"),
  comments ("Drop a question below"), profile/bio visits ("Link in bio"), or DM ("DM us
  'GUIDE'").
- Match the ask to the awareness level — never ask a cold audience for a purchase.

### Hashtag block
Deliver a 3-tier block separated from caption by one blank line. Do not embed hashtags in
caption body (breaks read flow; reduces algorithmic reach [verify]).

| Tier | Volume | Count | Purpose |
|---|---|---|---|
| Niche | < 100K posts | 5–7 | Rank fast, own the conversation |
| Mid-range | 100K–1M posts | 5–7 | Sustainable impressions |
| Broad | 1M+ posts | 2–3 | Reach exposure, not ranking |

Total: 12–17 hashtags. Never hit 30 (spam signal). Always include at least one branded hashtag
if the brand has one.

**Save output to:** `./instagram/captions/[brand-slug]-[date].md` when producing a batch
(>2 captions). Single captions are inline.

---

## Reel script mode — the 4-beat structure

Instagram Reels rank by **watch time and completion rate**. The entire structure is front-loaded
to earn continued watch time.

### Beat 1: Hook (0:00–0:03)
The first 3 seconds determine whether the viewer watches or swipes. One of:
- **Counterintuitive statement** — "Most email marketers are optimizing the wrong metric."
- **Visual pattern interrupt** — cue reads: "[CUT TO: dramatic close-up / unexpected prop]"
- **Direct address** — "If you run an eCommerce store and push open rates are under 15%..."
- **A transformation tease** — "Here's how we doubled retention in 30 days [verify]"

On-screen text for Beat 1: the hook line verbatim, large, centered, ≤8 words.

### Beat 2: Agitation / Setup (0:03–0:12)
Expand the problem or the stakes. Why does this matter right now? Keep it tight — every second
here is borrowed from the payoff. No more than 2–3 sentences of VO.

### Beat 3: Payoff / Value (0:12–0:45)
The tutorial, reveal, or proof. For a 30-second Reel, compress to 2–3 tight steps. For 60
seconds, allow 3–5.
- Each step gets: **VO line + on-screen text label + B-roll cue**.
- Use specific numbers and named features when real proof exists; `[verify]` otherwise.
- Maintain the brand's voice — if banned words include filler like "amazing" or "game-changer,"
  the VO must avoid them.

### Beat 4: CTA (0:45–0:60 or last 5 seconds)
- One action — save, follow, link in bio, DM keyword, comment.
- VO CTA + on-screen text CTA + closing graphic direction.
- Sync with the caption CTA when both modes are running.

### Script format
```
[00:00–00:03] HOOK
VO: "..."
ON-SCREEN: "..."
B-ROLL: [...]

[00:03–00:12] SETUP
VO: "..."
ON-SCREEN: "..." (optional)
B-ROLL: [...]

[00:12–00:45] VALUE
VO: "..."
ON-SCREEN: "Step 1 — ..."
B-ROLL: [...]
(repeat per step)

[00:45–00:60] CTA
VO: "..."
ON-SCREEN: "..."
B-ROLL: [...]
```

**Save Reel scripts to:** `./instagram/reels/[brand-slug]-[descriptor]-[date].md`.

---

## Instagram format rules (hard constraints)

- **Caption length:** 2,200 character limit; first 125 characters are visible before "more."
- **Hashtags:** 12–17 max; placed in a separate block below caption, not in the body.
- **Reel length:** 15–90 seconds native; 30–60 is the sweet spot for completion rate [verify].
- **On-screen text:** ≤8 words per card; center safe zone only (avoid bottom 25% — covered by
  UI chrome).
- **No external links in caption** — Instagram does not make them clickable; drive to bio link
  or DM keyword instead.
- **Aspect ratio for Reels:** 9:16 (1080×1920). Note in B-roll cues when a shot must be
  vertical.

---

## Principles

- **Brand-brain first.** No copy before the digest returns. Voice and banned-words are
  non-negotiable overrides.
- **Optimize for save + completion, not likes.** Save rate and watch-through rate drive
  algorithmic reach; likes are a vanity signal.
- **Hook earns watch time; value earns the save; CTA earns the action.** Each beat has one job.
- **Real proof or `[verify]`.** Never invent a stat or outcome; flag unconfirmed claims.
- **Platform constraints are hard limits, not guidelines.** No external links in captions;
  on-screen text in the safe zone; 9:16 for Reels.
- **One CTA per piece.** Two competing actions split attention and depress both.
- **Awareness ceiling.** Cold audiences get soft CTAs (save, comment, follow); warm audiences
  get conversion CTAs (link in bio, DM for offer). Match the ask to the stage.

---

## What not to do

- Don't write any caption or script before `brand-brain` returns.
- Don't reimplement brand-voice logic or ICP scanning here — that lives in `brand-brain`.
- Don't embed hashtags inside the caption body — algorithmic and readability penalty.
- Don't use more than 17 hashtags or fewer than 5 — both hurt reach.
- Don't write a hook that's vague brand positioning ("We believe in growth") — it has zero
  scroll-stopping power.
- Don't invent proof points or conversion stats; `[verify]` anything unconfirmed.
- Don't use banned words from the brand's brain even if they "sound good."
- Don't produce a generic "link in bio" as the only CTA option — call `cta-variant-generator`
  for angle variation.
- Don't write Reel on-screen text longer than 8 words; it won't read in 3 seconds.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; voice + banned-words applied; only real proof used (rest `[verify]`)?
- Caption hook: first 125 characters stop the scroll, no branded filler, no period?
- Caption body: delivers a saveable insight; clear HOOK → VALUE → PROOF → CTA arc?
- CTA angle-varied via `cta-variant-generator` (or synthesized inline); awareness ceiling
  respected; no unclickable external link in caption body?
- Hashtag block: 12–17, 3-tier structure, separated from body by blank line?
- Reel script: 4 beats present (Hook / Setup / Payoff / CTA); each beat has VO + on-screen
  text + B-roll cue; on-screen text ≤8 words and within safe zone?
- Reel total runtime within 30–60s and explicitly annotated?
- Caption + Reel CTAs aligned when producing both?
- Output saved to correct path (batch captions → `./instagram/captions/`; scripts →
  `./instagram/reels/`) or inline for a single piece?
