---
name: ugc-creator-brief-writer
description: >
  Campaign goal, product, creator tier, and platform → a full, production-ready UGC or creator
  brief that a nano-, micro-, or mid-tier creator can act on immediately without a briefing call.
  Outputs a complete deliverable spec (format, dimensions, duration, file-type, caption length),
  platform-correct hooks and talking-point rails, explicit dos/don'ts, brand-safe language guide,
  usage-rights clause language, and a review/approval workflow. Built on the Agency-Model UGC
  Brief Framework: Mandate → Hook Rails → Guardrails → Deliverable Spec → Rights. Brand context
  loads from `brand-brain` so every brief is on-voice without re-deriving the brand from scratch.
  Calls `icp-persona-builder` for audience context when no persona is available, and
  `campaign-brief-builder` when the campaign itself is not yet scoped. Outputs are saved to
  ./ugc-briefs/[slug]-[creator-handle]-brief.md so creators receive a link, not a chat export.
  Use when the user says "write a creator brief," "brief a UGC creator," "influencer brief,"
  "send to a creator," "organic-style ad brief," "creator deliverable spec," "UGC hook ideas,"
  or drops a product + platform and asks what to tell the creator.
---

# UGC & Creator Brief Writer

A brief a creator can act on without a phone call. Takes campaign goal, product, platform, and creator tier; returns a complete, professional brief a freelancer, nano creator, or micro influencer can open, read once, and start shooting. Every brief is on-brand because the brand context comes from `brand-brain` — not from guessing.

This skill writes briefs. It does not negotiate rates, vet creator audiences, or publish assets. If the campaign itself is unscoped, it calls `campaign-brief-builder` first.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, banned words, offer mechanics, and real proof before a single line of brief copy is written.
- **`icp-persona-builder`** (when no persona is on file) — confirms the audience the creator is speaking to so hooks and talking points match real buyer language.
- **`campaign-brief-builder`** (when the campaign context is missing) — scopes the campaign goal, key message, and window before the brief is authored.
- **`headline-hook-generator`** (optional) — generates extra hook options when the user wants a deep hook bank. Synthesize inline when absent.
- **`cta-variant-generator`** (optional) — supplies the in-video or caption CTA when a stronger CTA is needed. Synthesize inline when absent.

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain (bootstraps on first use)
Step 1  Gather inputs       ──► campaign goal · product · platform · creator tier · any existing persona
Step 2  Fill gaps           ──► call icp-persona-builder if no persona; call campaign-brief-builder if no goal scoped
Step 3  Write the brief     ──► Agency-Model UGC Brief Framework (see below)
Step 4  Self-review         ──► checklist pass; flag any missing field or off-brand element
Step 5  Save + deliver      ──► write to ./ugc-briefs/[brand-slug]-[handle-or-tier]-brief.md; present inline summary
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before writing anything. It returns the active brand's digest — voice adjectives, banned words, offer mechanics, destination URLs, real proof, ICP, positioning. Obey the voice and banned-words as hard overrides; mark any unconfirmed proof `[verify]`.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if neither exists, ask the user to install `brand-brain` or answer four quick questions (what the product is, ICP + awareness, offer mechanics + destination, 3 voice adjectives + banned words), then proceed.

### Step 1 — Gather inputs

Required (ask if missing in ≤5 questions, batched):
- **Campaign goal:** awareness / trial / conversion / UGC-for-ads (organic-style)
- **Product or offer:** what the creator is promoting; any current promo or hook offer
- **Platform:** TikTok · Instagram Reels · YouTube Shorts · Facebook Reels · Pinterest (pick one primary)
- **Creator tier:** nano (1K–10K) · micro (10K–100K) · mid (100K–500K) · macro (500K+)

Optional but used when present:
- Creator's handle and niche
- Existing persona doc or segment name
- Any past-performing hook angles or scripts
- Hard usage-rights requirement (paid amplification / whitelisting / paid partnership label)

### Step 2 — Fill gaps

If no persona exists for the audience: invoke `icp-persona-builder`.
If the campaign goal is vague or the brief context is absent: invoke `campaign-brief-builder`.
Neither blocks the run if the user provides enough context directly.

---

## Agency-Model UGC Brief Framework

Five mandatory sections; every brief ships all five. No section is optional; a brief missing any section requires a revision.

### Section 1 — Mandate (one-paragraph north star)

One tightly written paragraph the creator reads first. Covers: what the brand is, the one thing the creator must communicate, who is watching (audience), and what the viewer should feel/do after watching. Mandate ends with the single most important message — not a list.

### Section 2 — Hook Rails (3–5 hooks the creator can use verbatim or riff on)

Hooks are the first 1–3 seconds (TikTok/Reels) or first line of caption (static). Use proven hook structures; name the structure so the creator understands the pattern.

Hook structures to draw from (pick those fitting the platform and goal):
| Structure | Example |
|---|---|
| Pain interrupt | "If you're still doing X, this is why it's not working…" |
| Curiosity gap | "Nobody talks about this, but [product] actually…" |
| Contrarian | "Everyone says X. Here's what actually happened when I tried [product]…" |
| Social proof / number | "I've tried 14 [category] tools. This is the only one I kept." |
| Before/after tease | "I went from [bad state] to [good state] in [timeframe]. Here's how." |
| How-to promise | "How to [outcome] without [cost/friction] — I'll show you in 60 seconds." |

Write 3–5 hooks. At least one must be low-awareness (curiosity/pain) and one must be product-aware (social proof/result). Mark which are optimized for paid amplification (front-load the hook, assume no sound for first 3 frames).

