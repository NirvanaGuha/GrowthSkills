---
name: pricing-page-copywriter-reviewer
description: >
  Writes and critiques pricing page copy for SaaS, eCommerce, and service businesses — working from
  a tier structure and ICP pains (Write mode) or a draft/live page (Review mode). Produces plan names
  and taglines, the hero headline + positioning line, per-tier value propositions, feature-highlight
  rationale copy, FAQ set tuned to objection patterns, and CTA copy via cta-variant-generator. In
  Review mode it runs a structured critique against the Pricing Page Persuasion Stack: clarity, tier
  logic, anchoring, objection coverage, social proof placement, and CTA strength. Brand context,
  voice, banned words, ICP, offer mechanics, and real proof all come from brand-brain — never
  re-derived here. Use when the user says "write my pricing page," "improve / critique my pricing
  page," "name my plans," "pricing page copy," "pricing page review," "tier messaging," "plan
  descriptions," "pricing FAQs," "audit pricing page," or hands over a tier structure and asks what
  the page should say.
---

# Pricing Page Copywriter & Reviewer

The pricing page is the highest-stakes copy in a SaaS or product business — it is where position,
proof, objection-handling, and CTA mechanics converge at the moment of purchase intent. Most pricing
pages fail not because the pricing is wrong but because the copy makes the wrong trade-off legible.
This skill fixes that.

Two modes: **Write** (tier structure + ICP pains → complete pricing page copy) and **Review** (draft
or live page → structured critique with rewrite priorities). Both modes run the Pricing Page
Persuasion Stack as their evaluative framework, compose upward from `offer-pricing-brain` for offer
mechanics and from `cta-variant-generator` for button copy, and anchor everything in the active
brand via `brand-brain`.

---

## Skills this calls

- **`brand-brain`** (required, always first) — voice, banned words, ICP, offer mechanics, real proof,
  positioning. This skill never reimplements brand resolution.
- **`offer-pricing-brain`** — in Write mode, called to confirm tier logic, feature bundling rationale,
  anchoring strategy, and guarantee / trial mechanics before writing any plan copy. In Review mode,
  called when the tier structure itself looks like the root cause of weak copy.
- **`cta-variant-generator`** — called to generate button copy for each tier (plan-level and page-level
  hero CTA); pass the brand digest + awareness stage + offer destination.
- *(optional)* `proof-vault` — for social proof microcopy (testimonials, logos, stat lines) to drop
  into the trust band and FAQ answers. Synthesize inline when absent.
- *(optional)* `objection-library-builder` — for a richer FAQ objection set when the page is
  high-consideration or the review surfaces objection coverage as a gap. Synthesize inline when absent.
- *(optional)* `icp-persona-builder` — when the user's ICP is unknown or multi-segment, to establish
  the primary buyer and their awareness stage before writing tier positioning.

---

## How a run works

```
Step 0  Load the brand      ──► brand-brain (always; bootstraps on first use)
Step 1  Determine mode      ──► Write (tier structure in) | Review (draft/live page in)
Step 2  Confirm offer logic ──► offer-pricing-brain (Write) / tier audit (Review)
Step 3  Do the work         ──► Pricing Page Persuasion Stack drives output
Step 4  Generate CTAs       ──► cta-variant-generator for each tier + hero
Step 5  Self-review & present
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand digest: voice
adjectives, banned words, offer mechanics + destination URLs, real proof, positioning line, ICP +
awareness tendency. Obey voice and banned-words as hard overrides; use only returned real proof
(everything else is `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly. If none exists, ask for brand name, ICP description, 3 voice adjectives +
banned words, and the offer structure before proceeding.

### Step 1 — Determine mode

- **Write mode** — triggered by: tier structure / plan list, no existing page copy, "write my
  pricing page," "name my plans," "what should my pricing page say."
- **Review mode** — triggered by: existing copy pasted in, a URL, "review / audit / critique my
  pricing page," "what's wrong with," "improve this."
- **Ambiguous** — if only a URL is given, fetch the page, show a 2-line summary of what you found,
  and ask whether the user wants a critique or a rewrite of specific sections.

---

## The Pricing Page Persuasion Stack (the framework)

Every pricing page lives or dies on six layers. Work through them in order — gaps in earlier layers
compound into failures in later ones.

| Layer | What it does | Common failure |
|---|---|---|
| **1 — Clarity** | Buyer reads page once and knows exactly what each plan gives them and what it costs | Jargon tier names, ambiguous feature language, hidden limits |
| **2 — Anchoring** | The highest-priced plan reframes the middle plan as the sensible choice | No anchor, or anchor is invisible/buried |
| **3 — Tier logic** | Each plan has a coherent buyer identity — it serves a specific person with a specific situation | Plans defined by features, not by buyer; arbitrary gates |
| **4 — Objection coverage** | The five pricing-page objection families are addressed in the FAQ/proof band before the buyer has to ask | Missing contract/cancellation, missing ROI/value, missing trust signals |
| **5 — Social proof placement** | Proof appears at the right moments — at the plan decision, not only in the footer | Proof in wrong zone; generic logos without context |
| **6 — CTA strength** | Per-tier CTAs match commitment to awareness stage; the hero CTA does not compete with plan CTAs | All plans say "Get started"; free/trial/purchase conflated |

Use this stack as the evaluation spine in Review mode and as the construction checklist in Write mode.

---

## Write mode

### Inputs needed
- Tier structure: plan names (working or final), price points, key feature differences
- ICP: who buys each tier (or the primary buyer if undifferentiated)
- Any existing brand context not already in `brand.md`

Call `offer-pricing-brain` to validate the tier logic (anchoring, natural upgrade paths, gate
placements) **before** writing. If it flags structural problems (e.g., middle plan has no buyer
identity, anchor tier has no visible stretch benefit), surface them to the user as a gate — offer to
fix the structure first, or write around the constraint and flag the risk.

