---
name: lead-nurture-drip-builder
description: >
  Builds multi-touch lead nurture drip sequences from persona, funnel stage, and content assets —
  producing email and push copy, send-timing logic, and conditional branch rules in one structured
  run. Accepts a persona description (or ICP notes), a declared funnel stage, and any available
  content assets (blog posts, case studies, feature pages, webinars, PDFs); outputs a ready-to-configure
  sequence with subject lines, preview text, body copy, CTA, send delay, and an explicit branch map
  covering the main engagement and non-engagement paths. Built on PASTA (Problem → Agitate →
  Solution → Trust → Action), our working drip model that extends the classic PAS copywriting
  structure, which keeps each email playing a distinct role
  in the decision journey instead of repeating the same pitch in different fonts. Loads the active
  brand from `brand-brain` before writing a single word; obeys voice + banned words as hard
  overrides; maps real proof through `proof-vault`; pulls CTA copy from `cta-variant-generator`.
  Saves sequences to ./sequences/ and offers an ESP-spec handoff for platform configuration.
  Use when the user says "build a nurture sequence," "write a drip for [persona],"
  "map content to a nurture flow," "set up lead nurture emails," "nurture drip from [funnel stage],"
  "what emails should I send between MQL and SQL," or hands over a persona and asks what to send next.
---

# Lead Nurture Drip Builder

Persona + funnel stage + content assets in. A complete, branch-mapped nurture sequence — with subject lines, body copy, timing, and engagement forks — out. Every email is on-brand and on-strategy because context comes from the brand brain, not guesswork.

This skill writes and structures sequences. It does not build ESP automations, manage list hygiene, or score leads. If those are needed, point to `esp-platform-builder` or `lead-scoring-routing-model-designer`.

---

## Skills this calls

- **`brand-brain`** (required) — resolves active brand; loads voice, banned words, ICP, offer/pricing, proof, positioning. No copy before this returns.
- **`proof-vault`** (optional) — surfaces real proof points (case studies, metrics, testimonials) to anchor trust emails. Synthesize inline when absent; mark unconfirmed numbers `[verify]`.
- **`cta-variant-generator`** (optional) — generates the CTA for each email's button/link. Call when the skill is installed; otherwise draft CTAs inline following the awareness-ceiling rule.
- **`icp-persona-builder`** (optional) — if no persona exists, call this first; pass its output as the persona input here.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (always first)
Step 1  Define the sequence scope
Step 2  Map content assets to PASTA roles
Step 3  Write the sequence (subject, preview, body, CTA, delay, branch logic)
Step 4  Self-review against brand + checklist
Step 5  Present + offer to save
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency. Do not write any email until it returns.

Obey voice and banned-words as hard overrides. Use only real proof (mark anything else `[verify]`). Anchor CTAs to the brand's real offer destinations.

Fallback if brand-brain is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If none exists, ask the user to install `brand-brain` or provide a 5-field mini-brief (product + ICP + voice adjectives + banned words + primary CTA destination).

### Step 1 — Define scope

