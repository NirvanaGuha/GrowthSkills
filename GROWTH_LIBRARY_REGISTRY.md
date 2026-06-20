# Growth Skill Library — Registry & Call Graph

The live map of built skills and how they interconnect. As the library grows, every new skill (a) reads brand context from `brand-brain` and (b) records its `calls` / `called-by` here. This is the "manage the project" artifact — graphify can ingest it for a queryable dependency graph once there are more nodes.

Last updated: 2026-06-20

**Library home:** `~/GrowthSkills/` — canonical source in `~/GrowthSkills/skills/<slug>/`, each symlinked into `~/.claude/skills/` for global discovery (so skills are usable in every project, not tied to PushEngage). Edit the canonical files in `~/GrowthSkills/`; the symlinks reflect changes live.

---

## Architecture rules (every skill follows these)

1. **One brand brain, many readers.** No skill re-derives brand context. Every copy/marketing skill calls **`brand-brain`** to load the active brand before producing output.
2. **Data lives outside skill folders.** Brand data → `~/.brandbrain/` (per-project `./.brandbrain/` overrides). Survives `amskills` updates (rm-rf safe).
3. **Skills call skills.** Prefer composing existing skills over re-implementing. Declare `calls` and `called-by` here and in each SKILL.md.
4. **Graceful degradation.** If a called skill is absent, fall back inline — never hard-fail.

---

## Data roots

| Purpose | Path |
|---|---|
| Brand context (all brands) | `~/.brandbrain/brands/<slug>/brand.md` · active pointer `~/.brandbrain/brands/.active` |
| Built brands | `pushengage` (ACTIVE, confidence high) |

---

## Built skills

### `brand-brain` — Layer 0 (Foundation)
- **Provides:** resolve active brand · serve `brand.md` digest · bootstrap (scan→interview→write) · refresh · audit · multi-brand.
- **Calls (downward, when installed):** `brand-voice-codifier`, `icp-persona-builder`, `positioning-messaging-architect`, `competitive-intelligence-dossier`, `offer-pricing-brain`, `proof-vault`, `objection-library-builder`, `editorial-style-guide` *(all planned — synthesizes inline until they exist)*.
- **Called by:** every copy/marketing skill (currently `cta-variant-generator`).
- **Location:** `~/GrowthSkills/skills/brand-brain/` (symlinked → `~/.claude/skills/brand-brain`) · **Published:** not yet.

### `cta-variant-generator` — Layer 5/8 (Content / Paid)
- **Provides:** quick mode (copy→CTA / CTA→better CTA) · battery mode (variants + A/B).
- **Calls:** `brand-brain` (required) · `proof-vault`, `headline-hook-generator` *(optional, when installed)*.
- **Called by:** *(none yet)* — future orchestrators (e.g. Full Campaign Launch) will call it.
- **Location:** `~/GrowthSkills/skills/cta-variant-generator/` (symlinked → `~/.claude/skills/cta-variant-generator`) · **Published:** AM Skills, public, v1 (predates the brand-brain split — needs a v2 re-publish alongside `brand-brain`).

---

## Call graph

```
                 ┌─────────────────────────────┐
   (callers) ───►│        brand-brain          │◄─── every copy/marketing skill
                 │  (Layer 0 · system of record)│
                 └──────────────┬──────────────┘
                                │ delegates sections (when installed)
                                ▼
   brand-voice-codifier · icp-persona-builder · positioning-messaging-architect ·
   competitive-intelligence-dossier · offer-pricing-brain · proof-vault · …  [planned]

   cta-variant-generator ──calls──► brand-brain
   cta-variant-generator ──(optional)──► proof-vault · headline-hook-generator  [planned]
```

---

## Publish status / TODO

- `cta-variant-generator` v1 is public on AM Skills but still bundles the old embedded brand logic. Re-publish as **v2** (delegates to `brand-brain`) — and publish **`brand-brain`** first so the dependency exists for installers. Use `amskills publish ... --update cta-variant-generator --changelog "…"`.
- Next Tier-1 builds should each: call `brand-brain` in Step 0, and register here.
