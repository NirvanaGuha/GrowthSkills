---
name: new-competitor-detected-response-kit
description: >
  A new or moving competitor showed up — research them, then ship the whole response. Give it a
  competitor URL (and optionally what spooked you: a sales loss, a pricing change, an ad you saw)
  and it runs the full chain a growth team would run over a week, in one pass: deep competitive
  intelligence → an updated battlecard with objection handling → a review-gap map of pains they
  ignore that you can own → counter-content (a comparison/FUD-buster page + a LinkedIn post) →
  revised positioning suggestions, all gated by a positioning review before anything ships. It does
  NOT re-derive brand context or re-implement any stage — it calls `brand-brain` for voice/ICP/offer/
  proof and chains the specialist sibling skills, each via the Skill tool, then compiles their output
  into one bundled deliverable folder you can hand to sales and marketing. Use whenever the user says
  "new competitor," "a competitor just launched/raised/repositioned," "respond to [competitor]," "we
  keep losing to X," "build a competitive response," "they changed their pricing/messaging," "react to
  this competitor," or pastes a competitor URL and asks what to do about it. Produces a response kit,
  not a single asset.
---

# New-Competitor-Detected Response Kit

A competitor moved. Point this at their URL and it gives you back what a junior marketer would take a week to assemble: the intelligence, the sales battlecard, the wedge no one else is defending, the counter-content, and the positioning tweak — compiled, reviewed, and ready to use. One run, one folder.

This is an **orchestrator**. It does not write copy, build dossiers, or score positioning itself — it sequences the skills that do, hands each one the last stage's output, and assembles the result. If a stage's input is thin, it says so and stops rather than papering a weak response over weak intel.

---

## Skills this calls

The pipeline, in order. Every name is a real, installed sibling skill — invoke each via the **Skill tool**; never re-implement its work here.

- **`brand-brain`** (required, Layer-0) — loads the active brand's voice, ICP, offer/pricing, proof, banned words, and any `competitors.md` / `objections.md` companions. Run first; everything downstream reads from it.
- **`competitive-intelligence-dossier`** — turns the competitor URL into the structured intel brief (positioning, pricing, features, GTM, strengths/weaknesses).
- **`battlecard-objection-handler`** — turns the dossier + your offer into a sales battlecard with "why we win / why we lose / trap-setting questions / objection rebuttals."
- **`competitor-review-gap-spotter`** — finds pains in their review themes they ignore that you could own; feeds the counter-content wedge.
- **`content-format-writer-suite`** — writes the counter-content (a comparison / "X vs You" page or FUD-buster) in the brand's voice.
- **`linkedin-post-writer`** — writes the public, founder-voice angle on the wedge (the social shot, not a press release).
- **`positioning-reviewer`** (the gate) — scores the revised positioning + counter-messaging for clarity, differentiation, believability; returns Approve or Revise.
- *(optional, when present)* `competitor-ad-library-spy` for live creative/offers, `competitor-price-benchmarking-analyst` for a tier-by-tier price gap, `positioning-messaging-architect` to draft the revised positioning the reviewer then grades. Synthesize inline only if absent.

---

## How a run works

