---
name: offer-pricing-brain
description: >
  Captures the canonical offer, packaging, and pricing mechanics that gate every CTA — the tiers
  (good-better-best), the value metric that scales price, the feature fences between plans, the
  free/trial/demo/guarantee mechanics, and the primary conversion actions with their exact
  destination URLs. It is a `brand-brain` COMPONENT: callable standalone, or invoked by `brand-brain`
  during bootstrap/refresh to author two brand.md sections — "Offer & pricing essentials" and
  "Common CTAs & destinations" — that copy skills (CTAs, landing pages, email, pricing pages) depend
  on for message-match and honest offer framing. Use when the user says "offer," "pricing brain,"
  "plans and tiers," "what's our pricing," "free trial details," "packaging," or whenever any skill
  needs to know the real offer mechanics and conversion destinations before writing.
---

# Offer & Pricing Brain

The one place the offer is written down so no skill ever guesses it. What's free, what's gated, what the trial actually is, what the primary action is, and exactly where the button points — captured once, served everywhere. A CTA is only as honest as the offer behind it; this is that offer.

This skill captures and structures the offer. It does **not** set strategy (it won't tell you to add a tier), write the pricing page, or invent prices. Where a number isn't confirmed, it's marked `[verify]` — never filled in to look complete.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode only) — resolves the active brand and loads its current `brand.md` for context. This skill does not implement brand resolution, scanning, or storage; that lives in `brand-brain`, once. When `brand-brain` calls *this* skill, it passes the context — do **not** call back (no recursion).
- *(none else)* — no companion file; everything lives in the two owned `brand.md` sections.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED (context passed in) | STANDALONE (resolve via brand-brain)
Step 1  Gather the offer ──► scan passed/loaded inputs, then ask only the gaps
Step 2  Build the two sections (Offer & pricing essentials · Common CTAs & destinations)
Step 3  CALLED → return the section blocks  |  STANDALONE → persist into brand.md, confirm
```

### Step 0 — Detect the mode

- **CALLED BY brand-brain.** The request carries the active **slug**, the current **brand.md** content, and **scanned raw inputs** (pricing page text, plan tables, checkout URLs, session/memory facts). Use that context, do the work, and **return** your two section blocks for `brand-brain` to fold in. Do **not** invoke `brand-brain` (no recursion) and do **not** write any file — `brand-brain` owns the write.
- **STANDALONE.** No brand context was passed. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. Do the work. Then **persist** into the active brand's file at `~/.brandbrain/brands/<slug>/brand.md` (the data root `brand-brain` resolves — global `~/.brandbrain/`, or a per-project `./.brandbrain/` when present) — read it, replace **only** your two owned section blocks, bump `updated`, append yourself to `sources`, and confirm the save path.

**Fallback if `brand-brain` is not installed (standalone):** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for the brand and the offer basics, then proceed. Always prefer the call.

### Step 1 — Gather (scan, then ask only the gaps)

Pull from the passed/loaded inputs first: plan/tier tables, the live pricing page, checkout and signup URLs, the existing `brand.md` Offer section, and any session/memory facts. Map each to a field below. **Then ask only what's still missing or low-confidence** — never re-ask what the scan already established. Batch the questions. Minimum to be usable: the primary conversion action + its destination URL, and the free/trial/guarantee mechanics.

---

## Offer & pricing essentials (section 1 — the framework)

Build this on three named moves. A junior operator who fills these three tables in order produces an expert offer map.

### 1. Tiering — good-better-best

Capture every plan as a row. Name the **job each tier is for** (not just its features) and the **one reason** a buyer steps up — that step-up reason is the single most important fact for upsell copy.

| Tier | Price (+ value metric) | Who it's for / job | Step-up trigger (why upgrade) |
|---|---|---|---|
| Free / entry | $0 (cap: …) | … | hits the cap on … |
| Good | $X / [metric] | … | needs … |
| Better *(anchor)* | $Y / [metric] | … | needs … |
| Best / enterprise | $Z or "Contact us" | … | needs … |

Rules: name the **anchor tier** (the one you want most buyers on — usually "Better"). If pricing is "Contact us," say so and note what gates a quote. Annual vs monthly framing and any "most popular" badge belong here. Unconfirmed prices → `[verify]`.

### 2. Value-metric selection — what scales the price

Pin the single axis the customer pays more along, and where it's measured/capped. This is what makes pricing feel fair and what the buyer self-selects on.

| Field | Capture |
|---|---|
| Primary value metric | e.g. subscribers / seats / sends / orders / GB |
| Why it's the right axis | scales with the value the customer gets, not your cost |
| Free-plan cap on that metric | the number that triggers the first upgrade |
| Overage / hard-wall behavior | soft overage, hard cap, or forced upgrade |
| Secondary metrics, if any | seats, environments, add-ons |

If a brand prices on a vanity axis that doesn't track value, **say so once** as a note — don't redesign it.

### 3. Feature fences — what's gated where

The fences are the upgrade story. List the 3–6 features that actually move buyers up a tier — not the full matrix.

| Gated capability | First available in | Buyer pain it unlocks |
|---|---|---|
| … | Better | … |
| … | Best | … |

### Free / trial / demo / guarantee mechanics (the CTA-critical facts)

The exact terms copy must state truthfully. Get every cell right — these are the lines a CTA's risk-reducer microcopy is built from.

| Mechanic | Capture exactly |
|---|---|
| Free plan | yes/no · what's included · the cap that triggers upgrade |
| Free trial | yes/no · length (e.g. 14 days) · **card required?** · what happens at end |
| Freemium vs trial | which model (don't promise a trial if it's freemium, or vice-versa) |
| Demo / sales-assist | self-serve, or demo-gated above a tier? who books? |
| Guarantee / refund | money-back window · "cancel anytime" · no-lock-in language |
| Discounts | annual %, nonprofit/edu, launch — only if real and current |

Every number here that isn't confirmed is `[verify]`. Copy skills will state these verbatim, so a wrong cell becomes a wrong promise on a button.

---

## Common CTAs & destinations (section 2 — the message-match anchors)

The conversion actions and **where each one points**. This is what CTA, landing-page, and email skills match against so the button's promise equals the page it lands on. Get the URLs exact — a CTA that says "Start free trial" pointing at a pricing wall is a broken promise.

| Action | Button label(s) in use | Commitment | Destination URL | Notes |
|---|---|---|---|---|
| Primary conversion | "Start free trial" | high | https://… | the one action you want most |
| Secondary | "See plans" / "Compare" | medium | https://…/pricing | evaluate step |
| Low-commitment | "See it in action" / "Watch demo" | low | https://… | for less-aware traffic |
| Sales / enterprise | "Talk to sales" / "Get a quote" | high (assisted) | https://…/contact | gates the Best tier |

Rules: name the **single primary action** (one per surface — never two competing primaries). Map each action to the **right awareness/commitment level** so downstream CTA skills don't over-ask. Flag any destination that doesn't message-match its label as a `[verify]` to fix. If a button label is unknown, leave the action + URL and let the CTA skill write the label.

---

## Principles

- **Capture, don't strategize.** Record the offer as it is; mark gaps `[verify]`. Don't invent a tier, a price, a trial length, or a guarantee.
- **Own exactly two sections.** "Offer & pricing essentials" and "Common CTAs & destinations" — nothing else in `brand.md`, ever.
- **The trial line is sacred.** Card-required vs not, trial vs freemium, the exact day count, the exact cap — these become button promises. One wrong cell is a lawsuit-shaped lie.
- **One primary action per surface.** Name the anchor tier and the single primary CTA; everything else is secondary.
- **URLs are facts, not vibes.** Every destination is a real, current URL or `[verify]`. Label and destination must match.
- **Value metric over feature list.** Pricing is understood through the axis that scales it, not a 40-row matrix.
- **brand-brain owns the write in CALLED mode.** Return blocks; never touch the file.

## What not to do

- Don't write or overwrite any `brand.md` section other than your two. Where the spec says you "feed" pricing-page or CTA work, you **suggest** content — you don't author their sections.
- Don't call `brand-brain` when `brand-brain` called you (no recursion).
- Don't store offer data inside the skill folder — it lives in the resolved brand data root (`~/.brandbrain/brands/<slug>/`, or a per-project `./.brandbrain/`).
- Don't invent prices, trial terms, discounts, or guarantees; don't dump the full feature matrix — capture only the fences that drive upgrades.
- Don't redesign the pricing model (no "you should add a mid-tier"); note a real flaw once, then move on.
- Don't promise a trial when the brand runs freemium (or vice-versa).

## Quality checklist (self-review before returning/saving)

- Mode detected correctly — CALLED returns blocks (no file write, no brand-brain call); STANDALONE resolved via brand-brain and persisted?
- Scanned passed/loaded inputs first; asked **only** the missing/low-confidence fields?
- Section 1 has all three moves — tiering table with a named **anchor tier** + step-up triggers, the value metric pinned, the feature fences?
- Free/trial/demo/guarantee table filled cell-by-cell, with **card-required** and trial-vs-freemium unambiguous?
- Section 2 names a **single primary action**, each action mapped to a commitment level, every destination a real URL (label↔destination match), unknowns `[verify]`?
- Every unconfirmed number/term marked `[verify]`; nothing invented?
- STANDALONE only: persisted to `~/.brandbrain/brands/<slug>/brand.md`, replacing **just** the two owned section blocks, bumping `updated`, appending self to `sources`, and confirmed the save path?
