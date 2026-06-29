---
name: faceless-short-form-video-producer
description: >
  Takes a topic, a short-form script, or a published blog post and produces a complete, ready-to-render
  faceless TikTok / Instagram Reel / YouTube Short — on-brand voice, captioned, with a voiced-over
  narration track, B-roll direction, and an end-card CTA. It does not just write a script and stop;
  it coordinates the full production pipeline: script via the Hook-Bridge-Point-CTA framework, caption
  file (.srt), VO prompt for ElevenLabs / Murf / similar, B-roll shot list, and a Remotion or FFmpeg
  render command so the user can produce the actual video with native tooling. Brand voice, banned words,
  ICP, and offer destinations all come from `brand-brain` — they are never re-derived here. Calls
  `short-form-video-script-writer` if a polished script is not already in hand, `cta-variant-generator`
  for the end-card CTA, and `de-slop-humanize-pass` before finalizing VO narration. Saves the full
  production bundle to `./social/video/[slug]/` on request. Trigger phrases: "make a short-form video,"
  "produce a Reel / TikTok / Short," "turn this post into a video," "faceless video from this article,"
  "create a narrated video," "generate VO and captions," "video from blog post," "build the render command."
---

# Faceless Short-Form Video Producer

Give it a topic or a blog post; get back a production bundle you can actually render — script, narration, captions, B-roll list, and the shell command to stitch it into video. No talking-head camera required.

The hard part of faceless short-form isn't the hook or the script — it's the coordination tax: write the VO, sync captions, source B-roll, set render parameters, then realize the CTA is off-brand. This skill runs all four lanes in one pass, brand-checked before any word is written, so a junior operator can ship a senior-quality video in 20 minutes.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's voice, banned words, ICP, offer mechanics + destination URLs, and real proof. Never re-derive brand context here.
- **`short-form-video-script-writer`** (call when no script is in hand) — produces a platform-tuned, hook-led script; pass the brief and brand digest, fold the returned script into Step 2.
- **`cta-variant-generator`** (call for the end-card CTA) — writes a committed, on-brand closing CTA from the offer mechanics; synthesize inline only if unavailable.
- **`de-slop-humanize-pass`** (call before finalizing VO narration) — strips AI-sounding cadence from the narration so it sounds like a person, not a text-to-speech demo.
- *(optional)* **`proof-vault`** — pull real stats for the body if the source lacks them; mark all unconfirmed proof `[verify]`.
- *(optional)* **`content-repurposer-atomizer`** — if the input is a long blog post and the user wants a video extracted from it rather than from the whole article, call this first to pull the video atom, then pass that atom as the brief.

---

## How a run works

