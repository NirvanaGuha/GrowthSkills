---
name: tiktok-script-hook-generator
description: >
  Turns a trend, topic, product angle, or campaign brief into a hook-forward TikTok script
  (15–60 seconds) ready to shoot. Uses the Pattern-Interrupt Hook Framework: every script
  leads with a ranked shortlist of opening-line variants built on distinct interrupt mechanics
  (curiosity gap, counter-intuition, call-out, social proof, bold claim), then delivers the
  body and CTA in tight TikTok pacing. Two modes: Quick (one script + ranked hooks) and
  Battery (3 scripts across distinct angles). Calls brand-brain so every script stays on
  voice and uses real proof. Calls headline-hook-generator for the opening-line battery when
  available. Calls post-quality-reviewer-voice-auditor for a voice pass before presenting.
  Calls content-repurposer-atomizer when the user wants to spin a finished script into captions,
  a LinkedIn post, or Reels. Saves output to ./tiktok/[slug]-scripts.md on request.
  Use when the user says "write a TikTok script," "TikTok hook," "script for TikTok,"
  "short-form video script," "hook variants," "write a 30-second video," "TikTok for [product],"
  or hands over a trend or campaign brief and needs video content.
---

# TikTok Script & Hook Generator

Attention is not captured at the second sentence. It is won or lost in the first 1–3 seconds. This skill builds every script hook-first: generate the strongest opening line, then write the body and CTA to pay it off. Everything is calibrated to TikTok's real format constraints, algorithm behavior, and viewer psychology — not generic short-form video advice.

Brand context is always loaded first. The brand's voice, ICP, and real proof shape every word. No generic hooks, no invented stats.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand; returns voice adjectives, banned words, offer mechanics, real proof, ICP + awareness tendency. No script before it returns.
- **`headline-hook-generator`** (Step 2, when installed) — generates the ranked opening-line battery using its persuasion-angle coverage. Synthesize inline if absent.
- **`post-quality-reviewer-voice-auditor`** (Step 4, when installed) — scores hook strength, platform-fit, and brand-voice consistency before presenting output. Run inline review if absent.
- **`content-repurposer-atomizer`** (optional, on request) — repurposes a finished script into platform variants (captions, Reels, LinkedIn post, email snippet).

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain; block until it returns
Step 1  Pick the mode       ──► Quick (default) | Battery (on request)
Step 2  Generate hooks      ──► call headline-hook-generator OR build inline
Step 3  Write the script(s) ──► Pattern-Interrupt Hook Framework (below)
Step 4  Voice review        ──► call post-quality-reviewer-voice-auditor OR inline check
Step 5  Present + offer     ──► output + optional save + repurpose offer
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns: active slug, voice adjectives, banned words, offer mechanics + destination URLs, real proof, ICP + awareness tendency. Do not write a single word of script until it returns.

Obey the returned voice and banned-words as hard overrides. Use only real proof from the brand; mark anything unconfirmed `[verify]`. Match the awareness tendency to the hook type — a cold audience needs curiosity/call-out hooks, not "here's why you need X" closers.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or run a 4-question mini-setup (what it is · ICP + awareness · real proof · 3 voice adjectives + banned words). Always prefer the call.

### Step 1 — Pick the mode