### Section 3 — Talking-Point Rails (the middle, not a script)

NOT a word-for-word script. Rails the creator speaks through naturally. Three to five bullet points covering: (a) the setup/problem context, (b) the product moment (show don't just tell), (c) one real proof point from `brand-brain` (or `[verify]` if not confirmed), (d) the offer/CTA. Add one "must include" (e.g., "show the app/product in active use for minimum 3 seconds") and one "feel" note (e.g., "conversational, slightly skeptical-to-convinced arc — not hype").

### Section 4 — Guardrails (dos and don'ts)

Keep short, scannable, unambiguous. Format: two columns.

**Dos (always include):**
- Use your real voice; don't read from a script
- Show the product in real use (not just packaging)
- Disclose per platform rules (#ad, #sponsored, Paid Partnership label) [verify: check brand's specific disclosure requirement]
- Tag [brand handle] in the caption
- Include the CTA link in bio / swipe-up / caption as specified

**Don'ts (tailor to brand's banned-words + vertical):**
- Do not mention competitor names
- Do not make health/safety claims not on the product page (`[verify]` any claim before including)
- Do not use music not cleared for commercial use if the content will be whitelisted
- Do not edit the mandated disclosure language
- [Brand-specific bans from `brand-brain`]

### Section 5 — Deliverable Spec (unambiguous production requirements)

One table. Never approximate — use the real platform limits.

| Field | Requirement |
|---|---|
| Platform | [primary platform] |
| Format | Vertical video / horizontal / square / carousel / static |
| Dimensions | [platform-correct px — see Platform Specs below] |
| Duration | [range in seconds — see Platform Specs below] |
| File type | .mp4 (H.264) / .jpg / .png |
| Caption length | [limit — see Platform Specs below] |
| Hashtags | [count + any required brand hashtags] |
| Thumbnail | Required / optional |
| Raw footage | Yes/No — brand requires raw footage for paid amplification |
| Due date | [from campaign window] |
| Submission method | [Google Drive folder URL / email / direct DM — set by user] |
| Usage rights | [license clause — see Usage Rights below] |

**Platform Specs (real limits as of this writing — mark `[verify]` if checking current spec):**

| Platform | Recommended dimensions | Duration range | Caption limit |
|---|---|---|---|
| TikTok | 1080×1920 (9:16) | 15–60s (sweet spot 21–34s [verify]) | 2,200 chars |
| Instagram Reels | 1080×1920 (9:16) | 15–90s | 2,200 chars |
| YouTube Shorts | 1080×1920 (9:16) | ≤60s | 100 chars title |
| Facebook Reels | 1080×1920 (9:16) | 3–90s | 63,206 chars |
| Pinterest Idea Pin | 1080×1920 (9:16) | up to 60s | 250 chars |

**Usage Rights clause (adapt to campaign type):**

> Creator grants [Brand] a non-exclusive, royalty-free license to use, repurpose, and amplify the Deliverable(s) across [Brand]'s own paid and organic channels for [duration, e.g., 12 months] from the date of delivery. Creator retains ownership. [Brand] will not grant sub-licenses without written consent. [Verify: have legal confirm for paid whitelisting campaigns.]

---

## Saving the brief

Save the full brief to `./ugc-briefs/[brand-slug]-[creator-handle-or-tier]-brief.md` relative to the user's CWD. Tell the user the path in one line. Present the brief inline as well so there's no round-trip.

If the user is briefing multiple creators in one session, append a creator index to `./ugc-briefs/[brand-slug]-index.md` (handle, tier, platform, due date, status: Draft).

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No brief copy before `brand-brain` returns. Voice + banned-words override everything.
- **Platform-correct specs.** Never approximate dimensions, duration, or caption limits. Mark stale data `[verify]`.
- **Rails, not scripts.** Talking points are direction, not dictation. Over-scripted UGC sounds scripted.
- **Hooks vary by structure, not vocabulary.** Five variations of "check out [product]" is not a hook bank.
- **Disclosure is mandatory.** Every brief includes the correct disclosure requirement for the platform. Never omit.
- **Real proof only.** One real, sourced stat beats three invented ones; mark anything unconfirmed `[verify]`.
- **Usage rights are explicit.** Every brief ships a plain-English rights clause. Ambiguity costs money.

## What Not to Do

- Don't write a word-for-word script and call it "talking points" — creators who sound scripted lose credibility.
- Don't omit the disclosure instruction; a non-disclosed paid post is an FTC violation [verify jurisdiction].
- Don't invent platform specs; use the real numbers and mark any that need checking.
- Don't reimplement brand scanning or ICP derivation here — call `brand-brain` and `icp-persona-builder`.
- Don't write "authentic" as a brand instruction — it means nothing to a creator; describe the *tone arc* instead.
- Don't skip the usage rights clause because it feels legal; missing rights language is the #1 UGC dispute trigger.
- Don't let a brief exceed two pages for a nano/micro creator; longer briefs go unread.

## Quality Checklist (self-review before saving)

- `brand-brain` called; active brand loaded; voice + banned-words applied?
- All five framework sections present: Mandate · Hook Rails · Talking-Point Rails · Guardrails · Deliverable Spec?
- ≥3 hooks present, at least one low-awareness + one product-aware; hooks vary by structure?
- Platform dims, duration, and caption limit are real numbers (or flagged `[verify]`)?
- Disclosure instruction present and platform-correct?
- Usage rights clause included?
- Unconfirmed stats and claims marked `[verify]`; no invented proof?
- Brief saved to `./ugc-briefs/` and path surfaced to user?
