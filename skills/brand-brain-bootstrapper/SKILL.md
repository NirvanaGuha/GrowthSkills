---
name: brand-brain-bootstrapper
description: >
  Flagship orchestrator: company URL + optional PDF/deck → a fully populated brand brain (voice,
  ICP, positioning/differentiators, offer/pricing, real proof points, objections, editorial style)
  that every other skill in the library then reads. This is the "set up my brand once, properly"
  run — it drives `brand-brain`'s own Mode B bootstrap and orchestrates the eight component skills
  (`brand-voice-codifier`, `icp-persona-builder`, `positioning-messaging-architect`,
  `competitive-intelligence-dossier`, `offer-pricing-brain`, `proof-vault`,
  `objection-library-builder`, `editorial-style-guide`) in disciplined waves, then validates the
  result. It does NOT re-implement brand scanning, interviewing, or storage — it INVOKES
  `brand-brain` as the system of record. Use when the user says "set up my brand brain," "bootstrap
  my brand," "onboard a brand from our site," "build the brand foundation," "do the full brand
  setup," "give me a brand brain from this URL/deck," or hands over a company URL + assets and wants
  the whole foundation built end-to-end. The output is a reusable `brand.md` plus companion files
  (`personas.md`, `competitors.md`, `proof.md`, `objections.md`, `style-guide.md`) that unlock every
  downstream copy, SEO, paid, lifecycle, and sales skill. One brand per run; multi-brand and
  agency-ready via slugs.
---

# Brand-Brain Bootstrapper

Point this at a company URL (and any deck, PDF, or one-pager you have) and walk away with a complete, reusable brand brain: voice and banned words, ICP and personas, positioning and differentiators, offer and pricing mechanics, real proof, the objection map, and an editorial style guide — all written into the library's system of record so every other skill reads it instead of re-deriving it. This is the foundation that turns "ten disconnected copy tools" into "a growth team that knows your brand."

It is an **orchestrator, not an author**. The brand of record is owned by `brand-brain`; the deep work of each section is owned by the eight specialist skills. This skill's job is to **drive `brand-brain`'s bootstrap, sequence the specialists in the right waves, hand off context cleanly between them, gate on quality, and get the human's sign-off** — not to re-write any stage's work. It is the one place in the library allowed to *drive* `brand-brain`'s bootstrap (every other skill only reads the served digest).

This replaces a junior marketer's first week: the "go read the site, study the competitors, write up our voice and ICP, pull our proof, list the objections, and document a style guide" project — done in one supervised run.

---

## Skills this calls

The pipeline. Each is a real, installed skill — **invoke it via the Skill tool; never re-implement its work.**

- **`brand-brain`** (required, the spine) — the system of record. This orchestrator drives its **Mode B bootstrap** (scan → synthesize → interview → write `brand.md`) and reads back its served digest. All resolution, scanning, interviewing, and storage live there.
- **`brand-voice-codifier`** — voice adjectives, tone spectrum, lexicon, banned words/phrases → the `## Voice` section.
- **`icp-persona-builder`** — ICP definition + awareness tendency → the `## ICP` section and `personas.md`.
- **`positioning-messaging-architect`** — category, positioning line, value prop, messaging pillars → the `## Positioning` section.
- **`competitive-intelligence-dossier`** — competitor set, alternatives, differentiators → `competitors.md`.
- **`offer-pricing-brain`** — offer mechanics (free/trial/card/guarantee), pricing, primary CTAs + destination URLs → the `## Offer` section.
- **`proof-vault`** — real proof points (metrics, logos, testimonials, awards) → the `## Proof` section and `proof.md`.
- **`objection-library-builder`** — the objection → reframe + proof map → `objections.md`.
- **`editorial-style-guide`** — formatting, mechanics, structure rules → `style-guide.md`.

Optional finishing pass when installed: **`content-qa-reviewer`** (validate the assembled brain reads cleanly and on-voice) and **`data-qa-measurement-gotcha-checker`** (sanity-check any numbers pulled from analytics before they harden into "proof"). Synthesize their checks inline if absent.

---

## How a run works

A staged pipeline. Each stage names what it **receives** and what it **passes on** (the handoff). Run the four scan-dependent foundations first, then the two stages that depend on them, then companions, then assemble and gate.

