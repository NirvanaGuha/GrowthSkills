---
name: abm-account-to-outreach-kit
description: >
  Target account list + ICP → personalized research brief, LinkedIn connection note, and cold email
  sequence per account — hand-crafted at scale. The flagship ABM orchestrator: a "growth team in a box"
  that chains four sibling skills end-to-end into ONE bundled deliverable a buyer can run cold. It scores
  the list (account-list-builder-icp-scorer), builds a dossier per Tier-1 account (account-dossier-builder),
  writes a multi-touch sequence personalized to each (cold-outreach-sequence-architect), and packs a
  pre-call brief + follow-up for booked meetings (meeting-prep-follow-up-pack). It does NOT re-implement
  any stage — it invokes each via the Skill tool and manages handoffs, gates, and approvals between them.
  Brand context (voice, ICP, offer, proof) comes from the brand-brain skill, loaded once and passed down.
  Use whenever the user says "run ABM," "build an outreach kit," "turn this account list into outreach,"
  "personalized cold email at scale," "ABM campaign for these accounts," "account-based outreach,"
  "research and write outreach for my target list," or hands over a list of target accounts and an offer
  and wants research + sequences, not just one email. Produces a saved campaign folder, not a single asset.
---

# ABM Account-to-Outreach Kit

Point it at a list of target accounts and your offer. Get back a ready-to-run account-based campaign: a scored, tiered account file; a sourced research dossier for every Tier-1 account; a personalized multi-touch email + LinkedIn sequence written off that dossier; and a meeting-prep + follow-up pack for the accounts that book. One run replaces a junior SDR's week of list-cleaning, company research, and first-draft sequencing.

This is an **orchestrator**, not a writer. It owns the run order, the handoffs, the gates, and the bundled output. Every stage is done by the sibling skill that specializes in it — this skill never re-implements list scoring, account research, sequence copy, or meeting prep. It loads the brand once, feeds each stage the previous stage's output, stops at the human gates, and compiles the result into a campaign folder.

---

## Skills this calls

The pipeline, in order. All are real, installed sibling skills — invoke each via the **Skill tool**; never re-derive their work here.

1. **`brand-brain`** (required, Layer 0) — resolves and loads the active brand: voice, banned words, ICP + awareness tendency, offer mechanics + destination URLs, real proof, positioning. Loaded **once** at Stage 0 and threaded into every later stage so the whole kit is on-voice and offer-true.
2. **`account-list-builder-icp-scorer`** — raw account list + ICP → a clean, deduped, Tier 1/2/3 scored account file with a score rationale per row.
3. **`account-dossier-builder`** — one company → a sourced intelligence dossier (business model, signals, stack, org hints, recent news, priorities, pain hypotheses tied to the offer). Runs **once per Tier-1 account**.
4. **`cold-outreach-sequence-architect`** — ICP segment + offer + channel mix → a full multi-touch email + LinkedIn sequence, personalized per account from its dossier.
5. **`meeting-prep-follow-up-pack`** — dossier (or booked-meeting notes) → a pre-call brief + a personalized post-meeting follow-up. Runs on demand for accounts that reply or book.

*Optional, when the run calls for it (invoke only if installed and relevant):* `competitive-intelligence-dossier` (when an account has an incumbent to displace), `battlecard-objection-handler` (to arm reps for the objections a dossier surfaces), `positioning-reviewer` (to sanity-check the offer framing before sequencing at scale). Skip silently when absent.

---

## How a run works

