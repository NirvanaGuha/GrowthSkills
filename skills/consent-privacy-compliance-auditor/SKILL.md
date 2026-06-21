---
name: consent-privacy-compliance-auditor
description: >
  Audits forms, banners, pop-ups, data-collection touchpoints, and privacy policies against
  GDPR (EU 2016/679), CCPA/CPRA (California), PECR/ePrivacy, and CAN-SPAM/CASL where relevant —
  then produces a gap report, a dark-pattern flag list, and a draft data governance policy tailored
  to the brand's actual data flows. Operates from a structured compliance framework (the "Consent
  Audit Matrix") that maps every collection surface to the legal basis claimed, the data retained,
  and the disclosure required — so gaps are never just opinion, they're traceable to a specific
  article or regulation. Composes `email-compliance-auditor-gdpr-can-spam` for email-specific
  controls and `tracking-plan-taxonomy-builder-auditor` for tag/event data-flow mapping; does NOT
  reimplement those. Brand context (voice, legal entity name, jurisdiction) is loaded from
  `brand-brain` on every run. Use when the user says "GDPR audit," "CCPA review," "privacy gap
  report," "consent banner check," "dark patterns," "data governance policy," "cookie audit,"
  "privacy policy review," "opt-in compliance," or hands over a form, banner, or policy doc and
  asks whether it's legally sound.
---

# Consent & Privacy Compliance Auditor

Privacy compliance fails silently — until it doesn't. This skill runs a structured audit of every
data-collection surface and privacy disclosure against the actual text of GDPR, CCPA/CPRA, and
relevant email/ePrivacy laws, then hands back a gap report mapped to specific articles, a
dark-pattern flag list, and a ready-to-edit data governance policy. It cites real regulation. It
does not paper over genuine legal risk with reassuring copy.

This skill audits and drafts policy scaffolding. It is not a substitute for qualified legal counsel;
advise the user to have a licensed attorney review any policy before publishing.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's legal entity name, jurisdiction(s),
  data-collection practices, and any known compliance posture. Obey voice + banned-words; use only
  real proof (else `[verify]`). Fallback if brand-brain is absent or returns no brand: read
  `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user
  for their legal entity name, operating jurisdictions, primary data-collection surfaces, and email
  sending regions before proceeding.
- **`email-compliance-auditor-gdpr-can-spam`** (compose, not duplicate) — for email-specific
  opt-in, unsubscribe, sender-ID, and CAN-SPAM/CASL controls. Call this when the audit scope
  includes email capture forms or sending practices; fold its output into the gap report under
  "Email & Push Channel Controls."
- **`tracking-plan-taxonomy-builder-auditor`** (compose, not duplicate) — for tag/event data-flow
  mapping and consent signal propagation through GA4, GTM, Meta Pixel, etc. Call when the scope
  includes tracking pixels or analytics tags; fold its output into the gap report under "Analytics &
  Tag Data Flows."
- **`gtm-tag-builder-server-side-conversion-setup`** — reference for Consent Mode v2 implementation
  guidance if remediation of GTM/GA4 consent blocking is needed.
- **`form-friction-auditor`** — for UI-level friction analysis of opt-in forms (complementary; does
  not overlap on legal requirements).
- **`martech-stack-auditor-mapper`** — maps the full tool footprint; call when the user needs to
  understand what data is flowing where before auditing consent coverage.

---

## How a run works

```
Step 0  Load brand context        ──► call brand-brain (entity name, jurisdiction, data surfaces)
Step 1  Scope & inventory         ──► enumerate collection surfaces and map to regulation(s)
Step 2  Consent Audit Matrix      ──► surface × legal basis × disclosure × gap
Step 3  Dark-pattern scan         ──► pre-ticked boxes, nudge language, hidden opt-out, forced consent
Step 4  Compose siblings          ──► email-compliance (if in scope), tracking-plan-auditor (if in scope)
Step 5  Produce outputs           ──► Gap Report + Dark-Pattern Flags + Data Governance Policy draft
Step 6  Self-review               ──► every gap cites a real article; policy scaffolding is internally consistent
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Use the returned digest for:
the brand's legal entity name (needed in policy drafts), active jurisdictions (determines which
regulations apply), known data-collection surfaces, and any existing privacy posture. Do not begin
the audit until brand-brain returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for their legal entity name, operating
jurisdictions, primary data-collection surfaces, and email sending regions before proceeding.

