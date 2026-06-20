---
name: campaign-brief-builder
description: >
  Turns a business goal, audience, budget, and channel mix into a structured, ready-to-execute
  campaign brief — covering objective (SMART), audience targeting, budget allocation, channel
  strategy, creative spec, KPIs, and a completeness review. It does NOT manage brand context
  itself — it calls the `brand-brain` skill first so every brief is on-voice, on-positioning,
  and uses the right ICP and offer. Two modes: Full Brief (default) — produces the complete
  brief document; and Quick-Scope (on request or when inputs are sparse) — produces a scoped
  one-pager to align stakeholders before full build. Optional: chains to `ad-copy-variant-generator`
  for creative copy and `campaign-qa-launch-checklist-generator` for the pre-launch gate.
  Use when the user says "write a campaign brief," "build a campaign plan," "I need a brief for
  this campaign," "scope this campaign," "help me plan a paid campaign," "what should this
  campaign look like," or provides a goal + budget + channel and needs it structured.
---

# Campaign Brief Builder

A brief is a contract — between the campaign and the business goal, between the team and the
budget, between creative and audience. This skill produces that contract. It does not produce
vague campaign "plans" full of generic best practices. It produces a filled-in brief with
a stated objective, a real audience definition, a justified budget split, channel-specific
creative specs, and measurable KPIs — grounded in the brand's real positioning and ICP.

Every brief is a downstream artifact from brand context. A brief written without brand context
is a template exercise. This skill calls `brand-brain` first so the objective, message framing,
offer mechanics, and proof are drawn from the live brand, not invented on the spot.

---

## Skills this calls

| Skill | When |
|---|---|
| **`brand-brain`** | Always first — loads voice, ICP, offer/pricing, proof, positioning, banned words |
| **`icp-persona-builder`** | When the persona for the target segment is thin or absent in `brand.md` |
| **`offer-pricing-brain`** | When the offer or promotion mechanic needs to be defined or validated |
| **`competitive-intelligence-dossier`** | When competitive context is relevant to positioning in the brief |
| **`ad-copy-variant-generator`** | To populate the Creative Spec section with actual headlines/copy variants |
| **`cta-variant-generator`** | To populate primary CTA + landing page CTA fields in the brief |
| **`campaign-qa-launch-checklist-generator`** | After the brief is done — generates the pre-launch QA gate from it |
| **`utm-parameter-bulk-builder`** | To pre-build tracking URLs for all placements in the brief |
| **`ltv-cac-payback-calculator`** | When budget justification requires a CAC ceiling or LTV-to-CAC target |

---

## How a run works

```
Step 0  Load the brand   ──► call brand-brain (always first)
Step 1  Pick the mode    ──► Full Brief (default) | Quick-Scope (sparse inputs / explicit request)
Step 2  Clarify gaps     ──► ask only what's missing to fill the framework
Step 3  Build the brief  ──► GSOT + audience + budget + channels + creative spec + KPIs
Step 4  Self-review      ──► completeness check + offer-weakness flag
Step 5  Offer next steps ──► ad-copy-variant-generator, QA checklist, UTM builder
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before writing a single
brief field. It returns the active brand's digest: voice, banned words, ICP + awareness
tendency, offer mechanics + destination URLs, real proof, and positioning. If the brand is
new, `brand-brain` bootstraps it first; do not start the brief until it returns.

Obey voice and banned-words throughout. Use only the returned proof (mark anything else
`[verify]`). Anchor the objective and offer language to the brand's actual positioning, not
to generic marketing claims.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none, ask the user to install `brand-brain` or answer a
4-question mini-setup before proceeding.

---

## Full Brief mode (default)

Build the brief using the **GSOT framework** (Goal → Strategy → Objective → Tactic) as the
structural spine, expanded into eight sections. GSOT prevents the common failure where a
campaign has tactics but no coherent objective, or an objective with no traceable connection
to a business goal.

### Section structure

```
## Campaign Brief — [Campaign name / slug]
Brand: [slug, via brand-brain]     Date: [today]     Version: 1.0

