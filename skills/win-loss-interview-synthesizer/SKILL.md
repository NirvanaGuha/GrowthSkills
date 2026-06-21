---
name: win-loss-interview-synthesizer
description: >
  Turns raw win-loss interview transcripts, recorded call summaries, or CRM close/lost notes into a
  structured synthesis: decision factors ranked by frequency and weight, competitive evaluation criteria
  (what the buyer actually compared and why), verbatim key quotes with tagging, and a categorized
  objection inventory. Works from a single interview or an entire batch. Produces a deal-room-ready
  summary plus downstream-ready structured data that feeds objection-library-builder, positioning updates,
  and competitive briefs. Two modes: Single-Deal (fast, one transcript or note block → immediate
  synthesis) and Batch (multiple transcripts or a CSV of CRM notes → cross-deal theme hierarchy with
  frequency counts). Calls brand-brain to anchor synthesis to the active brand's positioning, ICP,
  and known proof before drawing any conclusions — so insights are immediately actionable, not generic.
  Use when the user says "analyze this win/loss," "what did the buyer say," "why did we lose this deal,"
  "synthesize these customer interviews," "extract themes from deal notes," "what objections are
  showing up," "what did the prospect care about," "here are my interview transcripts," or pastes
  CRM notes and asks for meaning.
---

# Win-Loss Interview Synthesizer

Raw deal transcripts and CRM close notes contain the most honest market signal a growth team will ever see — but unprocessed they just pile up in Notion. This skill extracts signal from noise: decision factors, competitive criteria, verbatim quotes, and objections, all mapped against the active brand's current positioning so gaps surface immediately and feed directly into the skills that act on them.

This skill synthesizes. It does not run the interviews, coach reps, or rewrite positioning itself — it surfaces the insight and hands exact inputs to the skills that do those jobs next.

---

## Skills this calls

- **`brand-brain`** (required, first) — loads the active brand's positioning, ICP, offer, and proof so synthesis is anchored to what the brand actually claims, not to a generic framework. Highlights where deal signals confirm or contradict current positioning.
- **`objection-library-builder`** (optional, downstream) — pass the objection inventory from this skill's output to add or update the brand's canonical objection library.
- **`positioning-messaging-architect`** (optional, downstream) — pass decision-factor ranking and positioning-gap flags to inform a messaging refresh.
- **`competitive-intelligence-dossier`** (optional, downstream) — pass competitive criteria and head-to-head quotes when a competitor analysis needs primary deal-room data.
- **`battlecard-objection-handler`** (optional, downstream) — pass the competitive criteria section to refresh or create a battlecard for the named competitor.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain (always first)
Step 1  Classify the input    ──► Single-Deal or Batch; Win, Loss, or Ghost
Step 2  Pre-process           ──► segment the raw material by speaker/section
Step 3  Extract with MEDDIC+  ──► run the structured extraction framework
Step 4  Synthesize            ──► build the output artefact, flag positioning gaps
Step 5  Route downstream      ──► offer next-skill handoffs
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns: the active brand slug, voice, ICP + typical awareness stage, positioning line, current differentiators, known proof, and banned words. Load any companion `competitors.md` and `objections.md` if present.