### Step 1 — Scope and surface inventory

Enumerate every data-collection surface in scope. Standard surfaces to check:

| Surface | Typical data | Regulation hook |
|---|---|---|
| Cookie/consent banner | Device IDs, behavioral | GDPR Art. 6/7; ePrivacy; CCPA §1798.100 |
| Sign-up / lead-gen form | Email, name, company | GDPR Art. 13; CAN-SPAM §7704; CASL S.10 |
| Checkout / billing form | PII + payment data | GDPR Art. 13; CCPA §1798.100 |
| Newsletter / email opt-in | Email, preferences | GDPR Art. 7; CAN-SPAM; CASL S.10 |
| Push notification prompt | Device token + behavior | GDPR Art. 6(1)(a); PECR Reg. 6 |
| Analytics / pixel tags | Behavioral, cross-site | GDPR Art. 6; CCPA §1798.140(t) (sale/share) |
| Third-party integrations | Varies by tool | GDPR Art. 28 (DPA requirement); CCPA §1798.140(v) |
| Privacy policy / ToS page | Disclosure only | GDPR Art. 13/14; CCPA §1798.100(b) |

Ask the user to paste or link each surface in scope if not already provided.

---

## The Consent Audit Matrix (the core framework)

For every surface, populate one row. This is the deliverable skeleton — fill it from the user's
materials, then identify gaps against the cited articles.

```
| Surface | Data collected | Legal basis claimed | Disclosed in PP? | Consent mechanism | Regulation | Gap / Finding | Severity |
```

**Legal basis options (GDPR Art. 6):** Consent (Art. 6(1)(a)), Contract (6(1)(b)), Legal obligation
(6(1)(c)), Legitimate interests (6(1)(f)). Legitimate interests requires a balancing test (Art. 6(1)(f)
+ Recital 47); flag if the basis is asserted without a documented LIA.

**Severity tiers:**
- **Critical** — enforcement risk; fine or injunction probable (e.g., no valid consent for cookies,
  no unsubscribe mechanism, missing Art. 13 disclosures, CCPA "Do Not Sell" link absent).
- **High** — likely non-compliant; remediation required before next audit cycle.
- **Medium** — gap or ambiguity; fix recommended; lower immediate risk.
- **Low** — best-practice gap; document and monitor.

---

## Dark-Pattern Scan (GDPR EDPB Guidelines 3/2022)

After the Matrix, scan for the seven dark patterns identified in EDPB Guidelines 03/2022 on dark
patterns in social media platforms (generalized to all consent UIs):

1. **Overloading** — excessive prompts or information to nudge consent.
2. **Skipping** — design makes users skip privacy-friendly settings.
3. **Stirring** — emotional language or visuals to steer toward consent.
4. **Obstructing** — blocking navigation until consent is given (walls).
5. **Fickle** — inconsistent interface making it hard to withdraw consent.
6. **Left in the dark** — vague or absent information (missing purposes, no retention period).
7. **Pre-ticked boxes** — opt-in is pre-selected (explicitly invalid under GDPR Art. 7(2)).

For each pattern found: quote the exact UI text or describe the element, cite the EDPB section,
assign severity (Critical/High/Medium), and give a concrete remediation note.

---

## Output structure

### A. Gap Report

```
## Consent & Privacy Gap Report
Brand: [slug] | Jurisdiction(s): [list] | Audit date: [today]
Surfaces audited: [count]

### Critical findings ([count])
[Surface] — [gap description] — [Regulation: exact Article] — Remediation: [what to do]

### High findings ([count])
...

### Medium / Low findings ([count])
...

### Summary table
[Consent Audit Matrix — all rows]
```

### B. Dark-Pattern Flags

```
## Dark-Pattern Flags
Framework: EDPB Guidelines 03/2022
[Pattern name] — [Element] — [Quote or description] — Severity — [Remediation]
```

### C. Data Governance Policy Draft

A ready-to-edit plain-English policy scaffold — not boilerplate legalese, but the brand's actual
practices written accurately. Sections:

1. **Who we are** (legal entity, contact, DPO if required — GDPR Art. 13(1)(a))
2. **What we collect and why** (per-surface, legal basis for each — Art. 13(1)(c/d))
3. **How long we keep it** (retention periods — Art. 13(2)(a); no "as long as necessary" without specifics)
4. **Who we share it with** (processors, sub-processors, international transfers — Art. 13(1)(e),
   Art. 46; for US: CCPA §1798.140(v) service-provider language)
