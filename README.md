# Growth Skill Library

A sellable ($50/mo) library of AI agent skills covering the **entire** day-to-day of a growth + content marketer — not just content, but decision-making, productivity, internal comms, PR, growth engineering, analytics, and the meta-work in between.

This folder is the whole project: strategy docs, the built skills, and the registry that maps how they interconnect.

## Layout

```
~/GrowthSkills/
├── README.md                     ← you are here
├── GROWTH_LIBRARY_REGISTRY.md    ← live call-graph / dependency map (every new skill registers here)
├── docs/
│   ├── GROWTH_MARKETER_SKILL_LIBRARY.md      v1 catalog (170 skills, 16 layers)
│   ├── GROWTH_MARKETER_SKILL_LIBRARY_v2.md   v2 master catalog (35 layers, ~1,137 skills)
│   └── GROWTH_SKILL_LIBRARY_PRIORITIZED.md   de-duped build list (412 canonical, scored + tiered)
└── skills/                      (120 skills — Tier-1 complete)
    ├── brand-brain/              Layer 0 — system of record; every skill calls it
    ├── <8 brand-brain components>    the "organs" (voice, icp, positioning, competitive, offer, proof, objections, style)
    ├── <106 Tier-1 skills>           across 31 layers (SEO, content, lifecycle, paid, CRO, analytics, ABM, PR, ops, decision, …)
    └── <8 flagship orchestrators>    L35 — chain the library into "growth team in a box" pipelines
```
*Full catalog by layer + the orchestrator pipelines: see `GROWTH_LIBRARY_REGISTRY.md`.*

## How the skills are wired

- **One brand brain, many readers.** Brand context (voice, ICP, offer, proof, banned words) is captured once by `brand-brain` and read by every other skill. No skill re-derives it.
- **Skills call skills.** Composition over re-implementation — calls are by skill name (location-independent). Every new skill registers its `calls` / `called-by` in `GROWTH_LIBRARY_REGISTRY.md`.
- **Graceful degradation.** If a called skill isn't installed, the caller falls back inline rather than failing.

## Data lives outside this folder (by design)

Brand data is at **`~/.brandbrain/brands/<slug>/brand.md`** (+ `.active` pointer), **not** inside this project. That's deliberate: it's shared runtime data that skills read from *any* working directory, and it must survive `amskills`/marketplace updates (which rm-rf the skill folder). Think of it as the database, not the source. (Per-project override: a `./.brandbrain/` in any repo takes precedence when present.)

## Discovery / install

Canonical source lives here; each skill is **symlinked into `~/.claude/skills/<slug>`** for global discovery (usable in every project, not tied to PushEngage). Edit the files here — the symlinks reflect changes live.

**To add a new skill:** create `skills/<slug>/SKILL.md` (call `brand-brain` in step 1), `ln -sfn ~/GrowthSkills/skills/<slug> ~/.claude/skills/<slug>`, and add it to the registry.

## Status

- **Built: Tier-1 COMPLETE — 120 skills.** brand-brain + 8 components + 106 Tier-1 skills + 8 flagship orchestrators, across 31 layers. Built in 7 verified waves; every skill brand-brain-contract-checked, symlinked, and pushed.
- **Publishing:** GitHub public (this repo). AM Skills publishing paused — `cta-variant-generator` v1 remains public there from earlier.
- **Brands built:** `pushengage` (`~/.brandbrain/brands/pushengage/brand.md`, confidence high).
- **Next candidates:** a library-wide quality polish pass · Tier-2 (152 skills from the prioritized list) · package/sell.
