# Component Skills — downward delegation map

During a deep Bootstrap or Refresh, `brand-brain` can delegate individual sections of `brand.md` to specialized Layer-1 skills when they're installed. This keeps the brain authoritative (it owns the file) while letting deeper skills do deeper work. **Graceful rule:** if a component skill is not installed, synthesize that section inline from the scan + interview instead. Never hard-fail on a missing component.

## How to call a component
Invoke it with the Skill tool (`skill: <slug>`), pass the scanned raw inputs for its section, and fold its structured output into the matching `brand.md` section. Note it in `sources`.

## The map

| brand.md section | Component skill (slug) | Status |
|---|---|---|
| Voice & tone / banned words | `brand-voice-codifier` | planned (Tier 1) |
| ICP(s) | `icp-persona-builder` | planned (Tier 1) |
| Positioning / core frame / value prop | `positioning-messaging-architect` | planned (Tier 1) |
| Differentiators + competitors | `competitive-intelligence-dossier` | planned (Tier 2) |
| Offer & pricing essentials | `offer-pricing-brain` | planned (Tier 2/3) |
| Proof assets | `proof-vault` | planned (Tier 3) |
| Objection handling | `objection-library-builder` | planned (Tier 2) |
| Editorial style | `editorial-style-guide` | planned (Tier 3) |

(Slugs are the canonical library names. As these ship, brand-brain auto-prefers them; until then it synthesizes inline.)

## Detecting availability
A component is "installed" if it appears in the available-skills list for the session (Claude Code surfaces installed skills) or its folder exists under `.claude/skills/<slug>/`. If present, prefer it for its section; otherwise synthesize inline and move on.

## Refresh interplay
On `brand-brain refresh`, re-run only the component(s) whose source data changed (e.g. new competitors → `competitive-intelligence-dossier`), diff their output against the current section, and write on approval.