```
Step 0  Load the brand        ──► call `brand-brain`            → voice/ICP/offer/proof + companions
Step 1  Frame the trigger     ──► what moved, who's affected     → scope: sales / marketing / both
Step 2  INTEL                  ──► competitive-intelligence-dossier  → dossier.md
Step 3  WEDGE (parallel)       ──► review-gap-spotter ∥ price/ad spy → wedge + proof-of-difference
Step 4  ARM SALES              ──► battlecard-objection-handler   → battlecard.md
Step 5  DRAFT POSITIONING      ──► (positioning-architect or inline) → revised-positioning.md
Step 6  GATE                   ──► positioning-reviewer           → Approve | Revise → loop to 5
Step 7  COUNTER-CONTENT        ──► content-format-writer-suite ∥ linkedin-post-writer
Step 8  COMPILE + human review ──► one ./competitor-response-[slug]/ folder + README
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. Use the returned digest — voice + banned words, ICP + awareness, offer mechanics + destination URLs, real proof, positioning line — as hard overrides for every stage, and pass it forward so no downstream skill re-derives it. If `competitors.md` already exists for this brand, hand it to Step 2 so the dossier extends rather than restarts.

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; else ask the user for the brand name, ICP + awareness stage, offer mechanics + primary destination URL, and 3 voice adjectives + banned words, then proceed. This thin read is the sanctioned fallback — it is *not* a license to reimplement scanning/interviewing/storage. Always prefer the call. Never write `brand.md` yourself.

### Step 1 — Frame the trigger

Establish three things before spending a single sub-skill call:
1. **The competitor** — confirm the URL resolves to one company; if the user pasted a category or several, ask which one (one kit per competitor).
2. **What moved** — a launch, a raise, a repricing, a repositioning, a feature, an ad, or "we keep losing deals to them." This sets which stages matter most (a repricing weights the price-gap + battlecard; a repositioning weights the dossier + positioning review).
3. **Who consumes it** — sales (battlecard-heavy), marketing (counter-content-heavy), or both (default: build all, label each artifact's owner).

Write this into a one-paragraph **run brief** that every downstream skill receives alongside the brand digest. If the URL is dead or the trigger is "I have a vague feeling," say so and ask for one concrete signal before proceeding — do not invent a competitor move.

### Step 2 — INTEL

Invoke **`competitive-intelligence-dossier`** with the competitor URL + run brief + brand digest. Expect: positioning, pricing/packaging, ICP overlap, feature surface, GTM motion, and an honest strengths/weaknesses read. **Handoff:** the dossier is the spine — Steps 3–6 all read from it. If the dossier comes back thin (paywalled site, no public pricing), record the gaps as `[verify]` and continue; do not fabricate to fill them.

### Step 3 — WEDGE (parallel)

Run these in parallel — they're independent reads of the same competitor:
- **`competitor-review-gap-spotter`** — the pains in their reviews they don't address; this is your ownable wedge.
- *(if installed)* `competitor-price-benchmarking-analyst` for a tier gap, `competitor-ad-library-spy` for the angles/offers they're spending on.

**Handoff:** synthesize one **wedge statement** — "the thing they can't or won't say that we can prove" — backed only by brand-brain's real proof (everything else `[verify]`). This wedge feeds the battlecard's "why we win," the counter-content's thesis, and the LinkedIn angle. If no defensible wedge survives the proof filter, flag it: the honest move may be a feature/positioning gap to close, not a campaign to ship.

### Step 4 — ARM SALES

Invoke **`battlecard-objection-handler`** with the dossier + wedge + brand digest (+ any `objections.md`). **Handoff:** expect a battlecard — why-we-win / why-we-lose / landmine questions / objection rebuttals — that reuses the *same* wedge and proof as the content, so sales and marketing tell one story.

### Step 5 — DRAFT POSITIONING

Produce the **revised positioning suggestions** the kit promises: how our positioning shifts (if at all) now that this competitor exists. Prefer invoking `positioning-messaging-architect` if installed; otherwise draft a tight positioning delta inline (for/who/unlike/we-uniquely) from the dossier + wedge. **Handoff:** this draft goes straight to the gate.

### Step 6 — GATE (positioning review → loop)

Invoke **`positioning-reviewer`** on the revised positioning + the counter-content thesis. It returns a verdict:
- **Approve** → proceed to Step 7.
- **Revise** → take its specific notes back to Step 5, regenerate, and re-submit. **Loop max twice.** If still not approved after two passes, stop and surface the reviewer's unresolved objections to the human — do not ship un-approved positioning. This is the quality gate the whole kit hinges on: counter-content built on shaky positioning is worse than no counter-content.

### Step 7 — COUNTER-CONTENT (parallel, post-gate only)

Only after Approve, run in parallel:
- **`content-format-writer-suite`** — the comparison / "vs" page or FUD-buster, in the chosen format, on the approved positioning + wedge.
- **`linkedin-post-writer`** — the public founder-voice angle (a perspective, not a teardown).

**Handoff:** both consume the *approved* positioning, the wedge, and the same proof — so the page, the post, and the battlecard are mutually consistent.

### Step 8 — COMPILE + human review

Assemble everything into one folder (below) with a README that states the trigger, the wedge, the positioning verdict, and a per-artifact owner + next action. Present a summary and **explicitly ask the human to approve before anything goes external** — counter-positioning is brand-risky; a person signs off, not the orchestrator.

---

## Bundled deliverable

Save to a **project-relative path in the user's CWD** (never the skill folder):

```
./competitor-response-[competitor-slug]/
  README.md                  run brief · trigger · wedge · positioning verdict · owners/next actions
  00-dossier.md              competitive-intelligence-dossier output
  01-wedge.md                review-gap + price/ad synthesis → the ownable angle
  02-battlecard.md           sales battlecard + objection rebuttals
  03-positioning.md          revised positioning + positioning-reviewer verdict (Approve, w/ history)
  04-comparison-page.md      counter-content (vs / FUD-buster), brand voice
  05-linkedin-post.md        public founder-voice angle
