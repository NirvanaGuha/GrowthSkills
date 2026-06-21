---
name: social-ad-copy-writer
description: >
  Turns a product brief, audience definition, and campaign objective into production-ready paid social
  ad copy — per platform, per format, per awareness stage. Covers Meta (Facebook/Instagram), LinkedIn,
  TikTok, and X. For each format (single image, carousel, video/UGC) it delivers a headline, primary
  text, and description/CTA with character-count compliance baked in. Uses the Hook-Bridge-CTA
  framework as its core ad-copywriting spine and the Schwartz awareness ladder to gate the ask.
  Brand context comes from the `brand-brain` skill; proof comes from `proof-vault` when available;
  CTA labels are sharpened through `cta-variant-generator`. Does NOT design the creative — it writes
  the copy that goes on top of it, and flags creative direction as a note only. Use when the user says
  "write social ads," "Facebook/Instagram/LinkedIn/TikTok ad copy," "write ads for [product]," "ad
  variants," "write the primary text," "write the headline," "paid social copy," or hands over a
  product and audience and asks for ad copy.
---

# Social Ad Copy Writer

Product + audience + objective in. Platform-correct, brand-on-voice ad copy out — for every format
you need to run. Headline, primary text, and description written to the character limit, structured
around a named framework, and matched to the audience's awareness stage so the ask never outruns
the reader's readiness.

This skill writes copy. It does not design the creative, manage campaigns, or set budgets. If the
offer or landing page is weak, it surfaces that — it does not paper over a thin offer with clever
copy.

---

## Skills this calls

- **`brand-brain`** (required, always first) — resolves and loads the active brand's voice, ICP,
  offer mechanics, proof, and banned words. No ad is written before this returns.
- **`proof-vault`** (optional) — fetches real, citable proof points for microcopy and social proof
  overlays. If absent, uses proof from the `brand-brain` digest; marks unconfirmed claims `[verify]`.
- **`cta-variant-generator`** (optional) — sharpens CTA button labels for single-image and
  carousel formats. Synthesize inline when absent.
- **`audience-targeting-spec-writer`** (optional) — if the user also needs a targeting spec to
  accompany the copy, delegate rather than rebuild.
- **`ad-to-landing-page-message-match-auditor`** (optional) — after copy is drafted, flag if the
  landing page URL is provided and message match is at risk.

---

## How a run works