```
Step 0  Load brand        ──► call brand-brain (bootstraps on first use)
Step 1  Intake            ──► topic / script / blog post + platform + duration
Step 2  Script            ──► call short-form-video-script-writer (or refine existing)
Step 3  VO narration      ──► write + humanize via de-slop-humanize-pass
Step 4  Caption file      ──► generate .srt from the narration
Step 5  B-roll shot list  ──► map each scene to a visual directive
Step 6  End-card CTA      ──► call cta-variant-generator
Step 7  Render command    ──► output Remotion / FFmpeg / CapCut directive
Step 8  Self-review, then present and offer to persist the bundle
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency. If the brand is new, `brand-brain` bootstraps it before returning; **do not produce any output until it returns.**

Obey voice + banned-words as hard overrides. Point end-card CTAs only at the brand's real destination URLs. Use only real proof; mark anything unconfirmed `[verify]`.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user to install `brand-brain` or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed.

### Step 1 — Intake

Ask only what is missing:
- **Source** — a topic/headline, a short script draft, or a blog post URL/path.
- **Platform** — TikTok, Instagram Reel, YouTube Short (default: all three; slight caption-style and duration differences noted).
- **Duration** — 30s, 45s, 60s (default: 45s).
- **Render tooling preference** — Remotion (code-based), FFmpeg (CLI), CapCut (manual export), or "just the assets" (default: Remotion command + FFmpeg fallback).

---

## HBPC: our working production scaffold

Every faceless video follows **Hook → Bridge → Points → CTA** (HBPC) — our house shorthand for the same structure as high-performing organic short-form, adapted for voiceover delivery. Most faceless video fails not in production but in scripting: the hook doesn't earn the scroll, or the body lists features while the viewer is still problem-aware. HBPC fixes the awareness mismatch first, then builds the production bundle around it.

### Step 2 — Script (Hook → Bridge → Points → CTA)

If a polished script is not already in hand, **call `short-form-video-script-writer`** (Skill tool), passing the topic/source + the brand digest + the target duration. Fold the returned script in and proceed.

If a draft script exists: evaluate it against the HBPC scaffold and rewrite only what fails the check.

| Beat | Duration | Job | What makes it fail |
|---|---|---|---|
| **Hook** | 0–3s | Stop the scroll — one concrete claim, question, or number | Generic opener ("In this video…"); slow wind-up |
| **Bridge** | 3–8s | Connect the hook to why *this viewer* cares — name the pain or the stake | Jumping straight to tips; no stakes set |
| **Points** | 8–40s | 3–4 tight beats, each with one idea + one visual cue; no padding | Listing features, not outcomes; "also…" connectors |
| **CTA** | 40–45s | One action, low friction, anchored to the brand's real offer | "Follow for more"; vague "check the link"; no risk-reducer |

**Script format to produce:**

```
[HOOK — spoken]
[BRIDGE — spoken]
[POINT 1 — spoken] / [on-screen text]
[POINT 2 — spoken] / [on-screen text]
[POINT 3 — spoken] / [on-screen text]
[CTA — spoken + on-screen]
Estimated duration: Xs  ·  Word count: N (~Ns at 150 wpm)
```

Duration math: ~150 words/minute for comfortable VO pacing. 45s ≈ 112 words. 60s ≈ 150 words.

### Step 3 — VO Narration (humanize before finalizing)

Rewrite the script into VO-ready narration: short sentences (≤12 words average), spoken contractions, no markdown, no bullet punctuation. Then **call `de-slop-humanize-pass`** (Skill tool) on the narration draft before finalizing. The returned narration is what goes into the .srt and the VO prompt.

Provide a **VO synthesis prompt** the user pastes into ElevenLabs, Murf, Play.ht, or Eleven's API:

```
Voice: [tone] — [energy level] — [pacing note from the brand voice]
Style: conversational, not announcer. Slight emphasis on [hook word / key stat].
Pause: 0.4s after the hook. Natural micro-pauses at commas.
No filler sounds. No breath artifacts. Clip to clean silence at the end.
```

### Step 4 — Caption File (.srt)

Generate a syntactically valid `.srt` file from the finalized narration. Rules:
- ≤6 words per caption line (readability on mobile, especially TikTok's narrow safe zone).
- Sync timestamps to the estimated VO pacing (150 wpm = ~1 word per 0.4s); flag that timestamps need adjustment after actual VO render.
- Highlight the hook word in ALL CAPS (TikTok native style) if the brand doesn't ban capitalization for emphasis.
- Produce the `.srt` block inside a fenced code block labeled `srt`.

### Step 5 — B-Roll Shot List

Map each script beat to a concrete visual directive. Faceless video lives or dies on B-roll specificity — "woman on laptop" is not a direction; "close-up of a Shopify dashboard loading with a cursor hovering over the Orders tab" is.

| Beat | Visual directive | Source suggestion | Duration |
|---|---|---|---|
| Hook | [concrete scene] | Stock (Pexels / Pixabay / Envato) or screen-record | Xs |
| Bridge | [concrete scene] | … | Xs |
| Point 1 | [concrete scene] | … | Xs |
| Point 2 | [concrete scene] | … | Xs |
| Point 3 | [concrete scene] | … | Xs |
| CTA | End card: brand logo + CTA text + destination | Static / animated overlay | Xs |

Stock source defaults: **Pexels** (free, no attribution), **Pixabay** (free), **Envato Elements** (subscription). Screen recordings and product demos are preferred for SaaS brands — they show, not just tell.

### Step 6 — End-Card CTA

**Call `cta-variant-generator`** (Skill tool) with: placement = "short-form video end card", brand digest, and the awareness stage for this video. Use the recommended primary CTA as both the spoken CTA (in the script) and the on-screen overlay text. Keep the on-screen text ≤6 words. If `cta-variant-generator` is unavailable, write the CTA inline following the HBPC CTA rules above.

### Step 7 — Render Command

Output the render directive for the user's chosen tooling.

**Remotion (default — code-based, most flexible):**
```
# Install: npx create-video@latest  or add to existing project
# Template: faceless-short-form (set duration, caption overlay, B-roll sequence)
npx remotion render FacelessVideo --props='{"script":"[slug]-script.json","captions":"[slug].srt","broll":"[slug]-broll.json","duration":<Xs>,"cta":"[CTA text]","ctaUrl":"[destination]"}' out/[slug].mp4
```

**FFmpeg fallback (for users with pre-rendered B-roll clips + VO .mp3):**
```bash
# Concat B-roll clips into a base track
ffmpeg -f concat -safe 0 -i broll-list.txt -c copy base.mp4
# Burn in captions
ffmpeg -i base.mp4 -vf "subtitles=[slug].srt:force_style='FontName=Arial,FontSize=18,Bold=1,PrimaryColour=&HFFFFFF&,OutlineColour=&H000000&,Outline=2'" -c:a copy captioned.mp4
# Mix VO audio
ffmpeg -i captioned.mp4 -i vo.mp3 -filter_complex "[0:a]volume=0.3[bg];[bg][1:a]amix=inputs=2:duration=shortest" final-[slug].mp4
```

**CapCut / manual note:** output the asset bundle and instruct the user to import B-roll clips, drop the .srt in the captions panel, import the VO .mp3, and drop the end-card overlay on the top track.

---

## Platform differences (apply at Step 2 + Step 4)

| Platform | Aspect ratio | Duration sweet spot | Caption style | First-frame note |
|---|---|---|---|---|
| TikTok | 9:16 | 30–45s (hook < 3s is critical) | Bold white, black outline, ≤6 words/line | Algorithm reads first 3s; hook must be visible text + audio |
| Instagram Reel | 9:16 | 30–60s | Same — Reel captions auto-generated but .srt upload improves accuracy | Cover frame pulled from 0:01 — make it count |
| YouTube Short | 9:16 | ≤60s | Styled via YouTube's own caption layer; export clean .srt | Title card in first 2s outperforms no title |

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No script, no VO, no CTA before `brand-brain` returns. Voice + banned-words override everything here.
- **HBPC always.** Every video has a hook that earns 3 more seconds, a bridge that sets stakes, points that deliver outcomes (not features), and one CTA. No beat is skipped.
- **Produce assets, not suggestions.** Deliver the .srt block, the B-roll table, and the render command — not a description of what the user should go make. This skill is a production run, not a strategy deck.
- **VO sounds human.** Run `de-slop-humanize-pass` before finalizing narration. Nothing ships that sounds like a text-to-speech demo.
- **CTA matches awareness.** The end-card CTA cannot ask more than the awareness stage the video's viewer is in. A problem-aware viewer gets a low-commitment CTA; a product-aware viewer gets a trial/buy CTA.
- **Real proof only.** Every stat, quote, or customer name must come from the brand's real proof vault or the source asset. Mark unconfirmed numbers `[verify]`.

## What Not to Do

- Don't produce any output before `brand-brain` returns the active brand.
- Don't re-derive brand voice, ICP, or offer mechanics — that lives in `brand-brain`, once.
- Don't deliver only a script and call it done — this skill produces the full production bundle.
- Don't write VO narration in bullet points, headers, or markdown — it renders literally in TTS engines.
- Don't invent B-roll descriptions that imply branded footage the user doesn't have; suggest free stock with specific search terms.
- Don't use emojis, exclamation marks, or hashtags unless the brand explicitly allows them.
- Don't pad to duration — if the core idea lands in 30s, the video is 30s.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any output?
- Voice + banned-words honored in script, VO, captions, and CTA?
- HBPC all four beats present; hook ≤3s; CTA matches the viewer's awareness stage?
- VO narration passed through `de-slop-humanize-pass`; reads naturally spoken aloud?
- `.srt` block is syntactically valid; ≤6 words per line; timestamps plausible at 150 wpm?
- B-roll table has a concrete visual directive for every beat (not "person on laptop")?
- `cta-variant-generator` called for end card; CTA points at brand's real destination URL?
- Render command present (Remotion or FFmpeg) and uses the correct output slug/paths?
- Offered to save the bundle to `./social/video/[slug]/` — not inside the skill folder, not overwriting `brand.md`?
