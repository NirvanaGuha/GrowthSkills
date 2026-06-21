---
name: prompt-engineering-library-suite
description: >
  Raw prompt + sample outputs + regression cases → rewritten prompt with annotated fixes, refined
  system prompt, before/after comparison, and versioned library storage. A systematic workbench for
  writing, critiquing, refining, and maintaining AI prompts across any use case — from one-off
  rewrites to a full team library of production prompts. Applies structured prompt-engineering
  methodology (role/task/format/constraints/examples/chain-of-thought scaffolding) to diagnose why
  a prompt underperforms and produce a version that reliably does better, with documented reasoning
  a junior can learn from. Saves versioned prompt files to ./prompts/ for library management,
  regression testing, and diff review over time. Use when the user says "improve this prompt,"
  "write a system prompt," "my AI keeps getting this wrong," "help me build a prompt library,"
  "version control my prompts," "prompt engineering," "write a meta-prompt," "chain-of-thought
  prompt," "few-shot prompt," or "review / audit my prompts."
---

# Prompt Engineering Library Suite

Turn flaky AI instructions into reliable, versioned production prompts. Give it a prompt that disappoints and it diagnoses the failure, rewrites with annotated reasoning, and saves a versioned file to your library. Build the discipline once; every prompt you ship after is faster and better.

Two modes: **Rewrite** (default — one prompt in, one stronger prompt out, before/after annotated) and **Library** (build, version, organize, and regression-test a collection of prompts). Both produce real, runnable output — not coaching.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, ICP, banned words, and offer so any prompt targeting user-facing output stays on-brand without manually pasting brand guidelines into every prompt.
- *(compose when relevant)* `automation-workflow-designer-debugger` for multi-step prompt chains embedded in no-code workflows; `sop-builder-reviewer` to capture the prompt-engineering process as a team SOP; `data-qa-measurement-gotcha-checker` when evaluating prompt output quality against structured data.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (voice + banned words for any user-facing prompt)
Step 1  Pick the mode       ──► Rewrite (default) | Library
Step 2  Diagnose            ──► classify failure; score against the CRAFT rubric
Step 3  Rewrite / act       ──► produce real output; annotate every change
Step 4  Save & version      ──► write ./prompts/<slug>-v<N>.md + update index
Step 5  Offer regression     ──► stub test cases for the key failure modes caught
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before producing any prompt that will generate user-facing content. Use the returned voice adjectives, banned words, ICP context, and offer mechanics as hard constraints embedded directly into the rewritten prompt's system instructions.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, ask the user for voice adjectives, banned phrases, and ICP in a single question block, then proceed.

---

## Rewrite mode (default)

Triggered by: a raw prompt pasted in, or "improve / fix / rewrite this prompt."

### Diagnosis pass — the CRAFT rubric

Score the submitted prompt on five axes (1–5 each). Flag every axis below 4:

| Axis | What it checks |
|---|---|
| **C — Clarity** | Is the task unambiguous? Could a careful person misread the instruction? |
| **R — Role & context** | Does the model have a role, persona, or relevant background to operate from? |
| **A — Anchors** | Are constraints, length, format, and tone explicitly stated? Implicit ≠ reliable. |
| **F — Few-shot / examples** | For complex or stylistically specific output, are 1–3 examples provided? |
| **T — Test against failure** | Would this prompt survive the most obvious adversarial or edge input? |

Show the rubric table with scores and one-line diagnosis per axis. If a submitted sample output exists, classify the failure type before scoring:

- **Scope drift** — model follows a different interpretation of the task
- **Format bleed** — output structure wrong (markdown where plain text needed, wrong length, etc.)
- **Hallucination / invention** — model fills missing context with confident fabrication
- **Style mismatch** — tone, register, or vocabulary wrong for the audience
- **Ambiguity fork** — multiple valid readings, model chose the wrong branch
- **Chain break** — multi-step reasoning lost the thread mid-task

### Rewrite pass

1. **Restructure using the six-block scaffold:**
   ```
   [ROLE]       Who the model is + relevant expertise framing
   [TASK]       One precise instruction sentence (verb-first, no "please")
   [CONTEXT]    Everything the model cannot infer: audience, channel, constraints
   [FORMAT]     Exact output shape: sections, length, tone, what to omit
   [EXAMPLES]   1–3 gold examples (required when style/edge cases are load-bearing)
   [CHAIN]      Step-by-step reasoning instruction, only when needed
   ```
   Omit blocks that add noise without value — not every prompt needs [CHAIN].

2. **Annotate every change.** Inline comments (`# ← reason`) or a change table below the prompt. Name the failure type each change addresses. A junior reading this should understand why, not just what.

