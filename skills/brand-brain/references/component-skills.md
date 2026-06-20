# Component Skills — downward delegation map

During a deep Bootstrap or Refresh, `brand-brain` delegates individual parts of the brand to specialized Layer-1 skills. This keeps the brain authoritative (it owns `brand.md` and the brand folder) while letting deeper skills do deeper work. **Graceful rule:** if a component isn't installed, synthesize that part inline from the scan + interview instead. Never hard-fail on a missing component.

All eight components below are **built** and live in this library (`~/GrowthSkills/skills/<slug>/`, symlinked into `~/.claude/skills/`).

## How to call a component
Invoke it with the Skill tool (`skill: <slug>`) and **pass the context** so it doesn't recurse back into `brand-brain`: the active **slug**, the current `brand.md` content, and the **scanned raw inputs** for its part. It returns its `brand.md` section block(s) and/or writes its companion file; fold the returned blocks into `brand.md` and note the component in `sources`.

## The map

| Brand part | Component (slug) | Writes to | Status |
|---|---|---|---|
| Voice & tone · Banned words · Preferred lexicon | `brand-voice-codifier` | 3 brand.md sections | built ✓ |
| ICP(s) | `icp-persona-builder` | brand.md "ICP(s)" + optional `personas.md` | built ✓ |
| Positioning / core frame · Value proposition & differentiators | `positioning-messaging-architect` | 2 brand.md sections | built ✓ |
| Competitor dossiers + battlecards (feeds differentiators) | `competitive-intelligence-dossier` | companion `competitors.md` | built ✓ |
| Offer & pricing essentials · Common CTAs & destinations | `offer-pricing-brain` | 2 brand.md sections | built ✓ |
| Proof assets | `proof-vault` | brand.md "Proof assets" + `proof.md` | built ✓ |
| Objection library (top-objections pointer in Notes) | `objection-library-builder` | companion `objections.md` | built ✓ |
| Editorial style (coordinates banned words) | `editorial-style-guide` | companion `style-guide.md` | built ✓ |

## Companion files
Deeper artifacts that don't fit the `brand.md` cheat sheet live alongside it in `<data-root>/brands/<slug>/`: `personas.md`, `competitors.md`, `proof.md`, `objections.md`, `style-guide.md`. `brand.md` should carry a one-line pointer to each that exists (see the brand template's "Companion files" section). When serving a caller, mention any companion file relevant to their task.

## Recommended bootstrap order
Some parts feed others, so delegate in waves rather than all at once:
1. **Wave 1 (parallel, independent):** `brand-voice-codifier`, `icp-persona-builder`, `offer-pricing-brain`, `proof-vault`, `competitive-intelligence-dossier`.
2. **Wave 2 (needs Wave 1):** `positioning-messaging-architect` (reads the competitive differentiation summary + ICP), then `objection-library-builder` (reframes pull from positioning + proof).
3. **Wave 3 (last):** `editorial-style-guide` (suggests banned-word additions back to the voice section).

## Detecting availability
A component is "installed" if it appears in the session's available-skills list or its folder exists under `~/.claude/skills/<slug>/` (or `~/GrowthSkills/skills/<slug>/`). If present, prefer it for its part; otherwise synthesize inline and move on.

## Refresh interplay
On `brand-brain refresh`, re-run only the component(s) whose source data changed (e.g. new competitors → `competitive-intelligence-dossier`; repricing → `offer-pricing-brain`), diff their output against the current section/companion, and write on approval.
