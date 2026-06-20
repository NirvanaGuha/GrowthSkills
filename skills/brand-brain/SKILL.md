---
name: brand-brain
description: >
  The library's single source of truth for brand context — voice, ICP, positioning, offer/pricing,
  proof, banned words — captured once and read by every other marketing/copy skill. Resolves the
  active brand (multi-brand / agency-ready, with per-brand folders like Thoth), and on first use
  BOOTSTRAPS a brand by scanning existing context (previous sessions, local graphs, memory skills,
  brand-brain skills, Thoth personas, project config) and asking only the few questions still
  missing — then saving a reusable brand.md. Other skills CALL this skill to load brand context
  instead of rebuilding it. Use when the user says "set up my brand," "brand brain," "onboard a
  brand," "what's my brand voice/positioning," "switch / add / refresh / audit brand," or whenever
  any copy/marketing skill needs the active brand's voice, ICP, offer, or proof before producing
  output. This is the Layer-0 dependency for the whole skill library.
---

# Brand Brain — the library's system of record

Every marketing skill writes better when it knows the brand. Rather than have each skill re-derive voice, ICP, offer, and proof from scratch, this skill captures that **once** into a reusable `brand.md` and **serves** it to every other skill on demand. Build the brand here; the rest of the library reads it.

Three jobs:
1. **Serve** — given an active brand, hand any caller the brand's `brand.md` (the fast path; the steady state).
2. **Bootstrap** — when a brand has no brain yet, scan what already exists, ask only the gaps, and write `brand.md`.
3. **Maintain** — list / switch / add brands (multi-brand ops), and refresh / audit a brain over time.

---

## Where brand data lives

Brand data is **mutable user data and lives outside the skill folder** — so a marketplace/`amskills` update (which does rm-rf + re-extract) can never wipe it, and the user can grant read/write on the data root without exposing Claude's config. The skill code (`SKILL.md`, `references/`) stays immutable in `.claude/skills/brand-brain/`.

**Resolve the data root** on every run, in order:
1. If `./.brandbrain/brands/` exists relative to the CWD → root is `./.brandbrain/` (per-project mode — teams can check brand context into a repo).
2. Else if `~/.brandbrain/brands/` exists → root is `~/.brandbrain/` (global default).
3. Else → create `~/.brandbrain/brands/` and use it.

Paths written `brands/<slug>/...` resolve to `<root>/brands/<slug>/...`. The active-brand pointer is `<root>/brands/.active` (one line: the active slug). A brand is **onboarded** when `brands/<slug>/brand.md` exists with `status: ACTIVE`.

---

## Resolving the active brand (multi-brand / agency)

On each run, resolve the target brand in this order:
1. **Explicit in the request / caller** — a named brand ("for LedgerPort") → match to `brands/<slug>/` (fuzzy on name/slug). No match → treat as a NEW brand → Bootstrap.
2. **`.active` pointer** — else read `brands/.active`; if it points to an onboarded brand, use it.
3. **Single brand** — else if exactly one onboarded brand exists, use it and set it active.
4. **Ambiguous / none** — else list brands and ask which; if none exist, Bootstrap a new one.

---

## Commands

Honor these when the user (or a caller) uses them:

| Command | Action |
|---|---|
| `brand-brain` (no args) | Resolve the active brand; serve its `brand.md`. Bootstrap if none. |
| `brand-brain list` | Show every `brands/<slug>/` with status, confidence, and `updated`. |
| `brand-brain use <brand>` | Write `<slug>` to `brands/.active`. |
| `brand-brain new <brand>` / `onboard <brand>` | Bootstrap a new brand. |
| `brand-brain show <brand>` | Print the brand's `brand.md`. |
| `brand-brain refresh <brand>` | Re-scan for changed facts, diff, confirm, write (bump `updated`). |
| `brand-brain audit <brand>` | Health check: flag staleness, gaps, `[verify]` items, internal contradictions. |
| `brand-brain where` | Print the resolved data root. |

---

## Mode A — Serve (the contract other skills call)

This is how the rest of the library uses the brain. When invoked **by another skill** (or any time a caller just needs context):

1. Resolve the data root and active brand (above).
2. If the brand has an `ACTIVE` `brand.md` → load it. **Return to the caller:**
   - the active **slug**,
   - the absolute **path** to `brand.md`,
   - a compact **digest**: voice adjectives, banned words/phrases, offer mechanics (free/trial/card/guarantee + primary destination URLs), real proof points, positioning line, ICP + awareness tendency.
   - pointers to any **companion files** present that fit the caller's task (`competitors.md`, `objections.md`, `proof.md`, `personas.md`, `style-guide.md`).
   The caller reads the full `brand.md` (and any relevant companion) if it needs more.
3. If the brand has **no** `ACTIVE` brain → run **Bootstrap** (Mode B) first, then return as above. Tell the user in one line that you're setting the brand up once so every future run is instant.

