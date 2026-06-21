---
name: full-campaign-launch-orchestrator
description: >
  The flagship "growth team in a box" — campaign goal + ICP + budget in, and a launch-ready campaign
  bundle out the door in one session: a structured brief, a big creative idea, channel-correct ad copy,
  landing/product-page copy, a welcome/onboarding email sequence, a social content calendar, a complete
  UTM-tagged tracking-link set, and a pre-launch QA checklist. It does NOT do the underlying work itself
  — it CHAINS the library's specialist skills in order, handing each stage's output to the next, with
  human approval gates and a Revise loop on weak output. It calls `brand-brain` first so every asset is
  on-voice, ICP-aligned, and uses only real proof. Use whenever the user says "launch a campaign,"
  "run a full campaign," "I need everything for [campaign]," "build me the whole campaign," "spin up a
  launch," "end-to-end campaign," "campaign in a box," or hands over a goal + audience + budget and wants
  assets ready to ship, not advice. Use the single-purpose siblings directly when the user only needs one
  artifact (just ad copy, just a UTM set); use this when they want the whole launch assembled and QA'd.
---

# Full Campaign Launch Orchestrator

Point it at a campaign goal, an ICP, and a budget. Get back a launch-ready bundle — brief, big idea, ad copy across channels, landing-page copy, a welcome sequence, a social calendar, UTM-tagged links, and a QA checklist — produced in one session by chaining the specialist skills that already exist. This is the skill a buyer points at and says "this replaces a junior marketer's week."

It is an **orchestrator, not a re-implementer**. It does not write ad copy itself, build UTMs itself, or invent a QA list — it invokes the sibling skill that owns each job and threads the output forward. Its value is the *run order, the handoffs, the gates, and the assembled deliverable* — the connective tissue a single skill can't provide.

This skill produces launch assets. It does not buy media, set bids, or push anything live — handoff to the human and to ops at the end.

---

## Skills this calls

In pipeline order (each is a separate, already-built skill invoked via the Skill tool):

1. **`brand-brain`** (required, always first) — loads the active brand's voice, ICP, offer, proof, positioning, banned words. The orchestrator never writes `brand.md` itself.
2. **`campaign-brief-builder`** — turns goal + audience + budget + channels into a structured brief (objective, targeting, KPIs, creative spec).
3. **`campaign-concept-developer`** — turns the brief into a big idea, hero message, and per-channel execution angles.
4. **`ad-copy-variant-generator`** — channel-format-correct, char-limit-hard ad copy per channel (it internally calls `cta-variant-generator`).
5. **`landing-product-page-copy-writer`** — the destination page the ads point to (message-matched to the concept).
6. **`welcome-onboarding-email-sequence-builder`** — the post-conversion email sequence that activates new signups.
7. **`social-content-calendar-builder`** — the organic amplification calendar across channels and dates.
8. **`utm-parameter-bulk-builder`** — one consistent UTM-tagged link set for every destination above (the single tracking-link source of truth).
9. **`campaign-qa-launch-checklist-generator`** — the channel-specific pre-launch pass/fail checklist run against the whole bundle.

