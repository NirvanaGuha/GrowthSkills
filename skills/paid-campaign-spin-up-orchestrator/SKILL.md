---
name: paid-campaign-spin-up-orchestrator
description: >
  Product + audience + budget → a complete, QA-gated paid campaign ready to activate: a campaign
  brief, a match-type-segmented keyword list (search) AND/OR an audience-targeting spec (social),
  on-voice ad copy variants per format, a validated UTM tracking sheet, and a launch checklist with
  a go/no-go verdict. This is a FLAGSHIP ORCHESTRATOR — a "growth team in a box" that chains six
  sibling skills end-to-end into one bundled deliverable a buyer can run cold, with human approval
  gates and a revise-loop when any stage is weak. It does NOT re-implement any stage; it invokes
  each specialist skill via the Skill tool and wires the handoffs. Brand context comes from the
  shared `brand-brain` skill so every asset is on-voice, ICP-aligned, and multi-brand aware. Use
  when the user says "spin up a paid campaign," "launch a Google/Meta campaign," "build me an ad
  set," "set up a search campaign," "I have a product/budget, get me ready to launch," "end-to-end
  paid campaign," or hands over a product + audience + budget and wants launch-ready assets, not
  advice. Orchestrates and assembles only — it does not place spend or touch ad-platform accounts.
---

# Paid-Campaign Spin-Up Orchestrator

Hand it a product, an audience, and a budget. Get back a folder a media buyer could activate the same afternoon: the strategic brief, the keyword list or audience spec, the ad copy, the tracking, and a QA gate that says go or no-go. One run replaces a junior marketer's week of stitching tools together.

This skill is a **conductor, not a soloist**. It owns the *run order, the handoffs, the gates, and the assembled deliverable* — and nothing else. Every unit of real work (writing the brief, building keywords, speccing audiences, writing ad copy, building UTMs, running the QA checklist) is done by a dedicated sibling skill invoked through the Skill tool. It never re-derives brand context and never reimplements a stage.

---

## Skills this calls

Brand context first, then the ordered pipeline. All names are real, installed siblings.

- **`brand-brain`** (required, Layer-0) — resolves and loads the active brand's voice, banned words, ICP + awareness tendency, offer mechanics + destination URLs, and real proof. Called once up front; the digest threads into every stage.
- **`campaign-brief-builder`** — Stage 1. Turns product + audience + budget + objective into the campaign brief (goal, KPI + target, channel decision, budget split, messaging angle, offer, dates).
- **`keyword-list-builder-segmenter`** — Stage 2a (search channels). Seed terms + intent → match-type-segmented keyword list grouped by ad group, with suggested bids and negative seeds.
- **`audience-targeting-spec-writer`** — Stage 2b (social/display channels). Goal + ICP → platform-ready targeting + retargeting spec (Meta/LinkedIn/Google) with window, signal, and exclusion logic.
- **`ad-copy-variant-generator`** — Stage 3. Brief + targets → headline/description variants per format and channel, on-voice via the brand digest.
- **`utm-parameter-bulk-builder`** — Stage 4. Destination URLs + the brief's naming convention → a validated, deduped UTM tracking sheet, one tagged URL per ad/variant.
- **`campaign-qa-launch-checklist-generator`** — Stage 5 (gate). The full assembled bundle → a launch checklist with pass/fail items and a go/no-go verdict.

*Optional, when installed (deepen a stage; skip silently if absent):* `competitor-ad-library-spy` (creative angles before Stage 3), `ad-to-landing-page-message-match-auditor` (Stage 4.5, ad↔LP alignment), `bid-budget-pacing-checker` (sanity-check the budget split in Stage 1), `sample-size-calculator` (size any A/B you set up), `cta-variant-generator` (sharpen the in-ad CTA). Invoke them too — never inline their work.

---

## How a run works

The pipeline is a relay: each stage consumes the prior stage's artifact and emits the next. **Invoke each via the Skill tool; do not write any stage's output yourself.**