```
Step 0  Load the brand     ──► call brand-brain (always)
Step 1  Scope the run      ──► which platforms + formats + how many variants
Step 2  Set the framework  ──► Hook-Bridge-CTA × Schwartz awareness ceiling
Step 3  Write the copy     ──► per platform, per format, character-count compliant
Step 4  Self-review        ──► brand voice, offer gate, char limits, proof honesty
Step 5  Deliver            ──► table + annotation; save to ./ads/ if batch
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's request and
any named brand. It returns voice adjectives, banned words, offer mechanics + destination URLs, real
proof, positioning, and ICP + awareness tendency. Do not write a single line of ad copy until it
returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer a
4-question mini-setup (what it is · ICP + awareness tendency · offer mechanics + destination ·
3 voice adjectives + banned words). Always prefer the call.

### Step 1 — Scope the run

Identify from the user's request:
- **Platform(s):** Meta (Facebook/Instagram), LinkedIn, TikTok, X — or "all"
- **Formats:** single-image, carousel (per-card copy), video/UGC script — default to single-image
  if unspecified
- **Objective:** awareness / traffic / lead generation / conversion / retargeting — this controls the
  commitment ceiling
- **Variant count:** default 2 creative directions per format; up to 3 if explicitly asked

When objective is unstated, infer from the funnel stage implied by the product/landing page.

---

## The framework: Hook-Bridge-CTA × Schwartz awareness

Every ad unit is built on three parts:

**Hook** — stops the scroll. Opens with the reader's pain, desire, or a pattern interrupt.
Must work in the first 3 seconds (video) or the first line of primary text (static).

**Bridge** — earns the click. Connects the hook to the offer: credibility, mechanism, proof, or
demonstration. This is where social proof, numbers, and guarantees live.

**CTA** — closes with the right ask for the awareness stage. Matches commitment level to where
the audience actually is.

### Awareness ceiling (Schwartz — mandatory)

| Awareness stage | Max commitment | Example CTA range |
|---|---|---|
| Unaware | curiosity only | "See why 12,000 stores switched" |
| Problem-aware | low — educate | "Watch how it works" / "Read the guide" |
| Solution-aware | medium — evaluate | "Compare plans" / "See it in action" |
| Product-aware | high — trial/demo | "Start free" / "Book a demo" |
| Most-aware | transact | "Get started today" / "Claim your trial" |

Retargeting audiences are Product-aware or Most-aware by default. Cold audiences should rarely
exceed Solution-aware unless a compelling proof point earns the higher ask.

---

## Platform specs and hard limits

Internalize these before writing. Never exceed a limit — truncation kills the message.

### Meta (Facebook / Instagram)

| Field | Limit | Notes |
|---|---|---|
| Primary text | 125 chars visible (up to 500 total) | First 125 display before "see more"; front-load the hook |
| Headline | 27 chars (desktop feed) / 40 chars (mobile) | Use 27 as the safe ceiling |
| Description | 27 chars | Below headline in link previews; optional |
| CTA button | Platform list (Shop Now, Learn More, Sign Up, etc.) | Match button to objective |
| Carousel card headline | 40 chars | Per card |
| Carousel card body | 125 chars | Per card |

**Format notes — Meta:**
- Single image: hook in first 125 chars; bridge + proof in the next 375 if needed; headline
  closes the promise.
- Carousel: each card tells its own micro-story (feature, use case, or proof point); card 1 is
  the hook card; the final card is always the CTA card.
- Video/Reel: first 3 seconds on-screen text is the hook (≤8 words); primary text supports;
  captions mandatory (85%+ watch without sound [verify]).

### LinkedIn

| Field | Limit | Notes |
|---|---|---|
| Introductory text | 150 chars visible (600 total) | 150-char rule is aggressive; 600 is the hard cap |
| Headline | 70 chars | Shown under the image |
| Description | 100 chars | Below headline on desktop |
| Carousel headline | 255 chars | Per card (rarely use the full length) |
| Sponsored InMail subject | 60 chars | — |

**Format notes — LinkedIn:**
- B2B audience; professional pain + ROI proof outperforms lifestyle angles.
- Lead Gen Forms: the CTA is the form; headline should name exactly what they get.
- Carousel: can carry a narrative arc across 3–8 cards; use "swipe to see" in the intro text
  to prime the behavior.

### TikTok

| Field | Limit | Notes |
|---|---|---|
| Ad text | 100 chars | Shown below the video; supports ≤1 emoji |
| CTA button | Platform list | "Shop Now," "Learn More," "Sign Up," etc. |

**Format notes — TikTok:**
- Native-first: copy must match UGC/creator energy. Polished brand copy is penalised
  by TikTok's algorithm (lower delivery quality) [verify specific penalty mechanism].
- Hook is on-screen: the first 2–3 seconds of spoken/text-on-screen content is the real
  headline. Ad text is secondary.
- Write the hook line as if a creator is saying it, not as a brand announcement.
- Sound-on default (opposite of Meta/LinkedIn).

### X (Twitter)

| Field | Limit | Notes |
|---|---|---|
| Ad copy (tweet body) | 280 chars total | Links eat 23 chars of the budget |
| Carousel card | 70-char headline + 200-char body | Per card |

---

## Single-image ad (the core unit)

For each creative direction, deliver:

```
### Ad [N] — [Direction label, e.g. "Pain-led" / "Social proof" / "Curiosity"]
Platform: [Meta / LinkedIn / TikTok / X]
Awareness stage: [Unaware → Most-aware]
Objective: [Awareness / Traffic / Lead / Conversion / Retargeting]

Primary text (≤125 chars visible):
[The hook line — written to land in ≤125 chars]
[Optional bridge/proof expansion — clearly marked as below-the-fold for Meta]

Headline (≤27 chars for Meta / ≤70 for LinkedIn):
[Value-forward, closes the promise or names the outcome]

Description (≤27 chars):
[Optional; supporting detail or risk-reducer]

CTA button: [platform-native option, e.g. "Sign Up" / "Learn More" / "Get Quote"]

