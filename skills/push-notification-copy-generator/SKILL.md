---
name: push-notification-copy-generator
description: >
  Generates ready-to-send web and app push notification copy — title, body, CTA action label,
  and emoji variants — that passes character limits, earns the tap, and drives the session.
  Takes a campaign goal, target segment, and channel (web/app) and returns production-ready
  push variants with A/B pairs, a lead recommendation, and a compliance spot-check. Every
  notification is written in the brand's real voice against the brand's real offer and real
  proof, because brand context is loaded from the `brand-brain` skill — not re-derived here.
  Two modes: Quick (one campaign → best variant + 1 A/B pair + rationale) and Battery (one
  campaign → full variant set across persuasion angles with ranked A/B recommendation).
  Optionally calls `cta-variant-generator` when the action label needs multi-angle pressure-testing.
  Use when the user says "write a push notification," "push copy," "web push variants,"
  "push for cart abandon / re-engagement / flash sale / new post / feature announce,"
  "push A/B test," or hands over a campaign brief and asks for the notification.
---

# Push Notification Copy Generator

Campaign goal + segment in → production-ready push copy out. Every variant fits the character window, earns the tap on its own merit, and sounds like the brand — not a generic "Don't miss out!" blast.

This skill writes push notifications. It does not design the campaign flow, configure the platform, or set send-time logic. If the trigger, audience, or offer is the root problem, it says so — it does not paper over a weak setup with snappy copy.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's context. Voice, banned words, ICP, offer/pricing, proof, and positioning all come from here. Do not reimplement brand resolution or storage.
- **`cta-variant-generator`** (optional, Battery mode) — when the action label needs multi-angle pressure-testing beyond 2–3 alternatives. Call it, pass the channel + awareness stage; fold its output into the action-label column.
- **`proof-vault`** (optional) — when a campaign calls for a specific stat or testimonial snippet to anchor urgency. Use the returned proof as-is; mark anything unconfirmed `[verify]`.

---

## How a run works

```
Step 0  Load the brand   ──► call brand-brain; get voice, banned words, offer, proof, ICP
Step 1  Pick the mode    ──► Quick (default) | Battery (on request)
Step 2  Set the channel  ──► Web push | App push (limits differ; see table below)
Step 3  Do the work      ──► PESO framework guides the copy; see sections below
Step 4  Self-review      ──► checklist pass, then present
Step 5  Persist          ──► offer to save sequences to ./push/
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns: voice adjectives, banned words, offer mechanics + destination URLs, real proof points, ICP + awareness tendency, and the path to `brand.md`. Do not write a single line of push copy until it returns.

Obey the returned voice and banned-words as hard overrides. Use only real, returned proof — mark anything else `[verify]`. Anchor the destination URL to the brand's known offer pages.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (product · ICP + awareness tendency · offer + destination URL · voice adjectives + banned words), then proceed.

### Step 1 — Pick the mode

- **Quick mode (default).** Campaign brief in → one recommended push notification (title + body + action label) + one A/B variant on a *different* angle + one-line rationale.
- **Battery mode.** Triggered by "variants," "full set," "give me options," "A/B," an explicit count, or a sequence request → full angle-spread table + ranked A/B recommendation + optional sequence plan.

When unsure, default to Quick and offer Battery at the end.

---

## Channel character limits

| Channel | Title | Body | Action label | Notes |
|---|---|---|---|---|
| Web push (Chrome/Edge) | 50 chars | 125 chars | 20 chars | Truncation varies by OS/browser — treat as hard limits |
| Web push (Firefox) | 50 chars | 200 chars | 20 chars | |
| Web push (Safari / macOS) | 50 chars | 100 chars | — | No custom action button on older macOS |
| App push (iOS) | 65 chars | 240 chars | — | Alert text only; lock-screen truncates at ~110 chars body |
| App push (Android) | 65 chars | 240 chars | — | Expanded view shows full body; collapsed ~45 chars |

When the channel is unspecified, default to the tightest window (web/Chrome) and flag the assumption.

---

## The PESO Push Framework (named framework — all modes use this)

Every notification earns its tap through one dominant motivational lever. Vary the lever across variants — not just the vocabulary.

| Lever | Trigger psychology | Best push moment | Example title pattern |
|---|---|---|---|
| **P — Pain/Problem** | Loss aversion, status quo threat | Abandon flows, expiry alerts, risk reminders | "Your [X] is at risk" |
| **E — Exclusivity** | FOMO, insider status | Flash sales, early access, loyalty tiers | "Early access: [X] just for you" |
| **S — Social proof** | Conformity, trust | Feature launches, milestone campaigns | "[N] teams switched this week" |
| **O — Offer/Urgency** | Scarcity, deadline | Promotional, win-back, cart abandon | "[X] ends tonight" |

Subframes within each lever:

- **Curiosity gap** — open a loop the body closes: title withholds; body delivers.
- **Specificity** — numbers anchor credibility. "3x faster" beats "much faster." Real numbers only; mark invented ones `[verify]`.
- **Identity / transformation** — speak to who they want to become, not just what to do.
- **Low-friction** — reduce the perceived cost of tapping: "takes 2 minutes," "no card needed."

---

## Quick mode (default)

1. **Identify the dominant lever** (PESO) given the campaign goal + segment awareness.
2. **Draft the title** — lead with the outcome or the open loop, not the brand name. Fit the tightest applicable limit.
3. **Draft the body** — pay off the title's promise. One idea only. Close with the action or the stake.
4. **Draft the action label** — verb-led, specific (not "Tap here"). Match the destination.
5. **Check emoji** — offer one emoji variant (prepend or append title) only if the brand allows it. One emoji maximum; never substitute emoji for words.
6. **Write one A/B variant** on a *different* PESO lever — not a synonym of the primary.
7. **One-line rationale** — which lever, why it fits this segment + awareness stage.

Quick-mode output: primary notification (title / body / action), A/B variant, rationale, destination URL, char counts. No table unless asked.

---

## Battery mode (on request)

### Gate: pressure-test the setup first

Before generating variants: does the campaign have a clear trigger event, a defined segment, and a real destination? If not, name the gap and offer a fix. Clever copy on a misconfigured campaign is waste.

### Battery output structure

```
## Push variants — [campaign name / goal]
Context: [brand · segment · awareness stage · channel · destination]
Brand: [slug, via brand-brain]
[⚠ setup gate note, if any]

