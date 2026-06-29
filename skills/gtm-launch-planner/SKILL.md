---
name: gtm-launch-planner
description: >
  Product or feature brief + launch size (S/M/L) → launch plan with audience, channels,
  timeline, and go/no-go criteria. Works through a LAUNCH checklist (our acronym: Lock audience, Anchor
  message, Unify channels, Nail timing, Check readiness, Hand off) around three tiers:
  S (single-channel drop, < 2 weeks), M (coordinated multi-channel, 2–6 weeks), L
  (full-market program, 6+ weeks with staged rollout). Calls brand-brain for brand
  context, icp-persona-builder for segment sharpening, positioning-messaging-architect
  for the message hierarchy, and campaign-brief-builder for any paid channel. Exports
  a structured plan with timeline, RACI, and go/no-go gates. Use when the user says
  "plan a launch," "GTM for this feature," "launch checklist," "we're shipping X —
  how do we go to market," "build a GTM plan," or hands over a product brief and asks
  what to do with it.
---

# GTM Launch Planner

Launches fail from three root causes: wrong audience targeted, message that doesn't land, and channels that aren't ready when the product is. This skill eliminates all three. Give it a product or feature brief and a size (S/M/L) and it returns a launch plan a team can execute — audience, message, channels, timeline, go/no-go gates, and RACI — without reinventing brand context or message hierarchy from scratch.

This skill plans and coordinates. It does not write the channel copy itself — it calls the right specialist skills for that. If the brief reveals the product isn't ready to launch, it says so and proposes what has to be true first.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — voice, ICP, offer mechanics, proof, positioning.
- **`icp-persona-builder`** — sharpens segment + persona for the specific launch if the brief names a new or narrow audience.
- **`positioning-messaging-architect`** — message hierarchy and value prop for the launch if one doesn't exist in brand.md.
- **`campaign-brief-builder`** — structured brief for any paid channel included in the plan.
- **`campaign-concept-developer`** — big idea + hero message when the launch warrants a creative concept.
- **`cta-variant-generator`** — primary and secondary CTAs for each channel.
- **`lifecycle-journey-mapper`** — maps the post-launch activation sequence for product-led or freemium contexts.
- **`a-b-multivariate-test-designer`** — test brief for any launch variant being A/B'd.
- **`content-brief-builder`** — briefs for launch-day and warm-up content.
- **`pre-mortem-post-mortem-generator`** — pre-mortem for L-tier launches; post-mortem template scaffolded at plan export.

---

## How a run works

```
Step 0  Brand context     ──► call brand-brain (always first)
Step 1  Size the launch   ──► S / M / L (ask if not stated)
Step 2  Run LAUNCH        ──► Lock · Anchor · Unify · Nail · Check · Hand off
Step 3  Build the plan    ──► timeline, RACI, go/no-go gates
Step 4  Compose or call   ──► call specialists for deep channel work
Step 5  Export            ──► save to ./plans/[slug]-gtm-[date].md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Wait for its digest (voice, ICP, offer, proof, positioning, banned words) before producing any plan output. Obey voice and banned words as hard overrides; use only real proof (else `[verify]`).

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If neither exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (product · ICP · offer mechanics · 3 voice adjectives + banned words), then proceed.

### Step 1 — Size the launch

| Tier | Scope | Timeline | Typical signals |
|---|---|---|---|
| **S** | One channel, one message, one segment | < 2 weeks | Bug fix, minor feature, niche segment drop |
| **M** | 2–4 coordinated channels | 2–6 weeks | Feature release, pricing update, new segment entry |
| **L** | Full-market program, staged rollout | 6+ weeks | New product, major rebrand, market expansion |

If the user doesn't name a size, ask one question: "Is this a small drop, a coordinated multi-channel release, or a major market program?" then map the answer. Never pad a S launch into an M or L.

---

## The LAUNCH Checklist (our working acronym)

Each letter is a mandatory gate. For S-tier, answers can be one sentence. For L-tier, some gates require calling a specialist skill.

### L — Lock the audience

Who, specifically, is this for? One primary segment + one secondary (if applicable). Confirm this aligns with the brand's ICP from `brand-brain`. If the launch targets a new or narrower segment, call `icp-persona-builder`. Output: named segment + awareness stage + first touchpoint channel.

### A — Anchor the message

One hero message. What is the singular job-to-be-done this launch solves, expressed in the audience's language, not product language? Pull from `brand.md` if the positioning already covers it. If not, call `positioning-messaging-architect`. Derive a supporting message hierarchy (primary claim → proof point → differentiator → CTA direction). Output: hero message + 3-level hierarchy.

### U — Unify the channels

Which channels carry this launch, in what order, with what role? Assign each a role:

| Role | Purpose | Example |
|---|---|---|
| **Spearhead** | First contact, awareness/demand | Organic post, press drop, email blast |
| **Amplifier** | Extends reach after spearhead | Paid retargeting, partner mention, push |
| **Converter** | Closes the action | Landing page, in-app modal, pricing page |
| **Sustainer** | Keeps momentum post-launch | Nurture drip, SEO content, retargeting |

For any paid channel, call `campaign-brief-builder`. For CTAs at each converter touchpoint, call `cta-variant-generator`. For content at spearhead/sustainer nodes, call `content-brief-builder`.

### N — Nail the timing

Map the launch arc:

```
Pre-launch warm-up  →  Launch day  →  Amplification window  →  Sustain / handoff
```

- **S:** warm-up optional; launch day + 48h amplification.
- **M:** 1–2 week warm-up; launch day; 1–2 week amplification.
- **L:** 3–6 week warm-up (teasers, waitlist, partner seeding); launch day; 2–4 week push; sustain.

Anchor dates to business-context constraints (avoid major holidays, competing launches, billing cycles). For L-tier, call `pre-mortem-post-mortem-generator` now — a structured pre-mortem reduces launch-day surprises more than any checklist.

### C — Check readiness (go/no-go gates)

Hard gates that must pass before launch day. Flag any that are unresolved as blockers.

**Universal gates (all tiers):**
- [ ] Hero message approved by stakeholder with sign-off authority
- [ ] Landing / destination page live and converting (UTMs tested)
- [ ] Tracking in place: conversion event fires in GA4 or equivalent
- [ ] Legal / compliance review completed (if applicable)
- [ ] Support / CS briefed on launch scope and expected volume

**M/L additional gates:**
- [ ] Paid campaigns in review or approved; budgets confirmed
- [ ] Email sequence built, tested, and scheduled
- [ ] Partner or co-launch commitments confirmed in writing
- [ ] Rollback or hotfix plan documented if product-side issue surfaces

**L additional gates:**
- [ ] Staged rollout plan (% cohort or geo) defined with escalation criteria
- [ ] Press/media embargo lifted and coverage confirmed
- [ ] Executive comms approved and scheduled

### H — Hand off

Who owns what after launch? A launch with no owner on sustain dies by week two.

RACI template:

| Activity | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Launch-day post | | | | |
| Paid campaign management | | | | |
| Conversion monitoring | | | | |
| Weekly performance review | | | | |
| Post-launch content | | | | |

Fill roles from the user's team context. If unknown, label placeholders and flag.

---

## Plan output format

```markdown
# GTM Launch Plan — [Product/Feature Name]
**Brand:** [slug] | **Tier:** S/M/L | **Launch date:** [date] | **Owner:** [name]

