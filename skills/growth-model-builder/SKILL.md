---
name: growth-model-builder
description: >
  Business context + current metrics → documented growth model with acquisition loop, retention loop,
  and lever map. Takes any combination of inputs — product description, current funnel metrics, channel
  mix, pricing tier data, churn/retention numbers, cohort curves — and produces a documented growth
  model: the acquisition loop (how strangers become customers), the retention loop (how customers stay
  and expand), a prioritized lever map (what to pull and in what order), and a set of leading indicator
  KPIs with owner assignments. Built on the Reforge Growth Accounting + Loop Framework so a junior
  marketer gets output a senior would sign off on. Does NOT generate channel copy, OKRs, or a full
  marketing plan — it produces the strategic growth architecture those tools build on top of.
  Use when the user says "build my growth model," "map our acquisition loop," "what are our growth
  levers," "document how we grow," "retention loop," "where should we focus for growth," "growth
  framework," or hands over funnel metrics and asks what they mean strategically.
---

# Growth Model Builder

Turn your funnel data and business context into a documented growth model — the acquisition loop, the retention loop, and a ranked lever map — so every campaign, OKR, or experiment is pulling the same strategic rope.

This skill builds the architecture. It does not write channel copy, generate OKR sets, or produce a marketing plan. When those are ready, hand the model to `okr-suite`, `channel-strategy-selector`, or `marketing-plan-generator` — they read what this outputs.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand's ICP, offer mechanics, positioning, and proof. The growth model is meaningless without knowing who the customer is and what they're buying.
- **`icp-persona-builder`** *(call if ICP is missing from brand-brain)* — builds the persona that anchors loop design.
- **`ltv-cac-payback-calculator`** *(call if CAC/LTV data is present)* — quantifies the unit economics that size each lever's impact.
- **`funnel-drop-off-analyzer`** *(call if funnel step data is present)* — pinpoints the biggest drop-off to anchor lever prioritization.
- **`kpi-tree-builder`** *(from .all-skills.txt, invoke when available)* — extends the lever map into a formal KPI tree with owner assignments.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (ICP, offer, proof, positioning)
Step 1  Gather inputs            ──► collect or request business context + metrics
Step 2  Map the loops            ──► acquisition loop → retention/expansion loop
Step 3  Build the lever map      ──► identify all growth levers; score and rank them
Step 4  Define leading KPIs      ──► one metric per loop stage; flag what to track first
Step 5  Document + save          ──► write growth-model.md; present the model
```

---

### Step 0 — Load the brand (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`) before touching any inputs. Use the returned digest for:
- **ICP + awareness tendency** → shapes which acquisition channels make sense and where the loop starts.
- **Offer mechanics** → free trial / freemium / self-serve / sales-assisted / product-led determines which loops are available.
- **Proof + positioning** → anchors the retention value statement; unconfirmed proof is `[verify]`.

**Fallback:** if `brand-brain` is not installed, read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if absent, ask the user for: (1) ICP one-liner, (2) acquisition motion (self-serve / sales / product-led), (3) pricing model (free/trial/paid/expansion), (4) rough MoM retention. Then proceed.

---

### Step 1 — Gather inputs

Ask only for what is missing. The more the user provides, the more specific the model. Minimum viable input set:

| Input | Why it matters |
|---|---|
| What the product does + who it serves | Anchors both loops |
| Primary acquisition channel(s) | Identifies loop entry mechanics |
| Current conversion / activation rate (optional) | Sizes acquisition loop efficiency |
| 30/60/90-day retention or churn rate (optional) | Sizes retention loop health |
| Revenue model: one-time / subscription / usage | Determines expansion loop structure |
| ARR / MRR range (optional) | Calibrates lever sizing |

If none of these are provided, run the model qualitatively (directional, not quantified) and mark every sizing note `[verify or measure]`.

---

## The framework: Reforge Growth Accounting + Loop Model

Every durable growth model has two interlocking loops. Build both.

### Acquisition loop

The self-reinforcing mechanism that brings new users or customers in. Loops compound; channels don't. A channel (paid search, SEO, outbound) is one input to an acquisition loop — not the loop itself.

**Identify the loop type first:**

| Loop type | Mechanic | Trigger signal |
|---|---|---|
| Viral / WOM | Every activated user creates at least one new user | Referral, sharing, network effects |
| Content / SEO | Published content attracts searchers who become users | Organic traffic compounding |
| Sales / outbound | Reps create pipeline from ICP lists; closed revenue funds more reps | CAC-positive payback period |
| Paid acquisition | Ad spend → trial → revenue → reinvest in ads | LTV:CAC ratio > 3 |
| Product-led | Usage itself drives acquisition (free tier, embeds, notifications) | Viral coefficient k > 0.1 |
| Partnership | Integration partners send qualified users | % of new signups from partner sources |

**Map the acquisition loop in 5 nodes:**
```
[Awareness trigger] → [Acquisition surface] → [Activation event] → [Value realization] → [Loop amplifier → back to Awareness]
```
Label each arrow with the conversion rate if known, or `[measure]` if not.

---

### Retention + expansion loop

Retention is the multiplier on acquisition. A company with 5% monthly churn is on a treadmill. Map the retention loop before optimizing acquisition spend.

**Retention loop structure (Jobs-to-be-Done anchor):**
```
[Customer achieves outcome] → [Habit formation / repeated use] → [Expansion trigger] → [Advocacy or referral] → [Deepened product dependency]
```

