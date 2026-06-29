---
name: upgrade-expansion-prompt-writer
description: >
  Turns usage signals into upgrade revenue. Give it a trigger event (feature-limit hit,
  usage spike, plan expiry approaching, expansion signal like a second seat added) plus
  the plan limits and what the next tier unlocks — and it returns: (a) a battery of upgrade
  modal copy variants (headline + body + CTA + microcopy), (b) a paywall-moment copy spec
  that replaces a hard block with a contextual sell, and (c) a prioritized trigger spec
  (which event fires which prompt, delay/timing logic, suppression rules, and success
  metric per trigger). Organized around our Moment-Fit Expansion model (a working framework
  used in this skill) — the working principle that expansion prompts succeed or fail based on
  timing precision and benefit specificity, not persuasion intensity. Every asset is written in the brand's live voice via brand-brain and
  composed with cta-variant-generator and in-app-microcopy-writer-auditor rather than
  rebuilding them. Use when the user says "write upgrade modal," "paywall copy," "upsell
  prompt," "expansion trigger," "limit-hit message," "trial conversion prompt," "upgrade
  moment," "PQL prompt copy," "what to say when a user hits a limit," or hands over a
  usage trigger and asks what the product should say.
---

# Upgrade & Expansion Prompt Writer

The right message at the limit hit converts. The wrong one — or the right one at the wrong moment — trains users to dismiss. This skill writes expansion copy that lands because it is specific to the trigger, honest about what the upgrade buys, and timed for the moment of maximum perceived value.

Scope: modal copy (variants), paywall-surface spec, and a trigger prioritization map. It does not build the full email nurture sequence (that is `usage-triggered-message-sequencer`), the feature adoption campaign (that is `feature-adoption-campaign-planner`), or the onboarding flow (that is `onboarding-flow-builder`). It handles the prompt in the product at the moment of expansion intent.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, offer mechanics, pricing tiers, and real proof before any copy is written. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, plan names + limits, and the next-tier headline benefit before proceeding.
- **`cta-variant-generator`** (compose) — generates the modal's primary CTA and friction-reducer microcopy from the upgrade offer; do not rewrite CTA logic here.
- **`in-app-microcopy-writer-auditor`** (compose) — writes and audits the paywall surface copy (empty-state label, disabled-feature tooltip, progress bar label, hard-block message); call it for that surface rather than writing microcopy from scratch.
- **`offer-pricing-brain`** (optional) — if the brand's `brand.md` lacks current tier limits and unlocks, call this to load a canonical pricing object before writing any tier-specific copy. Mark any unconfirmed plan detail `[verify]`.
- **`lifecycle-email-push-copy-reviewer`** (optional) — if output will be shared with a reviewer pass, route the completed copy through this skill; do not self-approve.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (always first)
Step 1  Classify the trigger  ──► Moment-Fit matrix
Step 2  Write the three deliverables
         a) Modal copy battery (3–5 variants)
         b) Paywall surface spec
         c) Trigger prioritization map