**Optional, when the run needs it (invoke only if relevant — never bloat the bundle):** `proof-vault` / `objection-library-builder` (when concept or pages need real proof or objection handling), `icp-persona-builder` / `positioning-messaging-architect` (when brand-brain's ICP/positioning is thin), `headline-hook-generator` (extra hook spread), `subject-line-preview-text-optimizer` (sharpen email subjects), `push-notification-copy-generator` (if push is a channel), `content-qa-reviewer` / `de-slop-humanize-pass` (deeper copy QA before the checklist), `landing-page-heuristic-live-cro-auditor` (if a live URL exists), `a-b-multivariate-test-designer` (turn the variant spread into a real test). Synthesize inline only if a required sibling is somehow unavailable, and say so.

---

## How a run works

```
Stage 0  Brand        ─► brand-brain                         → digest (voice, ICP, offer, proof) + brand.md path
Stage 1  Brief        ─► campaign-brief-builder              → BRIEF (goal, audience, KPIs, channels, budget split)
            └─ GATE A: human confirms brief before any asset is written
Stage 2  Concept      ─► campaign-concept-developer          → BIG IDEA + hero message + per-channel angles
            └─ GATE B: human picks the concept direction
Stage 3  Assets (parallel fan-out, all fed Brief + Concept + brand digest):
            ├─ ad-copy-variant-generator                     → ad copy per channel
            ├─ landing-product-page-copy-writer              → destination page copy
            ├─ welcome-onboarding-email-sequence-builder     → activation email sequence
            └─ social-content-calendar-builder               → organic calendar
            └─ content-qa-reviewer loop on each → Revise/Approve  (see Orchestration logic)
Stage 4  Tracking     ─► utm-parameter-bulk-builder          → UTM link set for every destination from Stages 3–5
Stage 5  QA gate      ─► campaign-qa-launch-checklist-generator → pass/fail checklist over the whole bundle
            └─ GATE C: human launch approval
Stage 6  Assemble     ─► write the bundle to ./campaigns/[slug]/ and present the index
```

### Stage 0 — Load the brand (always first)
**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the `brand.md` path. If the brand is new, brand-brain bootstraps it first. **Produce nothing downstream until it returns.** Pass the digest to every later stage so no sibling re-derives brand context.

*Fallback if brand-brain is absent:* read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words). Always prefer the call.

### Stage 1 — Brief (`campaign-brief-builder`)
Hand it the goal, ICP, budget, timeline, and candidate channels (plus the brand digest). It returns the structured brief. **GATE A:** show the brief and get an explicit confirm/edit before spending tokens on assets — a wrong objective here poisons everything downstream. If inputs are missing (no budget, no goal, no deadline), ask the 3–4 blocking questions now, not later.

### Stage 2 — Concept (`campaign-concept-developer`)
Pass the confirmed **brief** + brand digest. It returns the big idea, hero message, and per-channel execution angles. **GATE B:** present 1–2 concept directions and let the human pick one — the concept is the creative spine every Stage-3 asset must stay true to. Lock the chosen hero message; it is the message-match anchor for ads → page → emails.

### Stage 3 — Asset fan-out (parallel where the tools allow)
Feed **every** Stage-3 sibling the same package — the brief, the locked concept/hero message, and the brand digest — so the ad headline, the landing-page hero, and the welcome email all tell one story. Each sibling owns its craft (char limits, format rules, sequence logic); the orchestrator only supplies context and collects output. Run the assets that don't depend on each other in parallel.

### Stage 4 — Tracking (`utm-parameter-bulk-builder`)
Collect every destination URL produced in Stage 3 (each ad's landing URL, each social post's link, each email CTA target) and hand the full list to the UTM builder with one consistent naming convention. It is the single source of truth for tracking links — do not let any earlier stage invent its own UTMs.

### Stage 5 — QA gate (`campaign-qa-launch-checklist-generator`)
Hand the assembled bundle (brief + concept + all assets + UTM set) to the checklist generator. It returns a channel-specific pass/fail list covering copy, creative, UTMs, targeting, tracking, and legal. **GATE C:** present the checklist and the bundle index for human launch approval. Any **Fail** blocks launch and routes back to the owning stage.

### Stage 6 — Assemble & deliver
Write the bundle to a **project-relative** folder (see *Bundled deliverable*) and present a one-screen index with the path to every artifact and the QA verdict.

---

## Orchestration logic (gates, branches, the Revise loop)

- **Gates are human checkpoints, not auto-passes.** GATE A (brief), GATE B (concept), and GATE C (launch) each require an explicit human OK. Default to pausing; only auto-continue if the user said "run the whole thing, don't stop."
- **Revise loop on weak copy.** After each Stage-3 asset, run it through `content-qa-reviewer` (or a focused self-review if absent). If it returns **Revise**, send the specific notes back to the *owning* sibling and regenerate — up to **2 loops**. Still weak after 2? Stop the loop, flag the asset `[needs human]` in the bundle, and keep going — never block the whole launch on one stubborn asset.
- **Upstream-weakness branch.** If a sibling reports the *input* is the problem (thin offer, weak ICP, no real proof), don't paper over it: branch to the relevant foundation skill (`offer-pricing-brain`, `icp-persona-builder`, `proof-vault`) or surface it at the nearest gate. A clever button can't fix a broken offer.
- **Message-match enforcement.** The locked hero message from Stage 2 is the through-line. At Stage 5, verify ad → landing page → email all carry it; a mismatch is a QA **Fail**, not a stylistic nit.
- **Channel pruning.** Only generate assets for channels in the confirmed brief. If budget can't support a channel, drop it at GATE A rather than producing copy nobody will run.
- **Truth discipline across the chain.** Every unconfirmed number, proof point, or claim stays `[verify]` end-to-end; the orchestrator never "cleans up" a `[verify]` into a hard claim during assembly.

