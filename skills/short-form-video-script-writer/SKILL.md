---
name: short-form-video-script-writer
description: >
  Writes platform-native short-form video scripts for TikTok, Instagram Reels, YouTube Shorts,
  and LinkedIn Video — from a product feature, campaign message, topic, or repurposable asset.
  Outputs a timestamped script with hook line, value body, on-screen text cues, scene direction,
  voiceover/caption split, and a closing CTA, calibrated to the target platform's norms and the
  specified duration (15 s / 30 s / 60 s / 90 s). Brand voice, ICP language, banned words, and
  proof points all come from the `brand-brain` skill — this skill writes, it does not rebuild brand
  context. Calls `cta-variant-generator` for the closing action, and `de-slop-humanize-pass` on
  any draft that reads like AI copy. Offers to save the finished script to `./social/` as a
  reusable asset. Trigger phrases: "write a TikTok script," "script a Reel," "short-form video
  script," "YouTube Shorts script," "LinkedIn video script," "script for a 30-second video,"
  "video for [feature / campaign / topic]," "turn this into a video."
---

# Short-Form Video Script Writer

Product feature, campaign message, or topic → platform-calibrated, timestamped video script with hook, value body, on-screen text cues, scene direction, and a proven-framework CTA — all in the brand's real voice.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, ICP language, banned words, proof, and positioning before a single word is written.
- **`cta-variant-generator`** (required) — generates the closing CTA line and on-screen button/text variant so the ask is conversion-tested, not improvised.
- **`de-slop-humanize-pass`** (optional, recommended) — strips AI cadence and generic phrasing from the final draft; call when the output reads polished-but-flat.
- **`content-repurposer-atomizer`** (optional) — if the source asset is a long-form article or blog post, call it first to extract the core angle and key lines before scripting.
- **`headline-hook-generator`** (optional) — if the hook needs multiple angle options before committing, call it to generate a ranked set.

---

## How a run works

```
Step 0  Load brand        ──► call brand-brain (voice, banned words, ICP, proof, positioning)
Step 1  Gather inputs     ──► platform + duration + source material / topic + goal
Step 2  Build the script  ──► apply the Hook–Value–Pattern Interrupt–CTA (HVPC) framework
Step 3  Generate CTA      ──► call cta-variant-generator for the close
Step 4  Humanize (if needed) ──► call de-slop-humanize-pass on the full draft
Step 5  Present + persist ──► show formatted script; offer to save to ./social/
```

### Step 0 — Load brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics, real proof, positioning, ICP vocabulary and awareness tendency. Do not write a single word of script until it returns.

Obey voice + banned words as hard overrides. Use only real proof — anything unconfirmed gets `[verify]`. Anchor the ICP's language (their words, not product-team words) in the hook and value body.

Fallback if `brand-brain` not installed: read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (product + ICP + voice adjectives + banned words), then proceed.

### Step 1 — Gather inputs

Collect (in one message if missing from the request):
- **Platform:** TikTok / Instagram Reels / YouTube Shorts / LinkedIn Video / unspecified (platform-agnostic)
- **Duration:** 15 s · 30 s · 60 s · 90 s (default: 30 s)
- **Source material:** topic, product feature, campaign angle, article URL, or paste-in copy
- **Goal:** awareness / consideration / conversion / retention / engagement
- **Delivery format:** voiceover narration · talking-head · text-on-screen only · B-roll + VO

If the user pastes source material from a long article or blog post, call `content-repurposer-atomizer` to extract the sharpest angle before scripting — do not just paraphrase the article.

---

## The HVPC framework (Hook → Value → Pattern Interrupt → CTA)

Every high-performing short-form video follows this spine. Build the script around it; do not deviate.

| Beat | Purpose | Target timing (30 s) | Target timing (60 s) |
|---|---|---|---|
| **Hook** | Stop the scroll; state or tease the payoff | 0–3 s | 0–5 s |
| **Value** | Deliver the promise in tight, concrete chunks | 3–22 s | 5–45 s |
| **Pattern Interrupt** | Reset attention mid-video (cut, stat, visual shift, reframe) | 10–14 s | 20–30 s |
| **CTA** | One clear ask, on-brand, low-friction | 22–30 s | 45–60 s |

**Hook rules (non-negotiable).** The first 1–3 seconds are the entire game on TikTok and Reels. The hook must do one of: make a bold claim, name an uncomfortable truth, tease a surprising outcome, or ask a question the ICP already has in their head. It must not start with "Hey guys," a brand name, or a product overview. On LinkedIn Video the grace period extends to ~5 s.

**Pattern Interrupt.** At the midpoint, break the rhythm: introduce a counterintuitive stat, cut to a different visual, pivot from problem to solution, or reframe the stakes. Without this, watch-time drops at the halfway mark.

**CTA rules.** One action only. Call `cta-variant-generator` (skill: cta-variant-generator) with the video goal and platform to produce the closing line — do not invent a CTA inline. Use the recommended primary from that output; include the friction-reducer microcopy as a lower-third text cue.