```
W0  Intake & scan      ──► drive brand-brain Mode B: resolve slug, scan all sources, draft brand.md
W1  Foundations (∥)    ──► voice · ICP · competitors · offer   (independent — run together)
W2  Synthesis          ──► positioning   (needs ICP + competitors + offer)
W3  Evidence & rules(∥)──► proof · objections · style-guide
W4  Assemble & write   ──► brand-brain folds all sections + companions into brand.md, activates
W5  QA gate            ──► content-qa-reviewer → Approve | Revise (loop the weak stage)
W6  Human sign-off     ──► show the digest + gaps, confirm, hand off
```

### W0 — Intake and scan (drive `brand-brain` Mode B)
**Receives:** the company URL + any uploaded PDF/deck/one-pager + a named brand (if given).
Collect inputs, then **invoke `brand-brain`** to bootstrap. It resolves the data root and slug (multi-brand aware), scans every present source (`[verify]` the URL and any uploaded asset — never trust memory of the site), and produces a **draft `brand.md`** with `sources` and `confidence`. **Do not author anything yourself.** If `brand-brain` reports an existing `ACTIVE` brain for this brand, switch to a *refresh* posture: offer to update rather than overwrite, and run only the stages whose facts changed.
**Passes on:** the active slug + the draft `brand.md` + the raw scanned inputs, to every component skill — so none of them re-scans or recurses.

### W1 — Foundations (parallel)
Run these four together; they don't depend on each other, only on W0's draft.
- **`brand-voice-codifier`** → `## Voice` + banned words. **Handoff:** voice adjectives + banned list become a hard constraint on every later stage's copy.
- **`icp-persona-builder`** → `## ICP` + `personas.md`. **Handoff:** ICP + awareness tendency feed positioning and proof relevance.
- **`competitive-intelligence-dossier`** → `competitors.md`. **Handoff:** the competitor/alternative set + differentiators feed positioning and objections.
- **`offer-pricing-brain`** → `## Offer` + CTAs/destinations. **Handoff:** offer mechanics + pricing feed positioning (value prop) and objections (price/risk).

