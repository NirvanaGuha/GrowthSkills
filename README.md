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
└── skills/
    ├── brand-brain/              Layer 0 — system of record; every copy skill calls it
    └── cta-variant-generator/    quick + battery CTA generation; calls brand-brain
```

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

- **Built:** `brand-brain` (Layer 0) · `cta-variant-generator`.
- **Published (AM Skills):** `cta-variant-generator` v1, public. `brand-brain` not yet (publish it before re-publishing CTA v2, which now depends on it).
- **Brands built:** `pushengage` (`~/.brandbrain/brands/pushengage/brand.md`, confidence high).
- **Next:** the rest of Tier 1 from the prioritized list — each calling `brand-brain` from day one.