### Variants
| # | Lever | Title (chars) | Body (chars) | Action label | Emoji Y/N |
|---|---|---|---|---|---|

### A/B recommendation
Primary: #_  ·  Variant B: #_
Lever contrast: [what motivational difference the test resolves]
Success metric: [CTR / session start / conversion — be specific]

### Sequence option (if applicable)
[Day 0 / Day 1 / Day 3 trigger logic and lever escalation]
```

**Generate ≥ 5 variants** across at least 3 PESO levers. Include:
- At least 1 specificity/number variant (real data only or `[verify]`)
- At least 1 curiosity-gap variant
- At least 1 low-commitment / low-friction variant
- At least 1 urgency/scarcity variant (honest — no fake deadlines)

**A/B recommendation:** pick a Primary and a deliberately *different-lever* Variant B. State the hypothesis the test resolves (e.g., "Does loss aversion outperform social proof for lapsed users in this segment?").

**Sequence option:** if the campaign is a multi-touch flow (abandon, re-engagement, onboarding), suggest a 3-step lever escalation (e.g., P → O → E) with timing gaps and the sunset condition.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No push copy before `brand-brain` returns. Its voice + banned-words override everything here.
- **Earn the tap, respect the lock screen.** Every notification must justify the interruption. If it can't pass the "would I want this?" test for the ICP, rewrite it.
- **Vary the lever, not the vocabulary.** Different PESO angles = different reasons to tap. Synonyms of the same angle are not variants.
- **Honest urgency only.** No fake countdowns, manufactured scarcity, or invented social proof. Unconfirmed numbers are `[verify]`.
- **Fit the window first.** A brilliant body that truncates on the lock screen is not brilliant. Count before you ship.
- **One idea per notification.** Title makes a claim; body closes it. Never two competing hooks.
- **Emoji: purposeful or absent.** One emoji that adds signal. Never multiple; never a substitute for words.

## What Not to Do

- Don't write push copy before `brand-brain` returns the active brand.
- Don't reimplement brand scanning, voice derivation, or storage here — call `brand-brain`.
- Don't write fake urgency ("Only 3 left!" when inventory is unlimited) or unverified stats.
- Don't exceed character limits — count every variant before presenting.
- Don't use the brand name as the title — the sender field already shows it.
- Don't stack two calls to action in one notification.
- Don't use banned words from the brand voice, even if they "test well" generically.
- Don't generate near-identical variants with different wording — if you can't find distinct levers, say the offer is too thin.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy written?
- Voice + banned-words honored; only real proof used (rest `[verify]`)?
- Every variant counted against the correct channel's character limits?
- Quick: one primary + one different-lever A/B + one-line rationale + destination URL?
- Battery: ≥ 5 variants, ≥ 3 PESO levers, including specificity, curiosity-gap, low-friction, and honest-urgency types?
- A/B pair is genuinely different-lever, not synonyms? Hypothesis stated?
- Emoji present only if brand allows; never more than one per notification?
- Setup gate checked in Battery mode — trigger, segment, and destination confirmed before generating?
- Sequence option offered when campaign is multi-touch?
- Output offered for save to `./push/[slug]-push.md` if it's a reusable sequence or battery?