- **Quick (default):** one script (15–60s, user's choice or inferred from the brief) + ranked hook battery. Use when the user gives a single topic, trend, or video brief.
- **Battery:** three scripts, each built on a distinct hook mechanic and angle. Triggered by "options," "3 scripts," "test," "multiple," or "battery." Each gets its own hook shortlist.

When duration is unspecified: 15–30s for awareness/DTC/single-point content; 30–60s for explainer, social proof, or multi-step content.

---

## The Pattern-Interrupt Hook Framework

Every TikTok script is structured as: **Hook → Bridge → Body → CTA**. The hook is the entire job. Everything else pays it off.

### Hook types (distinct mechanics — not synonyms)

| Mechanic | Structure | Best for |
|---|---|---|
| **Curiosity gap** | Open a loop the viewer can't close without watching | Information, tips, reveals |
| **Counter-intuition** | Contradict a common belief in the category | Thought-leadership, repositioning |
| **Direct call-out** | Name the viewer's exact situation or identity | Targeted ICP segments, pain-point products |
| **Proof / social validation** | Lead with a real outcome number or social fact | High-awareness segments, DTC |
| **Bold claim** | Make the payoff unmissable before explaining it | Strong-proof brands; use only with real numbers |

Generate **3–5 ranked hook variants** per script using distinct mechanics (not vocabulary variants). The #1 hook is the recommended take — state why it fits this ICP and awareness level. The alternates are genuinely different bets, not fallbacks.

### Script structure (all durations)

```
[HOOK]     1–3s   Pattern interrupt — the single job. One sentence max.
[BRIDGE]   2–4s   Earn the next 10 seconds. Connect the hook to the payoff.
[BODY]     varies  Deliver the value in tight, scannable beats (1 idea per beat).
[CTA]      3–5s   One action only. Match the awareness ceiling; never exceed it.
```

**Pacing rules:**
- 15–30s: hook + bridge + 2–3 body beats + CTA. Zero fat.
- 31–60s: hook + bridge + 4–6 body beats + CTA. Every beat earns the next.
- On-screen text cues matter: call them out (`[TEXT OVERLAY: …]`) when the hook or proof lives on screen.
- Spoken word first, on-screen second — TikTok auto-captions; the script is the audio track.

### Body and CTA craft

- **Body beats:** each beat is one discrete idea, ~5–10 spoken words. Number them.
- **CTA ceiling:** cold/unaware audience → "follow for more" / "save this"; warm/product-aware → "link in bio" / "try it free." Never ask cold viewers to buy.
- **Proof use:** real numbers from `brand-brain` only. Illustrative metrics are `[verify]`. Social proof ("over X customers") requires confirmation.
- **Native feel:** conversational, not scripted-sounding. Read it aloud — if it sounds like ad copy, rewrite.

---

## Output format

### Quick mode

```
## TikTok script — [topic / angle]
Brand: [slug, via brand-brain]  |  Duration: [Xs]  |  Awareness stage: [stage]

### Hook variants (ranked)
1. ★ [Recommended hook — mechanic label]
   Why: [one line — why it fits this ICP and awareness level]
2. [Alternate hook — different mechanic]
3. [Alternate hook — different mechanic]
(+ 1–2 more if generated)

### Script

[HOOK]     [line]
[BRIDGE]   [line]
[BODY]
  1. [beat]
  2. [beat]
  3. [beat]
[CTA]      [line]

[TEXT OVERLAY notes, if any]
[Tone/delivery note — 1 line]
```

### Battery mode

Three full blocks in the Quick-mode format, each with its own hook battery. A summary table at the end: which angle to test first, what each tests for, and what a "win" looks like.

Save to `./tiktok/[brand-slug]-scripts.md` when asked.

---

## Platform realities (non-negotiable)

- **The 3-second cliff:** TikTok's completion rate drops sharply at 3 seconds. If the hook does not create a reason to stay, the rest is irrelevant. Test hooks, not scripts.
- **Captions are crawled:** TikTok's search index reads captions. Keyword-rich captions extend organic reach beyond FYP distribution.
- **Trend decay is fast:** trend-driven hooks have a short shelf life. If the brief references a specific sound or trend, write a timeless alternate.
- **Sound-off baseline:** ~40% of TikTok is watched without audio [verify]. Text overlays on the hook and key beats are not optional for DTC/informational content.
- **15–30s content completes more and gets pushed harder** by the algorithm for cold reach; 31–60s is better for warm segments and conversion-oriented content.
- **Banned words and brand voice are hard overrides.** Not suggestions.

---

## Principles

- **Brand-brain first.** No script before the brand context returns.
- **Hook wins or the script doesn't exist.** Generate the hook battery before writing the body; never start with the body.
- **Distinct mechanics, not synonym lists.** Five hooks built on five different psychological mechanisms; never five ways to say the same thing.
- **Real proof or `[verify]`.** Never invent a stat. If the brand has no real proof yet, write curiosity/call-out hooks — not proof hooks.
- **Awareness ceiling on the CTA.** Cold viewers save and follow; warm viewers click. Never exceed the stage.
- **Conversational audio test.** Read every script aloud. If it sounds like a banner ad, rewrite it.

## What not to do

- Don't write the body before generating the hook battery.
- Don't produce five hooks that all open with "Did you know…" — that is vocabulary variation, not mechanic variation.
- Don't use invented proof ("millions of people," "97% effectiveness") without a confirmed source.
- Don't write a CTA that asks cold audiences to buy or book — match the ask to the stage.
- Don't reference a specific TikTok trend or sound without providing a timeless alternate — trends die; the brief usually outlasts them.
- Don't reimplement brand scanning, voice capture, or ICP research — call `brand-brain`.
- Don't skip the voice review pass (inline or via `post-quality-reviewer-voice-auditor`).

## Quality checklist

- `brand-brain` called and the active brand loaded before any script was written?
- Hook battery has 3–5 variants built on *distinct mechanics* (not vocabulary), with a stated #1 recommendation and a one-line rationale?
- Script follows Hook → Bridge → Body → CTA structure; each beat is one tight idea?
- Duration fits the content type and audience awareness stage?
- CTA matches the awareness ceiling — no cold-audience buy asks?
- Only real proof used; unconfirmed numbers marked `[verify]`?
- Text overlay cues noted where the hook or proof lives on screen?
- Voice adjectives honored; banned words absent?
- Script reads naturally aloud — not like ad copy?
- Battery mode: three distinct angles + summary test recommendation?
