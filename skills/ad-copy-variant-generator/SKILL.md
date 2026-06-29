---
name: ad-copy-variant-generator
description: >
  Product description, value props, and character limits → headline/description and
  copy/image/CTA variants per ad format across distinct angles. Covers search (Google/Bing RSAs,
  DSAs), social (Meta, LinkedIn, X), display, and YouTube bumper/in-stream — output is always
  channel-format-correct and char-limit-hard. Works in two modes: Focused (default) for a single
  channel/format with a clear A/B recommendation, and Full Battery for multi-channel variant sets
  across ≥6 motivation angles. Calls brand-brain for voice, ICP, and proof before writing a
  single word; calls cta-variant-generator for the action component rather than reimplementing CTA
  craft. Applies the Eugene Schwartz Five Stages of Market Awareness + a working checklist of
  direct-response craft principles in the David Abbott tradition as its core framework — so a junior produces output that a senior media
  buyer can run without editing. Use when the user asks for "ad copy," "Google ads copy," "Meta ad
  variants," "ad creative," "write me ads," "headline variants for [campaign]," "paid copy,"
  "performance copy," or hands over a brief and asks what to test.
---

# Ad Copy Variant Generator

Platform format in, on-brand performance copy out. Every variant is channel-correct, char-limited, and built across distinct persuasion angles — not synonyms of the same claim. Brand context, ICP, and proof come from `brand-brain`; the CTA component comes from `cta-variant-generator`. This skill's job is the copy in between.

---

## Skills this calls

- **`brand-brain`** (required, always first) — voice, banned words, ICP, offer mechanics, real proof, positioning, awareness tendency.
- **`cta-variant-generator`** (required for CTA component) — generate the button/headline action when the format includes a standalone CTA field.
- *(optional, when installed)* `proof-vault` for live proof microcopy snippets; `icp-persona-builder` if persona depth is needed beyond the brand-brain digest; `competitive-intelligence-dossier` for comparative angle positioning.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain (always, always first)
Step 1  Get the format spec  ──► channel + format → char limits, field count, field names
Step 2  Pick the mode        ──► Focused (default) | Full Battery
Step 3  Gate the offer       ──► is the value exchange clear? commitment = awareness stage?
Step 4  Generate variants    ──► across angles, not vocabulary (Schwartz + Abbott framework)
Step 5  Validate             ──► char limits, voice, banned words, proof honesty
Step 6  Recommend A/B pair   ──► primary angle + different-angle Variant B + stated hypothesis
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Do not write a single headline until it returns. Obey voice adjectives and banned words as hard overrides. Use only real proof from the returned digest; mark everything else `[verify]`.

Fallback if `brand-brain` is not installed: read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if neither exists, ask the user to install `brand-brain` or answer four inline questions (product + ICP + offer mechanics + 3 voice adjectives/banned words) before proceeding.

---

## Step 1 — Format spec

Before writing, confirm the channel + format combination and hard-lock the character limits. Use the table below. If the user hasn't specified, ask (or infer from the brief) and confirm.

| Channel | Format | Fields | Hard char limits |
|---|---|---|---|
| Google Search | RSA | 15 headlines / 4 descriptions | H: 30 · D: 90 |
| Google Search | DSA | 2 ad descriptions | D: 90 × 2 |
| Meta (FB/IG) | Single Image/Video | Primary text · Headline · Description | PT: 125 · H: 40 · D: 30 |
| Meta | Carousel | Per-card headline + CTA | H: 40 per card |
| LinkedIn | Single Image | Intro copy · Headline | IC: 150 · H: 70 |
| LinkedIn | Document/Thought Leader | Intro copy | IC: 150 |
| X (Twitter) | Promoted post | Body (incl. URL) | 280 − URL (23) = 257 |
| Display (GDN) | Responsive Display | 5 short headlines · 1 long · desc | SH: 30 · LH: 90 · D: 90 |
| YouTube | Bumper (6s) | Companion banner headline | H: 15 (spoken ≤10 words) |
| YouTube | In-stream (skippable) | Hook line (0–5 s) + body | Hook: ~8 words before skip |

**RSA-specific rule:** treat the 15 headlines as an asset library, not 15 sequential ads. Write 5 core + 5 proof/feature + 5 angle-test. Google combines them; ensure every pairing is coherent.

---

## Step 2 — Modes

**Focused mode (default):** one channel + format → the recommended set of headlines/descriptions/copy lines + one clean A/B pair. No sprawl. Triggered by a single-channel request or default behavior.

**Full Battery mode:** multiple channels or explicit request for "all variants," "give me options," "battery." Produces a section per channel/format, each with ≥6 angle variants and a channel-specific A/B recommendation.

When unsure, default to Focused and offer Full Battery at the end.

---

## The core framework — Schwartz Awareness × Abbott Craft

### Schwartz: match the angle to the awareness stage

The brand-brain digest tells you the ICP's awareness tendency. Don't exceed the ceiling.

| Awareness stage | Right angle | Opening move |
|---|---|---|
| Unaware | Problem agitation — name the pain before naming you | "Still losing 40% of cart abandoners?" |
| Problem-aware | Solution education — category lead, not brand | "Web push wins back cart abandoners in 15 min" |
| Solution-aware | Differentiation — why yours over the category | "No other push tool learns send-time per user" |
| Product-aware | Proof + risk removal — reviews, guarantees, trial | "2,500 brands, zero setup fee, cancel anytime" |
| Most-aware | Offer + urgency — transact now | "Start free · 500 subs on us · upgrade later" |

