---
name: email-compliance-auditor-gdpr-can-spam
description: >
  Audits email copy, acquisition method, and sending region against the actual text of
  GDPR (Regulation (EU) 2016/679), CAN-SPAM Act (15 U.S.C. § 7704), CASL (Canada's
  Anti-Spam Legislation, S.C. 2010, c. 23), and PECR (UK Privacy and Electronic
  Communications Regulations 2003 / UK GDPR post-Brexit). Produces a per-regulation
  pass/fail checklist with specific statutory citations, a prioritized fix list (P0 = legal
  exposure / P1 = deliverability risk / P2 = best practice), and optionally writes a
  compliant rewrite of flagged copy elements.
  Inputs accepted: raw email copy (paste or file), sequence brief, acquisition method
  description (how the list was built), and target sending region(s).
  Call this skill whenever the user says "compliance check," "GDPR audit," "CAN-SPAM
  review," "is this legal to send," "check my opt-in / unsubscribe," "cold email legal,"
  "email disclaimer," or hands over an email draft and asks if it's safe to send.
  This skill reviews; it does not rewrite copy from scratch — call lifecycle-email-push-copy-reviewer
  for brand voice and flow, or subject-line-preview-text-optimizer for subject line craft.
---

# Email Compliance Auditor (GDPR / CAN-SPAM)

Every email that ships is a legal document. This skill audits it like one — against the actual statutes, not vibes. Pass it an email (or a sequence), tell it how the list was built and where it's going, and it returns a regulation-by-regulation pass/fail checklist with citations, a P0/P1/P2 fix queue, and optional compliant rewrites for flagged copy.

It does not write campaigns. It checks them. Use it as the last gate before you hit send, or as the first gate when a legal or ops team asks for evidence of due diligence.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, legal entity name, physical address, and country of incorporation; these anchor the physical-address check (CAN-SPAM § 5(a)(5)), the data-controller identity (GDPR Art. 13), and the sender-identification check (CAN-SPAM § 5(a)(1)). Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the legal entity name, physical mailing address, and country of operation before proceeding.
- **`lifecycle-email-push-copy-reviewer`** *(optional, when installed)* — hand off after the compliance pass to catch brand-voice and CTA issues separately; do not conflate legal review with copy critique here.
- **`sms-whatsapp-message-writer`** *(optional)* — when the audit surfaces TCPA/SMS-channel issues on a multi-channel sequence, note the gap and suggest routing to that skill.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (legal entity name, address, sender identity)
Step 1  Scope the audit     ──► determine regulation(s) from sending region + list origin
Step 2  Run the checklist   ──► per-regulation pass/fail with citations
Step 3  Prioritize fixes    ──► P0 (legal exposure) / P1 (deliverability) / P2 (best practice)
Step 4  Offer rewrites      ──► on request, produce compliant copy for flagged elements only
Step 5  Save artifact       ──► write audit to ./compliance/[slug]-email-audit-[YYYY-MM-DD].md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill to load the active brand. Pull: legal entity name, physical address, country of incorporation. These feed the sender-ID and physical-address checks. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the legal entity name, physical mailing address, and country of operation before proceeding.

### Step 1 — Scope the audit

Determine which regulations apply before touching the checklist. Ask if not provided:

| Signal | Regulation triggered |
|---|---|
| Sending to EU/EEA recipients | GDPR (Reg. EU 2016/679) |
| Sending to UK recipients | UK GDPR + PECR (SI 2003/2426) |
| Sending to US recipients | CAN-SPAM Act (15 U.S.C. §§ 7701–7713) |
| Sending to Canadian recipients | CASL (S.C. 2010, c. 23) |
| Cold / purchased / scraped list | Heightened scrutiny on all of the above |
| B2B "legitimate interest" claim | GDPR Art. 6(1)(f) must be documented |

Never assume the user's list is clean. If they can't describe the acquisition method (opt-in form, purchased, webinar sign-up, etc.), treat it as unknown and flag that as P0 for GDPR and CASL.

---