### 1. Goal (business context)
[One sentence: the business-level outcome this campaign serves.
 Must connect to revenue, pipeline, retention, or activation —
 not "increase brand awareness" unless that's the explicit brief.]

### 2. Objective (SMART)
[Specific · Measurable · Achievable · Relevant · Time-bound.
 Format: "Achieve [metric] of [number] by [date] with [budget]."
 If the user has not set a KPI number, surface a benchmark or mark [verify].]

### 3. Audience
| Field | Value |
| Segment | [persona name / ICP tier from brand-brain] |
| Awareness stage | [Schwartz: unaware / problem / solution / product / most-aware] |
| Platform behavior | [where they actually are; relevant to channel choice] |
| Key pain / desire | [one line from ICP; the lever the campaign pulls] |
| Excluded audiences | [who NOT to reach — suppression lists, existing customers if acquisition, etc.] |

### 4. Offer & Messaging
| Field | Value |
| Offer mechanic | [from offer-pricing-brain / brand-brain: trial, discount, demo, content lead] |
| Primary message | [single governing message — the one thing the audience must believe] |
| Proof point(s) | [real proof from brand-brain; else [verify]] |
| Primary CTA | [from cta-variant-generator or inline] |
| Landing destination | [specific URL; not "the website"] |

### 5. Budget & Allocation
| Channel | Allocation % | Budget $ | Rationale |
| [Channel 1] | | | |
| [Channel 2] | | | |
| Total | 100% | [total] | |
[Note CAC ceiling if ltv-cac-payback-calculator data is available.
 If budget is not given, ask — do not invent one.]

### 6. Channel Strategy
For each channel in the budget table:
| Channel | Objective | Format | Bid strategy | Targeting levers | Flight dates |
| | | | | | |
[Be specific: "Meta Advantage+ Shopping with 7-day-click 1-day-view attribution"
 beats "run Meta ads." If the channel is not in the brand's known toolkit, flag it.]

### 7. Creative Spec
| Placement | Format | Size / Length | Headline | Primary CTA | Visual direction |
| | | | | | |
[Fill headline + CTA fields only after calling ad-copy-variant-generator /
 cta-variant-generator, or note "to be populated in creative brief."
 Include character limits per platform (Google: 30-char headline; Meta: 255-char primary;
 LinkedIn: 150-char intro). Never exceed them.]

### 8. KPIs & Measurement
| KPI | Type | Target | Measurement method | Attribution |
| Primary conversion | Lagging | [number] | [GA4 event / pixel / CRM] | [window] |
| Secondary signal | Leading | | | |
| Efficiency metric | Unit econ | [CPA / ROAS target] | | |
[Encode the attribution window explicitly — "7-day click, 1-day view" not just "last-click."
 Flag if the measurement method requires setup before launch.]

### 9. Dependencies & Risks
- [ ] [Creative assets not yet produced — blocks launch]
- [ ] [Landing page CRO / UTM setup]
- [ ] [Pixel / conversion event verified in platform]
- [Any offer, legal, or copy approval dependency]

### Completeness review
[A one-paragraph verdict: what is solid, what is missing or assumed,
 what must be resolved before the brief goes to execution.]
```

---

## Quick-Scope mode

Triggered when inputs are genuinely sparse ("I'm thinking about a campaign for X, can you help
me scope it?"), when the user explicitly asks for a one-pager, or when a Full Brief would be
premature before stakeholder alignment.

Produce a condensed scoping document:
- **What:** one-line campaign concept
- **For whom:** audience segment + awareness stage
- **Why now:** business reason / trigger event
- **How:** 2–3 channel candidates with a brief rationale each
- **Success looks like:** one primary KPI with a realistic benchmark
- **Open decisions:** the 3–5 questions that must be answered before a Full Brief can be written

Keep it under one page. Offer to build the Full Brief once the open decisions are answered.

---

## Framework note: GSOT + Schwartz awareness

GSOT (Goal → Strategy → Objective → Tactic) is the structural spine. It prevents two
failure modes: campaigns that have tactics with no traceable business goal, and campaigns
with a goal but no SMART objective to hold them accountable.

Schwartz awareness stages govern message and channel selection in sections 3–4 and 7. A
campaign aimed at problem-aware buyers should not use conversion-optimized copy or
bottom-of-funnel CTAs — that mismatch is one of the most common reasons briefs produce
low-quality leads. Always surface the awareness stage explicitly; do not leave it implied.

| Awareness stage | Right message angle | Right channel/format |
|---|---|---|
| Unaware | Category problem framing | Broad social, display, content |
| Problem-aware | Education, benchmarks, cost of inaction | Search (informational), social content |
| Solution-aware | Comparison, differentiation, social proof | Search (comparison), retargeting |
| Product-aware | Offer, risk-reversal, proof | Retargeting, email, branded search |
| Most-aware | Incentive / urgency | Email, high-intent search, cart recovery |

---

## Principles

- **Brand-brain first.** No brief field before the brand digest is loaded. Voice, banned
  words, and ICP override user instinct.
- **SMART objectives are non-negotiable.** A brief with a vague objective ("increase
  awareness") is not a brief. Push the user toward a measurable target or surface a
  benchmark; mark it `[verify]` if unconfirmed.
- **Specific beats template.** "Meta Advantage+ Shopping, 7-day click attribution, $80 CPA
  target" is a brief. "Run paid social" is not.
- **Attribution window is a first-class field.** Briefs that omit the attribution window
  produce campaigns that can't be evaluated consistently.
- **Weak offer blocks everything.** If the offer won't close at the stated awareness stage,
  flag it before filling in the creative spec — don't build a tower on a bad foundation.
- **Compose, don't reinvent.** ICP → `icp-persona-builder`. Creative copy → `ad-copy-variant-generator`.
  CAC math → `ltv-cac-payback-calculator`. Call them; don't reimplement.

## What Not to Do

- Don't produce a brief before `brand-brain` returns the active brand.
- Don't invent a budget if the user hasn't provided one — ask.
- Don't write generic "best practice" channel advice detached from the brand's audience.
- Don't omit the attribution window or leave KPIs as qualitative ("improve conversions").
- Don't pad the brief with strategic preamble that doesn't appear in an execution field.
- Don't confuse a campaign brief with a content brief — if the user needs a content brief,
  route to `content-brief-builder`.

## Quality Checklist

- [ ] `brand-brain` called and active brand loaded before any field is written?
- [ ] Objective is SMART with a stated metric, number, and date?
- [ ] Audience section includes awareness stage (Schwartz) and an exclusion criterion?
- [ ] Offer is real (from brand-brain / offer-pricing-brain) with a named destination URL?
- [ ] Budget table sums to 100%; rationale given per channel?
- [ ] Every channel has a bid strategy and a flight date — not just a name?
- [ ] Creative spec includes character limits; headline/CTA populated (or explicitly deferred)?
- [ ] Attribution window stated explicitly in the KPI table?
- [ ] Completeness review flags any missing inputs or unresolved decisions?
- [ ] Voice + banned words honored throughout; all unconfirmed proof marked `[verify]`?
- [ ] Brief offered for save to `./briefs/[slug]-campaign-brief.md`?
