---
name: transactional-email-copywriter
description: >
  Writes production-ready transactional email copy — order confirmations, password resets,
  receipts, shipping notices, trial expirations, account alerts, welcome triggers — from a
  trigger event spec and brand tone. Applies our FIRE working model (Functional clarity, Instructional
  precision, Reassurance microcopy, Emotional temperature-setting), a house framework, so every transactional email
  both delivers its payload AND advances the brand relationship. Works at the email level (subject,
  preheader, salutation, body blocks, CTA, footer) rather than the sequence level. On request,
  produces a full trigger-event matrix covering an entire product lifecycle. Does NOT manage brand
  context itself — calls `brand-brain` to load voice, banned words, and proof; calls
  `subject-line-preview-text-optimizer` for subject/preheader tuning; calls
  `email-compliance-auditor-gdpr-can-spam` for compliance review before marking work done.
  Use whenever the user says "write a transactional email," "order confirmation copy," "password
  reset email," "receipt template," "system email," "trigger email," "account alert," "write my
  shipping confirmation," "welcome email copy," or hands over a trigger spec and asks for the email.
---

# Transactional Email Copywriter

Transactional emails have an open rate 4–8× higher than marketing sends [verify exact multiple per ESP]. That makes them the most-read brand touchpoint most companies treat as an IT ticket.

This skill applies **FIRE**, our working model (a house framework, not an industry standard) — Functional clarity, Instructional precision, Reassurance microcopy, Emotional temperature-setting — to every trigger email so it fulfills its functional job AND moves the relationship forward. The copy is production-ready: structured into labeled blocks, character-counted where limits matter, compliance-flagged before handoff.

It writes individual emails, not full sequences. For multi-step drip or lifecycle sequences, compose this skill with `welcome-onboarding-email-sequence-builder`, `abandon-flow-writer`, or `usage-triggered-message-sequencer`.

---

## Skills this calls

- **`brand-brain`** (required, always first) — resolves the active brand's voice, banned words, offer mechanics, and proof. Does not implement brand scanning or storage here.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, 3 voice adjectives + banned words, and primary product/offer before proceeding.
- **`subject-line-preview-text-optimizer`** — call after drafting to sharpen the subject and preheader pairing (open-rate angle, preview truncation, mobile preview).
- **`email-compliance-auditor-gdpr-can-spam`** — call before finalizing; transactional emails have specific CAN-SPAM/GDPR treatment (transactional exemption has real limits; physical address, opt-out mechanics for marketing content embedded in transactional sends).
- **`cta-variant-generator`** — call for the primary CTA button when multiple angles are worth testing (e.g., upgrade nudge within a receipt email).
- **`proof-vault`** — call for reassurance microcopy referencing real proof points (e.g., "Trusted by 50,000 merchants [verify]").
- **`lifecycle-email-push-copy-reviewer`** — call for a final editorial pass on brand voice, flow, and CTA strength before handoff.

---

## How a run works

```
Step 0  Load the brand     ──► call brand-brain (mandatory before any copy)
Step 1  Classify the trigger ──► what type, what emotional moment, what functional payload
Step 2  Apply FIRE          ──► draft each labeled block
Step 3  Tune subject line   ──► call subject-line-preview-text-optimizer
Step 4  Compliance check    ──► call email-compliance-auditor-gdpr-can-spam
Step 5  Present & persist   ──► deliver copy, offer matrix mode on request
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics + URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`.

Obey the returned voice and banned-words as hard overrides. Use only real proof; mark anything else `[verify]`. Do not write a single line of email copy until `brand-brain` returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, 3 voice adjectives + banned words, and primary product/offer before proceeding.

---

## The FIRE framework

Every block in a transactional email serves one of four roles. Apply in order; don't skip any.

### F — Functional clarity
The email's primary job is information delivery. State what happened in the first two lines: action taken, outcome confirmed, and the key reference data (order number, amount, next step). No creative wind-up before the functional payload. If the reader has to search for the thing the email is about, the email has failed before the brand layer matters.

