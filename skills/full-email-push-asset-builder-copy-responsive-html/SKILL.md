---
name: full-email-push-asset-builder-copy-responsive-html
description: >
  Takes approved copy and turns it into two shippable, channel-native assets in a single pass:
  (1) a responsive HTML email — paste-into-ESP-ready, dark-mode-aware, with inline styles and
  a plain-text fallback — and (2) a platform-ready push notification payload (web or app) with
  title, body, icon/badge notes, action labels, and deep-link URL, fully within character
  limits. Loads the active brand's voice, colors, proof, and offer from brand-brain before
  writing a single line of copy or code. Calls cta-variant-generator for buttons and
  subject-line-preview-text-optimizer (when installed) for subject/preview pairs. Built on our
  Email Code Architecture (ECA) checklist (our working model) — a five-layer production checklist
  that separates structure, style, content, dark-mode, and QA so a junior operator produces
  deliverable-quality output without knowing CSS email quirks. Use when the user says "build the email HTML,"
  "make this email sendable," "code up the email," "push payload," "give me the ESP file,"
  "finalize the email and push," "turn this copy into the real asset," or hands over a
  finished/approved brief and asks for the production-ready deliverable.
---

# Full Email / Push Asset Builder — Copy + Responsive HTML

Copy in, shippable asset out. This skill does one thing well: take approved or near-approved copy and turn it into the actual HTML email file and the push notification payload a marketer pastes into their ESP or push platform. It does not write strategy, plan sequences, or decide what to say — if copy is still in draft, use `blog-post-drafting-engine`, `lead-nurture-drip-builder`, or `abandon-flow-writer` first. When copy is ready, come here.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand: voice, colors (primary/secondary/accent hex), banned words, offer + destination URLs, real proof, ICP. All HTML color choices and copy overrides come from here.
- **`cta-variant-generator`** (required) — produces the button label(s) and friction-reducer microcopy line; do not invent CTAs without calling it.
- **`subject-line-preview-text-optimizer`** (call when installed) — generates ranked subject line / preview text pairs. If absent, produce 3 pairs inline using the Subject-Line Shelf (see below).
- **`proof-vault`** (call when installed) — surfaces the strongest available social proof for the testimonial block. If absent, use proof from the brand-brain digest, marking anything unconfirmed `[verify]`.

---

## How a run works

```
Step 0  Load brand          ──► call brand-brain (colors, voice, proof, offer, ICP)
Step 1  Clarify scope       ──► email only / push only / both? channel variants?
Step 2  Build subject line  ──► call subject-line-preview-text-optimizer (or inline fallback)
Step 3  Build CTAs          ──► call cta-variant-generator for every button
Step 4  Produce the email   ──► ECA checklist, five layers in order
Step 5  Produce push payload ──► Platform Payload Matrix
Step 6  Self-QA             ──► run the QA checklist before presenting
Step 7  Present + persist   ──► offer to save to ./emails/ and ./push/
```

Never produce HTML or push copy before Steps 0–3 complete.

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Wait for the returned digest:

- **Hex colors:** primary (buttons, links), secondary (header BG or accent band), text, muted/secondary text.
- **Voice adjectives + banned words** — hard overrides on every copy touch.
- **Real proof** — testimonial quote, customer count, or G2 rating for the social-proof block. Mark anything not in the digest `[verify]`.
- **Offer mechanics + destination URL** — the CTA target and what the offer actually is.

If brand-brain is absent: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none, ask for: brand colors (hex), primary font stack, CTA destination URL, and one piece of real proof. Prefer the call.

---

## Step 1 — Clarify scope