Step 3  Compose CTA via cta-variant-generator
Step 4  Compose paywall microcopy via in-app-microcopy-writer-auditor
Step 5  Self-review, then present
```

### Step 0 — Load brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, pricing tiers + limits, primary upgrade destination URL, real proof, ICP, and awareness tendency. Do not write a single word of copy before it returns.

Obey voice and banned-words as hard overrides. Mark any pricing detail not confirmed by `brand.md` or `offer-pricing-brain` as `[verify]`. If the brand has multiple tiers, ask the user to confirm the source tier and the upgrade target tier if they are ambiguous.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, plan names + limits, and the next-tier headline benefit before proceeding.

---

## The Moment-Fit Expansion model (our working framework)

Every upgrade or expansion prompt sits in one of four trigger categories. The category determines copy temperature, urgency type, and CTA commitment level. Misidentifying the category is the most common root cause of dismissed prompts.

| Category | Trigger signal | Perceived value moment | Copy temperature | Urgency type |
|---|---|---|---|---|
| **Limit-Hit** | Feature cap reached, quota exhausted | High — user is blocked from something they just tried | Warm, helpful | Functional (remove the block) |
| **Usage Spike** | Rapid consumption of allowance (>70 % used in <30 % of billing window) | High — momentum is present, upgrade prevents interruption | Encouraging | Preventive (protect progress) |
| **Aha Expansion** | User discovered a premium-feature surface (clicked a locked feature, activated a trigger that touches an upsell) | Medium-high — curiosity or intent already active | Aspirational | Value-pull (show what they'd gain) |
| **Renewal/Expiry** | Trial end approaching, plan anniversary, downsell risk | Medium — user knows the clock exists | Factual + urgent | Time-bounded (honest deadline) |

**Moment-Fit gate:** if the user's trigger does not map cleanly to one of the four categories, surface the ambiguity and ask before writing. A mismatched prompt burns trust more than no prompt.

---

## Deliverable A — Modal copy battery (3–5 variants)

Write one variant per persuasion angle, not per synonym. Minimum angles for a full battery:

1. **Value-unlock angle** — lead with the specific feature or outcome the upgrade reveals; name it exactly, not generically ("Unlock behavioral segmentation" not "Unlock more features").
2. **Progress-protection angle** — frame the upgrade as preventing the loss of work or momentum already invested ("You've built 3 active flows — keep them running without interruption").
3. **Social proof angle** — use a real proof point from `brand.md` (customer count, stat, named outcome); if none confirmed, leave a `[verify]` slot rather than inventing.
4. **ROI / specificity angle** — tie the upgrade to a measurable outcome the ICP cares about ("Teams that upgrade to [tier] see X% higher [KPI]" — `[verify]` if not confirmed).
5. **Low-friction angle** — reduce the commitment ("Try [tier] free for 14 days; cancel any time") when the brand's offer mechanics support it.

Each variant: `Headline` (≤12 words) + `Body` (≤40 words, one clear benefit claim) + `Primary CTA` (call `cta-variant-generator` for this) + `Friction-reducer microcopy` (1 line, closes the remaining objection).

```
## Modal variant [#] — [angle name]
Headline:
Body:
CTA: [from cta-variant-generator]
Microcopy:
Trigger category: [Limit-Hit | Usage Spike | Aha Expansion | Renewal/Expiry]
Best for: [placement note — e.g., "fires on the 11th-project-create attempt for Growth plan"]
```

---

## Deliverable B — Paywall surface spec

A hard block that says "Upgrade to continue" without context is a conversion fumble. This deliverable specifies every copy surface on the blocked/locked state so the product team has exact strings to implement.

For each locked surface identified (or handed by the user), specify:

| Surface element | String | Character limit | Notes |
|---|---|---|---|
| Feature-gate label | exact string | ≤6 words | In-context; appears near the locked element |
| Tooltip / hover copy | exact string | ≤20 words | Fires on hover of the locked element |
| Empty-state headline | exact string | ≤10 words | Replaces the empty state if no items exist yet |
| Empty-state body | exact string | ≤30 words | What they'd have if they upgrade |
| Hard-block header | exact string | ≤8 words | Top of modal or inline block |
| Hard-block body | exact string | ≤35 words | Specific benefit of the upgrade at this surface |
| CTA | exact string | ≤25 chars | From `cta-variant-generator` |

Call `in-app-microcopy-writer-auditor` for the paywall surface copy; synthesize inline only if the skill is unavailable.

---

## Deliverable C — Trigger prioritization map

Not all triggers are equal. This map tells the product/engineering team which events fire which prompt variant, and prevents simultaneous or redundant prompts from degrading trust.

For each trigger event supplied (or inferred from the modal copy), specify:

| Priority | Event | Trigger category | Delay | Suppression rules | Modal variant # | Success metric |
|---|---|---|---|---|---|---|
| 1 | e.g., `project_limit_reached` | Limit-Hit | immediate | suppress if upgrade shown in last 7 days | Variant 2 (Progress-protection) | upgrade click / dismiss rate |
| 2 | e.g., `quota_at_70pct` | Usage Spike | on next page load | suppress if already upgraded | Variant 1 (Value-unlock) | upgrade click |
| 3 | e.g., `premium_feature_hover` | Aha Expansion | 500 ms hover | suppress if shown 3× in 30 days | Variant 5 (Low-friction) | free-trial start |

**Suppression rules are mandatory.** A trigger map without suppression logic ships a harassment pattern. Always specify: recency window (do not re-show within N days), count cap (max N impressions per billing period), and exclusion state (do not fire if user is already at target tier).

Save trigger map to `./upgrade-prompts/[slug]-trigger-map.md` on request; modal copy to `./upgrade-prompts/[slug]-modal-copy.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before the brand is loaded and tier limits confirmed.
- **Trigger category before copy.** Misclassifying Limit-Hit as Aha Expansion produces the wrong tone and urgency — diagnose before you write.
- **Specificity over enthusiasm.** "Unlock behavioral segmentation" outperforms "Unlock powerful features" every time. If you cannot name the specific unlock, ask.
- **Suppression is product quality.** Every trigger spec ships with suppression rules or it is incomplete.
- **Honest urgency only.** Trial expiry is real; "offer expires soon" with no expiry date is dark-pattern territory — do not use it.
- **Real proof or `[verify]`.** Never invent customer counts, stats, or outcome claims.
- **Compose, don't duplicate.** CTA logic lives in `cta-variant-generator`; microcopy lives in `in-app-microcopy-writer-auditor`; call them, don't rewrite them.

## What Not to Do

- Don't write modal copy before `brand-brain` returns the active brand and tier limits.
- Don't produce five synonyms instead of five angles — if you can't find five distinct motivations, say why and produce the legitimate ones.
- Don't ship a trigger map without suppression rules — a suppression-free trigger map is a bug, not a feature.
- Don't use "Upgrade now" as the primary CTA without calling `cta-variant-generator` first.
- Don't write paywall copy that names pricing (`$X/month`) unless `offer-pricing-brain` or `brand.md` has confirmed the exact figure — price surprises in modals destroy trust.
- Don't conflate the upgrade modal with the post-upgrade onboarding sequence (that is `usage-triggered-message-sequencer` + `onboarding-flow-builder`).
- Don't use dark patterns: fake countdown timers, implied data-loss threats that aren't real, or manufactured scarcity.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand, voice, banned words, and tier limits loaded?
- Trigger category correctly identified via Moment-Fit matrix, and named in each variant?
- Deliverable A: ≥3 variants, each on a genuinely distinct persuasion angle (not synonyms), each with headline + body + CTA + microcopy?
- `cta-variant-generator` called for CTAs; `in-app-microcopy-writer-auditor` called for paywall surface copy?
- Deliverable B: every copy surface specified with exact strings and character limits?
- Deliverable C: trigger prioritization map includes event, category, delay, suppression rules (recency + count + exclusion state), variant #, and success metric?
- No invented proof or pricing — unconfirmed values marked `[verify]`?
- No dark patterns: all urgency is real, no fake scarcity, no data-loss threats that don't reflect actual product behavior?
- Voice and banned-words from `brand-brain` honored throughout?
