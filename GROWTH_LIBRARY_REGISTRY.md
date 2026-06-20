# Growth Skill Library — Registry & Call Graph

The live map of built skills and how they interconnect. Every new skill (a) reads brand context from `brand-brain` and (b) records its `calls` / `called-by` here. This is the "manage the project" artifact — graphify can ingest it for a queryable dependency graph as the library grows.

Last updated: 2026-06-20

**Library home:** `~/GrowthSkills/` — canonical source in `~/GrowthSkills/skills/<slug>/`, each symlinked into `~/.claude/skills/` for global discovery (usable in every project, not tied to PushEngage). Edit the canonical files in `~/GrowthSkills/`; the symlinks reflect changes live.

---

## Architecture rules (every skill follows these)

1. **One brand brain, many readers.** No skill re-derives brand context. Every copy/marketing skill calls **`brand-brain`** to load the active brand before producing output.
2. **Data lives outside skill folders.** Brand data → `~/.brandbrain/` (per-project `./.brandbrain/` overrides). Survives `amskills` updates (rm-rf safe).
3. **Skills call skills.** Prefer composing existing skills over re-implementing. Declare `calls` / `called-by` here and in each SKILL.md.
4. **Graceful degradation.** If a called skill is absent, fall back inline — never hard-fail.
5. **No recursion.** When `brand-brain` calls a component it passes context; the component must NOT call `brand-brain` back.

---

## Data roots

| Purpose | Path |
|---|---|
| Brand context (all brands) | `~/.brandbrain/brands/<slug>/brand.md` · active pointer `.active` · companions `personas.md` / `competitors.md` / `proof.md` / `objections.md` / `style-guide.md` |
| Built brands | `pushengage` (ACTIVE, confidence high) |

---

## Built skills (10)

### `brand-brain` — Layer 0 (Foundation / system of record)
- **Provides:** resolve active brand · serve `brand.md` digest · bootstrap (scan→delegate→interview→write) · refresh · audit · multi-brand.
- **Calls (downward):** all 8 Layer-1 components below *(now BUILT — delegates in waves; synthesizes inline only if one is unavailable)*.
- **Called by:** every copy/marketing skill (currently `cta-variant-generator`).
- **Location:** `~/GrowthSkills/skills/brand-brain/` · **Published:** not yet.

### Layer-1 components — the brand-brain "organs" (all built ✓)
Each is callable **standalone** (it calls `brand-brain` to resolve the active brand) or **by `brand-brain`** during bootstrap/refresh (uses passed context, returns its part, no recursion). Each owns only its part; none re-derives context.

| Skill | Owns / writes | Companion file |
|---|---|---|
| `brand-voice-codifier` | brand.md: Voice & tone · Banned words · Preferred lexicon | — |
| `icp-persona-builder` | brand.md: ICP(s) | `personas.md` (optional) |
| `positioning-messaging-architect` | brand.md: Positioning · Value proposition & differentiators | — |
| `competitive-intelligence-dossier` | feeds differentiators (suggests, no overwrite) | `competitors.md` |
| `offer-pricing-brain` | brand.md: Offer & pricing essentials · Common CTAs & destinations | — |
| `proof-vault` | brand.md: Proof assets | `proof.md` |
| `objection-library-builder` | brand.md: Notes top-objections pointer | `objections.md` |
| `editorial-style-guide` | coordinates banned words (suggests, no overwrite) | `style-guide.md` |

All at `~/GrowthSkills/skills/<slug>/` (symlinked into `~/.claude/skills/`). Published: none yet.

### `cta-variant-generator` — Layer 5/8 (Content / Paid)
- **Provides:** quick mode (copy→CTA / CTA→better CTA) · battery mode (variants + A/B).
- **Calls:** `brand-brain` (required) · `proof-vault`, `headline-hook-generator` *(optional; headline skill not built yet)*.
- **Called by:** *(none yet)* — future orchestrators (e.g. Full Campaign Launch) will call it.
- **Location:** `~/GrowthSkills/skills/cta-variant-generator/` · **Published:** AM Skills, public, v1 (predates the brand-brain split).

---

## Call graph

```
   every copy/marketing skill ──calls──► brand-brain   (Layer 0 · system of record)
                                              │
        brand-brain ──delegates (bootstrap/refresh, in waves)──► 8 Layer-1 components:
          brand-voice-codifier · icp-persona-builder · positioning-messaging-architect
          competitive-intelligence-dossier · offer-pricing-brain · proof-vault
          objection-library-builder · editorial-style-guide
        each component, run standalone, ──calls──► brand-brain to resolve the active brand
        companions written alongside brand.md: personas.md · competitors.md · proof.md ·
          objections.md · style-guide.md

   cta-variant-generator ──calls──► brand-brain
```

Delegation waves (from `brand-brain/references/component-skills.md`): **W1** voice · icp · offer · proof · competitive → **W2** positioning (uses competitive + icp) · objections (uses positioning + proof) → **W3** editorial-style-guide.

---

## Publish status / TODO

- **AM Skills publishing: paused** (per user, 2026-06-20). `cta-variant-generator` v1 remains public on AM Skills from earlier; no further marketplace pushes for now.
- **GitHub:** whole project public at https://github.com/NirvanaGuha/GrowthSkills (single repo, no collaborators).
- Next Tier-1 builds: each calls `brand-brain` in Step 0 and registers here. Candidate next skills include `headline-hook-generator` (CTA already references it as optional).