```
Stage 0  Load the brand        ──► brand-brain  → brand digest + brand.md path
            │  (passes: voice, banned words, ICP, offer mechanics, proof, positioning)
            ▼
Stage 1  Score & tier the list ──► account-list-builder-icp-scorer
            │  HANDOFF: Tier 1/2/3 account file (company, URL, score, rationale)
            ▼
       ┌─ GATE A (human) ──► confirm Tier-1 set + per-run cap before any research ─┐
            ▼
Stage 2  Build dossiers        ──► account-dossier-builder  (LOOP: 1 call / Tier-1 account)
            │  HANDOFF: one sourced dossier per account (facts vs [infer], pain hypotheses)
            ▼
Stage 3  Write sequences       ──► cold-outreach-sequence-architect  (1 sequence / account)
            │  input: brand digest + that account's dossier + offer + channel mix
            │  HANDOFF: per-account multi-touch sequence (LinkedIn note + email steps)
            ▼
       ┌─ GATE B (human) ──► review a sample sequence; approve voice/personalization ─┐
            ▼
Stage 4  Meeting packs         ──► meeting-prep-follow-up-pack  (on reply / booked call)
            ▼
Stage 5  Compile & save        ──► ./abm/<campaign-slug>/ campaign folder + index
```

### Stage 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest (voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, real proof, positioning) and the `brand.md` path. If the brand is new, `brand-brain` bootstraps it first. **Do not run any stage until it returns.** Thread the digest into Stages 2–4 so every dossier pain-hypothesis, every subject line, and every follow-up is on-voice and offer-true.

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; else ask the user for the brand's voice/banned words, ICP + awareness stage, offer mechanics + destination URL, and 2–3 real proof points, then proceed. This thin read is the sanctioned fallback — never write `brand.md` yourself.

### Stage 1 — Score & tier the list
Invoke **`account-list-builder-icp-scorer`** with the raw list (CSV / pasted rows / seed criteria) + the brand's ICP from Stage 0. It returns a deduped, Tier 1/2/3 file with a score and one-line rationale per account. The orchestrator does not score rows itself.

### GATE A — human confirms the target set (before any research)
Dossier research is the expensive stage. Present the Tier-1 set and ask the human to confirm which accounts to research and a **per-run cap** (default: top 10 Tier-1). Do not silently research a 500-row list. Tier-2/3 stay queued for a later batch.

### Stage 2 — Build one dossier per Tier-1 account (loop)
For each confirmed Tier-1 account, invoke **`account-dossier-builder`** with the company name + URL (and the brand digest, so pain hypotheses map to *this* offer). It returns a sourced dossier separating facts from `[infer]`. One call per account — these are independent and may run in parallel where the harness allows. Carry each dossier's `[infer]`/`[verify]` flags downstream untouched.

### Stage 3 — Write a sequence per account
For each account, invoke **`cold-outreach-sequence-architect`** with: the brand digest (voice + offer + proof), that account's dossier (the personalization fuel), and the channel mix (default email + LinkedIn). It returns a multi-touch sequence — LinkedIn connection note + email steps with subject lines, bodies, and timing. The orchestrator supplies inputs and collects output; it does not write copy.

