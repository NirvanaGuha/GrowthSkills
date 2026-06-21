---
name: initiative-business-case-writer
description: >
  Turns a rough initiative idea — a channel test, a tool purchase, a headcount ask, a campaign
  investment, a product bet — into a tight, CFO-defensible one-pager. Takes the initiative
  description, a rough cost estimate, and the expected outcome, then applies the Problem–Solution–
  Financials–Risk framework to produce a structured business case: a crisp problem statement grounded
  in data, a solution with rationale and scope, a quantified financial model (cost, expected return,
  payback period, confidence tier), a risk register with mitigations, and a clear recommendation. The
  output is shareable as-is with a budget owner or executive sponsor. Calls `brand-brain` to load
  business context, `proof-vault` to ground proof points, and the `ltv-cac-payback-calculator` for
  unit-economics inputs. Use when someone says "write a business case," "make the case for this,"
  "justify the spend," "I need to get budget approved," "build the one-pager for leadership," "help
  me pitch this initiative," "get sign-off on," or hands over an initiative brief and asks for
  financial justification.
---

# Initiative Business Case Writer

Turn a rough idea into a budget-approved one-pager. This skill takes an initiative, cost estimate, and expected outcome, then builds the case a CFO or executive sponsor will actually act on — not a slide deck filled with hopeful adjectives, but a structured document with a quantified problem, a scoped solution, a model with stated assumptions, and a risk register.

The framework is borrowed from the best corporate strategy practices and made usable by a solo growth marketer in under 30 minutes.

---

## Skills this calls

- **`brand-brain`** (required) — loads business context: ICP, positioning, current offer mechanics, proof points, and any stated company goals or strategic priorities the case should align to. Do not substitute with guessing.
- **`proof-vault`** (when installed) — retrieves real customer proof, benchmark data, or performance numbers to anchor the problem statement and projected returns.
- **`ltv-cac-payback-calculator`** (when installed) — provides unit-economics inputs (LTV, CAC, blended margin) needed for the financial model. Pull inline from the user if the skill is absent.
- **`positioning-messaging-architect`** (optional) — consult if the initiative involves repositioning or a new segment, to ensure the case's messaging rationale is coherent.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain; note strategic priorities + proof
Step 1  Clarify scope       ──► confirm initiative, cost estimate, and expected outcome
Step 2  Build the case      ──► run the PSFR framework (see below)
Step 3  Self-review         ──► quality-check against the checklist, then present
Step 4  Save artifact       ──► write to ./business-cases/[slug]-business-case.md
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns: the brand's ICP, current product/pricing, existing proof, strategic direction, and voice. Use any stated company goals (growth rate targets, CAC/LTV benchmarks, strategic themes) to frame the case's alignment section. Banned words from the brand override copy here too.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If none exists, ask the user for: (1) the company's core strategic goal this quarter, (2) the budget owner's key metric (revenue, CAC, payback period), (3) any real benchmark numbers they have. Proceed only after you have these.

### Step 1 — Clarify scope (before writing)

If any of the following are missing, ask — batch the questions:

1. **Initiative:** one-sentence description of what exactly is being proposed.
2. **Cost:** total spend (or range) — hard cost (tools, ads, contractors) + internal time at a loaded rate if relevant.
3. **Expected outcome:** what changes, by how much, by when. Rough is fine; you will add confidence tiers.
4. **Decision owner:** who signs off (CFO, CMO, founder)? This sets the framing register.
5. **Urgency:** is there a window (competitive timing, seasonal, contract renewal)?

Do not write the case against blank fields — a business case built on assumptions the user hasn't confirmed will fail at the meeting.

---

## The PSFR Framework

Every business case this skill produces follows four sections in this order. Each section has a hard job; do not blend them.

### Section 1 — Problem (the cost of inaction)

**Job:** make the decision-maker feel the pain of doing nothing, using numbers.

- State what is currently broken, constrained, or underperforming. Anchor to data: a metric that is off-target, a competitive gap, a cost that is rising, a revenue pool being left on the table.
- Quantify the problem in the decision-maker's preferred currency (revenue at risk, CAC above benchmark, payback period drifting). Pull from `proof-vault` or user-supplied numbers. Mark unconfirmed estimates `[verify]`.
- Name the strategic stakes: how does this problem compound if left unaddressed for 6 / 12 months?
- Keep it to 2–3 sentences + one anchor stat. Do not editorialize or use adjectives that are not supported by the data.

### Section 2 — Solution (the scoped proposal)

**Job:** define what you are actually proposing — no ambiguity, no scope creep.

- One-paragraph description: what the initiative is, how it works, what it requires (headcount, tools, timeline), and what it explicitly does NOT include.
- Explain the mechanism: why does this solution address the root cause identified in Section 1? Link the lever to the outcome.
- State the primary success metric and a secondary metric. These must be measurable from systems the team already has.
- Note strategic fit: how does this align to a current company priority loaded from `brand-brain`? If it doesn't align to a stated priority, name that honestly — the case is weaker and should acknowledge it.