```

If the user prefers one file, compile the same sections into a single `competitor-response-[slug].md`. Confirm the saved path in one line. Every unverified number, claim, or competitor fact carries `[verify]`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No stage runs before `brand-brain` returns; its voice + banned words override every sub-skill's output.
- **Orchestrate, don't reimplement.** Each stage is a real sibling skill invoked via the Skill tool. This file owns sequencing, handoffs, gates, and compilation — nothing else.
- **One wedge, told consistently.** The battlecard, the page, the post, and the positioning all reuse the same proof-backed wedge. No two artifacts contradict each other.
- **The positioning gate is real.** Counter-content ships only after `positioning-reviewer` approves; Revise loops back, max twice, then a human decides.
- **Truth discipline.** Real proof only; every unconfirmed competitor fact or number is `[verify]`. Never invent a pricing tier, a customer, or a competitor weakness.
- **Punch up on substance, not mud.** Counter-positioning attacks the gap you can prove, never the competitor's character. The honest read sometimes is "close a product gap," not "run a campaign."
- **A human approves external use.** The kit is drafted and reviewed; a person signs off before it leaves the building.

## What Not to Do

- Don't produce any artifact before `brand-brain` returns the active brand.
- Don't re-implement a stage's work (no inline dossier, battlecard, or review-scoring) when its skill exists — call it.
- Don't ship counter-content on un-approved positioning; don't loop the gate more than twice without escalating to the human.
- Don't fabricate competitor pricing, features, claims, or reviews to fill a thin dossier — mark gaps `[verify]`.
- Don't let the battlecard and the public content tell different stories.
- Don't save artifacts into the skill folder; write to the project CWD.
- Don't run the kit on a vague hunch — require one concrete competitor signal first.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or the sanctioned fallback used) before any stage?
- Run brief written (competitor confirmed, what moved, who consumes) and passed to every sub-skill?
- Each stage invoked as a real Skill-tool call — dossier, review-gap, battlecard, content suite, LinkedIn, positioning reviewer — with the prior stage's output handed forward?
- One proof-backed wedge, reused consistently across battlecard + page + post + positioning?
- `positioning-reviewer` returned **Approve** before counter-content was written? (Revise looped ≤2, else escalated?)
- Every unconfirmed competitor fact/number marked `[verify]`; no invented tiers, customers, or weaknesses?
- Bundle written to `./competitor-response-[slug]/` with a README (trigger · wedge · verdict · owners) and the path confirmed?
- Human approval explicitly requested before any external use?
