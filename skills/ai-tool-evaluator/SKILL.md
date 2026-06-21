---
name: ai-tool-evaluator
description: >
  Evaluates AI tools and platforms against a specific job-to-be-done — so you buy the right tool
  once instead of paying for three that overlap. Give it a job (e.g., "AI writing for long-form SEO
  content," "AI image generation for social ads," "AI meeting transcription and CRM sync") plus a
  candidate list or a named category, and it returns: a scored comparison matrix (capability, fit,
  pricing, integration, risk), a clear recommendation with rationale, a free-trial test plan you can
  run in 30–60 minutes per tool, and a pricing summary formatted for budget approval. Composes with
  `martech-stack-auditor-mapper` (detects overlap with your existing stack before recommending),
  `go-no-go-gate-evaluator` (runs a formal pass/fail on the finalist), and `vendor-cost-comparison-renewal-brief`
  (produces the budget memo). Evaluation only — it does not build, configure, or integrate the chosen
  tool; use `automation-workflow-designer-debugger` or `martech-stack-auditor-mapper` for that.
  Trigger phrases: "which AI tool should I use for," "compare AI tools," "evaluate these tools,"
  "help me pick an AI," "AI tool recommendation," "is [tool] worth it," "best AI for [job],"
  "we're evaluating," "AI tool shortlist," "replace [tool] with AI."
---

# AI Tool Evaluator

Pick the right AI tool once. This skill takes a job-to-be-done and a candidate list (or category), applies a structured evaluation framework, and returns a scored matrix, a ranked recommendation, a free-trial test plan, and a budget summary — so the decision is defensible, not a gut call.

It evaluates and recommends. It does not configure, build integrations, or run implementation. Once a tool is chosen, hand off to `automation-workflow-designer-debugger` or `martech-stack-auditor-mapper` for the build-out.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's context: stack, ICP, team, budget constraints, banned vendors, and voice requirements. Every tool recommendation is anchored to this brand's actual context, not a generic buyer profile.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the job-to-be-done, current toolstack, team size, monthly budget ceiling, and any hard exclusions (vendor, pricing model, data-residency)].
- **`martech-stack-auditor-mapper`** *(compose when installed)* — surface existing tools before recommending a new one; detect overlap and redundancy early.
- **`go-no-go-gate-evaluator`** *(compose when installed)* — run a formal pass/fail gate on the finalist tool before the final recommendation.
- **`vendor-cost-comparison-renewal-brief`** *(compose when installed)* — produce a budget-approval memo after the finalist is selected.
- **`roi-business-case-calculator`** *(compose when installed)* — quantify the ROI case for the top tool when stakeholder buy-in requires a financial model.

---

## How a run works

```
Step 0  Load the brand           ──► call brand-brain; load stack, budget, team context
Step 1  Define the job           ──► confirm JTBD + must-haves + dealbreakers
Step 2  Build the longlist       ──► surface candidates (user-supplied or category scan)
Step 3  Score the matrix         ──► Capability · Fit · Pricing · Integration · Risk
Step 4  Shortlist + compose      ──► call martech-stack-auditor-mapper for overlap check
Step 5  Finalize + gate          ──► call go-no-go-gate-evaluator on the finalist
Step 6  Deliver outputs          ──► matrix + recommendation + trial plan + pricing summary
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest including: current martech stack, team structure and size, monthly budget range, integration constraints (CRM, ESP, CMS, SSO), data-residency or compliance requirements, and any hard vendor exclusions. **Do not score any tool before brand-brain returns.**

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the job-to-be-done, current toolstack, team size, monthly budget ceiling, and any hard exclusions (vendor, pricing model, data-residency)].

### Step 1 — Define the job

Pin the job-to-be-done (JTBD) before touching any tool. A weak JTBD produces a useless matrix.

Confirm three things:
1. **The exact job** — what the tool must do, for whom, at what frequency.
2. **Must-haves** — non-negotiables (e.g., API access, SOC 2, specific integrations, self-hosted option).
3. **Dealbreakers** — automatic disqualifiers (pricing model, vendor lock-in, data export limits, conflicting existing contract).

If the user's request is underspecified ("best AI writing tool"), ask one clarifying question before proceeding: *"What's the primary job — blog drafting, product descriptions, email copy, or something else?"* One question, not a form.

---

## The JTBD×Fit Evaluation Framework (shared across all modes)

The framework is derived from Clayton Christensen's Jobs-to-Be-Done theory crossed with a five-dimension vendor-fit rubric. Tools do not have intrinsic quality; they have fit for a job. Score fit, not hype.

### Five scoring dimensions

| Dimension | What to score | Weight |
|---|---|---|
| **Capability** | Does it do the core job well — depth, accuracy, output quality? | 30% |
| **Fit** | Does it match this team's workflow, skill level, and stack without heavy adaptation? | 25% |
| **Pricing** | Does the realistic usage cost fit the budget ceiling? Is the model (per-seat, per-usage, per-output) appropriate? | 20% |
| **Integration** | Does it connect natively to the existing stack? REST API, Zapier/Make, native connectors? | 15% |
| **Risk** | Data ownership, vendor stability, lock-in depth, compliance (SOC 2, GDPR, HIPAA if relevant), exit cost? | 10% |

Score each dimension 1–5. Compute the weighted total (max 5.0). Flag any dimension scored ≤2 as a **red flag** regardless of total score — a tool that scores a 1 on Risk can't be rescued by a 5 on Capability.

### Longlist → Shortlist filter

Apply dealbreakers as hard exits before scoring. Any tool that trips a dealbreaker is removed from the matrix with a one-line explanation. The scored matrix covers only shortlisted tools (≤5; more clutters the output).

---

## Output format

### Scored comparison matrix

```
## AI Tool Evaluation — [Job: …]
Brand: [slug, via brand-brain]
Job-to-be-done: [confirmed JTBD]
Must-haves: [list]  |  Dealbreakers applied: [list + removed tools]