### I — Instructional precision
If the reader needs to do something (confirm email, reset password, track shipment, contact support), give the exact step — verb + destination + what to expect. One primary action. Button label leads with the verb + the specific outcome, not generic ("Reset my password" not "Click here"). When the action has a time constraint (reset link expires in 2 hours), state it plainly and early.

### R — Reassurance microcopy
Every transactional trigger carries latent anxiety: "Did my order go through?", "Was my account hacked?", "Will this work?" Reassurance microcopy addresses the most predictable anxiety for that trigger type in 1–2 short lines near the CTA or beneath key data. Use the brand's real proof or guarantees (from `proof-vault`). Mark unconfirmed claims `[verify]`.

| Trigger type | Predictable anxiety | Reassurance angle |
|---|---|---|
| Order confirmation | "Did it actually go through?" | Order ID + delivery estimate + contact hook |
| Password reset | "Was my account compromised?" | Expiry window + "if you didn't request this" line |
| Receipt / invoice | "Is this amount correct?" | Itemized breakdown + refund/dispute link |
| Shipping notice | "Where is it?" | Carrier + tracking link + expected window |
| Trial expiration | "Will I lose my data/work?" | Data persistence policy + upgrade path |
| Account alert | "Am I being hacked?" | "You (or someone) did X" framing + security link |
| Welcome / verify | "Is this legit?" | Sender trust signal + what happens next |

### E — Emotional temperature-setting
Transactional emails carry an emotional register whether you set it or not. Match it to the trigger moment. A password reset email is not the place for brand warmth — it's the place for speed and calm. An order confirmation is a moment of delight; the brand voice can show up there. A failed payment email is a moment of friction; the tone should be matter-of-fact and non-accusatory.

| Register | When | Voice guidance |
|---|---|---|
| Warm / celebratory | Order confirm, welcome, upgrade success | Brand voice at full volume; 1 congratulatory beat then pivot to functional |
| Calm / efficient | Password reset, email verify, 2FA | Brevity; minimal brand flavor; action in sentence 1 |
| Neutral / matter-of-fact | Receipt, invoice, account alert | Professional, no sentiment; pure functional + reassurance |
| Empathetic / low-friction | Failed payment, refund, cancellation confirm | Acknowledge the moment; do not lecture; give the exit or fix clearly |

---

## Output format — labeled blocks

Deliver every email in labeled blocks. Designers and ESP builders need discrete copy; don't embed it in prose.

```
## [Email type — trigger name]
Brand: [slug, via brand-brain]
Trigger: [what fires this email + send timing]
Emotional register: [from FIRE table]
Compliance note: [any flags from email-compliance-auditor, or "run auditor before deploy"]

### Subject line
[subject line copy]

### Preheader
[preheader copy — 85–100 chars; different from subject, extends it]

### Salutation
[Hi {{first_name}}, / Hello, / etc. — note personalization variable]

### Body block 1 — Functional payload
[The what + the key data: order ID, amount, link, etc.]

### Body block 2 — Instructional step
[The action, button label, time constraint if any]
[CTA button: "[label]" → [destination or variable]]

### Body block 3 — Reassurance
[1–2 lines addressing predictable anxiety; real proof or [verify]]

### Body block 4 — Emotional close / brand beat (optional)
[Brief on-brand sign-off — omit for calm/neutral register]

### Footer
[Required: physical address, unsubscribe note (if marketing content is embedded),
 support contact. Flag if transactional exemption applies or if CAN-SPAM compliance
 needs auditor review.]

---
Character counts: Subject [n] | Preheader [n] | CTA label [n]
Personalization variables used: [list]
Compliance flags: [list or "none — run email-compliance-auditor before deploy"]
```

---

## Trigger-event matrix mode (on request)

When the user asks for a full matrix ("all my transactional emails," "map out the triggers," "what emails does my product need") rather than a single email:

1. **Map the product lifecycle** against standard trigger categories: acquisition (welcome, verify), activation (onboarding milestones), transaction (confirm, receipt, shipping), account management (password reset, alerts, plan changes), retention (trial expiration, re-engagement), and support (refund, cancellation).
2. **For each trigger:** name, timing spec, emotional register, primary functional payload, reassurance angle, and whether a secondary marketing nudge is appropriate (e.g., cross-sell in receipt footer).
3. **Flag the five highest-impact triggers** (typically: welcome/verify, order confirm, receipt, password reset, trial expiration) for first-pass drafting.
4. **Draft in priority order** — apply FIRE to each, calling `subject-line-preview-text-optimizer` per email, running `email-compliance-auditor-gdpr-can-spam` once across the batch.

Save the matrix to `./transactional-emails/[slug]-trigger-matrix.md`; save individual email copy to `./transactional-emails/[slug]-[trigger-slug].md`.

---

## Transactional vs. marketing copy — the critical line

Transactional emails have different legal treatment under CAN-SPAM and GDPR than marketing sends. Key rules:

- **Transactional exemption (CAN-SPAM):** an email triggered by a customer's action (purchase, reset request) with a primary purpose of fulfilling that transaction is exempt from opt-out requirements — but if marketing content dominates, it loses that exemption. [Source: 15 U.S.C. § 7702(17)]
- **The "primary purpose" test:** if the marketing content (upsells, promotions) in the subject line or above-the-fold body would make a reasonable recipient see the email as primarily commercial, it's a commercial email, not transactional.
- **GDPR / legitimate interest:** sending transactional emails to existing customers under legitimate interest is generally defensible, but embedding marketing segments to third parties or re-using the address for new campaigns requires separate legal basis.
- Always run `email-compliance-auditor-gdpr-can-spam` before deploying. Flag any email embedding a cross-sell or upsell block for auditor scrutiny.

---

## Personalization variable discipline

- Use standard ESP variable syntax as a placeholder: `{{first_name}}`, `{{order_id}}`, `{{amount}}`, `{{reset_link}}`. State the variable name clearly; the user maps it to their ESP.
- Never write personalization variables without a fallback defined: `{{first_name | fallback: "there"}}` — note the fallback inline.
- Flag any variable that requires a segment or behavioral attribute beyond basic profile data as "requires ESP segment" in the compliance/variable block.

---

## Principles (Non-Negotiable)

- **brand-brain first.** No copy before `brand-brain` returns. Its voice + banned-words override every framework rule here.
- **Functional payload in the first viewport.** The reader should know exactly what happened before they scroll.
- **One primary action.** A transactional email with two CTAs is a transactional email that underperforms on both.
- **Honest time constraints.** If a link expires, say when. Never fake urgency.
- **Reassurance is not optional.** Every trigger has a predictable anxiety; address it.
- **Real proof only.** If you can't confirm the number from `proof-vault` or `brand.md`, mark it `[verify]`.
- **Compliance before deploy.** Call `email-compliance-auditor-gdpr-can-spam`; never self-certify.

## What Not to Do

- Don't write copy before `brand-brain` returns the active brand.
- Don't apply marketing-email register (hype, countdown urgency, benefit-stacking) to calm/neutral triggers like password resets or account alerts.
- Don't embed marketing content above the functional fold in a transactional send — the CAN-SPAM exemption depends on primary purpose.
- Don't invent proof, guarantees, or delivery estimates; mark anything unconfirmed `[verify]`.
- Don't merge two trigger types into one email — one trigger, one job.
- Don't deliver copy without labeled blocks; designers need the structure, not a prose paragraph.
- Don't skip the compliance flag; running the auditor is the sending team's risk gate.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any copy?
- Voice + banned-words honored; only real proof used (rest `[verify]`)?
- FIRE applied: functional payload in first viewport, one instructional action, reassurance line present, register matches trigger type?
- All output in labeled blocks with character counts and personalization variable list?
- `subject-line-preview-text-optimizer` called for subject + preheader?
- `email-compliance-auditor-gdpr-can-spam` called or compliance flag set for sending team?
- Marketing content in transactional send (if any) flagged for primary-purpose test?
- Personalization variables include fallbacks?
- If matrix mode: triggers mapped by lifecycle stage, top 5 flagged, files saved to `./transactional-emails/`?
