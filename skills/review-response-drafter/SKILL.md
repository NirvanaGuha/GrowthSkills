---
name: review-response-drafter
description: >
  Drafts professional, on-brand public responses to customer reviews on G2, Capterra, Google Business,
  and App Store / Play Store. Sentiment-aware: positive reviews get acknowledgment + a subtle expansion
  hook; neutral/mixed reviews get a specific close of the gap; negative reviews get de-escalation,
  accountability, and a concrete resolution path — never defensiveness. Handles batch-mode (paste a
  list, get a table of responses) and single-review mode. Brand voice, proof points, and banned words
  come from brand-brain, so responses sound human and on-brand rather than generic corporate. Calls
  voice-of-customer-mining-pipeline to surface pain-to-copy patterns when batch size is large enough to
  warrant theme analysis. Calls proof-vault when a response would benefit from a real stat or win to
  reinforce credibility. Use when the user says "respond to this review," "draft review replies,"
  "review response," "reply to G2/Capterra/Google/App Store," "review management," "handle negative
  review," or pastes one or more review texts and asks for a response.
---

# Review Response Drafter

Public review responses are marketing copy that the whole world can read — but they have to feel like
a real human wrote them in two minutes, not a brand manager in two hours. This skill threads that
needle: every response is on-brand, sentiment-calibrated, platform-appropriate, and short enough that
a reader actually finishes it.

It does not write generic "Thanks for your feedback!" filler. It does not get defensive. It does not
invent resolutions that don't exist. It acknowledges specifically, closes the gap concretely, and
exits cleanly.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, ICP, positioning, and real proof before
  any response is drafted. No response before this returns.
- **`proof-vault`** (optional) — called when a positive or neutral response benefits from a real stat
  or customer-win reinforcement to add credibility without fabrication.