---

## Platform calibration

Apply these constraints on top of the framework. Never ignore them; they are platform hygiene, not style preferences.

| Platform | Optimal duration | Caption style | Hook window | On-screen text density | Key constraint |
|---|---|---|---|---|---|
| TikTok | 21–34 s (sweet spot) | Captions on always; punchy sentence-per-line | 1.5 s | High — text overlays carry meaning | Trending audio overlay is out of scope; note slot for it |
| Instagram Reels | 15–30 s for discovery; up to 60 s for saves | Captions on; subtitle-style | 2 s | Medium-high | Ratio is 9:16; first frame must work as a thumbnail |
| YouTube Shorts | 15–60 s | Auto-captions; manual for accuracy | 3 s | Medium | No end screen; CTA must be spoken + on-screen |
| LinkedIn Video | 30–90 s | 85 % watch without sound — captions are mandatory | 5 s | Medium | Professional register; no trending sound; native upload wins over links |
| Platform-agnostic | 30 s default | Assume always-on captions | 2–3 s | Moderate | Optimize for sound-off comprehension |

---

## Script format

Deliver every script in this exact format so it can be handed directly to a creator or edited in a teleprompter app:

```
## [Brand slug] | [Platform] | [Duration] | [Goal]
Topic: [one line]
ICP: [persona from brand-brain]
Voice: [3 adjectives from brand-brain]

---

[0:00–0:03] HOOK
Visual cue: [scene / text card / talking head direction]
Spoken: "[exact words]"
On-screen text: "[overlay or lower third]"

[0:03–0:22] VALUE — BEAT 1: [sub-beat label]
Visual cue: [scene direction]
Spoken: "[exact words]"
On-screen text: "[overlay]"

[0:12–0:16] PATTERN INTERRUPT
Visual cue: [cut / text smash / reframe direction]
Spoken: "[exact words, if any]"
On-screen text: "[stat / bold claim]"

[0:22–0:30] CTA
Visual cue: [talking head or graphic]
Spoken: "[CTA from cta-variant-generator primary]"
On-screen text: "[friction-reducer microcopy from cta-variant-generator]"

---
Director's note: [1–2 sentences on pacing, energy, or delivery — brand voice specific]
```

Adjust timestamps proportionally for 15 s / 60 s / 90 s. For 90 s add a second Pattern Interrupt at ~55 s.

---

## Quick mode vs. Battery mode

- **Quick (default):** One platform + one duration → one finished script.
- **Battery (on request):** "Give me variations" / "multi-platform" / "two angles" → produce two scripts on different hooks/angles for the same core topic, or one script per specified platform, each in the format above. Present a one-line recommendation on which to shoot first and why (platform audience size, goal-fit, or brand-voice match).

Default to Quick. Offer Battery at the end if the topic is a hero campaign or multi-platform push.

---

## Principles

- **Brand-brain loads first.** No script before the brand digest returns. Voice + banned words are non-negotiable overrides — not suggestions.
- **Hook is the product.** The first 1–3 seconds determine whether the other 27 get watched. Spend disproportionate effort here.
- **ICP language, not product-team language.** Write the hook and value beats in the words the ICP uses to describe their problem — not the way the brand describes its solution.
- **Sound-off must still work.** Every key claim appears as on-screen text. Captions are always on. Never bury the message in audio alone.
- **One CTA, one action.** Call `cta-variant-generator`; never stack competing asks.
- **Concrete over clever.** A specific number, outcome, or named pain beats a witty abstraction every time. If you don't have proof, mark it `[verify]` — do not invent.
- **Pattern interrupts are structural, not decorative.** They exist to reset attention at the watch-time cliff; they must land at the midpoint, not wherever it feels natural.

---

## What not to do

- Do not start the hook with the brand name, "Hey everyone," a product overview, or a mission statement.
- Do not write the CTA inline — always call `cta-variant-generator` so the ask is tested against the brand's funnel stage and awareness level.
- Do not pad the script to hit the duration ceiling — short is better than padded; flag if the material is thin.
- Do not use banned words from `brand-brain` even if they feel right for the platform.
- Do not invent social proof, stats, or customer outcomes — mark anything unconfirmed `[verify]`.
- Do not skip the on-screen text column — creators drop it; don't make it easy.
- Do not produce a script before `brand-brain` returns the active brand.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and digest loaded before any word was written?
- Hook lands within the platform's scroll-stop window (≤3 s TikTok/Reels, ≤5 s LinkedIn)?
- Hook uses ICP language — not product-team language?
- Pattern interrupt placed at the midpoint, not end?
- CTA produced by `cta-variant-generator` — one action, on-brand, low-friction?
- Every key claim present as on-screen text (sound-off comprehension passes)?
- No banned words; no invented proof (`[verify]` on anything unconfirmed)?
- Timestamps are accurate for the chosen duration?
- `de-slop-humanize-pass` called (or flagged to the user) if the draft reads flat?
- Offer to save to `./social/[brand-slug]-[platform]-[topic-slug].md` made?