5. **Your rights** (access, rectification, erasure, portability, objection, restriction — Art. 15–21;
   CCPA §1798.100–145 rights; do-not-sell/share opt-out link if applicable)
6. **Cookies and tracking** (categories, purposes, consent mechanism)
7. **Contact and complaints** (supervisory authority right — Art. 77; AG contact for CCPA)

Mark every field the brand must fill in as `[FILL: description]`. Mark every item that needs legal
confirmation as `[legal review required]`. Do not invent retention periods, DPO details, or
sub-processor lists — leave them as `[FILL]` stubs.

Save the draft to `./privacy/[brand-slug]-data-governance-policy.md`.

---

## Regulation quick-reference (what this skill cites)

| Reg | Key obligation | Article |
|---|---|---|
| GDPR | Lawful basis required for processing | Art. 6 |
| GDPR | Consent must be freely given, specific, informed, unambiguous | Art. 7 |
| GDPR | Transparency notice at collection | Art. 13 |
| GDPR | Data subject rights | Art. 15–21 |
| GDPR | DPA required for processors | Art. 28 |
| GDPR | Transfers require safeguards | Art. 46 |
| CCPA/CPRA | Right to know, delete, opt-out of sale/share | §1798.100–135 |
| CCPA/CPRA | "Do Not Sell or Share" link required if applicable | §1798.135 |
| CCPA/CPRA | Service provider contract required | §1798.140(v) |
| ePrivacy/PECR | Prior consent for non-essential cookies | Reg. 6 (UK PECR); Dir. 2002/58/EC |
| CAN-SPAM | Physical address, unsubscribe mechanism, no deceptive headers | §7704 |
| CASL | Express or implied consent before sending; identify sender; unsubscribe | S.6, S.10 |

Note: GDPR fines up to €20M or 4% global annual turnover (Art. 83(5)); CCPA civil penalty up to
$7,500 per intentional violation [verify current figure]; CASL up to CAD $10M per violation
[verify current figure]. Always cite current enforcement figures from the relevant authority.

---

## Principles (Non-Negotiable)

- **Cite the article, not the vibe.** Every gap maps to a specific regulation article or section.
  Opinion without citation is not a compliance finding.
- **Brand-brain first.** Do not begin the audit before the active brand's entity name and
  jurisdictions are known — the applicable regulations differ.
- **Compose, don't duplicate.** Email-specific controls live in `email-compliance-auditor-gdpr-can-spam`;
  tag/event flows live in `tracking-plan-taxonomy-builder-auditor`. Call them, fold results in.
- **Flag real risk, not hypothetical.** Severity tiers are about enforcement probability, not
  theoretical reading.
- **Policy drafts are scaffolding.** Never present a generated policy as final or legally reviewed.
  Always include the disclaimer and `[legal review required]` stubs.
- **No invented facts.** Retention periods, sub-processor names, and DPO details are always `[FILL]`
  unless the user provides them.

---

## What Not to Do

- Don't produce a policy or gap report before brand-brain returns the entity name and jurisdictions.
- Don't reassure users that a practice "is probably fine" without a citation.
- Don't duplicate email-opt-in or tag-data-flow logic — compose the relevant sibling skills.
- Don't present a policy draft as legally sufficient — always include the legal-review caveat.
- Don't fabricate enforcement statistics — mark current fine levels `[verify]` if not confirmed.
- Don't treat "legitimate interests" as a free pass — flag if no LIA is documented.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and returned entity name + jurisdictions before audit began?
- Fallback line present and used if brand-brain was absent?
- Every gap finding cites a specific regulation article or section?
- Dark-pattern scan covers all 7 EDPB pattern types (not just pre-ticked boxes)?
- `email-compliance-auditor-gdpr-can-spam` and/or `tracking-plan-taxonomy-builder-auditor` composed
  where in scope, and their findings folded into the Gap Report?
- Policy draft uses `[FILL]` stubs for all brand-specific unknowns; `[legal review required]` on
  every legally consequential clause?
- Severity tiers assigned consistently (Critical = enforcement risk, not just any gap)?
- Saved to `./privacy/[brand-slug]-data-governance-policy.md` if policy draft was produced?
- Legal disclaimer included — "not a substitute for qualified legal counsel"?