## Target audience
[Segment · awareness stage · persona source]

## Hero message + hierarchy
[Message · proof · differentiator · CTA direction]

## Channel map
| Channel | Role | Owner | Ready? |
| ... |

## Timeline
| Phase | Dates | Key actions | Milestone |
| ... |

## Go/no-go gates
[Checklist — mark PASS / FAIL / OPEN]

## RACI
[Table]

## Downstream calls queued
[List any specialist skills called or to be called, with outputs]

## Post-launch: measure these
[3–5 KPIs with targets and measurement method]
```

Save to `./plans/[brand-slug]-[feature-slug]-gtm-[YYYY-MM-DD].md`. Never save to the skill folder or brand.md.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No plan output before `brand-brain` returns.
- **One hero message.** Multi-message launches are positioning failures, not strategy.
- **Size honestly.** Don't inflate a S into an M to look thorough; don't shrink an L to avoid the work.
- **Compose, don't rebuild.** Message hierarchy, channel copy, and paid briefs go to their specialist skills — this skill orchestrates and owns the plan structure only.
- **Gates are binary.** A gate is PASS or OPEN (blocker); there is no "probably fine."
- **Sustain has an owner.** Every launch plan exits with named accountability for post-launch momentum.
- **Real proof only.** Unconfirmed metrics in the plan are `[verify]`, not assertions.

## What Not to Do

- Don't write channel copy — call `cta-variant-generator`, `campaign-brief-builder`, `content-brief-builder`, etc.
- Don't reimplement brand scanning or message positioning — that's `brand-brain` and `positioning-messaging-architect`.
- Don't produce a plan before `brand-brain` returns and the launch size is confirmed.
- Don't save to brand.md or the skill folder; save to `./plans/`.
- Don't list 12 channels for an S-tier launch — right-size every recommendation.
- Don't skip the pre-mortem for L-tier launches; overconfidence is the most common GTM failure mode.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any plan output?
- Launch tier confirmed (S/M/L) and scope consistent with the tier?
- LAUNCH checklist completed: all six gates answered (one-liners acceptable for S)?
- Hero message singular, in audience language, anchored to brand positioning?
- Each channel assigned a role (Spearhead / Amplifier / Converter / Sustainer)?
- Specialist skills called or queued for channel copy, paid briefs, and personas?
- Go/no-go gates listed; blockers explicitly flagged OPEN?
- RACI populated or flagged for owner input?
- Post-launch KPIs named with measurement method?
- Plan saved to `./plans/` with brand-slug + date in filename?
