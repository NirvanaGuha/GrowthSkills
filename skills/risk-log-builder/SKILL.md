---
name: risk-log-builder
description: >
  Turns a project plan, campaign brief, or initiative description into a structured risk register —
  every entry carrying a likelihood score, impact score, risk-priority number (RPN), mitigation
  strategy, contingency trigger, and a named owner. Two modes: Quick Register (one tight table from
  any brief, default) and Deep Register (full narrative per risk, pre-wired to mitigation tasks,
  with an executive summary heat-map). Calls brand-brain to keep risk framing and mitigation
  language on-voice and grounded in the brand's real constraints (banned promises, proof gaps,
  regulatory exposure). Composes with pre-mortem-post-mortem-generator for assumption mining,
  deadline-drift-detector for schedule risks, go-no-go-gate-evaluator for launch-gate integration,
  and project-plan-generator-reviewer for plan-quality context. Saves the register to a
  project-relative ./risks/ path so it ages alongside the plan. Use when the user says "build a
  risk log," "risk register," "what could go wrong," "identify campaign risks," "risk matrix,"
  "risk and mitigation," "map our risks," or hands you a brief and asks for risks before launch.
---

# Risk Log Builder

Every project plan is a theory. A risk log is the honest accounting of where that theory might break — before the breaking costs money, reputation, or a missed launch window. This skill takes a plan or brief and returns a prioritized, owner-assigned risk register that a team can act on, not just file away.

It does not moralize or pad. It identifies real, specific risks, scores them on a consistent 1–5 × 1–5 grid using FMEA-style RPN (Risk Priority Number = Likelihood × Impact), writes a mitigation that reduces the score (not just a platitude), names a trigger for the contingency, and assigns an owner. Anything unconfirmed is marked `[verify]`.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's constraints, voice, proof, and positioning. Risk language (especially customer-facing and legal risks) must obey the brand's banned words, proof standards, and regulatory context. Do NOT re-derive brand context here.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for the brand name, its product category, primary compliance/regulatory exposure, and any known banned claims or proof constraints.
- **`pre-mortem-post-mortem-generator`** (optional, on request or when input is sparse) — run a pre-mortem pass to mine implicit assumptions; feed the output as risk seeds into Step 2 below. Call it before scoring if the user says "pre-mortem" or the brief has fewer than three explicit dependencies.
- **`deadline-drift-detector`** (optional) — when a project plan with dates is provided, call it to flag schedule-risk inputs (milestones already at risk or with zero float); import those directly into the Schedule bucket.
- **`go-no-go-gate-evaluator`** (optional) — on request, use it to map the completed risk register against launch-gate criteria (pass/fail per criterion). A well-mitigated risk log is direct input to a go/no-go gate.
- **`project-plan-generator-reviewer`** (optional) — when available, use its quality-gate output (missing owners, over-optimistic durations, no test time) as additional risk seeds in Step 2.

---

## How a run works

```
Step 0  Load brand context      ──► call brand-brain
Step 1  Classify mode           ──► Quick Register (default) | Deep Register (on request)
Step 2  Mine risks              ──► structured extraction across 6 buckets
Step 3  Score & rank            ──► RPN = Likelihood (1–5) × Impact (1–5)
Step 4  Write mitigations       ──► reduce the score, name a trigger, assign an owner
Step 5  Format + save           ──► table (Quick) or full narrative (Deep); save to ./risks/
```

---

## Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill before any risk analysis. It returns the active brand's constraints — regulatory exposure, proof standards, banned claims, ICP fragility, and known operational gaps — that make certain risk categories higher-likelihood or higher-impact for this brand specifically. A generic risk log ignores these; this one doesn't.

Use the returned context to calibrate: a brand with `[verify]` claims in its proof vault carries higher reputation risk on launches; a brand in a regulated category (fintech, health) carries higher compliance risk; a scrappy brand with no dedicated ops owner carries higher execution risk.

---

## Step 1 — Pick the mode

- **Quick Register (default).** A clean risk table: one row per risk, six columns (Risk, Bucket, Likelihood, Impact, RPN, Mitigation + Trigger, Owner). Ranked by RPN descending. Inline output; saved to `./risks/[project-slug]-risk-log.md` if the user names the project.
- **Deep Register (on request).** Full narrative entry per risk: context paragraph, scoring rationale, mitigation task list (actionable, owner-assigned), contingency plan with explicit trigger condition, residual-risk score after mitigation. Plus an executive heat-map summary (3×3 visual grid: high/medium/low on each axis) and a top-3 risks one-pager for stakeholders. Triggers: "deep register," "full risk register," "narrative risk log," "board-ready," an explicit request for heat-map or executive summary.

---

## Step 2 — Mine risks across 6 buckets

Work systematically through the plan or brief. Every risk needs a specific failure mode, not a category label. "Timeline slippage" is a category; "third-party API integration takes 3 weeks instead of 1 because vendor documentation is incomplete" is a risk.

| Bucket | What to probe |
|---|---|
| **Execution** | Dependencies, single points of failure, hand-offs, resource constraints, tool/vendor reliability |
| **Schedule** | Zero-float milestones, external deadlines, parallel workstreams competing for the same owner, approval cycles |
| **Scope** | Unanchored requirements, stakeholder expectation gaps, feature creep, missing acceptance criteria |
| **Market / Audience** | Audience assumption errors, seasonality, competitor response, demand validation gaps |
| **Brand / Reputation** | Unverified claims, off-brand messaging, regulatory exposure, negative press surface, social blowback triggers |
| **Measurement** | Tracking not set up before launch, undefined success metrics, attribution black holes, no baseline |

