---
name: persona-to-full-funnel-messaging-kit
description: >
  Flagship orchestrator — a growth team in a box. Takes ONE raw persona description (or a CRM
  segment / VOC dump) and produces a complete, ready-to-ship messaging kit covering the entire
  buyer journey: a sharp ICP + named persona, a positioning message house, a TOFU blog angle/draft,
  a MOFU comparison page, and a BOFU email nurture sequence — all on-voice, all proof-backed, all
  mapped to the same persona's awareness path. It does NOT re-implement any stage; it CHAINS the
  library's specialist skills end to end (icp-persona-builder → positioning-messaging-architect →
  content-brief-builder/blog-post-drafting-engine → content-format-writer-suite →
  lead-nurture-drip-builder/welcome-onboarding-email-sequence-builder) and gates each handoff with a
  reviewer, looping on Revise. Brand context is loaded once from `brand-brain`; this skill never
  writes brand.md or re-derives voice/ICP/proof. Compiles everything into one bundled deliverable in
  the user's project. Use when the user says "build a full-funnel messaging kit," "one persona to a
  whole campaign," "messaging across every stage for [persona]," "TOFU/MOFU/BOFU for this segment,"
  "turn this persona into content + emails," "give me the whole funnel for [buyer]," or hands over a
  persona/segment and asks for end-to-end messaging. It orchestrates and compiles — the specialist
  skills do the writing.
---

# Persona-to-Full-Funnel Messaging Kit

Point this at one persona and walk away with a week of a junior marketer's output: who the buyer is, how you're positioned to them, a top-of-funnel article that earns their attention, a middle-of-funnel comparison that wins the evaluation, and a bottom-of-funnel email sequence that closes — every asset in the brand's real voice, anchored to the same persona's awareness journey, backed by real proof or flagged `[verify]`.

This skill is a conductor, not a soloist. It does not write personas, positioning, articles, or emails itself — it invokes the specialist skills that do, hands each one the previous stage's output, reviews the result at every gate, and compiles the lot into one folder you can ship. If a stage comes back weak, it loops that stage, not the whole pipeline.

---

## Skills this calls

The pipeline, in order. Every name is a real, installed sibling skill; invoke each via the **Skill tool** and pass it the upstream artifact. Never re-do a stage's work inline.

- **`brand-brain`** (Layer 0, required, first) — loads the active brand's voice, banned words, ICP tendency, offer + destinations, real proof, positioning. The single source of brand truth for every stage.
- **`icp-persona-builder`** (Stage 1) — turns the raw persona/segment into a sharp ICP + named persona card (goals, pains, triggers, objections, watering holes, language, awareness stage).
- **`positioning-messaging-architect`** (Stage 2) — builds the positioning statement + value-prop message house for *this* persona. Reviewed by **`positioning-reviewer`** (gate).
- **`content-brief-builder`** → **`blog-post-drafting-engine`** (Stage 3, TOFU) — brief then draft for the top-of-funnel article. Reviewed by **`content-qa-reviewer`** (gate).
- **`content-format-writer-suite`** (Stage 4, MOFU) — the comparison page (format = comparison). Reviewed by **`content-qa-reviewer`** (gate).
- **`lead-nurture-drip-builder`** (Stage 5, BOFU) — the bottom-of-funnel email sequence; use **`welcome-onboarding-email-sequence-builder`** instead when the persona enters via signup/trial. Reviewed by **`lifecycle-email-push-copy-reviewer`** (gate).
- *Pulled by the stages themselves, not by this skill:* `proof-vault` (proof), `cta-variant-generator` (CTAs), `subject-line-preview-text-optimizer` (subject lines), `editorial-style-guide` (house mechanics). Let the specialists call them; do not duplicate.

---

## How a run works