Notes: [Offer-gate warnings, creative direction hint (1 line), message-match risk, proof marked [verify] if not from brand-brain]
```

---

## Carousel ad (multi-card)

**Card structure rule:** Card 1 = hook (pain/desire). Cards 2–N = bridge (proof, features, use cases,
objections). Final card = CTA card.

For each card:

```
Card [N] — [Role: Hook / Feature / Proof / CTA]
Headline (≤40 chars Meta / ≤255 LinkedIn): [...]
Body (≤125 chars Meta): [...]
```

Minimum 3 cards, maximum 10 (Meta) / 10 (LinkedIn) / 35 (TikTok Spark — not standard carousel).
Default: 5 cards unless the user specifies.

---

## Video/UGC ad

Video copy has two layers: **on-screen text/script** and **caption copy** (the feed-level primary
text). Deliver both.

```
### Video Ad [N] — [Direction label]
Platform: [Meta / TikTok / LinkedIn / YouTube — note: YouTube not in scope unless requested]
Duration: [e.g. 15s / 30s / 60s]
Hook (0–3s, on-screen text or spoken line, ≤8 words): [...]
Body (3–[X]s, key proof/demo beats, 2–3 bullet points): [...]
CTA (final 3–5s, on-screen): [...]
Caption / primary text: [platform-limit-compliant feed copy]
```

---

## Offer gate (mandatory check before writing)

Before producing copy, confirm the offer clears these:

1. **Is there a clear value exchange?** (What does the user get + what do they give?)
2. **Is the commitment right for the audience's awareness stage?** (Cold traffic ≠ "Buy now")
3. **Is there a risk-reducer?** (Free trial, no credit card, money-back, free tier) — at least
   name it in the copy if one exists.
4. **Is there a credible landing page destination?** (Message-match risk)

If any gate fails, flag it in the `Notes` field of each ad unit — do not silently write copy that
papers over a broken offer.

---

## Principles

- **Brand-brain first.** No copy before `brand-brain` returns. Its voice and banned words are
  hard overrides for everything here.
- **Framework, not formula.** Hook-Bridge-CTA is the spine; vary the angle, not the structure.
- **Awareness ceiling is a ceiling.** Never ask for a higher commitment than the stage supports.
  Retargeting audiences can go higher; cold audiences almost never should.
- **Char limits are not suggestions.** Every field must comply. Truncated copy is broken copy.
- **Platform energy must match.** TikTok sounds like a creator. LinkedIn sounds like a practitioner.
  Meta sounds like a peer recommendation. Writing the same copy across all three is a failure.
- **Honest proof only.** Real numbers from `brand-brain` or `proof-vault`, or marked `[verify]`.
  No invented statistics, fake reviews, or manufactured urgency.
- **Offer gate before keys hit paper.** A great hook on a weak offer generates clicks that don't
  convert — flag it, don't hide it.

---

## What not to do

- Don't write copy before `brand-brain` returns the active brand.
- Don't produce identical copy across platforms — each platform has a distinct voice register and
  algorithm contract.
- Don't exceed character limits; don't approximate ("roughly 27 chars").
- Don't invent proof points, testimonials, or statistics not confirmed by brand-brain/proof-vault.
- Don't use a CTA that exceeds the audience's awareness-stage ceiling.
- Don't use emojis or exclamation marks unless the brand allows them.
- Don't redesign the creative or rewrite the landing page — flag the issue in Notes and stay in lane.
- Don't produce near-identical variants that only swap synonyms — different angles, different
  motivations.

---

## Output format and persistence

**Single-ad request:** inline in the conversation.

**Batch (3+ ads or multiple platforms):** save to `./ads/[brand-slug]-social-ads-[YYYY-MM-DD].md`
with a summary table at the top:

```
| # | Platform | Format | Direction | Awareness | Chars (primary) | Chars (headline) |
```

Ask before saving if the user hasn't specified.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy?
- Voice adjectives honored; banned words absent; only real proof used (rest `[verify]`)?
- Every field within its character limit — counted, not estimated?
- Hook stops the scroll in the first 3 seconds / 125 chars for the target platform?
- Awareness ceiling honored — commitment level matches the audience's stage?
- Offer gate checked — value exchange clear, risk-reducer present or flagged as absent?
- Each platform variant uses that platform's register (creator / practitioner / peer)?
- Carousel: Card 1 = hook, final card = CTA; each middle card earns its place?
- Video: on-screen hook ≤8 words; caption compliant with feed-copy limit?
- Variants differ by motivation/angle, not just vocabulary?