| Tool | Capability | Fit | Pricing | Integration | Risk | Weighted Score | Red Flags |
|------|------------|-----|---------|-------------|------|----------------|-----------|
| …    | /5         | /5  | /5      | /5          | /5   | /5.0           | —         |
```

### Recommendation

One clear pick. The recommendation section covers:
- **Winner** — tool name + one-sentence rationale anchored to the JTBD.
- **Runner-up** — when the winner has a notable gap the runner-up avoids (e.g., no API tier → runner-up has API).
- **Stack overlap note** — if `martech-stack-auditor-mapper` found overlap, note what gets deprecated or consolidated.
- **Conditions** — any assumption the recommendation depends on (e.g., "assumes the $X/mo budget is approved"; "assumes team size stays under Y seats").

### Free-trial test plan

A 30–60 minute structured test the user runs themselves to validate the recommendation before committing. Format:

```
## Trial Test Plan — [Winner tool]
Duration: [X minutes]  |  Who runs it: [role]

Task 1 — [Core JTBD task]: [exact prompt or action to run] → Pass if: [measurable criterion]
Task 2 — [Integration test]: [action] → Pass if: [criterion]
Task 3 — [Edge case / deal-risk test]: [action] → Pass if: [criterion]
Decision gate: if ≥2/3 pass, proceed to purchase. If Task [N] fails, escalate to runner-up.
```

### Pricing summary

Real pricing from known public tiers `[verify if changed]`. Format for budget approval:

```
## Pricing Summary
| Tool | Plan needed | Monthly cost | Annual cost | Per-unit note |
```

Flag any tool with usage-based pricing that has meaningful variance at scale (e.g., "at 100K words/mo, cost is $X [verify]"). Never invent pricing — mark unknown tiers `[verify current pricing at vendor site]`.

### Save artifact

Save the full evaluation to `./ai-tool-evals/[job-slug]-eval.md` when the user asks to save it, or when the matrix has 3+ tools. Inline for quick 1–2 tool comparisons.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No scoring before the brand's stack, budget, and constraints are loaded. Tool fit is relative to this brand.
- **JTBD first, tools second.** Never recommend a tool without confirming the exact job. "Best AI tool" is not a job.
- **Score fit, not hype.** A tool with a 5 on Capability that scores 2 on Integration or Risk is not the right answer.
- **Dealbreakers are hard exits.** A tool that trips a dealbreaker is removed from the matrix — it cannot be rehabilitated by high scores elsewhere.
- **Red flags surface regardless of total score.** Any single dimension ≤2 gets called out explicitly.
- **Honest pricing.** Known public tiers only; unconfirmed numbers are `[verify]`. Usage-based pricing gets a modeled estimate at the user's expected usage.
- **Compose, don't rebuild.** Overlap detection lives in `martech-stack-auditor-mapper`; pass/fail gating lives in `go-no-go-gate-evaluator`; budget memos live in `vendor-cost-comparison-renewal-brief`. Call them.

## What Not to Do

- Don't score tools before brand-brain returns the stack and constraints.
- Don't reimpose the JTBD from the category name — confirm the actual job with the user.
- Don't include tools that trip dealbreakers in the matrix; note them as excluded.
- Don't write pricing from memory for tools with frequent plan changes — mark as `[verify]`.
- Don't recommend and then immediately configure — evaluation ends at the recommendation; implementation is a separate skill.
- Don't produce a matrix with more than 5 tools — longlist → shortlist first.
- Don't bury a red-flag dimension in the weighted average; surface it explicitly.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand's stack, budget, and constraints loaded (or fallback applied)?
- JTBD confirmed with must-haves and dealbreakers before scoring began?
- Dealbreaker tools removed from the matrix with a one-line explanation?
- Five dimensions scored with weights applied; weighted totals accurate?
- Any dimension ≤2 flagged explicitly as a red flag?
- Winner + runner-up + conditions + stack-overlap note present in recommendation?
- Trial test plan has 3 tasks with measurable pass criteria and a decision gate?
- Pricing sourced from public tiers; unconfirmed numbers marked `[verify]`?
- Artifact saved to `./ai-tool-evals/[job-slug]-eval.md` when warranted?
- Compose calls noted: `martech-stack-auditor-mapper`, `go-no-go-gate-evaluator`, `vendor-cost-comparison-renewal-brief` invoked when installed and relevant?