### Section 3 — Financials (the model)

**Job:** give the budget owner a number they can defend upward.

Build the model in this structure:

```
## Financial model
Cost
  Hard cost:          $X  (tools/ads/contractors — itemized)
  Internal time:      $Y  (hours × loaded hourly rate — or "not modeled")
  Total investment:   $Z

Expected return (12-month horizon unless stated otherwise)
  Primary lever:      [metric] × [delta] × [value per unit] = $A
  Secondary lever:    [metric] × [delta] × [value per unit] = $B  (optional)
  Total expected return: $A + $B

Model output
  Expected ROI:       [(return − cost) / cost] × 100 = X%
  Payback period:     [cost / monthly return] = N months
  Confidence tier:    Low / Medium / High  (see below)

Key assumptions
  1. [assumption 1]
  2. [assumption 2]
  …
```

**Confidence tiers:**
- **High** — primary lever is already proven in a past test or via analogous case (cite it); model uses actuals or benchmark from a credible source.
- **Medium** — lever is plausible based on industry benchmarks `[verify source]`; one or two assumptions are untested.
- **Low** — greenfield; model is directional only; recommend a discovery sprint or pilot before full commitment.

Mark every unconfirmed number `[verify]`. Never fabricate benchmarks. If a number is illustrative, say "illustrative — replace with actuals."

If `ltv-cac-payback-calculator` is available, call it for unit-economics inputs and cite the returned values directly.

### Section 4 — Risk & Recommendation

**Job:** disarm the objections before the meeting and give a clear ask.

**Risk register (table):**

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| [e.g. adoption lower than projected] | Medium | High | Pilot 30 days before full rollout |
| [e.g. dependency on vendor timeline] | Low | Medium | Confirm SLA before signing |

Limit to the 3–5 real risks, not a kitchen-sink list. A risk register with 12 items signals the author hasn't prioritized; it undermines the case.

**Recommendation block:**
- **Ask:** exact spend, timeline, and owner (no vague "allocate resources").
- **Next step:** the single next action if approved (e.g., "sign contract by [date]," "launch pilot week of [date]").
- **If not approved:** what does the team do instead? A business case that ignores the no-path looks naive.
- **Urgency hook** (if real): state the time-bounded reason to decide now — contract renewal, competitive window, seasonal timing. Do not fabricate urgency.

---

## Output format

Deliver as a clean Markdown one-pager suitable for copy-paste into Notion, Google Docs, or a Slack message. Use this shell:

```markdown
# Business Case: [Initiative Name]
**Prepared for:** [Decision owner]  |  **Date:** [today]  |  **Owner:** [initiative owner]
**TL;DR:** [one sentence: the ask, the expected return, the confidence tier]

## 1. Problem
[Section 1 content]

## 2. Solution
[Section 2 content]

## 3. Financials
[Section 3 model block]

## 4. Risk & Recommendation
[Risk table + Recommendation block]
```

Save to `./business-cases/[initiative-slug]-business-case.md`. Tell the user the path.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No case before context is loaded. Strategic alignment requires knowing the company's actual priorities.
- **Numbers or nothing.** Every claim in Sections 1 and 3 must be sourced, estimated with stated assumptions, or marked `[verify]`. Adjectives without data are credibility killers in front of finance.
- **Confidence honesty.** Mark the tier accurately — a Low-confidence case presented as High will fail at the first question. A clearly Low-confidence case with a proposed pilot can still get approved.
- **Scope is sacred.** The solution section must say what is NOT included. Scope creep is the number-one reason business cases die in execution.
- **One ask.** The recommendation block makes exactly one ask. A menu of options signals the author hasn't made a decision.
- **No fake urgency.** If there is no real time-bounded reason to decide, don't invent one — it reads as pressure and weakens trust.

---

## What Not to Do

- Don't write a Section 3 without asking for cost and expected outcome — a case with blank financials is useless.
- Don't reimplement brand scanning or ICP derivation — call `brand-brain`.
- Don't use vague outcome language ("improve conversion," "increase brand awareness") without a metric and a delta.
- Don't list more than 5 risks in the register — prioritization is the skill, not enumeration.
- Don't recommend a High confidence tier when the primary data is unverified benchmarks.
- Don't editorialize in the Problem section — data earns urgency; adjectives don't.
- Don't deliver the output without saving the artifact to `./business-cases/`.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded; strategic priorities noted?
- Section 1 has at least one anchor stat (real or `[verify]`); cost-of-inaction quantified?
- Section 2 names the mechanism linking solution to root cause; success metrics are measurable?
- Section 3 model is itemized: hard cost, expected return, ROI %, payback period, confidence tier, and key assumptions listed?
- Every unconfirmed number marked `[verify]`; no invented benchmarks?
- Risk register has 3–5 prioritized risks with mitigations, not a kitchen-sink list?
- Recommendation block has a specific ask (amount + timeline + owner), a next step, a no-path, and urgency only if real?
- Output saved to `./business-cases/[slug]-business-case.md` and path reported to user?
