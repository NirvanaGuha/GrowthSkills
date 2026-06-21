---
name: partner-onboarding-checklist-generator
description: >
  Takes a partner type (affiliate, co-marketing, integration/tech, reseller, or influencer) plus
  product and deal details, and produces a step-by-step onboarding checklist with owner assignments,
  milestone gates, and a kickoff brief — so the first 30 days are structured, nothing falls through
  the cracks, and the partner is revenue-active as fast as possible. Builds on the MAPS framework
  (Milestones → Assets → People → Systems) to ensure every onboarding covers the four dimensions
  that determine partner time-to-value. Composes brand-brain for voice/ICP, sales-partner-collateral-creator
  for talk tracks and one-pagers, and co-marketing-sponsorship-campaign-planner for joint launch
  activities — rather than rebuilding those from scratch. Saves a reusable checklist artifact per
  partner type. Use when the user says "onboard a partner," "partner checklist," "affiliate setup,"
  "integration onboarding," "new reseller," "partner kickoff," "what do we need to activate
  a partner," or hands over a partner deal brief and asks what happens next.
---

# Partner Onboarding Checklist Generator

Most partner relationships die in the first 30 days — not from a bad deal, but from a missing asset, an unassigned owner, or a system never provisioned. This skill structures the gap between "deal signed" and "partner actively driving revenue" using the **MAPS framework**: Milestones, Assets, People, Systems. Every output is checklist-first, owner-assigned, and gated at the moments that actually predict partner activation.

This skill generates the checklist, kickoff agenda, and launch readiness gate. It does not write the collateral, build the co-marketing plan, or produce the talk tracks — it delegates those to purpose-built siblings.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, ICP, offer mechanics, and proof so every partner-facing asset and communication is on-brand. Does not reimplement brand resolution. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, core offer/pricing, and 3 voice adjectives before proceeding.
- **`sales-partner-collateral-creator`** — called in Phase 2 (Assets) to produce one-pagers, co-sell talk tracks, and Loom pitch script for the partner's reps. Compose, don't duplicate.
- **`co-marketing-sponsorship-campaign-planner`** — called for co-marketing partner types to plan joint launch activities, asset split, and shared calendar.
- **`campaign-brief-builder`** — called when the partner needs a launch campaign brief to take to their audience.
- **`lifecycle-journey-mapper`** — optional; used when the partner's end-customer journey needs mapping for integration or tech partners.
- **`sop-builder-reviewer`** — optional; called to produce a repeatable SOP for the onboarding process itself (useful when the same partner type recurs).

---

## How a run works

```
Step 0  Load the brand         ──► brand-brain (always first)
Step 1  Classify the partner   ──► affiliate | co-marketing | integration/tech | reseller | influencer
Step 2  Collect inputs         ──► 5 questions if not already provided
Step 3  Build the MAPS checklist
Step 4  Generate the kickoff brief + launch gate
Step 5  Delegate collateral/campaign tasks to siblings
Step 6  Save artifact
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Wait for the returned digest before producing any partner-facing content. Obey voice + banned-words; use only real proof (mark anything unconfirmed `[verify]`).

**Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, core offer/pricing, and 3 voice adjectives before proceeding.**

### Step 1 — Classify the partner

Identify the partner type from the request or ask. The type governs which MAPS lanes are critical:

| Type | Critical lanes | Typical TTV |
|---|---|---|
| **Affiliate** | Assets (links, creatives) + Systems (tracking, payout) | 7–14 days |
| **Co-marketing** | Milestones (joint campaign) + People (counterpart contacts) | 14–21 days |
| **Integration / Tech** | Systems (API keys, sandbox, webhook) + Assets (listing copy) | 21–30 days |
| **Reseller / VAR** | People (rep training) + Assets (sales kit) + Systems (portal access) | 14–21 days |
| **Influencer / Creator** | Assets (brief, links, codes) + Milestones (content deadlines) | 7–10 days |

### Step 2 — Collect inputs (if not supplied)

Ask only what's missing. Standard inputs:

1. **Partner name + type** — e.g. "Acme Widgets, affiliate"
2. **Deal terms summary** — commission rate, rev-share, joint offer, or integration scope
3. **Assigned internal owner** — the single accountable person on your side
4. **Partner's primary contact** — name, role, preferred channel
5. **Target activation date** — when the partner should be live and driving revenue

If the user provides a deal brief or contract summary, extract these fields from it rather than asking.

---

## The MAPS Framework

Every checklist is organized across four dimensions. Populate each lane based on the partner type, then assign owners and set gate criteria.

### M — Milestones

The sequenced events that define "activated." Each milestone must be binary (done / not done) and assigned to a single owner. Standard milestones by type:

- **All types:** Agreement signed → Kickoff call → System access granted → First live asset → First tracked conversion/action
- **Co-marketing add:** Joint campaign brief approved → Co-branded asset signed off → Launch date confirmed
- **Integration add:** Sandbox credentials provisioned → Integration test passed → Production credentials issued → Listing published
- **Reseller add:** Rep training completed → Deal registration system activated → First opportunity logged

### A — Assets

The collateral, links, and creative the partner needs to activate. Gap here is the most common reason a partner goes dark.

- **Affiliate:** Tracking links, creatives (banner sizes, email swipes), offer landing page URL, discount/coupon code, editorial guidelines
- **Co-marketing:** Co-branded one-pager, joint landing page copy, email copy, social assets, partner logo + brand guide
- **Integration:** API documentation link, sandbox credentials, webhook setup guide, marketplace listing copy + screenshots, support escalation path
- **Reseller:** Sales deck (partner-branded), battlecard, pricing/discount matrix, demo environment access, customer-facing one-pager
- **Influencer:** Creator brief (UGC Brief Writer handles this), unique tracking link, product samples/access, posting schedule, FTC disclosure language

**Flag to call `sales-partner-collateral-creator`** for talk tracks + one-pager if not already built. Flag to call `co-marketing-sponsorship-campaign-planner` for joint campaign planning.

### P — People

The human connections that make a partnership work. Missing an escalation contact or skipping rep training is a quiet killer.

- Internal owner (single accountable; not a committee)
- Partner primary contact + technical contact (if integration type)
- Your support/success contact for the partner
- Executive sponsor (for strategic/enterprise partners)
- Training schedule and completion confirmation for reseller reps

### S — Systems

Every platform, credential, and integration the partner needs provisioned before Day 1.

- **All types:** Partner portal or shared workspace access (Notion page, Slack channel, shared Drive)
- **Affiliate:** Affiliate platform provisioned (Impact, ShareASale, PartnerStack, or custom), payout method configured, tracking test completed
- **Integration:** Sandbox API keys, production API keys (after testing), webhook endpoint registered, rate-limit acknowledgment
- **Reseller:** CRM deal-registration portal, partner discount tier configured, demo environment
- **Co-marketing:** Shared campaign tracking UTMs built, shared content calendar, joint Slack channel

---

## Output format

### Checklist artifact

```markdown
# Partner Onboarding Checklist — [Partner Name] ([Type])
Brand: [slug] | Owner: [name] | Target activation: [date]

