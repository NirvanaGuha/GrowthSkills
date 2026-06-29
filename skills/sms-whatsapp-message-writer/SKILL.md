---
name: sms-whatsapp-message-writer
description: >
  Writes compliant, conversion-optimized SMS and WhatsApp messages for any lifecycle stage or
  campaign goal — welcome, promotional, transactional, re-engagement, cart-abandonment, or
  drip sequence. Inputs: campaign brief + compliance jurisdiction (US/CA/UK/EU/AU or global).
  Outputs: message copy with opt-out language, accurate character counts, segment counts for SMS,
  WhatsApp template category and button spec where needed, and a per-message compliance
  checksheet. Works across channels: carrier SMS (160-char GSM / 153-char per segment when
  concatenated), WhatsApp Business API (approved templates or session messages), and RCS
  (structured cards). Calls brand-brain for voice and proof, cta-variant-generator for CTA
  lines, and lifecycle-email-push-copy-reviewer for a review pass on multi-message sequences.
  Does NOT manage opt-in list building or ESP/CPaaS integration — use notification-opt-in-prompt-
  optimizer and esp-map-platform-builder for those. Use whenever the user says "write an SMS,"
  "WhatsApp message," "text blast," "SMS campaign," "WhatsApp template," "RCS message," "write
  a promo text," "SMS drip," or hands over a campaign brief asking for any channel that sends
  to a mobile number.
---

# SMS & WhatsApp Message Writer

Thirty characters decide whether this message gets read or ignored — and one missing opt-out
footer can result in a TCPA fine of up to $1,500 per message sent. This skill writes copy
that earns the tap AND survives legal review.

SMS and WhatsApp are the highest-intimacy channels in the stack. Open rates sit above 90%
[verify exact figure against your ESP/CPaaS data], but the trust budget is thin: a single
off-brand, spammy, or legally non-compliant message trains subscribers to opt out permanently.
This skill applies **BRIEF** (Brevity · Relevance · Intent · Evidence · Fallback) — our
working checklist for this skill — to every message so the copy is tight, compliant, and
brand-true.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's voice, banned words, offer
  mechanics, real proof, and ICP before any copy is written. Do not reimplement brand
  scanning or interviewing here.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
  + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP
  summary, offer mechanics, voice adjectives, and banned words before proceeding.

- **`cta-variant-generator`** (compose) — call for the tap-through CTA line (link, reply
  keyword, or call-to-action word) if the user wants multiple CTA angles or an A/B pair.
  Synthesize inline for single-message quick jobs.

- **`lifecycle-email-push-copy-reviewer`** (review) — run on completed multi-message
  sequences to check brand voice, flow, urgency/clarity, and character-limit compliance
  before presenting. Skip for single standalone messages.

- **`notification-opt-in-prompt-optimizer`** (upstream) — handles the consent capture
  surface; this skill starts *after* the subscriber has opted in.

- **`proof-vault`** (compose, when installed) — pull real social proof for endorsement
  lines. Use only proof returned from `brand-brain`'s digest if `proof-vault` is absent.

---

## How a run works