```
Stage 0  brand-brain ............... load brand digest (gate: must return before Stage 1)
Stage 1  campaign-brief-builder .... brief.md  ── HUMAN GATE A: approve brief ──┐
                                                                                 ▼
Stage 2  (channel branch, run the two tracks IN PARALLEL where both apply)
  2a search → keyword-list-builder-segmenter ...... keywords.md
  2b social → audience-targeting-spec-writer ...... audiences.md
                                                                                 ▼
Stage 3  ad-copy-variant-generator ............... ad-copy.md  (per format/channel)
Stage 4  utm-parameter-bulk-builder .............. utm-tracking.csv
Stage 5  campaign-qa-launch-checklist-generator .. qa-checklist.md + GO / NO-GO
                                       │
                  NO-GO / Revise ──────┘ loop back to the failing stage, re-run, re-QA
                                       │
                              GO ──► HUMAN GATE B: approve launch → assemble bundle
```

### Stage 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Use the returned digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness — as a hard override for every downstream stage, and pass it into each sub-skill call so none of them re-derive it. **Do not produce any campaign asset until it returns.**

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; else ask the user for the brand's name, ICP + awareness stage, offer mechanics + primary destination URL, and 3 voice adjectives + banned words, then proceed. This thin read is the *correct* fallback — it is not a reimplementation. Never write `brand.md` yourself; that is `brand-brain`'s job alone.

### Stage 1 — Campaign brief (the spine)