```
Stage 0  Brand        ──► brand-brain  ........... voice · ICP · offer · proof · positioning  (load once, share with every stage)
Stage 1  Persona      ──► icp-persona-builder ..... persona card + awareness stage
                 handoff: persona card ▼
Stage 2  Positioning  ──► positioning-messaging-architect ──► [gate] positioning-reviewer
                 handoff: message house + value pillars + proof ▼
              ┌───────────────────────────── parallel after Stage 2 ─────────────────────────────┐
Stage 3 TOFU │ content-brief-builder ─► blog-post-drafting-engine ─► [gate] content-qa-reviewer   │
Stage 4 MOFU │ content-format-writer-suite (format=comparison)     ─► [gate] content-qa-reviewer   │
              └─────────────────────────────────────────────────────────────────────────────────┘
                 handoff: published-shaped assets (titles, URLs, key claims) ▼
Stage 5  BOFU ──► lead-nurture-drip-builder (or welcome-onboarding-…) ──► [gate] lifecycle-email-push-copy-reviewer
                 ▼
Stage 6  Compile + human approval ──► ./funnel-kits/<persona-slug>/
```

### Stage 0 — Load the brand (always first)
**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest (voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning line, ICP + awareness tendency) and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first. **Produce nothing until it returns.** Every downstream Skill call gets the active slug + digest so no stage re-derives brand context.

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly (this thin read is correct — not a reimplementation); else ask the user for the brand name, ICP + awareness tendency, offer mechanics + destination URL, 3 voice adjectives + banned words, then proceed. Never write `brand.md` yourself.

### Stage 1 — Persona
Invoke **`icp-persona-builder`** with the raw persona description / CRM segment / VOC notes. Capture the returned **persona card** — especially the **awareness stage**, which sets the commitment ceiling for every later asset. This card is the spine; every other stage receives it.

### Stage 2 — Positioning (gated)
Invoke **`positioning-messaging-architect`** with the persona card → get the positioning statement + value-prop message house (three pillars, proof under each). **Gate:** invoke **`positioning-reviewer`** on the result. If verdict is *Revise*, hand its specific notes back to the architect and re-run (max 2 loops); if still weak, surface the blocker to the human rather than building three assets on a shaky foundation. The approved message house is the shared source for all three funnel assets — they must not contradict it.