```
Step 0  Load the brand    ──► call brand-brain; obey voice + banned-words; load real proof
Step 1  Classify the job  ──► Quick (1 message) | Sequence (2+ messages) | Template (WA API)
Step 2  Set compliance    ──► jurisdiction → required opt-out text + consent class
Step 3  Write the copy
Step 4  Count + check     ──► characters, segments, compliance checksheet
Step 5  Review + present  ──► call lifecycle-email-push-copy-reviewer for sequences
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active
brand's digest — voice adjectives, banned words, offer mechanics, real proof, ICP, and
positioning. **Do not write a single character of copy before it returns.**

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
+ that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP
summary, offer mechanics, voice adjectives, and banned words before proceeding.

### Step 1 — Classify the job

| Job type | Trigger | Output |
|---|---|---|
| **Quick** (default) | Single message request | 1 recommended message + 1 alternate angle + compliance checksheet |
| **Sequence** | "drip," "series," "campaign," or explicit count | Numbered sequence with delay logic, fallback branch, review pass |
| **WA Template** | "WhatsApp template," "Meta approve," or WA Business API context | Template body + category tag + button spec + variable list |

### Step 2 — Set compliance context

Ask if not stated. One jurisdiction is enough for a single brand — store it.

| Jurisdiction | Opt-out language required | Consent class |
|---|---|---|
| US (TCPA / CTIA) | "Reply STOP to unsubscribe" every promo message | Express written consent for marketing; implied for transactional |
| Canada (CASL) | "Reply STOP to unsubscribe" + sender ID | Express consent; proof required |
| UK/EU (PECR / ePrivacy) | "Reply STOP" or equivalent; sender ID | Opt-in required; soft-opt-in only for existing customers |
| Australia (Spam Act 2003) | Unsubscribe mechanism on every commercial message | Express or inferred consent; audit trail required |
| Global / unknown | Apply US+EU minimum: STOP language + sender ID every message | Treat all as marketing; flag for legal review |

TCPA statutory damages: $500–$1,500 per message [47 U.S.C. § 227]. CASL penalties up to
$10M CAD per violation [CASL s.20]. Flag any intent to send without confirmed opt-in as
a **[COMPLIANCE BLOCK]** — do not write the message until resolved.

---

## BRIEF — our working checklist for every message

**B — Brevity.** Under 160 GSM characters for single-segment SMS (see limits below). Every
word must earn its place; cut adjectives before cutting substance.

**R — Relevance.** The opening word or first clause must signal why *this person* got *this
message right now*. Name the trigger: order shipped, cart left behind, flash sale, renewal
due. Context = permission.

**I — Intent.** One action per message. Never stack two CTAs. The message knows what it
wants — a tap, a reply, a store visit.

**E — Evidence.** One real proof point (from brand-brain's digest or `proof-vault`) is
stronger than any adjective. "4.8/5 from 2,400 customers" outperforms "amazing quality."
Mark unconfirmed proof `[verify]`.

**F — Fallback.** Every sequence needs an exit: the opt-out line (mandatory), and for
sequences, a no-action branch. Write both.

---

## Character limits and segment math

| Channel / format | Single limit | Concatenated |
|---|---|---|
| SMS — GSM-7 characters | 160 chars = 1 segment | 153 chars/segment (UDH header costs 7) |
| SMS — Unicode (emoji, accented chars, non-Latin) | 70 chars = 1 segment | 67 chars/segment |
| WhatsApp body (session or template) | 1,024 chars (4,096 for template header) | N/A — single message, no segments |
| RCS rich card (short description) | ~2,000 chars | N/A |

**Practical rule:** target ≤155 GSM chars for SMS to stay single-segment on most carriers.
Emoji in SMS triggers Unicode encoding — one emoji pushes the limit to 70. Flag any emoji
usage and confirm the sender's platform handles Unicode encoding correctly before including.

For each message, report: `[Channel] [N chars / M segments] [Encoding: GSM / Unicode]`

---

## Quick mode (single message)

1. Read the brief for: trigger event, goal, audience segment, offer, destination.
2. Apply the BRIEF checklist — draft the body first without the opt-out line.
3. Prepend the brand identifier if not baked into the sender ID (e.g., "PushEngage:").
4. Append the required opt-out line for the stated jurisdiction.
5. Count: total chars, segment count, encoding type.
6. Write one alternate on a *different angle* (different motivator, not a synonym).
7. Output the compliance checksheet.

**Quick-mode output format:**

```
## SMS / [Channel] — [campaign name or trigger]
Brand: [slug] | Jurisdiction: [US / EU / etc.]

**Recommended**
[Message body]
[Chars: N | Segments: M | Encoding: GSM]

**Alternate** (angle: [motivation])
[Message body]
[Chars: N | Segments: M | Encoding: GSM]

**Compliance checksheet**
- [ ] Opt-out language present and correct for jurisdiction
- [ ] Sender ID or brand name in message body
- [ ] Single CTA only
- [ ] No confirmed proof marked [verify]
- [ ] No emoji unless platform confirmed Unicode-safe
- [ ] Consent class matches send type (transactional vs. marketing)
```

---

## Sequence mode (2+ messages)

1. **Map the sequence skeleton** — event trigger → message 1 → delay → condition → message
   2 / exit. Show the skeleton before writing copy.
2. **Write each message** using BRIEF, maintaining tonal arc (welcome warmth → social proof
   → urgency → soft re-ask — not identical urgency throughout).
3. **Tonal arc rule:** message 1 is benefit-led; middle messages add evidence or urgency;
   final message is low-pressure + clear exit. Never increase aggression past message 2.
4. **Sunset branch:** after the final message, specify the action for non-responders —
   move to a lower-cadence segment, not continued blasting.
5. Call `lifecycle-email-push-copy-reviewer` on the full sequence before presenting.
6. Save the sequence to `./sms-whatsapp/[slug]-[campaign].md` if the user requests archival.

```
## Sequence: [name] — [brand slug]
Trigger: [event]
Jurisdiction: [X] | Consent class: [marketing / transactional]