## The Compliance Checklist Framework (The ACID Test)

The audit runs four dimensions across every applicable regulation: **Acquisition legitimacy**, **Content requirements**, **Identity disclosure**, and **Departure rights** (unsubscribe / erasure). These form the mnemonic ACID — a useful mental model for your user, not marketing fluff.

### A — Acquisition legitimacy

**GDPR / UK GDPR (Art. 6, 7, Recitals 32, 47):**
- Was consent freely given, specific, informed, unambiguous, and documented? (Art. 7(1)) If not, what's the lawful basis (contract Art. 6(1)(b), legitimate interest Art. 6(1)(f))?
- For legitimate interest: was a three-part LIA (purpose test, necessity test, balancing test) conducted? Flag if no evidence.
- Pre-ticked boxes, bundled consent, and silence-as-consent all fail (Recital 32). Flag as P0.
- Is the consent request granular? (Art. 7(2)) Marketing vs. transactional must be separate consent.

**CAN-SPAM (§ 5(a)):**
- Commercial email has no opt-in requirement — but suppression list management is still a § 5(a)(3) obligation. Flag missing suppression management as P1.
- Transactional/relationship messages are exempt from most content rules but not the physical-address or sender-ID requirements.

**CASL (s. 6, 10):**
- Express consent required for commercial electronic messages (CEMs). Pre-checked boxes fail. Consent requests must state the purpose and identify the requester (s. 11).
- Implied consent (prior business relationship) expires 2 years after the last purchase (s. 10(9)(a)); or 6 months after inquiry (s. 10(9)(b)). Flag expired implied consent as P0 if the user cannot confirm date of last transaction.

### C — Content requirements

**CAN-SPAM (§ 5(a)(1)–(4)):**
- Subject line must not be deceptive (§ 5(a)(2)). Flag misleading subject lines (e.g., "Re:" / "Fwd:" if no prior thread) as P0.
- "From" name must accurately identify the initiating domain or advertiser (§ 5(a)(1)). Must match or relate to the sending domain.
- If the message is commercial but not explicit about it, a clear identifier is required (§ 5(a)(3)).
- FTC rules require "ADV:" prefixes in some jurisdictions [verify current FTC rule state]; flag as P2 to check.

**GDPR (Art. 13, 14):**
- First-contact emails where data was obtained not directly from the subject must disclose: controller identity, contact, purpose, legal basis, recipient categories, and retention period (Art. 14). Flag absence as P0.
- Profiling and automated decision-making must be disclosed if used (Art. 22). Flag any "personalized based on your behavior" language without disclosure.

**CASL (s. 6(2)):**
- Message must identify the sender and on whose behalf it's sent (s. 6(2)(a)).
- Must include contact information valid for 60 days (s. 6(2)(b)).
- Unsubscribe mechanism must be "readily performed" and honored within 10 business days (s. 6(2)(c), s. 11(3)).

### I — Identity disclosure

**CAN-SPAM (§ 5(a)(5)):**
- Physical postal address of the initiating domain or advertiser is mandatory. P.O. boxes are acceptable. No address = P0.
- The address must be a valid, deliverable address (not a defunct suite).

**GDPR (Art. 13(1)(a), Privacy Policy linkage):**
- Data controller name and contact details must appear or be accessible at the point of processing. Email footer linking to a GDPR-compliant privacy notice satisfies this; confirm the linked policy is current.

**CASL (s. 6(2)(a)–(b)):**
- Mailing address or link to a webpage with mailing address. Electronic address or phone number must be valid for 60 days after the message is sent.

### D — Departure rights (unsubscribe / erasure)

**CAN-SPAM (§ 5(a)(3), § 5(a)(6)):**
- Clear and conspicuous opt-out mechanism required (§ 5(a)(3)). "Conspicuous" = not hidden in small print or requiring login.
- Opt-out must be honored within **10 business days** (§ 5(a)(3)(B)). Charging for opt-out or requiring the subscriber to provide more than email address is prohibited.
- Suppression lists must be maintained indefinitely; no selling or transferring opted-out addresses (§ 5(a)(6)).