Use this to:
- Anchor every decision-factor against what the brand claims (gap = what the buyer cared about that the brand doesn't address; confirmation = what the brand says that the buyer validated).
- Know which competitor names to flag when they appear.
- Obey voice + banned words in all synthesized text.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user for positioning line, ICP description, and 2–3 differentiators before proceeding — then proceed with that inline context.

### Step 1 — Classify the input

| Signal | Mode |
|---|---|
| One transcript / one CRM note block | Single-Deal |
| Multiple transcripts, a CSV, a pasted list of notes | Batch |

Also tag each deal: **Win** (we closed), **Loss** (competitor or no-decision), **Ghost** (went dark). The synthesis framework applies to all three; ghost analysis focuses on unresolved friction rather than final decision factors.

### Step 2 — Pre-process

Segment the raw input by logical speaker/source chunks. For transcripts: caller vs. prospect. For CRM notes: closed-won vs. closed-lost field, rep commentary vs. prospect-attributed statements. Never mix attributed and unattributed claims in the final output.

If the input is too short to synthesize (fewer than ~3 substantive prospect statements), say so and ask for more material rather than hallucinating themes.

### Step 3 — Extract with MEDDIC+ Framework

The extraction runs against the **MEDDIC+ lens** — the six canonical B2B deal-qualification dimensions, extended with two win-loss-specific tracks:

| Dimension | Extract |
|---|---|
| **Metrics** | What measurable outcome did the buyer name? What number did they anchor to? |
| **Economic Buyer** | Who held final authority? Was there a blocker above the champion? |
| **Decision Criteria** | Explicit product/vendor requirements; evaluation rubric; must-haves vs. nice-to-haves |
| **Decision Process** | Steps, timeline, committee composition, vendor shortlist, scoring method used |
| **Identify Pain** | Stated problem, urgency driver, cost of inaction (explicit or implied) |
| **Champion** | Buyer advocate inside; how strong? Did they have internal support? |
| **+ Competitive Criteria** | What did they compare? Which competitors? On what dimensions? Verbatim if available |
| **+ Objection Inventory** | Every stated concern, hesitation, or "we'd need to see X" — verbatim + category tag |

Extract only from evidence in the transcript. Inferred items are marked `[inferred]`. Absent items are left blank, not fabricated.

### Step 4 — Synthesize the output

**Single-Deal output** (inline, ~1–2 pages):

```
## Win-Loss Summary — [Deal Name / Date / Outcome]

Brand: [slug, via brand-brain]
Outcome: Win | Loss (to: [competitor or reason]) | Ghost

### Decision Factors (ranked by buyer emphasis)
1. [Factor] — [evidence quote or paraphrase] — Positioning status: [confirms / gap / partial]
2. …

### Competitive Criteria
What they compared: [dimensions]
Competitors evaluated: [names]
Head-to-head moments: [verbatim quote or paraphrase → outcome]

### Key Quotes
| Quote | Speaker | Dimension | Tag |

### Objection Inventory
| Objection | Category | Verbatim | Resolved? |

### Positioning Gaps & Confirmations
[What this deal tells us about current messaging: where it landed, where it missed]

### Recommended next skills
- objection-library-builder → [new objections to add/update]
- positioning-messaging-architect → [gap to address]
- competitive-intelligence-dossier → [competitor to update]
```

**Batch output** (save to `./research/win-loss-synthesis-[slug]-[date].md`):

All of the above, plus:
- **Theme Hierarchy** — decision factors with frequency counts (n=X deals), segmented by Win vs. Loss vs. Ghost
- **Objection Frequency Table** — objection categories ranked by deal count
- **Competitive Win/Loss Rate** — by named competitor where data allows
- **Positioning Signal Summary** — confirms vs. gaps across the batch with a weighted view

Batch output is always saved to disk; single-deal output is inline unless the user asks to save.

---

## MEDDIC+ Extraction Rules

- **Quote fidelity is sacred.** Verbatim means verbatim — no paraphrasing a quote labeled as a quote. Paraphrases are labeled `[paraphrase]`.
- **Attribution discipline.** Rep-stated context and buyer-stated evidence are tagged separately. A rep saying "they cared about price" is not the same as a buyer saying "we couldn't justify the cost at that tier."
- **Evidence-only themes.** In Batch mode, a theme requires ≥2 independent deal-level occurrences to appear in the theme hierarchy. Single-deal mentions go in a "single mention" appendix — informative but not a signal yet.
- **Outcome-labeled framing.** Every insight in the positioning-gap section is labeled by its deal outcome — a gap in a won deal is informational; the same gap in a pattern of losses is a priority.
- **No coaching or editorializing.** This skill reports what happened in the deal. It does not assess whether the rep handled it well. Rep coaching belongs elsewhere.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No synthesis before the active brand's positioning is loaded — gaps can only be identified against what the brand currently claims.
- **Quote fidelity over brevity.** When a quote is powerful, keep it verbatim, even if long. Don't sanitize the buyer's voice to sound polished.
- **Evidence-only claims.** Every decision factor, competitive criterion, and objection traces to text in the input. Nothing inferred without labeling.
- **Attribution always.** Buyer said vs. rep inferred vs. synthesizer inferred are always distinguished.
- **Route, don't redo.** When the output calls for an objection library update or competitive brief, offer the downstream skill — do not re-derive what those skills own.
- **Honest about thin data.** Three deal notes do not make a pattern. Say so.

## What Not to Do

- Don't synthesize before `brand-brain` loads and returns the positioning digest.
- Don't invent quotes, themes, or objections not present in the input.
- Don't conflate rep attribution with buyer attribution.
- Don't declare a "pattern" from a single data point — mark it as a single mention.
- Don't write positioning copy or objection handlers here — offer `positioning-messaging-architect` and `objection-library-builder` for that.
- Don't reimplement brand scanning, voice coding, or objection library storage — call the sibling skills.
- Don't editorialize on rep performance — this is market intelligence, not call coaching.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand's positioning, ICP, and competitors loaded before extraction began?
- Every quote labeled verbatim vs. paraphrase; buyer-attributed vs. rep-attributed?
- MEDDIC+ dimensions extracted from evidence only; absent fields left blank, not fabricated?
- Positioning gaps and confirmations explicitly mapped to what the brand currently claims?
- Inferred items marked `[inferred]`; single-mention items not elevated to patterns?
- Single-Deal: inline output with recommended downstream skill handoffs?
- Batch: saved to `./research/win-loss-synthesis-[slug]-[date].md`; theme hierarchy requires ≥2 deal occurrences?
- Downstream skill handoffs offered (objection-library-builder, positioning-messaging-architect, competitive-intelligence-dossier, battlecard-objection-handler) with the exact inputs to pass?