Pull from the input explicitly. If the input is thin, call `pre-mortem-post-mortem-generator` to surface implicit assumptions, then treat each failed assumption as a candidate risk.

Aim for **6–15 risks** in Quick mode; more is not better — only include risks where a reader could do something. Duplicates, trivialities, and "risks" that are just features-not-yet-built belong in the scope log, not here.

---

## Step 3 — Score with FMEA-style RPN

Score each risk on two 1–5 scales:

| Score | Likelihood | Impact |
|---|---|---|
| 1 | Remote (≤5% chance) | Negligible — no deadline, budget, or customer effect |
| 2 | Unlikely (5–20%) | Minor — recoverable with no external visibility |
| 3 | Possible (20–40%) | Moderate — delays launch or requires workaround customers notice |
| 4 | Likely (40–70%) | Significant — budget overrun, customer-facing degradation, or team morale hit |
| 5 | Near-certain (>70%) | Critical — kills the launch, regulatory breach, or major customer churn |

**RPN = Likelihood × Impact.** Range: 1–25. Rank descending. Risks with RPN ≥ 15 are **Red** (immediate mitigation required before launch); 8–14 **Amber** (mitigation plan in place, monitored weekly); ≤7 **Green** (accepted or low-cost mitigation sufficient).

Do not inflate scores to seem thorough. A project with five Green risks is a well-planned project, not a failure of imagination.

---

## Step 4 — Write mitigations that actually reduce the score

Each mitigation entry has three parts:

1. **Mitigation action** — what you do *now* or *before* the risk materializes. Specific, assigned, time-boxed. E.g., "Lock API spec with vendor in writing by [date]; add 3-day integration buffer to sprint 2."
2. **Contingency trigger** — the specific observable signal that activates the contingency plan. E.g., "If vendor spec not confirmed by Day 5, escalate to PMO and activate backup library."
3. **Residual RPN** — the expected RPN *after* the mitigation is in place. If you can't reduce it, say so and recommend escalation.

Owner is a **role**, not a person's name (e.g., "Campaign PM," "Paid Media Lead") — keeps the log useful when teams change.

---

## Output format

### Quick Register

```
## Risk Register — [Project / Campaign Name]
Brand: [slug, via brand-brain]
Generated: [date]
Input: [brief title or first line]

| # | Risk | Bucket | L | I | RPN | Status | Mitigation + Trigger | Owner |
|---|------|--------|---|---|-----|--------|----------------------|-------|
| 1 | [specific failure mode] | [Bucket] | 4 | 5 | 20 | 🔴 Red | [action] · Trigger: [signal] | [Role] |
...

### Top 3 (RPN ≥ 15)
[One sentence per risk: what it is, why it's rated this way, what the mitigation buys you.]

### Accepted risks (Green, no action required)
[Brief list — shows the reader the scope was considered, not ignored.]
```

### Deep Register (on request, in addition to the table)

Per Red and Amber risk, add:
- **Context:** why this risk exists in this specific plan (one paragraph, no filler)
- **Mitigation task list:** 2–4 numbered actions with owner + due date
- **Contingency plan:** what happens if the trigger fires, who decides, in what timeframe
- **Residual RPN after mitigation:** and what risk remains accepted

Plus an executive heat-map (markdown 5×5 ASCII grid, L on x-axis, I on y-axis, risk numbers plotted) and a top-3 one-pager suitable for a stakeholder deck.

---

## Persistence

Save the risk log to `./risks/[project-slug]-risk-log.md`. Do not overwrite `brand.md` or any brand-brain companion file. On re-run (updated brief), diff against the existing log and surface new/changed/resolved risks — don't silently regenerate.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No risk scoring before brand context loads — brand constraints directly calibrate likelihood and impact in the Brand/Reputation bucket.
- **Specific failure modes only.** Categories are not risks. Name the mechanism of failure.
- **Mitigations must reduce the RPN.** If a "mitigation" doesn't change the score, it's not a mitigation — it's an acknowledgment. Flag it as accepted risk or escalate.
- **Honest scores.** Don't inflate to look thorough or deflate to avoid alarm. Use the rubric, show the rationale.
- **Owners are roles.** Not names; roles survive team changes.
- **Triggers are observable.** A contingency with no trigger is decoration. "If X happens" must be a thing a human can see.
- **Truth discipline.** Any risk grounded in an unverified claim (regulatory exposure, market assumption, competitor behavior) is marked `[verify]`.

---

## What Not to Do

- Don't pad to 20+ risks — more entries dilutes attention from the real ones.
- Don't use generic mitigation language ("monitor closely," "have a plan") — name the action, the owner, and the trigger.
- Don't re-implement brand context resolution; call `brand-brain`.
- Don't conflate a scope gap (missing feature) with a risk (failure mode of something that is planned).
- Don't score every risk as Red — calibrate honestly; a sea of Red means the rubric is broken, not the project.
- Don't save anything to the brand-brain data root; `./risks/` only.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand context loaded; reputation/compliance risks calibrated against brand constraints?
- Every risk is a specific failure mode, not a category label?
- RPN = L × I, scored against the defined rubric, with rationale?
- Every Red/Amber risk has a mitigation (action + trigger + residual RPN) and a role-based owner?
- Green risks listed as accepted (not silently omitted)?
- Quick: clean table ranked by RPN descending + top-3 summary? Deep: narrative + heat-map + one-pager?
- Saved to `./risks/[project-slug]-risk-log.md`; no brand-brain files touched?
- `[verify]` on any score grounded in an unconfirmed assumption?