**GDPR (Art. 7(3), 17, 21):**
- Right to withdraw consent at any time, as easy as giving it (Art. 7(3)).
- Right to erasure ("right to be forgotten") must be honored within **30 days** (Art. 17). Flag any footer that says "you can't delete your account" or similar as P0.
- Right to object to processing based on legitimate interest (Art. 21(2)); must be offered in every marketing email.

**CASL (s. 11):**
- Unsubscribe must be "readily performed" via the same electronic means used to send (or another means that is just as easy).
- Must process unsubscribes within **10 business days** (s. 11(3)).

---

## Output format

```markdown
## Email Compliance Audit — [brand slug] — [YYYY-MM-DD]

**Regulations audited:** [GDPR | CAN-SPAM | CASL | PECR — based on scoping]
**Acquisition method:** [as described by user]
**Sending region(s):** [as described by user]

---

### ACID Dimension: Acquisition legitimacy
| Check | Reg | Citation | Status | Finding |
|---|---|---|---|---|
| Consent documented | GDPR | Art. 7(1) | ✅ PASS / ❌ FAIL / ⚠ UNCLEAR | [one-line finding] |

### ACID Dimension: Content requirements
[same table structure]

### ACID Dimension: Identity disclosure
[same table structure]

### ACID Dimension: Departure rights
[same table structure]

---

### Fix queue

**P0 — Legal exposure (fix before sending)**
1. [Specific fix + citation]

**P1 — Deliverability risk**
1. [Specific fix + citation]

**P2 — Best practice**
1. [Specific fix + recommendation]

---

### Compliant rewrites (requested elements only)
[before → after for any flagged copy elements, on request]
```

Save to `./compliance/[brand-slug]-email-audit-[YYYY-MM-DD].md`.

---

## Principles (Non-Negotiable)

- **Cite the actual rule.** Every finding names the regulation, section, and article number — never "you should probably have an unsubscribe." Lawyers and ops teams need citations.
- **P0 before you write.** Do not offer rewrites or style feedback until P0 findings are listed. Legal exposure is the priority.
- **Unknown = unclean.** If the user cannot describe the acquisition method, default to the strictest applicable standard and flag it as P0.
- **Jurisdiction determines the floor.** Apply every regulation triggered by the sending region — do not let a lenient US-only frame excuse GDPR gaps when EU recipients are in the list.
- **Real numbers, real citations.** Deadlines (10 business days, 30 days, 60 days, 2 years) come from the statutes, not general memory. Mark any that may have changed `[verify]`.
- **Auditor, not author.** Produce pass/fail findings and targeted rewrites for flagged copy only — do not rewrite the entire email or volunteer brand-voice critique; that belongs to `lifecycle-email-push-copy-reviewer`.

## What Not to Do

- Don't skip Step 0 (brand-brain) — the legal entity name and physical address are required inputs for the identity-disclosure checks.
- Don't audit as "US only" without asking about the list composition — a single EU subscriber in a CAN-SPAM-only audit creates GDPR exposure.
- Don't invent citations or deadlines from general knowledge; look up the statute reference and mark uncertainty as `[verify]`.
- Don't conflate compliance review with copy critique — route brand voice and CTA feedback to `lifecycle-email-push-copy-reviewer` separately.
- Don't issue a PASS on any P0 finding just because the email "looks professional" — formality is not a defense.
- Don't let a "legitimate interest" claim pass without asking whether an LIA was documented.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; legal entity name + physical address confirmed (or fallback explicitly applied)?
- Regulations scoped based on sending region AND list composition (not just the brand's home country)?
- All four ACID dimensions covered for each applicable regulation?
- Every finding cites a specific article or section number?
- P0 findings listed before rewrites or P1/P2 items?
- Pass/fail on each row is binary — no "probably fine" without a supporting citation?
- Artifact saved to `./compliance/` (not inside the skill folder)?
- `lifecycle-email-push-copy-reviewer` flagged as next step for brand-voice pass?