### Stages 3 & 4 — TOFU + MOFU (parallel, each gated)
These two are independent of each other and run in parallel once Stage 2 is approved.
- **TOFU:** invoke **`content-brief-builder`** (persona + message house + a problem-aware/unaware angle) → then **`blog-post-drafting-engine`** on the approved brief. The brief-stage angle pays off the persona's *top-of-funnel* question, not a product pitch.
- **MOFU:** invoke **`content-format-writer-suite`** with `format = comparison` (persona + message house + the brand's real differentiators/alternatives) for the solution-aware evaluation moment.
- **Gate (both):** invoke **`content-qa-reviewer`** on each draft. *Ship* → keep; *Revise* → return its inline notes to the drafting skill and re-run that asset only (max 2 loops); *Reject* → escalate to the human.

### Stage 5 — BOFU sequence (gated)
Invoke **`lead-nurture-drip-builder`** (default) with the persona, funnel stage = decision, and the TOFU/MOFU assets as the content to reference inside the emails — so the sequence links to real pages, not placeholders. Use **`welcome-onboarding-email-sequence-builder`** instead when the persona converts via signup/trial. **Gate:** invoke **`lifecycle-email-push-copy-reviewer`**; loop on Revise (max 2), escalate on Reject.

### Stage 6 — Compile + human approval
Assemble all approved artifacts into the bundle (below), write the kit README that ties them to the one persona and one message house, then **stop and present to the human for final sign-off** before declaring done. The human is the release gate; the reviewers are the quality gates.

---

## Orchestration logic

- **One persona, one message house, one voice.** Stages 1–2 are the contract; Stages 3–5 consume it and may not invent new positioning or proof. If a downstream stage needs a claim not in the message house or proof vault, mark it `[verify]` — never fabricate to fill the kit.
- **Sequential where there's a dependency, parallel where there isn't.** 0→1→2 are strictly sequential (each feeds the next). 3 and 4 run in parallel after 2. 5 depends on 3+4 (it links to them).
- **Gate after every authored stage; author and review are separate passes.** The skill that wrote a stage never approves its own output — a distinct reviewer skill does. *Revise* loops that one stage with the reviewer's specific notes (cap at 2 loops per stage); persistent *Revise* or any *Reject* is escalated to the human, not papered over.
- **Degrade gracefully, never silently.** If an optional reviewer isn't installed, do a brief self-check against the brand digest and note it in the README; if a core pipeline skill is missing, tell the user which one and stop — do not improvise the stage inline.
- **Resumable.** Each approved artifact is written to disk as it passes its gate, so a re-run skips finished stages instead of regenerating the whole kit.

---

## Bundled deliverable

Saved to a **project-relative** path in the user's CWD (never the skill folder):

```
./funnel-kits/<persona-slug>/
  00-README.md            # the index: persona one-liner, message house summary, asset map, run/gate log, open [verify] items
  01-persona.md           # icp-persona-builder output (the spine)
  02-message-house.md     # positioning statement + value pillars + proof  (positioning-reviewer: approved)
  03-tofu-blog.md         # brief + draft + meta            (content-qa-reviewer: Ship)
  04-mofu-comparison.md   # comparison page                 (content-qa-reviewer: Ship)
  05-bofu-sequence.md     # email nurture sequence + ESP-spec handoff  (lifecycle reviewer: approved)
```

Each asset carries a one-line provenance header: which skill produced it, which reviewer cleared it, and the verdict. The README is the thing a buyer reads first — it must make the through-line obvious: *this* persona → *this* positioning → *these* three assets.

---

## Principles

- **Conduct, don't perform.** Every word of every asset is produced by a specialist skill via the Skill tool. This skill sequences, gates, hands off, and compiles — nothing more.
- **Brand-brain first, once.** Load brand context at Stage 0 and pass it down; never re-resolve, re-scan, or write `brand.md`.
- **The persona is the spine.** One persona, one awareness path, one message house, threaded through all five assets. No asset exceeds the persona's awareness/commitment ceiling for its stage.
- **Truth discipline.** Real proof and real differentiators only; anything unconfirmed is `[verify]`. The kit never invents customers, numbers, quotes, or comparisons to look complete.
- **A gate after every authored stage.** Author and review are separate passes; reviewers loop weak output, the human gives final sign-off.
- **Ship something runnable cold.** The bundle stands alone — a buyer who has never seen this conversation can read the README and execute.

## What not to do

- Don't write personas, positioning, articles, comparison pages, or emails inline — invoke the stage skill.
- Don't write or edit `brand.md`; don't re-implement brand scanning/interviewing — that's `brand-brain`.
- Don't let a downstream asset contradict the approved message house, or invent proof/differentiators to fill a gap.
- Don't self-approve a stage in the same pass that wrote it; don't skip a gate to "save time."
- Don't loop a stage forever — cap at 2 reviewer loops, then escalate to the human.
- Don't run Stage 5 before the TOFU/MOFU assets exist (the emails must link to real pages).
- Don't save the bundle inside the skill folder; always use the project-relative `./funnel-kits/<persona-slug>/`.
- Don't declare done before the human approves Stage 6.

## Quality checklist (self-review before presenting)

- `brand-brain` loaded at Stage 0 (or sanctioned fallback) and its digest passed to every stage?
- All five stages produced by their specialist skills via the Skill tool — nothing authored inline here?
- Every authored stage passed its named reviewer gate (positioning-reviewer, content-qa-reviewer ×2, lifecycle reviewer); Revise loops capped at 2; Rejects escalated?
- One persona + one message house threaded through all assets; no asset exceeds its stage's awareness ceiling; no asset contradicts the positioning?
- Only real proof used; every unconfirmed claim/number marked `[verify]`; nothing fabricated to fill the kit?
- BOFU sequence links to the actual TOFU/MOFU assets, not placeholders?
- Bundle written to `./funnel-kits/<persona-slug>/` with all six files, provenance headers, and a README that makes the through-line obvious?
- Run/gate log captured and presented to the human for final sign-off before declaring done?