Collect (from the user's message or by asking):

| Input | What to ask if missing |
|---|---|
| **Persona / ICP** | Who is this for? Role, company size, primary pain. (Or call `icp-persona-builder`.) |
| **Funnel stage** | MQL cold? Mid-funnel trial? Post-demo no-decision? Label it. |
| **Sequence goal** | Move to demo? Activate trial? Re-engage? Be explicit. |
| **Content assets** | URLs or titles of existing blog posts, case studies, webinars, PDFs to map in. |
| **Channel mix** | Email only? Email + push? (Push emails through `push-notification-copy-generator` if installed.) |
| **Sequence length** | Default: 5–7 touches; adjust to persona warmth + sales cycle length. |
| **ESP / platform** | Optional; affects token format, character limits, and spec output. |

If funnel stage is vague, use the Awareness Ladder (below) to calibrate.

---

## PASTA — our working drip model

PASTA is a house mnemonic, not an established framework — it extends the classic PAS (Problem-Agitate-Solution) copywriting structure with Trust and Action stages. Every email in the sequence has a declared role. Map content assets into these roles; don't stack two emails in the same role back-to-back.

| Role | Job in the sequence | Typical email # | Trigger to move on |
|---|---|---|---|
| **P — Problem** | Open the pain. Make the reader feel seen. No pitch. | 1 | Open / click |
| **A — Agitate** | Deepen the cost of inaction. Use a real proof story or stat. | 2 | Open / click |
| **S — Solution** | Introduce your product as the named solution to *that specific pain*. One angle only. | 3 | Click → key page |
| **T — Trust** | Social proof, case study, or comparison. Let someone else make the case. | 4 | Click → proof page |
| **A — Action** | Direct ask. Clear offer, friction-reducer, hard CTA. | 5 | Book / trial / upgrade |

For sequences longer than 5 emails: insert a second Agitate or Solution email between 2 and 3, or add a re-engagement nudge after the Action email. Never insert more than two consecutive emails in the same role.

### Awareness-ladder calibration

Match email tone and commitment ask to where the persona actually is, not where you wish they were (Schwartz stages):

| Funnel stage | Awareness | P email tone | Action email ask |
|---|---|---|---|
| Cold MQL | Problem- or solution-aware | Heavy on pain, zero product | Low commitment — "read this" |
| Mid-funnel / nurture | Solution-aware | Pain → category framing | Medium — "see how it works" |
| Post-demo / stalled | Product-aware | Outcome story | High — "let's pick a date" |
| Trial / freemium | Most-aware | Usage gap / value missed | Highest — "upgrade / activate" |

---

## Sequence output format

Present each email as a block. Deliver all emails before the branch map.

```
## Email [N] — [PASTA Role]
Send delay: [X days after email N−1, or trigger event]
Subject line: [subject]
Preview text: [preview]
Body:
[Opening line that earns the read]
[2–4 short paragraphs — PASTA role content]
[Transition to CTA]
CTA button: [label]  →  [destination URL from brand-brain]
Microcopy: [risk-reducer or expectation-setter, if applicable]
---
```

Keep emails short: 120–200 words body copy for cold/warm sequences; 80–150 for re-engagement. Every email has one CTA.

### Branch map

After all emails, output the branch map:

```
## Branch Map
Trigger: [enrollment event]
├── Email 1 sent
│   ├── OPENED / CLICKED → Email 2 on schedule
│   └── NOT OPENED (3 days) → Resend Email 1 with alt subject [provided], then Email 2
├── Email 2 sent
│   ├── CLICKED key asset → fast-track to Email 4 (skip Email 3)
│   └── NO CLICK → Email 3 on schedule
├── Email 5 (Action) sent
│   ├── CONVERTED → exit sequence, enroll in [onboarding / post-purchase flow]
│   └── NO RESPONSE (5 days) → sunset / re-engagement branch (Email 6 if applicable)
```

Adapt branches to the ESP / platform if specified. Flag any branch that requires a field or event the user hasn't confirmed exists.

---

## Content asset mapping

When the user provides existing assets, map each to a PASTA role:

| Asset type | Best PASTA role | Notes |
|---|---|---|
| Blog post (problem/pain) | P or A | Link as read-more; excerpt 2–3 lines in body |
| Case study / testimonial | T | Lead with outcome metric; proof-vault for real numbers |
| Feature page / demo | S | One feature angle per email — not a feature dump |
| Comparison page / vs competitor | T or S | Use only if persona is solution-aware or higher |
| Webinar / video | S or T | Clip the key 2-minute takeaway; link to full recording |
| Pricing page | A (final Action email only) | Never link to pricing before Trust email |

If an asset slot has no content: draft a native email that accomplishes the role without a linked asset, and flag it as `[asset gap — consider creating this]`.

---

## Timing and cadence defaults

These are starting points. Adjust to the product's sales cycle and persona warmth.

| Sequence type | Email spacing | Total window |
|---|---|---|
| Cold MQL (outbound-sourced) | Days 1, 4, 8, 13, 19, 26 | ~4 weeks |
| Mid-funnel / inbound | Days 1, 3, 6, 10, 15 | ~2 weeks |
| Post-demo stall | Days 1, 3, 7, 14 | 2 weeks |
| Trial / freemium activation | Days 1, 2, 4, 7, 14 | 2 weeks |
| Re-engagement (60-day dormant) | Days 1, 5, 12 | 2 weeks; sunset after 3 no-responses |

Never send two emails within 24 hours unless the persona is in an active trial and a behavioral trigger fires.

---

## Principles

- **Brand-brain first.** No copy before the brand digest returns. Voice and banned-words are non-negotiable overrides.
- **One role per email.** Each email does one job in the PASTA sequence. No blended pitch-plus-proof emails.
- **Vary motivation, not volume.** Different emails address different objections — not the same pitch reworded.
- **Awareness ceiling.** Never ask for more commitment than the funnel stage supports. Cold personas get curiosity-first emails; post-demo personas get direct asks.
- **Real proof only.** Every stat, case study name, or outcome metric must come from `proof-vault` or be marked `[verify]`. Never invent proof.
- **Branch logic is mandatory.** A sequence without engagement branches is an untested batch send, not a drip. Always output the branch map.
- **Asset gaps are honest.** Flag missing content; don't pretend an asset exists or make up a vague "resource."

## What Not to Do

- Don't write any email before `brand-brain` returns. Don't reimplement brand scanning here.
- Don't stack the same PASTA role twice in a row.
- Don't exceed the awareness ceiling — no "start your free trial" in Email 1 to a cold MQL.
- Don't invent proof, case study names, or metrics. Mark everything unconfirmed `[verify]`.
- Don't build the ESP automation workflow here — that belongs in `esp-platform-builder`.
- Don't write a sequence without a branch map.
- Don't use the same subject-line pattern (e.g., question + name) in more than two consecutive emails.
- Don't save anything inside the skill folder; save to `./sequences/` in the project directory.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest loaded (or bootstrapped) before any copy?
- Voice adjectives honored; banned words absent; all proof from `proof-vault` or `[verify]`?
- Each email assigned exactly one PASTA role; no two consecutive emails in the same role?
- Subject lines: distinct patterns, no near-duplicates; preview text adds information (not a repeat)?
- Every email has one CTA, pointing to a real brand destination URL?
- Awareness ceiling respected per funnel stage — no commitment overshoot in early emails?
- Branch map covers: engagement (open/click), non-engagement (resend / skip), conversion exit, and sunset?
- Asset gaps flagged where no content asset fills the slot?
- Sequence length appropriate to the sales cycle and persona warmth?
- Offer to save to `./sequences/[brand-slug]-[persona-slug]-nurture.md`?