- **`voice-of-customer-mining-pipeline`** (optional) — called in batch mode when 5+ reviews are
  present and theme-level pattern analysis would improve response strategy (e.g., a recurring UX
  complaint deserves a consistent framing, not 7 different ad-hoc phrasings).

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain skill
Step 1  Classify the input   ──► Single review or Batch?
Step 2  Score each review    ──► Sentiment + platform + public-response risk
Step 3  Apply the skeleton   ──► ACRA (Acknowledge / Close-the-gap / Redirect / Anchor)
Step 4  Calibrate per mode   ──► Positive / Neutral-Mixed / Negative
Step 5  Self-review          ──► checklist pass, then present
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before writing a single word. It returns voice adjectives, banned
words, ICP, offer mechanics, real proof, and positioning. Obey voice and banned words as hard
overrides; use only real proof (mark anything else `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly. If none exists, ask the user to install `brand-brain` or answer a 4-question
mini-setup (product description, ICP, 3 voice adjectives, banned words) before proceeding.

---

## ACRA — our working reply structure

ACRA (a house mnemonic, not an established model) is the four-beat skeleton we use here. Every
response — positive, neutral, or negative — follows the same four beats. Execution varies by
sentiment; the skeleton does not.

| Beat | What it does | Length |
|---|---|---|
| **A — Acknowledge** | Name what the reviewer actually said (not generic thanks). | 1 sentence |
| **C — Close the gap** | For positives: reinforce + invite more. For negatives/mixed: own the specific issue and name the concrete action taken or in progress. | 1–2 sentences |
| **R — Redirect** | Move the conversation somewhere productive: a support channel, a resource, a next step — never into a public debate. | 1 sentence (as needed) |
| **A — Anchor** | Sign off with something warm and brand-true. Never "Sincerely, The Team." | 1 sentence |

Total length: 3–5 sentences for most platforms. App Store / Play Store: tighter — 2–4 sentences.
G2 / Capterra: can run to a short paragraph when detailed feedback warrants it.

---

## Sentiment calibration

### Positive (4–5 stars, glowing tone)

Goal: convert a happy reviewer into an amplifier.

- Acknowledge the *specific* thing they praised — not "your kind words" but the actual feature or
  outcome they named.
- Add one brief genuine hook toward deeper value (a use case, a resource, a next milestone) only
  if it flows naturally — never a sales pitch.
- Keep it warm and short; over-engineering a happy response signals desperation.
- Do NOT: add a CTA to "share with friends," promise a feature that doesn't exist, or use their
  name without explicit permission cues on the platform.

### Neutral / Mixed (3 stars, or positive-but-with-caveats)

Goal: show the business listens and acts.

- Acknowledge both the positive and the specific friction — don't pretend the criticism isn't there.
- Offer a concrete next step for the pain point (a doc, a support channel, a roadmap note) rather
  than vague "we're always improving."
- If the gap has already been addressed in a product update, say so briefly (real info only — no
  `[verify]` items in a public claim).
- Invite them back to re-experience the product; don't demand a rating update.

### Negative (1–2 stars, complaint-led, or hostile tone)

Goal: de-escalate, own accountability, protect the brand's public credibility with other readers.

**Remember: the primary audience for a negative response is every future prospect reading the
review page — not just the unhappy reviewer.**

- **De-escalate first.** No defensiveness, no counter-arguments in the public reply, no "per our
  terms of service."
- **Acknowledge specifically.** Generic "we're sorry you feel that way" is brand-damaging.
  Name the actual pain.
- **Own it or explain it — not both.** Pick accountability (if the issue was real) or context
  (if there is a genuine misunderstanding) — never stack both, which reads as excuse-making.
- **Move offline fast.** Include a real support contact or escalation path (use the brand's actual
  support URL or email from `brand-brain` — never fabricate one).
- **Do not negotiate in public.** Offers of compensation, refunds, or exceptions belong in a private
  channel, never in a public response.
- **Short over long.** A 5-sentence tight response signals confidence; a 10-sentence one signals
  panic.

---

## Platform-specific adjustments

| Platform | Char / tone notes |
|---|---|
| **G2 / Capterra** | Professional B2B register; reviewers are practitioners — match their vocabulary. Can name features / integrations. 3–5 sentences. |
| **Google Business** | Broadest audience; most public. Slightly warmer. Avoid technical jargon. Keep to 3–4 sentences. |
| **App Store (iOS)** | Apple does not show reviewer names publicly in all contexts. Short and direct: 2–4 sentences. No markdown, no links. |
| **Play Store (Android)** | Same as App Store length. Google does surface developer responses prominently — crisp matters. |
| **Trustpilot** | Similar to Google Business; Trustpilot's own guidelines discourage incentivized reply language. |

---

## Batch mode

When the user pastes or provides 5+ reviews:

1. Optionally invoke `voice-of-customer-mining-pipeline` to identify recurring themes before drafting,
   so repeated complaints get a consistent strategic framing rather than 7 independent improvised
   answers.
2. Produce a response table:

```
## Review responses — [brand] · [platform] · [date]
| # | Stars | Key theme | Sentiment | Response |
|---|---|---|---|---|
```

3. After the table, add a **Pattern note** (2–3 sentences) on any theme appearing in 2+ reviews —
   this is actionable intel for the product or CS team, not just copy.

Save batch output to `./reviews/[brand-slug]-[platform]-responses-[YYYY-MM].md` when the user asks
for a saved artifact.

---

## Principles

- **Brand-brain first.** No response before the brand loads. Its voice and banned words override
  everything here.
- **Specific beats generic.** Name what they said. "Your point about the bulk-export workflow" beats
  "your feedback."
- **Primary audience is prospective buyers.** Especially on negative responses: write for the 1,000
  future readers, not just the one upset reviewer.
- **Own or explain, not both.** Stacking accountability and context reads as excuse-making.
- **Move negative conversations offline immediately.** Never negotiate in public.
- **Real proof only.** No invented resolutions, no fabricated metrics, no features that don't exist.
  Anything unconfirmed is `[verify]` — or omitted.
- **Short is confident.** 3–5 sentences is the ceiling for most responses.

---

## What not to do

- Don't write a response before `brand-brain` returns the active brand.
- Don't use "we're sorry you feel that way" — it's dismissive and publicly well-known as a non-apology.
- Don't promise a feature, a fix, or a refund in a public response unless it already exists.
- Don't use the reviewer's full name unless it's explicitly shown on the platform.
- Don't argue facts publicly, even when the review is factually wrong — address context, never debate.
- Don't include marketing copy in a negative response (pricing, trials, promo codes) — it reads as tone-deaf.
- Don't reimplement brand scanning or voice derivation here — call `brand-brain`.
- Don't let a response run over 6 sentences; if more is needed, the extra belongs in a support ticket.

---

## Quality checklist

- `brand-brain` called and active brand loaded before any response was drafted?
- Voice adjectives and banned words honored across every response?
- Each response names a specific element from the review (not generic praise or apology)?
- Negative responses: de-escalation first, one of (own / explain) not both, offline redirect present?
- No fabricated proof, unconfirmed resolutions, or invented support contacts — only `[verify]`-marked or omitted?
- Platform length and tone norms respected?
- Batch: pattern note included when recurring themes found?
- Save path offered for batch artifacts?