### GATE B — human approves a sample
Before accepting the full batch, surface **one** representative sequence (ideally the top-scored account) for the human to approve on voice, personalization depth, and offer accuracy. Approve → accept the batch. **Revise** → apply the note and re-invoke the sequence stage for the affected accounts (loop, don't hand-patch). This keeps authoring and review in separate passes.

### Stage 4 — Meeting packs (on demand)
When an account replies or books, invoke **`meeting-prep-follow-up-pack`** with that account's dossier (plus any meeting notes) for a pre-call brief + follow-up email. Optional at first run; always available as the campaign progresses.

### Stage 5 — Compile & save
Assemble everything into a **project-relative** campaign folder (the user's CWD, never the skill folder) and confirm the path.

---

## Orchestration logic (gates, branches, loops)

- **Sequential spine, parallel middle.** Stages run in order, but Stage 2 dossiers (and the Stage 3 sequences that follow each) are per-account and independent — fan them out where the harness allows; never block account B's research on account A's.
- **Two human gates, no auto-spend past them.** GATE A caps research before it starts; GATE B approves voice before the batch is accepted. Never skip a gate to "save a step."
- **Weak-output handling — loop, don't paper over.**
  - Stage 1 returns a thin or unscorable list (missing firmographics) → report the gap and ask for more seed criteria; don't fabricate tiers.
  - A dossier comes back thin (mostly `[infer]`, no real signals) → flag that account as **low-confidence**, drop it from the Tier-1 batch or down-tier it, and do **not** let Stage 3 invent specifics to compensate.
  - GATE B returns **Revise** → re-invoke `cold-outreach-sequence-architect` for the flagged accounts with the note; re-review. Loop until approved or the human stops.
- **Branch on incumbent.** If a dossier names a competitor to displace, optionally invoke `competitive-intelligence-dossier` / `battlecard-objection-handler` for that account before sequencing. Skip when no incumbent.
- **Idempotent & resumable.** Write each stage's output as it completes so a re-run skips finished accounts. Don't redo a dossier that already exists in the campaign folder unless asked to refresh.

---

## Bundled deliverable

Save to a **project-relative** path under the user's CWD — never the skill folder, never an absolute home path:

```
./abm/<campaign-slug>/
├── README.md                  # index: brand, offer, channel mix, account count, run date, stage status
├── 00-accounts-scored.md      # Stage 1: Tier 1/2/3 file + score rationale (CSV alongside if provided)
├── dossiers/
│   └── <account-slug>.md      # Stage 2: one sourced dossier per Tier-1 account
├── sequences/
│   └── <account-slug>.md      # Stage 3: LinkedIn note + email steps + timing, per account
├── meeting-packs/
│   └── <account-slug>.md      # Stage 4: pre-call brief + follow-up (as accounts book)
└── campaign-log.md            # gate decisions, revisions, low-confidence/skipped accounts, [verify] queue
```

`README.md` is the single entry point a rep opens. `campaign-log.md` records every gate decision and every `[verify]` item so nothing fabricated leaks into a real conversation. Confirm the folder path in one line when done.

---

## Principles

- **Brand-brain first, threaded everywhere.** No stage runs before `brand-brain` returns; its voice + banned words override all downstream copy.
- **Orchestrate, don't re-implement.** Every stage is a Skill-tool call to the specialist sibling. This skill owns sequence, handoffs, gates, and the bundle — nothing else.
- **Humans gate spend and voice.** Confirm the target set before research; approve a sample before the batch.
- **Personalization rides on real research.** A sequence is only as good as its dossier; carry `[infer]`/`[verify]` flags through untouched, never invent specifics to fill a thin dossier.
- **Truth discipline.** Real names, signals, and proof only — everything else is `[verify]`. Never fabricate a customer, a number, a quote, or a "I saw your recent…" hook the dossier didn't support.
- **One job per stage, separate passes.** Authoring (Stages 2–4) and review (GATE B) stay in distinct passes; the orchestrator never self-approves the copy it just generated.

## What not to do

- Don't write CTAs, dossiers, sequences, or briefs inline — invoke the sibling skills.
- Don't re-implement brand scanning/interviewing/storage — call `brand-brain` (or use the sanctioned thin-read fallback); never author `brand.md`.
- Don't research a full list before GATE A, or accept a batch before GATE B.
- Don't fabricate firmographics, signals, proof, or personalization hooks; don't strip a sibling's `[infer]`/`[verify]` flags.
- Don't hand-patch a rejected sequence — re-invoke the sequence stage and re-review.
- Don't save the deliverable inside the skill folder or to an absolute home path — use the project-relative `./abm/<campaign-slug>/`.

## Quality checklist (self-review before presenting)

- `brand-brain` invoked at Stage 0 and its digest threaded into Stages 2–4 (or the documented fallback used)?
- Stage 1 ran via `account-list-builder-icp-scorer` and produced a tiered, scored file — no orchestrator-invented tiers?
- GATE A passed: human confirmed the Tier-1 set + per-run cap before any dossier ran?
- One `account-dossier-builder` call per Tier-1 account; thin dossiers flagged low-confidence and down-tiered, not padded?
- One `cold-outreach-sequence-architect` sequence per account, built from its dossier + brand voice; `[verify]`/`[infer]` flags preserved?
- GATE B passed: a sample sequence approved; any Revise looped back through the sequence stage, not hand-edited?
- Bundle saved to project-relative `./abm/<campaign-slug>/` with README index + campaign-log; folder path confirmed?
- No fabricated accounts, numbers, quotes, or personalization hooks anywhere in the kit?