Ask once (batch questions, don't ping-pong):
1. **Assets needed:** email + push / email only / push only?
2. **Email type:** promotional / transactional / lifecycle nurture / digest — this sets layout and tone ceiling.
3. **Push channel:** web push (Chrome/Firefox/Safari) / iOS / Android / all three?
4. **Approved copy available?** Paste it. If not, this skill waits — it executes, it does not originate.
5. **Dark-mode preference?** Default yes; opt out explicitly.

---

## Step 2 — Subject line (gate)

Call `subject-line-preview-text-optimizer` (if installed). If absent, produce **3 subject/preview pairs** using the Subject-Line Shelf:

| Angle | Mechanic | Subject-line construction |
|---|---|---|
| **Curiosity gap** | Withhold the payoff | Name the problem, not the solution |
| **Specificity** | Real number or timeframe | Lead with the concrete figure |
| **Benefit-first** | What the reader walks away with | "Get X without Y" / "X in N days" |

- Subject: ≤50 chars (40 safe on mobile preview)
- Preview text: ≤90 chars, extends subject — never repeats it
- Avoid banned words from brand-brain; no all-caps; no manufactured urgency without a real deadline

---

## Step 3 — CTA production (gate)

Call `cta-variant-generator` for every clickable button. Pass: the copy, the offer, the destination URL, the awareness stage of this audience (from brand-brain ICP), and the channel (email). Use Quick mode unless the brief calls for A/B testing, in which case ask for Battery mode. Do not write button labels without this call.

---

## Step 4 — Email HTML: ECA Checklist

**Email Code Architecture (ECA)** is our working model — a five-layer pass performed in strict order. Each layer has a hard constraint. Skipping a layer produces a broken or off-brand file.

### Layer 1 — Structure (table skeleton)

Email clients ignore CSS floats, flexbox, and grid. The only safe layout primitive is nested tables.

```
outer wrapper table   width=600, center-aligned, bgcolor=#FFFFFF
  ├── header row        logo + nav (optional)
  ├── hero row          background color or image (with fallback color)
  ├── body rows         1-column default; 2-column max, inline-table split
  ├── social-proof row  testimonial or logo bar (optional but strongly recommended)
  ├── CTA row           single primary button, centered
  └── footer row        unsubscribe, mailing address (legal requirement)
```

- Max-width: **600px** — narrower than this on desktop looks cheap; wider breaks mobile
- Single-column layouts survive 97%+ of clients; only use 2-column for hero+text splits with a mobile-stacking fallback
- Never rely on `<div>` for layout; every block is a `<td>`

### Layer 2 — Inline styles (deliverability + client compat)

All CSS is inlined on every element that renders content. Embedded `<style>` is used only for dark-mode overrides and media queries (Gmail strips them; Outlook ignores media queries — inline covers the base case).

| Element | Required inline styles |
|---|---|
| `<body>` | `margin:0; padding:0; background-color:#F4F4F4` |
| Outer wrapper `<table>` | `border-collapse:collapse; mso-table-lspace:0; mso-table-rspace:0` |
| `<img>` | `display:block; border:0; outline:none; -ms-interpolation-mode:bicubic` |
| Headline `<td>` | `font-family:[brand stack],Arial,sans-serif; font-size:28px; color:#[brand]` |
| Body copy `<td>` | `font-family:...; font-size:16px; line-height:1.5; color:#[brand text]` |
| CTA button `<a>` | `display:inline-block; background-color:#[brand primary]; color:#FFFFFF; padding:14px 28px; border-radius:4px; text-decoration:none; font-weight:600` |

MSO (Outlook) button fallback — wrap every button in a VML conditional:
```html
<!--[if mso]>
<v:roundrect xmlns:v="urn:schemas-microsoft-com:vml" href="URL"
  style="height:48px;v-text-anchor:middle;width:200px;" arcsize="10%"
  fillcolor="#[PRIMARY_HEX]" strokecolor="none">
  <w:anchorlock/>
  <center style="font-family:Arial;font-size:16px;color:#ffffff;font-weight:bold;">CTA LABEL</center>
</v:roundrect><![endif]-->
<!--[if !mso]><!-->
<a href="URL" ...>CTA LABEL</a>
<!--<![endif]-->
```

### Layer 3 — Content assembly

Use the approved copy verbatim. Voice overrides from brand-brain apply to any microcopy written here (footer, alt text, preview text). Follow this slot map:

| Slot | Content source | Notes |
|---|---|---|
| Subject + preview | Step 2 output | Not in the HTML — provide separately |
| Logo | Brand-brain colors/assets | Alt text = brand name |
| Headline | Approved copy | H1 equivalent in `<td>`; ≤60 chars |
| Subhead / hero body | Approved copy | 1–2 sentences, ≤25 words each |
| Body copy | Approved copy | Max 3 paras before CTA; cut ruthlessly |
| Social proof | `proof-vault` or brand-brain digest | One quote + attribution, or a stat |
| Primary CTA button | Step 3 output | Centered, full-width on mobile |
| Secondary CTA | Step 3 output (if Battery mode) | Text link only, below primary |
| Footer | Legal boilerplate | Unsubscribe link is not optional |

Images: always include `alt` text. For decorative images, `alt=""`. For product/feature images, describe the benefit, not the feature.

### Layer 4 — Dark-mode awareness

Place a `<style>` block in `<head>` with:
```css
@media (prefers-color-scheme: dark) {
  .email-body { background-color: #1A1A1A !important; }
  .email-container { background-color: #2D2D2D !important; }
  .email-text { color: #E8E8E8 !important; }
  .email-muted { color: #A0A0A0 !important; }
}
```
Add corresponding `class` attributes to the matching elements alongside the inline styles. Use `!important` — this is one of the only safe uses in email CSS.

Button background on dark: test contrast. If the brand primary is dark (< 4.5:1 on dark BG), lighten 15% or add a 1px white border.

### Layer 5 — Plain-text fallback

Every HTML email must include a plain-text MIME part. The operator pastes this into their ESP's plain-text field.

Structure: Subject line → headline (ALL CAPS or surrounded by ===) → body copy with manual line breaks at 72 chars → CTA as full URL on its own line → unsubscribe instruction.

---

## Step 5 — Push Notification Payload

### Platform Payload Matrix

| Field | Web Push (Chrome/Firefox) | iOS (APNs) | Android (FCM) |
|---|---|---|---|
| **Title** | ≤50 chars (shown) | ≤65 chars | ≤65 chars |
| **Body** | ≤120 chars | ≤178 chars | ≤240 chars |
| **Icon** | 192×192 px PNG URL | App icon only | 72×72 px PNG URL |
| **Image (large)** | 360×240 px (Chrome) | Not standard | 1080×567 px (FCM) |
| **Action buttons** | Max 2, ≤20 chars each | Max 4 (categories) | Max 3 |
| **Deep link / URL** | Full HTTPS URL | `click_action` key | `click_action` or data key |
| **Badge** | 96×96 monochrome PNG | Badge count integer | Small icon (24dp mono) |
| **TTL** | Default 4 weeks | `apns-expiration` UNIX | `ttl` seconds |

Produce the payload as a JSON block (or structured output if the user specifies their platform/SDK):
```json
{
  "title": "...",
  "body": "...",
  "icon": "[URL or placeholder]",
  "badge": "[URL or placeholder]",
  "url": "[destination from brand-brain offer]",
  "actions": [
    { "action": "primary", "title": "[CTA label from Step 3]" },
    { "action": "dismiss", "title": "Not now" }
  ],
  "image": "[large image URL or [verify]]",
  "ttl": 86400
}
```

**A/B pair:** produce a second payload variant by default (different body copy angle — same brand, different motivation). Label them Variant A and Variant B and recommend which to run first and what the test resolves.

**Emoji policy:** use only if the brand allows them (brand-brain voice). One emoji maximum, at the start of the title or body — not both.

---

## Persistence

Offer to save at the end of every run:

- Email HTML → `./emails/[slug]-email.html`
- Plain-text fallback → `./emails/[slug]-email.txt`
- Subject/preview pairs → top of the HTML file as a comment block, or inline in the output
- Push payloads → `./push/[slug]-push.json`

Never save inside the skill folder. Never overwrite `brand.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No HTML color choices, no CTA labels, no proof before the digest returns.
- **Approved copy in, production asset out.** This skill executes — it does not originate or strategize. If copy is undecided, stop and route to the appropriate upstream skill.
- **ECA order is non-negotiable.** Structure → styles → content → dark mode → plain text, in that order. Skipping layers produces broken files.
- **Outlet for every image.** Every `<img>` has a meaningful `alt`. Decorative images get `alt=""`.
- **Real proof or `[verify]`.** Never invent social proof, ratings, or customer numbers.
- **Character limits are hard limits.** Truncated push notifications destroy click rates. Fail visibly rather than silently exceed limits.
- **MSO fallback on every button.** Outlook represents [verify]% of enterprise opens. A button that does not render in Outlook is a broken CTA for a significant segment.

---

## What Not to Do

- Don't write copy from scratch — this skill finishes assets, not briefs. Route undecided copy upstream.
- Don't produce CTAs without calling `cta-variant-generator` first.
- Don't use `<div>` or flexbox for layout in the email — table only.
- Don't embed images as base64 — use hosted URLs with fallback alt text.
- Don't use `style` blocks for layout (only for dark-mode `@media` overrides) — inline everything else.
- Don't produce only one push variant — always present an A/B pair.
- Don't invent proof, ratings, or customer logos not in the brand digest.
- Don't ship an HTML email without a plain-text fallback.

---

## Quality Checklist (self-review before presenting)

**Brand & copy**
- [ ] `brand-brain` called; colors, voice, proof, and CTA destination confirmed?
- [ ] `cta-variant-generator` called; button labels + microcopy returned?
- [ ] Voice and banned-words honored in all microcopy (footer, alt text, proof slot)?
- [ ] All proof is real and sourced from the digest (nothing invented, rest `[verify]`)?

**Email HTML**
- [ ] All five ECA layers present and in order?
- [ ] Every layout element uses `<table>` / `<td>` — no `<div>` layout?
- [ ] All styles inlined on content elements?
- [ ] MSO VML fallback on every button?
- [ ] Dark-mode `@media` block in `<head>` with `!important` overrides?
- [ ] Every `<img>` has `alt` text?
- [ ] `<meta name="viewport">` and charset meta in `<head>`?
- [ ] Plain-text fallback produced?
- [ ] Footer includes unsubscribe link and mailing address?

**Push payload**
- [ ] Title within platform limit (50 / 65 chars)?
- [ ] Body within platform limit (120 / 178 / 240 chars)?
- [ ] Deep-link URL present and matches brand-brain offer destination?
- [ ] Variant A + Variant B produced with a stated test hypothesis?
- [ ] Emoji policy respected (brand allows / one max)?

**Persistence**
- [ ] Offered to save `./emails/[slug]-email.html`, `.txt`, and `./push/[slug]-push.json`?