---

## Bundled deliverable

Write everything to a **project-relative path** — leading `./` resolves to the user's CWD/project, never the skill folder:

```
./campaigns/[campaign-slug]/
  00-INDEX.md                  ← bundle index: every artifact + path + QA verdict + open [verify]/[needs human] items
  01-brief.md                  ← from campaign-brief-builder
  02-concept.md                ← big idea, hero message, per-channel angles
  03-ad-copy.md                ← per-channel ad variants
  04-landing-page-copy.md      ← destination page copy
  05-welcome-sequence.md       ← onboarding email sequence
  06-social-calendar.md        ← organic calendar
  07-utm-links.csv             ← UTM-tagged link set (CRM/sheet-importable)
  08-qa-checklist.md           ← pass/fail launch checklist
```

`00-INDEX.md` is the cover sheet: the campaign in one screen — goal, brand, channels, the path to each artifact, the QA verdict, and a flagged list of every `[verify]` and `[needs human]` item still open. If the user prefers a single file, compile the same sections into one `./campaigns/[campaign-slug]/campaign-bundle.md`. Always confirm the save and print the absolute path.

---

## Principles (Non-Negotiable)

- **Brand-brain first, always.** No asset before brand-brain returns. Its voice + banned words override everything downstream.
- **Orchestrate, don't re-implement.** Every stage is a sibling skill's job — invoke it, don't rebuild it. The orchestrator's only craft is sequencing, handoffs, gates, and assembly.
- **One story, end to end.** The locked hero message threads ad → page → email → social. Message match is enforced, not hoped for.
- **Gates are real.** Brief, concept, and launch each get explicit human approval. Don't ship a bundle nobody signed off on.
- **Weak output loops, then flags.** Revise up to 2×, then mark `[needs human]` and continue — the pipeline degrades gracefully, it doesn't stall.
- **Truth survives assembly.** `[verify]` stays `[verify]`. No invented proof, numbers, customers, or urgency anywhere in the chain.
- **Project-relative output.** The bundle lands in the user's project, never inside the skill folder.

## What Not to Do

- Don't write ad copy, page copy, sequences, UTMs, or the QA list yourself — call the sibling that owns each.
- Don't write `brand.md` — that's brand-brain's job (only the bootstrapper drives brand-brain's own bootstrap).
- Don't skip gates because it's faster; don't auto-launch; this skill never pushes assets live or sets bids.
- Don't generate assets for channels not in the confirmed brief, and don't let any stage mint its own UTMs.
- Don't paper over a thin offer/ICP/proof with clever copy — branch to the foundation skill or flag it at a gate.
- Don't promote a `[verify]` to a hard claim, invent proof, or merge two brands' context.
- Don't loop forever on a weak asset — 2 Revise passes, then `[needs human]` and move on.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded/bootstrapped before any asset?
- Brief, concept, all four asset types, UTM set, and QA checklist each produced *by their owning sibling* — nothing re-implemented inline?
- GATE A (brief), GATE B (concept), GATE C (launch) each hit with explicit human approval (or an explicit "run it all" waiver)?
- Locked hero message carried through ad → landing page → email → social; message-match verified at QA?
- Every weak asset looped ≤2× then flagged `[needs human]`; every unconfirmed claim still `[verify]`?
- UTMs are one consistent set covering every destination; no stage minted its own?
- QA checklist run over the *whole* bundle; any **Fail** routed back to the owning stage, not waved through?
- Bundle written to `./campaigns/[slug]/` with a complete `00-INDEX.md`, save confirmed, absolute path printed?