### W2 — Synthesis
- **`positioning-messaging-architect`** *(receives W1's ICP + competitors + offer)* → `## Positioning` + messaging pillars. This is the keystone: it cannot run well until it knows who we're for, who we're against, and what we sell. **Handoff:** the positioning line + pillars steer proof selection and objection framing.

### W3 — Evidence and rules (parallel)
- **`proof-vault`** *(receives ICP + positioning)* → `## Proof` + `proof.md`, choosing proof that backs the positioning for this ICP. Every number is real or marked `[verify]`.
- **`objection-library-builder`** *(receives competitors + offer + positioning)* → `objections.md`.
- **`editorial-style-guide`** *(receives voice)* → `style-guide.md`, consistent with the codified voice.

### W4 — Assemble and write
Hand every returned section + companion back to **`brand-brain`** to fold into `brand.md` (it owns the schema, de-dupe, conflict resolution, `sources`, `confidence`, and activation). De-dupe across stages; on conflict prefer the most authoritative + recent source and flag it. **Do not write `brand.md` yourself** — that is `brand-brain`'s job, always.

### W5 — QA gate (the orchestration branch)
Invoke **`content-qa-reviewer`** on the assembled brain (or run the inline checklist below). It returns **Approve** or **Revise**:
- **Approve** → continue to W6.
- **Revise** → it names the weak section(s). **Loop only that stage** (re-invoke the owning component with the reviewer's notes + the rest of the brain as context), re-assemble via `brand-brain`, re-gate. Cap at **2 revision loops** per section; if still weak, leave the section marked `[verify]` / low-confidence and surface it as a gap for the human rather than faking strength.

### W6 — Human sign-off
Show the served **digest** (voice, banned words, ICP + awareness, positioning line, offer + destinations, top proof), the list of **companion files** written, and any **open gaps / `[verify]` items**. Get explicit confirmation. End with the one-line `brand-brain` confirmation (the saved path) and a pointer that every downstream skill now reads it.

### Fresh vs. refresh (the branch at W0)
- **No `ACTIVE` brain for this slug** → full bootstrap, all waves.
- **`ACTIVE` brain exists** → refresh posture: ask what changed (or re-scan for diffs), run **only** the affected stages, and route every change through `brand-brain`'s diff-and-confirm — never silently overwrite a human-confirmed field. A repositioning re-runs W2 (and the W3 stages that consume it); a new pricing tier re-runs offer + objections; a new case study re-runs proof only.

### What the parallel handoff looks like (W1)
Invoke the four foundation skills in **one batch**, each receiving the same packet — `{ slug, draft brand.md, raw scanned inputs }` — and each returning its section/companion. Concretely: `brand-voice-codifier` returns `## Voice` + banned words; `icp-persona-builder` returns `## ICP` + `personas.md`; `competitive-intelligence-dossier` returns `competitors.md`; `offer-pricing-brain` returns `## Offer` + CTAs. Collect all four before starting W2 — positioning must see the full foundation, not a partial one.

---

## Bundled deliverable

`brand-brain` writes to its resolved data root (`./.brandbrain/` per-project, else `~/.brandbrain/`). This orchestrator additionally compiles a **portable bootstrap report** to a project-relative path the buyer owns:

```
./brand-brain/<slug>/
  brand-brain-report.md     # one compiled overview: digest + every section + open gaps + sources
  brand.md                  # copy of the activated brain (canonical lives in the data root)
  personas.md  competitors.md  proof.md  objections.md  style-guide.md   # copies of companions
```

`./` is the **user's CWD / project**, never the skill folder. Note in the report that the canonical, mutable brain lives in the data root and is what downstream skills read; these are a shareable snapshot.

---

## Principles

- **Brand-brain is the spine — invoke it, don't impersonate it.** This is the one skill that *drives* `brand-brain`'s bootstrap, but `brand-brain` still owns resolution, scanning, interviewing, storage, and the `brand.md` schema. Never hand-write `brand.md`.
- **Orchestrate, don't re-author.** Every section is produced by its owning component skill via the Skill tool. If you're tempted to write voice/ICP/proof yourself, you've left your lane — invoke the specialist instead.
- **Respect the waves.** Run independent stages in parallel; never start positioning before ICP + competitors + offer exist; never select proof before positioning. The handoffs are the point.
- **Scan before you ask; ask only the gaps.** `brand-brain` exhausts the URL, uploads, and existing context first; the interview covers only what's still missing or low-confidence.
- **Truth discipline.** Real numbers, real differentiators, real proof — or `[verify]` / "illustrative." Never invent a metric, logo, customer, or quote to make a section look finished.
- **Gate, then loop the weak stage only.** A Revise verdict re-runs just the failing section with the reviewer's notes — not the whole pipeline.
- **Human owns the foundation.** The brain isn't "done" until the human confirms the digest and gaps at W6.
- **Multi-brand by default.** Everything is scoped to a slug; refreshing an existing brand diffs and confirms rather than overwriting.

## What not to do

- Don't write or edit `brand.md` (or any companion) directly — route all writes through `brand-brain`.
- Don't re-implement scanning, interviewing, resolution, or storage — that's `brand-brain`, once.
- Don't re-implement a component's craft (voice, ICP, positioning, competitive, offer, proof, objections, style) — invoke the skill that owns it.
- Don't run stages out of order or skip the handoff (e.g. positioning written blind to the competitor set).
- Don't invent proof, pricing, customers, or differentiators to fill a gap — mark it `[verify]` and surface it.
- Don't loop the whole pipeline on a single weak section; loop that section, capped at 2 passes.
- Don't write the deliverable into the skill folder; write to `./brand-brain/<slug>/`.
- Don't declare the brain ready without the W5 QA gate passing and the W6 human sign-off.

## Quality checklist (self-review before handing off)

- W0: inputs collected and `brand-brain` driven through Mode B — did *it* scan + draft, not me?
- Existing-brain case detected and handled as a refresh (diff + confirm), not a silent overwrite?
- W1 ran the four foundation skills (voice, ICP, competitors, offer); each returned its section/companion?
- W2 positioning ran *after* and *consumed* ICP + competitors + offer (not blind)?
- W3 proof was selected to back the positioning for the ICP; objections + style guide consistent with competitors + voice?
- Every stage produced by its owning skill via the Skill tool — nothing hand-authored here?
- W4 assembled through `brand-brain`; conflicts flagged, `sources` + `confidence` set, brand activated?
- W5 QA gate passed (Approve), or weak sections looped ≤2× then surfaced as gaps; every unconfirmed number `[verify]`?
- Deliverable written to `./brand-brain/<slug>/` (project, not skill folder), with report + brain + all present companions?
- W6: digest, companion list, and open gaps shown; human confirmed; saved-path line delivered?
