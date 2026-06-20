# Brand Context Sources — the scan catalog

When bootstrapping or refreshing a brand, probe these sources **in order (cheap → rich)**, using only the ones present and skipping the rest silently. Pull anything that maps to a `brand.md` field (what-it-is, positioning, ICP, differentiators, offer/pricing, proof, voice, banned words, destinations). Record the source for each fact. Stop early if a source already yields a near-complete picture. A brand-new user on a clean machine will have none of these — that's fine; the interview does the work.

The four categories the user expects scanned — plus config:

## 1. Skills that add brand context (richest first)
- **Content-pipeline voice file:** `<project>/.claude/skills/_shared/brand-and-voice.md` — often a full brain on its own (positioning, ICP, voice, banned words, differentiators, proof).
- **Brand-brain skills installed locally:** `~/.claude/skills/pushengage-brand-brain/` (+ feature/channel companion files), `~/.claude/skills/brand-brain-analyzer/`, `~/.claude/skills/brand-brain-wpchat/`. Read their `SKILL.md` and companion files. `brand-brain-analyzer` can be invoked for a deeper site/inputs analysis if the user wants it.
- **Thoth personas (voice gold):** `~/.thoth/personas/<slug>/persona.md` + `brand.yaml` — voice adjectives, archetype, banned phrases, tone, colors. A persona slug matching the brand is a strong seed.

## 2. Installed memory skills
- **Auto-memory:** `~/.claude/projects/<project-slug>/memory/MEMORY.md` (index) + the files it points to — pull ICP, positioning, acquisition-model, pricing, product facts (e.g. `icp_*.md`, `*_gtm.md`, `*_pricing_*.md`, `*_brand*.md`, `*_acquisition*.md`).
- **`wiki` / `writer-memory`** knowledge bases if present.

## 3. Previous sessions
- **claude-mem:** use the `claude-mem:mem-search` skill or the `get_observations` MCP tool to query past observations for the brand's positioning, ICP, pricing, voice. May be unavailable in headless/cron runs — degrade gracefully.
- **Session transcripts** under `~/.claude/projects/**/*.jsonl` are a last resort; parse only if nothing else exists and the user asks for a deep build.

## 4. Local graphs
- **Graphify:** if `<project>/graphify-out/` exists, query the local knowledge graph for brand / positioning / ICP / product / pricing entities (treat the request as a graphify query per that skill). This is the user's preferred project-graph tool — prefer it when present for a fast structured pull.

## 5. Project + global config
- `<project>/CLAUDE.md` and `~/.claude/CLAUDE.md` — brand colors, positioning lines, conventions, product facts, naming rules.

## Synthesis notes
- De-duplicate facts that appear in several sources; keep the most specific phrasing.
- On conflict, prefer the most authoritative (an explicit ICP brief beats an offhand mention) and most recent; surface the conflict to the user rather than silently picking.
- Tag every number `[verify]` unless the source is authoritative.
- Set `confidence: high` only when the core fields came from real sources, not assumption.