Most paid campaigns land in Solution-aware or Product-aware. Default there unless the brief or brand-brain says otherwise.

### Abbott: the direct-response copy principles

Direct-response craft principles in the tradition of David Abbott (reader self-interest, specificity over superlatives, one idea, honesty), operationalized here for digital ads — a working checklist, not a canonical Abbott list:

1. **One idea per ad.** If you need a comma to hold two claims together, cut one.
2. **The first line buys the second.** In Meta primary text and YouTube hooks, the opening earns attention — it does not sell.
3. **Specificity over superlatives.** "3× faster indexing" beats "industry-leading speed" every time.
4. **Honesty as a persuasion tool.** Naming a genuine limitation (then resolving it) builds more trust than hiding it.
5. **The reader's self-interest, not yours.** "You get / your store recovers / your team saves" — not "we built / our platform offers."
6. **Show the before.** The pain or friction state, named precisely, makes the benefit real.

---

## Angles to generate (vary motivation, not vocabulary)

Produce variants across distinct motivations — never ten rewrites of the same idea.

| Angle | What it activates | Example hook |
|---|---|---|
| Outcome/ROI | Greed, aspiration | "Recover $X in abandoned carts per month" |
| Specificity | Credibility, trust | "2,500 stores · 98.7% delivery rate" |
| Pain agitation | Problem urgency | "Every unsubscribed push is revenue left behind" |
| Low commitment | Risk reduction | "Free forever plan · no card · no catch" |
| Social proof | FOMO, conformity | "The push tool trusted by Shopify's top 500" |
| Competitive reframe | Dissatisfaction | "Tired of Klaviyo's push limits? You've outgrown it." |
| Speed/ease | Impatience, overwhelm | "Live in 10 minutes. No developer needed." |
| Transformation/identity | Aspiration | "Run retention like a seven-figure brand" |
| Curiosity/pattern interrupt | Attention | "The push stat your ESP doesn't show you" |
| Honest limitation → resolution | Trust | "Yes, push has low opt-in rates. Here's how to fix that." |

Minimum 6 angles per run. Always include ≥1 low-commitment and ≥1 specific-number angle.

---

## Output format

### Focused mode output

```
## Ad copy — [Channel · Format · Brand]
Context: [ICP · awareness stage · offer · destination]
[⚠ offer gate note, if any]

### Headlines
| # | Headline | Angle | Chars |

### Descriptions / Body copy
| # | Text | Angle | Chars |

### Recommended A/B pair
Primary: #_  (angle: ___)
Variant B: #_  (angle: ___)
Hypothesis: [what this test resolves — e.g. "Does ROI specificity outperform low-commitment for cold audiences?"]
Message-match note: [confirm destination page carries through the headline promise]
```

### Full Battery mode output

Repeat the Focused block per channel/format, then add a cross-channel summary of which angles to prioritize first based on the ICP + awareness stage.

Save Full Battery output to `./ads/[brand-slug]-[campaign]-variants.md` if the user asks; Focused is inline.

---

## RSA asset library structure (Google-specific)

For RSAs, group the 15 headlines into three tiers rather than presenting them as a flat list:

- **Core (5):** brand + offer + primary differentiator — always shown, high pinning priority.
- **Proof (5):** numbers, social proof, trust signals — rotate with core.
- **Angle test (5):** one per motivation angle — these are your actual A/B surface; watch Google's asset performance labels (Best / Good / Low) and cut Lows after 500+ impressions.

Flag which headlines should be pinned (position 1 / position 2) and why.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before it returns. Voice + banned words override every angle here.
- **One idea per ad.** If it takes a semicolon to hold it together, split it.
- **Hard limits are hard.** Never hand back copy that exceeds the platform's char limit, even by one character.
- **Vary motivation, not vocabulary.** Six synonyms for "easy" is one angle. Find six different reasons to click.
- **Specific beats superlative.** Real numbers or `[verify]` — never invent proof to fill a specificity angle.
- **Message match is non-negotiable.** The destination page must carry through the headline's promise. Flag mismatches.
- **State the A/B hypothesis.** A primary + a different-angle Variant B with a stated hypothesis, every time.

---

## What Not to Do

- Don't write copy before `brand-brain` returns the active brand.
- Don't hand back copy that exceeds platform char limits — validate before presenting.
- Don't write near-identical variants (same claim, different verb) — if you can't find distinct angles, say the offer is too thin.
- Don't use urgency language the brand hasn't authorized ("Act now," "Limited time") unless the offer is genuinely constrained.
- Don't invent proof, review counts, or customer numbers — mark unconfirmed `[verify]`.
- Don't paper over a weak or unclear value exchange with clever copy — name the offer gap and ask.
- Don't reimplement CTA logic here — call `cta-variant-generator` for standalone CTA fields.

---

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called and brand loaded before any copy written?
- [ ] Voice + banned words honored; only real proof used (rest `[verify]`)?
- [ ] Every field within platform char limit (count characters, don't estimate)?
- [ ] ≥6 distinct motivation angles — not synonyms?
- [ ] ≥1 low-commitment angle and ≥1 specific-number angle included?
- [ ] RSAs: 15 headlines organized in core/proof/angle-test tiers with pin recommendations?
- [ ] A/B recommendation: primary + different-angle Variant B + stated hypothesis?
- [ ] Message-match between headline promise and destination page addressed?
- [ ] Offer weakness (if any) flagged, not papered over?