M1 [delay: immediately]
[body] [chars / segments]

M2 [delay: +Xh / +Xd, condition: no reply / no click]
[body] [chars / segments]

...

Sunset: [action on no response after M_n]

Sequence compliance checksheet
- [ ] Opt-out language in every marketing message
- [ ] Escalating urgency capped at M2
- [ ] Sunset branch defined
- [ ] [verify] items flagged
```

---

## WhatsApp Business API template mode

WhatsApp templates require Meta pre-approval. Three categories:

| Category | Use case | Session window |
|---|---|---|
| **AUTHENTICATION** | OTP, verification codes | 24h from last user message |
| **UTILITY** | Order updates, delivery, transactional | 24h from last user message |
| **MARKETING** | Promotions, announcements, re-engagement | 24h from last user message; can initiate outside window |

Template output includes:
- **Category tag** (one of the three above)
- **Body text** with `{{variable}}` placeholders clearly named
- **Header** (text or media, optional)
- **Footer** (opt-out text — Meta enforces "Stop" button for MARKETING)
- **Button spec** (CTA button: type QUICK_REPLY or URL; text ≤25 chars; URL if applicable)

Mark any variables that will carry personal data (name, order ID) — these require your
privacy policy to cover their use under GDPR/CCPA.

---

## Compliance — non-negotiable rules

1. **Never write a marketing SMS/WhatsApp without confirmed opt-in.** If consent status is
   unknown, output a `[COMPLIANCE BLOCK]` and stop.
2. **Opt-out must be in every promotional message** — not just the first. TCPA/CTIA require
   it; CASL requires it; Spam Act 2003 requires it.
3. **Transactional messages** (order confirm, shipping update, password reset, OTP) do not
   require opt-out if they carry no promotional content. Any upsell line converts them to
   marketing — apply marketing rules immediately.
4. **Quiet hours (TCPA):** no marketing SMS before 8am or after 9pm in the recipient's local
   time zone [47 C.F.R. § 64.1200(c)]. Specify send-time rules in the sequence skeleton.
5. **Carrier filtering:** avoid all-caps, excessive punctuation, free/FREE, "WINNER," and
   URLs not from a registered short domain. Flag these in the compliance checksheet.
6. **RCS reputation:** Google's RCS spam scoring penalizes low-interaction senders — keep
   cadence conservative; flag any volume spike plan for carrier review.

---

## Principles

- **Brand-brain first.** No message before the brand digest is in hand. Voice and banned-
  words are hard overrides.
- **One action per message.** Two CTAs means zero conversions.
- **Compliance is part of the brief, not an afterthought.** Jurisdiction and consent class
  are inputs, not optional.
- **Brevity is craft.** If you can't land the message in ≤155 chars, the brief is unclear
  — flag it, don't bloat.
- **Honest proof only.** Real numbers from brand-brain or `proof-vault`; else `[verify]`.
- **Sequences need arcs, not repetition.** Varied motivation per message; sunset branch
  required.
- **Compose, don't duplicate.** CTA lines come from `cta-variant-generator`; review passes
  go to `lifecycle-email-push-copy-reviewer`.

---

## What not to do

- Do not write marketing copy before confirming opted-in consent — output `[COMPLIANCE BLOCK]`.
- Do not stack two CTAs in one message.
- Do not use emoji without flagging the Unicode encoding cost.
- Do not omit opt-out language from a promotional message, even in a sequence where M1 had it.
- Do not invent proof — mark unconfirmed numbers `[verify]`.
- Do not write the same angle across all messages in a sequence — vary motivation.
- Do not confuse transactional and marketing consent classes; any promo line = marketing.
- Do not implement brand scanning, voice analysis, or brand interviewing here — call brand-brain.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; voice + banned-words honored?
- Jurisdiction stated; correct opt-out language appended to every promotional message?
- Consent class identified; any ambiguity flagged as `[COMPLIANCE BLOCK]`?
- BRIEF applied: Brevity (≤155 GSM or correct limit), Relevance (trigger named), Intent
  (one action), Evidence (real proof or `[verify]`), Fallback (opt-out + sunset branch)?
- Character count and segment count reported per message with encoding type?
- Quick: recommended + one different-angle alternate?
- Sequence: tonal arc (benefit → evidence → urgency), sunset branch defined, reviewer called?
- WA template: category, variables named, button spec, Meta approval note included?
- No carrier-filter red-flags (all-caps, FREE, excessive punctuation)?