3. **Present before → after** with the score delta on the CRAFT rubric.

4. **Offer a regression stub:** 2–3 test input/expected-output pairs that cover the failure modes caught. Store them as `## Regression cases` in the saved file.

Output for a quick single-prompt rewrite stays compact: diagnosis table, annotated rewrite, before/after, regression stub. No padding.

---

## Library mode

Triggered by: "build a prompt library," "organize my prompts," "version this prompt," "prompt index," "template library," or ≥3 prompts submitted together.

### Library structure (saved to `./prompts/`)

```
./prompts/
  _index.md              ← manifest: slug, version, purpose, owner, last-tested date
  <slug>-v1.md           ← first approved version
  <slug>-v2.md           ← subsequent versions (never overwrite; always increment)
  <slug>-regression.md   ← regression cases growing over time
```

**File format for each versioned prompt:**

```markdown
---
slug: <slug>
version: <N>
purpose: one-line job description
owner: <name or team>
model-family: <e.g. claude-3-5, gpt-4o, gemini-1.5>
brand: <slug or "generic">
status: draft | approved | deprecated
last-tested: YYYY-MM-DD
---

## Prompt

[full prompt text here]

## Changelog
- v1: initial
- v2: fixed scope-drift issue (CRAFT-C), added format anchor for JSON output

## Regression cases
| Input | Expected output summary | Pass/Fail |
```

### Library operations

| Command | Action |
|---|---|
| `add <prompt>` | Diagnose, rewrite, version as v1, add to index |
| `update <slug>` | Rewrite existing prompt; increment version; log changelog entry |
| `audit` | Review all index entries; flag missing regressions, stale `last-tested`, deprecated prompts still in use |
| `test <slug>` | Run regression cases; mark Pass/Fail; update `last-tested` |
| `diff <slug> v1 v2` | Show structural diff between two versions with CRAFT delta |
| `deprecate <slug>` | Set status to deprecated in index + file; log reason |

---

## Chain-of-thought and meta-prompt patterns

For complex reasoning tasks, add [CHAIN] only when the model needs to hold intermediate state or the task has load-bearing sub-steps. Patterns worth naming:

- **Scratchpad-then-answer:** "Reason step by step in a <scratchpad> block, then provide only the final answer outside it." Prevents the model from committing to early wrong conclusions.
- **Self-critique loop:** "After drafting your answer, critique it against [criteria], then revise once." Effective for copy review, analysis, and structured outputs where the first pass is predictably shallow.
- **Role-then-task split:** Separate the role-setting (system prompt or first user turn) from the actual task (second turn). Keeps the model anchored when the task is long.
- **Anchor-first for constrained output:** State format/length/tone constraints before the task sentence — models weight earlier tokens more heavily.
- **Few-shot over explanation for style:** Showing three examples of the right voice beats three paragraphs describing it.

---

## Principles

- **Diagnose before rewriting.** Naming the failure type sharpens the fix and teaches the user something durable.
- **Annotate every change.** An unannotated rewrite is a black box; annotated rewrites transfer craft.
- **Version, never overwrite.** Prompt regression is real — always increment. The index is your audit trail.
- **Embed brand context, don't describe it.** Put the brand's actual voice adjectives and banned words into the prompt constraints, not a vague "be professional."
- **Prompts are code.** They need test cases, versioning, and ownership like any production artifact.
- **Six blocks, not six sections.** Omit blocks that add no signal. Sparse + precise beats padded + comprehensive.

## What not to do

- Don't rewrite before diagnosing — the fix should follow from the failure type, not instinct.
- Don't embed live brand data inside a versioned prompt file — reference the brand slug and load at runtime; brands change.
- Don't overwrite a prompt version — increment. The previous version is evidence if the new one regresses.
- Don't add [CHAIN] to simple tasks — it adds latency and can introduce false intermediate conclusions on low-complexity prompts.
- Don't call the rewrite "better" without showing the CRAFT score delta or a concrete reason.
- Don't skip regression stubs — the most common failure is fixing the known bug and breaking an adjacent case.

## Quality checklist (self-review before presenting)

- `brand-brain` called and digest applied as hard constraints in any user-facing prompt?
- CRAFT rubric scored; failure type(s) named before rewriting?
- Every change annotated with the reason and failure type it addresses?
- Before → after presented with CRAFT delta?
- Prompt saved to `./prompts/<slug>-v<N>.md` with frontmatter, changelog, and regression cases?
- `_index.md` updated if library mode or if this is a new slug?
- Regression stub includes ≥2 cases covering the primary failure mode caught?