**Classify the retention motion:**

| Motion | Signal | Expansion lever |
|---|---|---|
| Habit-loop retention | DAU/MAU > 0.3; users return without prompting | Add stickiness features; improve streak/habit triggers |
| Outcome-based retention | Usage spikes around a recurring business event | Build integrations and workflow entrenchment |
| Network-effect retention | Churn inversely correlated with team/network size | Drive multi-seat expansion; penalize downgrades structurally |
| Lock-in retention | Switching cost from data/integration accumulation | Deepen integrations; protect export friction deliberately |

For each retention motion identified, name:
- The **core action** the customer must repeat to stay.
- The **expansion trigger** (usage limit, seat count, feature gate, success milestone).
- The **churn risk signal** (what precedes a cancellation 30–60 days out).

---

### Step 3 — Build the lever map

A growth lever is a specific, ownable action that moves a loop metric. Enumerate all plausible levers, then score and rank them.

**Lever scoring rubric (ICE variant adapted for growth loops):**

| Criterion | 1 | 3 | 5 |
|---|---|---|---|
| **Impact** | Moves a lagging metric for one segment | Moves a leading metric across a loop stage | Compresses or accelerates an entire loop |
| **Confidence** | Directional hunch | One confirming data point | Multiple data points + prior evidence |
| **Speed** | > 90 days to see signal | 30–90 days | < 30 days |
| **Leverage** | Requires new infrastructure | Builds on existing motion | Reuses existing assets with small delta |

Score each lever 1–5 on each criterion; multiply for a composite score (max 625). Rank descending.

**Lever categories to cover (don't skip):**
1. Acquisition loop entry (top-of-funnel volume)
2. Activation rate (trial→paid, signup→first value)
3. Retention depth (habit formation, feature adoption)
4. Expansion (seat growth, tier upgrade, usage)
5. Resurrection (win-back of churned / lapsed users)
6. Loop amplifier (what makes the loop compound faster — referrals, SEO flywheel, integration density)

---

### Step 4 — Define leading KPIs

One leading indicator per loop stage. Leading indicators predict future loop health; lagging indicators (revenue, churn) confirm it.

| Loop stage | Example leading indicator | Owner |
|---|---|---|
| Acquisition entry | Qualified trial signups / week | Marketing |
| Activation | % of trials hitting activation event within 7 days | Product / Growth |
| Retention | 30-day retention rate by cohort | Product |
| Expansion | % of accounts at >60% plan limit | Revenue / CS |
| Loop amplifier | Referral sends per activated user / month | Growth |

Adapt to the brand's actual motion. Mark any KPI where baseline data is absent as `[measure first]`.

---

### Step 5 — Document + save

Write the completed model to `./growth-model.md` (project-relative). Structure:

```
# Growth Model — [Brand Name]
Generated: [date]

## Business context
[ICP, offer, revenue model, scale signal]

## Acquisition loop
[Diagram in plain text: 5-node loop with conversion rates or [measure]]
Loop type: [named type]

## Retention + expansion loop
[Diagram: 4-node loop]
Retention motion: [named type]
Core action: …
Expansion trigger: …
Churn risk signal: …

## Lever map
| Rank | Lever | Category | Impact | Confidence | Speed | Leverage | Score |
…
Top 3 immediate actions: …

## Leading KPIs
| Stage | KPI | Baseline | Owner |
…

## What to do first (30-day focus)
[One paragraph: highest-scored lever, why, what to measure to confirm it's working]
```

Confirm in one line after saving: *"Growth model saved to `./growth-model.md` — pass it to `channel-strategy-selector`, `okr-suite`, or `marketing-plan-generator` to build on top."*

---

## Principles (Non-Negotiable)

- **Loops before channels.** A channel is an input to a loop. Build the loop first; channels follow.
- **Two loops, always.** A model with only an acquisition loop is a model that leaks. Always map retention.
- **Lever ranking is the deliverable.** The map is what makes the model actionable; don't hand back a description of the business without it.
- **Real numbers or `[verify]`.** Size every lever with actual metrics if available; mark gaps `[measure first]`, not invented proxies.
- **Brand-brain context is load-bearing.** ICP, offer mechanics, and positioning are not decorative; they determine which loops are structurally available.

## What Not to Do

- Don't confuse channels with loops — don't output "our growth model is SEO + paid."
- Don't write the marketing plan or OKRs here; that's `marketing-plan-generator` and `okr-suite`.
- Don't skip the retention loop because the user only asked about acquisition.
- Don't invent CAC or LTV benchmarks; mark them `[verify]` and suggest how to calculate them.
- Don't produce a lever map with everything rated "high" — force differentiation; rank ruthlessly.
- Don't reimplement ICP, brand voice, or proof logic — call `brand-brain`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and ICP + offer mechanics loaded before any output?
- Acquisition loop has five named nodes with conversion rates (real) or `[measure]`?
- Loop type correctly classified (viral / content / sales / paid / PLG / partnership)?
- Retention loop mapped with core action, expansion trigger, and churn risk signal?
- Retention motion classified (habit / outcome / network / lock-in)?
- Lever map covers all six categories; ICE scores computed; top 3 called out explicitly?
- Leading KPIs include one per stage, with owner and `[measure first]` flags where baseline is absent?
- `growth-model.md` saved to project-relative path, not the skill folder?
- 30-day focus paragraph written — one lever, why, how to verify?