### What to produce

**Hero section**
- Headline (addresses the primary buyer's goal, not the product's feature list)
- Subheadline / positioning line (who it is for + the single most compelling differentiator)
- Trust line (social proof stat or customer count — real proof only, else `[verify]`)

**Plan names + taglines** (one per tier)
- Plan name: communicates buyer identity, not infrastructure (e.g., "Starter" < "For solo creators";
  "Enterprise" < "For teams that need control")
- Tagline: 6–10 words completing "This is right for you if…"
- Price framing copy: per-seat/per-month/annual-save note as appropriate

**Per-tier value proposition block** (2–3 sentences each)
- Leading benefit, not feature list
- The one thing the buyer at this tier cares most about (derived from ICP + offer tier logic)
- One real proof point or outcome claim at this tier — `[verify]` if unconfirmed

**Feature section guidance** (not the full feature table — that is product, not copy)
- For each tier: 2–3 feature callouts with rationale copy explaining *why* each feature matters to
  that buyer, not just what it is. Example: not "API access" but "Plug into your existing stack
  without a developer."

**Trust band / proof placement**
- Where to place: after the tier decision, before or alongside the FAQ
- 2–3 testimonial prompts (real quotes from `proof-vault` if available, else placeholder format:
  `[Customer, Role, Company — outcome in their words]`)
- Recommended logo strip note if applicable

**FAQ set** (6–10 questions)
- Structure around the five pricing-page objection families: (1) "Is this locked in?" / contract /
  cancellation, (2) "Will I get value from this?" / ROI / outcomes, (3) "Can I trust you?" / company
  / support quality, (4) "What happens if I outgrow / upgrade?", (5) "Is this the right tier for me?"
- Each answer is 2–4 sentences: direct answer first, then the reassurance, then the one-line CTA or
  next step. No FAQ answer is a sales pitch — it earns trust by being genuinely useful.

**CTAs** — call `cta-variant-generator` for each tier (passing: tier name, ICP awareness stage,
offer destination, brand digest). Present the returned recommendations. Also call for the page-level
hero CTA if separate.

Save output to `./pages/pricing-copy-[brand-slug].md`.

---

## Review mode

### Inputs
- The pricing page: paste the copy, provide a URL, or describe it. Fetch the URL if provided.

### What to produce

Run each layer of the Pricing Page Persuasion Stack against the page. For each layer:
- **Status**: Pass / Warn / Fail
- **Evidence**: quote the specific copy that earns the status (not a general claim)
- **Fix priority**: High / Medium / Low (High = direct conversion impact; Low = polish)
- **Recommended rewrite**: for every Fail and most Warns, provide the rewritten line/section — not
  just the diagnosis

Then produce a **Ranked Fix List** of the top 5 highest-leverage changes, in priority order, each
with: the layer it fixes, the current copy, the replacement, and the 1-line rationale.

If the root cause is the tier structure (not the copy), say so explicitly: "The copy cannot fix
this — the tier structure needs to change." Then offer to call `offer-pricing-brain` to work
through it.

Save critique to `./pages/pricing-review-[brand-slug].md` if asked.

---

## Principles

- **Brand-brain first.** No copy written, no review scored, before `brand-brain` returns.
- **Buyer identity before feature lists.** Plans serve a buyer, not a feature bundle. If a plan
  has no coherent buyer, the copy cannot fix it — say so.
- **Anchor deliberately.** The highest-priced plan's job is to reframe the middle plan as the
  obvious choice. Never bury it or de-emphasize it to seem humble.
- **Objections are not obstacles.** A good FAQ answers the question the buyer is embarrassed to ask
  aloud. Write it as if you want them to feel understood, not managed.
- **CTAs match commitment to stage.** A free-plan CTA and a paid-plan CTA should not be the same
  phrase. Call `cta-variant-generator` — do not hand-write CTAs.
- **Real proof only.** Customer counts, uplift numbers, and testimonials are `[verify]` unless
  sourced from `brand.md` or `proof-vault`.
- **Structural honesty.** When the tier logic is the problem, say it. Writing great copy onto a bad
  structure is malpractice.

---

## What not to do

- Don't write copy before `brand-brain` returns and the offer mechanics are confirmed.
- Don't reimplement brand resolution, ICP derivation, or CTA generation — call the skills.
- Don't write FAQ answers that are sales pitches in disguise — they reduce trust, they do not build it.
- Don't use generic plan names ("Starter / Pro / Enterprise") without validating they communicate
  buyer identity for this specific product; often they do not.
- Don't omit the anchor layer. A pricing page with no anchor is a race to the cheapest option.
- Don't invent proof numbers; don't use customer names or logos unless `brand.md` / `proof-vault`
  confirms they are approved for use.
- Don't stack-rank the Ranked Fix List arbitrarily — earn the priority with evidence from the page.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called first; voice, banned words, and ICP loaded and honored throughout?
- `offer-pricing-brain` consulted; tier logic validated or structural risk surfaced?
- All six Persuasion Stack layers addressed (Write: built in; Review: scored with evidence)?
- `cta-variant-generator` called for per-tier CTAs; no hand-written buttons?
- Real proof sourced from `brand.md` / `proof-vault`; everything else `[verify]`?
- FAQ covers all five objection families; answers are useful first, reassuring second?
- Write: hero, plan names + taglines, per-tier value props, feature rationale, trust band, FAQ, CTAs
  all present?
- Review: status + evidence + fix-priority + recommended rewrite per layer; Ranked Fix List of top 5?
- Output saved to `./pages/pricing-[mode]-[brand-slug].md` when applicable?