## Phase 1 — Agreement & Kickoff (Days 1–3)
- [ ] [Task] — Owner: [name] — Due: Day [N]
- [ ] ...
**Gate:** Kickoff call complete + deal terms confirmed in writing

## Phase 2 — Assets & Access (Days 3–10)
- [ ] [Task] — Owner: [name] — Due: Day [N]
- [ ] Call `sales-partner-collateral-creator` → one-pager + talk track for [partner reps / partner's audience]
- [ ] ...
**Gate:** All assets delivered + system access confirmed by partner

## Phase 3 — Training & Test (Days 10–20)
- [ ] ...
**Gate:** [type-specific: tracking test passed | rep quiz score ≥80% | integration test passed]

## Phase 4 — Launch & First Conversion (Days 20–30)
- [ ] ...
**Gate:** First tracked conversion / first attributed action / first content post live

## Ongoing Cadence
- [ ] [30/60/90-day check-in cadence] — Owner: [name]
- [ ] Refresh assets each quarter — Owner: [name]
```

### Kickoff brief (inline, ~150 words)

Agenda for the kickoff call: intro, deal terms review, asset handoff plan, system access timeline, first milestone target, escalation contacts, cadence agreement.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No partner-facing copy, messaging, or asset before `brand-brain` returns. Its voice + banned-words override everything.
- **One owner per task.** Shared ownership is no ownership. If the user hasn't named an owner, call it out — don't silently leave it blank.
- **Gates are binary.** A gate is a hard stop (pass/fail), not a progress percentage. If a gate can't be tested, rewrite it until it can.
- **Collateral is delegated, not duplicated.** Call the right sibling skill for talk tracks, co-marketing plans, and launch briefs. Don't rebuild that work here.
- **TTV is the north-star metric.** Every task in the checklist either directly reduces time-to-value or it should be removed.
- **Unconfirmed numbers are `[verify]`.** Realistic activation windows can differ wildly; never fabricate a benchmark.

## What Not to Do

- Don't write talk tracks, email sequences, or co-marketing plans here — call `sales-partner-collateral-creator` and `co-marketing-sponsorship-campaign-planner`.
- Don't produce any partner-facing copy before `brand-brain` returns.
- Don't leave any task without an owner assignment (even if the user must fill it in).
- Don't conflate onboarding with ongoing partner management — this skill covers Day 0 to activation gate; flag cadence items but don't build a full partner management program.
- Don't generate a generic checklist ignoring partner type — the MAPS lanes that matter vary significantly; adapt every output.
- Don't skip the launch gate — a checklist without a binary pass/fail condition is a wishlist.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback applied) before any output?
- Partner type classified and the right MAPS lanes prioritized?
- All five inputs collected (partner name/type, deal terms, internal owner, partner contact, activation date)?
- Every task in the checklist has a single named owner and a due day?
- Every phase ends with a binary gate criterion?
- `sales-partner-collateral-creator` flagged for talk tracks/one-pager where relevant?
- `co-marketing-sponsorship-campaign-planner` flagged for co-marketing partner types?
- Checklist saved to `./partners/[partner-slug]-onboarding.md`?
- TTV to activation realistic for the partner type; any benchmarks marked `[verify]`?