Keep Serve fast and silent on the happy path: no scan, no questions when a brain already exists.

---

## Mode B — Bootstrap (scan → synthesize → interview → write)

Run when the target brand has no `ACTIVE` brain. **Produce nothing downstream until this completes.**

1. **Scan** existing context — follow `references/brand-context-sources.md` in full. Probe each source only if present; skip silently otherwise. Pull anything mapping to a `brand.md` field, noting the source per fact.
2. **Delegate to the component skills** (all eight are built — see `references/component-skills.md`): call them in the recommended waves, passing each the scanned context (active slug + draft `brand.md` + raw inputs) so they don't recurse. Each returns its `brand.md` section(s) and/or writes a companion file. Synthesize a section inline only if its component is somehow unavailable.
3. **Synthesize** a draft `brand.md` from the `references/brand-template.md` schema. De-dupe across sources; on conflict prefer the most authoritative + recent and flag it. Mark every unconfirmed number `[verify]`. Set `confidence` and list `sources`.
4. **Interview the gaps only** — follow `references/interview.md`. Show the user what the scan already established, then ask **only** for still-missing / low-confidence fields (never re-ask what the scan found). Batch the questions.
5. **Write & activate** — write `brands/<slug>/brand.md` (`status: ACTIVE`, `sources`, `confidence`, today's dates); write `<slug>` to `brands/.active` (unless mid-task on another brand). Confirm in one line: *"Brand saved to `<root>/brands/<slug>/brand.md` — every library skill reads it now. Run `brand-brain refresh <slug>` anytime."*
6. **Return** the served digest (Mode A) to the caller and continue.

---

## Mode C — Refresh / Audit

- **Refresh:** re-scan for *changed* facts (new pricing, new proof, a repositioning), diff against the existing `brand.md`, show proposed changes, write on approval, bump `updated`. Never silently overwrite a user-confirmed field.
- **Audit:** read the brain and report staleness (old `updated`), gaps (empty core fields), unresolved `[verify]` items, and internal contradictions — before they corrupt downstream copy.

---

## How OTHER skills call this skill (interconnection — upward)

Any copy/marketing skill in the library should resolve brand context through this skill instead of implementing its own:

> **Before producing output, invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve and load the active brand. Use the returned digest + `brand.md`; obey its voice and banned-words as overrides; use only its real proof. **Do not implement brand resolution, scanning, or interviewing yourself** — that lives here, once.

If `brand-brain` is not installed, a caller may fall back to reading `~/.brandbrain/brands/.active` + that brand's `brand.md` directly, and if none exists, ask the user to install `brand-brain` (or run a minimal inline setup). Prefer the call.

## How this skill calls OTHER skills (interconnection — downward)

During a deep Bootstrap/Refresh, delegate to the **eight built component skills** (full map + recommended wave order in `references/component-skills.md`): `brand-voice-codifier` (voice/banned/lexicon), `icp-persona-builder` (ICP + `personas.md`), `positioning-messaging-architect` (positioning + value prop), `competitive-intelligence-dossier` (`competitors.md`), `offer-pricing-brain` (offer + CTAs), `proof-vault` (proof + `proof.md`), `objection-library-builder` (`objections.md`), `editorial-style-guide` (`style-guide.md`). Pass each the active slug + current `brand.md` + scanned inputs so it doesn't recurse; fold the returned section(s) into `brand.md`, keep companion files alongside, and note each in `sources`. Synthesize inline only when a component is unavailable. This keeps the brain authoritative while specialist skills do the deep work.

---

## Principles (Non-Negotiable)

- **One brain, many readers.** Brand context is captured once here and referenced everywhere — never re-derived per skill.
- **Scan before you ask.** Always exhaust existing sources before interviewing; never re-ask what's already known.
- **Truth only.** Real numbers, real differentiators, or `[verify]` / "illustrative." Never invent proof or positioning.
- **Data outside the skill.** Brand files live in the data root, never in the skill folder (survives updates).
- **Confirm before overwrite.** Refresh diffs and asks; it never silently changes a user-confirmed field.
- **Multi-brand by default.** Everything is scoped to a brand slug; agencies and multi-brand ops are first-class.

## What Not to Do

- Don't write brand data inside the skill folder.
- Don't produce or let a caller produce copy before an `ACTIVE` brain exists for the target brand.
- Don't re-interview fields the scan already answered.
- Don't merge two brands' data; keep slugs isolated.

## Quality Checklist

- Data root + active brand resolved; multi-brand handled?
- On a fresh brand: scanned all present sources, interviewed only the gaps, wrote `brand.md` with `sources` + `confidence`?
- Served digest includes voice, banned words, offer mechanics + destinations, real proof, positioning, ICP?
- Every unconfirmed number marked `[verify]`; conflicts flagged?
- Caller received the digest + path and can proceed?