Collect the three inputs the deliverable is named for — **product, audience, budget** — plus objective, channel(s), and timeline (ask only for what's missing; infer ICP from the brand digest). **Invoke `campaign-brief-builder`**, passing inputs + brand digest. It returns `brief.md`: objective, single primary KPI + target, channel decision, budget split, core messaging angle, offer, and launch window. *(Optional: run `bid-budget-pacing-checker` to sanity-check the split.)* **→ HUMAN GATE A:** present the brief and get explicit approval before any asset is built — the brief is the contract every later stage is graded against. If the user revises, re-run Stage 1 with their edits.

### Stage 2 — Targeting (branch by channel; parallelize)

Read the brief's channel decision and run the matching track(s). When the campaign spans both search and social, run **2a and 2b in parallel** — they are independent and each only needs the brief + brand digest.

- **2a — Search (Google/Bing).** **Invoke `keyword-list-builder-segmenter`** with the brief's seed themes + intent notes → `keywords.md` (ad groups, match types, suggested bids, negative seeds).
- **2b — Social/display (Meta/LinkedIn/Google Display/Demand Gen).** **Invoke `audience-targeting-spec-writer`** with the brief's ICP + objective → `audiences.md` (targeting layers, retargeting windows, exclusion/suppression logic).

*(Optional, before Stage 3: `competitor-ad-library-spy` on a named competitor for live angle intel — feed findings into the copy brief, don't copy creatives.)*

### Stage 3 — Ad copy

**Invoke `ad-copy-variant-generator`**, passing the brief (angle + offer), the Stage 2 targets (ad groups / audience segments — copy is written *per* group so message-match holds), and the brand digest. It returns `ad-copy.md`: headline + description variants per format and channel, inside each platform's character limits. *(Optional: `cta-variant-generator` to sharpen the in-ad CTA.)*

### Stage 4 — Tracking

**Invoke `utm-parameter-bulk-builder`** with the brief's naming convention and the destination URL(s) for every ad/variant → `utm-tracking.csv`: one validated, deduped tagged URL per ad. *(Optional Stage 4.5: `ad-to-landing-page-message-match-auditor` to confirm each ad's promise pays off on its destination page before you tag it.)*

### Stage 5 — QA gate (the go/no-go)

**Invoke `campaign-qa-launch-checklist-generator`** with the *entire* assembled bundle (brief + targets + copy + UTMs). It returns `qa-checklist.md` with pass/fail items and a verdict.

- **GO** → proceed to assembly + Human Gate B.
- **NO-GO / Revise** → identify the failing stage from the checklist, **re-run only that stage** (re-invoke its sub-skill with the QA notes as input), then re-run Stage 5. **Cap the loop at 2 iterations**; if it still fails, stop and surface the blocking issues to the user with the partial bundle — do not ship a campaign that failed QA, and do not fabricate a passing checklist.

### Assembly + Human Gate B

On GO, write the bundle (below), then present the manifest and the go/no-go verdict to the user for the **final launch approval**. The skill assembles and recommends; the human authorizes the launch. This skill never places spend or touches an ad-platform account.

---

## Bundled deliverable

Write a campaign folder to a **project-relative** path in the user's CWD (never the skill folder):

```
./paid-campaign/[brand-slug]-[campaign-slug]/
  00-README.md          run manifest: inputs, channel branch taken, stage status, QA verdict, gate decisions, [verify] list
  01-brief.md           ← campaign-brief-builder
  02a-keywords.md       ← keyword-list-builder-segmenter   (search track)
  02b-audiences.md      ← audience-targeting-spec-writer    (social track)
  03-ad-copy.md         ← ad-copy-variant-generator
  04-utm-tracking.csv   ← utm-parameter-bulk-builder
  05-qa-checklist.md    ← campaign-qa-launch-checklist-generator (incl. GO / NO-GO)
```

Include only the tracks that ran (search-only or social-only campaigns omit the other). `00-README.md` is the buyer's table of contents and audit trail: what went in, which stages ran, every approval, the QA verdict, and every `[verify]` flag still open. Confirm the saved path in one line when done.

---

## Principles

- **Conduct, don't perform.** Every stage's work is done by its sibling skill via the Skill tool. This skill owns sequencing, handoffs, gates, and assembly — nothing else. If you're tempted to write keywords or ad copy directly, you're in the wrong skill.
- **Brand-brain first, always.** No asset before `brand-brain` returns. Its voice + banned words override every stage; the digest threads into every sub-skill call so none re-derive context.
- **The brief is the contract.** Everything downstream is graded against the approved brief. No assets are built before Human Gate A.
- **Gate, don't guess.** Two human gates (brief, launch) and one machine gate (QA). The QA verdict is binding: NO-GO means loop, not ship.
- **Pass artifacts, not prose.** Each stage hands the next the actual file/object it produced, plus the brand digest — so message-match and naming stay consistent end-to-end.
- **Parallel where independent, sequential where dependent.** Search and social tracks run together; copy waits on targeting; tracking waits on copy.
- **Truth discipline.** Use only the brand's real proof, offer, and numbers. Anything unconfirmed (bid estimates, volumes, benchmark CPCs) is marked `[verify]` and carried into the README. Never invent metrics, audiences, competitors, or quotes.
- **Assemble, don't activate.** Output is launch-*ready*, not launched. The human authorizes spend.

## What not to do

- Don't write any stage's output yourself — invoke the sibling skill. No inlined keyword lists, audience specs, ad copy, UTM sheets, or QA checklists.
- Don't reimplement brand resolution/scanning/interviewing, or write `brand.md` — that lives in `brand-brain`.
- Don't skip Human Gate A and start building assets off an unapproved brief.
- Don't ship on a NO-GO, loop more than twice without surfacing blockers, or fabricate a passing QA checklist.
- Don't run the search and social tracks serially when both apply — parallelize them.
- Don't write the bundle into the skill folder; always use the project-relative `./paid-campaign/` path.
- Don't place spend, log into ad platforms, or claim the campaign is "live." It is ready to activate.
- Don't invent proof, audience sizes, bids, or competitor claims; mark estimates `[verify]`.

## Quality checklist (self-review before presenting)

- `brand-brain` invoked first and the active brand loaded (or the sanctioned fallback used)? Digest passed into every sub-skill call?
- Every stage produced by its **sibling skill via the Skill tool** — nothing inlined?
- Brief approved at Human Gate A before any asset was built?
- Correct channel branch taken — search → keywords, social → audiences — and both tracks run in parallel when the campaign spans both?
- Ad copy written per ad group / audience segment so message-match holds; inside each platform's character limits?
- One validated, deduped UTM per ad/variant, following the brief's naming convention?
- QA gate run on the *full* bundle; verdict honored (GO → assemble; NO-GO → loop the failing stage, capped at 2, else surface blockers)?
- Bundle saved to the project-relative `./paid-campaign/[brand]-[campaign]/` path with a `00-README.md` manifest, every track that ran, the QA verdict, gate decisions, and all open `[verify]` items?
- Final launch presented for Human Gate B; no spend placed, no platform touched?
