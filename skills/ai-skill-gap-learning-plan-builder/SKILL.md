---
name: ai-skill-gap-learning-plan-builder
description: >
  Takes a current role (or honest self-assessment), a target role or capability goal, and a weekly
  hour budget → produces a side-by-side skill gap analysis plus a structured 4–12 week AI and
  marketing learning plan with sequenced resources, weekly milestones, and a prioritized backlog of
  skills to build. Works equally for individual growth (marketer levelling up to "growth engineer")
  and team capability planning (manager auditing an entire function). Distinguishes AI-specific gaps
  (prompting, agent design, model selection, eval) from marketing-domain gaps (conversion, SEO,
  attribution, lifecycle) so each is addressed with the right resource type. Does NOT recommend tools
  in isolation — for a scored tool comparison use `ai-tool-evaluator`. Uses the brand brain only to
  anchor role context and voice in any stakeholder-facing deliverable. Use when the user says
  "what AI skills do I need to learn," "build me a learning plan," "skill gap analysis," "how do I
  level up from X to Y," "90-day upskilling plan," "my team needs AI training," "career roadmap for
  growth marketer," or pastes a job description and asks what's missing.
---

# AI Skill Gap & Learning Plan Builder

Diagnose the gap between where you are and where you need to be, then turn it into a sequenced learning plan you will actually complete. Combines honest gap analysis with a realistic schedule, because an 80-item reading list with no sequence or time-boxing is not a plan.

Powered by the **70/20/10 learning model** (Lombardo & Eichinger): 70% learning by doing (shipped work, live experiments), 20% social/peer learning (cohorts, communities, peer review), 10% formal instruction (courses, docs). Resources are slotted accordingly — not just a pile of links.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand context before writing any stakeholder-facing deliverable (team capability plan, exec summary, hiring scorecard attachment). Voice + banned words override any output.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [role context, target capability, and tone for any deliverable they want].
- **`ai-tool-evaluator`** (optional, when a resource comparison is needed) — scores candidate learning tools/platforms; call it when the user asks "which course platform is better" rather than inlining a DIY comparison.
- **`icp-persona-builder`** (optional) — if the plan is for a team and no role personas exist, call it to build a quick learner profile; synthesize inline when absent.
- **`quarterly-goal-decomposer`** (optional) — for plans spanning a full quarter, delegate the milestone-breakdown pass; synthesize inline when absent.
- **`marketing-roadmap-builder`** (optional) — when the output needs a roadmap format for a team or exec audience; synthesize inline when absent.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (for any stakeholder output)
Step 1  Gather inputs        ──► current role/skills, target, hours/week, deadline
Step 2  Run the gap analysis ──► side-by-side table, tiered by urgency
Step 3  Build the plan       ──► 4–12 week schedule, 70/20/10 mix, weekly milestones
Step 4  Deliver & save       ──► inline + optional save to ./learning/
```

### Step 0 — Load brand (when stakeholder output is in scope)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing any branded deliverable (team plan, exec summary, hiring-scorecard attachment). On personal learning plans with no brand audience, proceed directly to Step 1.

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [role context, target capability, and tone for any deliverable they want].

### Step 1 — Gather inputs (gate before proceeding)

Minimum viable inputs:
- **Current role/level** — title or honest self-assessment of working skills (not just the job title)
- **Target** — role, job description, capability goal, or promotion criteria
- **Available hours/week** — be conservative; the plan must fit real life
- **Deadline or urgency** — "I'm interviewing in 6 weeks" shapes a different plan than "next year"

If these are missing, ask for them in a single batch before running. Do not guess a gap analysis without knowing both poles.

Optional enrichers:
- Job description or LinkedIn posting (for precise skill extraction)
- Previous performance review or 360 feedback
- "Biggest fear" about the target role (often the highest-leverage gap)

---

## Gap Analysis (Step 2)

Apply the **Dreyfus Skill Model** (novice → competent → proficient → expert) to rate each skill on a 1–4 scale. Any skill rated at target-level or above is a strength — do not pad the plan with it. Any gap of ≥2 levels is Priority 1; ≥1 level is Priority 2; adjacent/nice-to-have is Priority 3.

Separate the gap space into two lanes:

**AI & Automation lane** — skills specific to working with AI systems:
| Skill | Current | Target | Gap tier | Domain |
|---|---|---|---|---|
| Prompt engineering (structured, role, chain-of-thought) | | | | AI |
| Multi-agent / orchestration design | | | | AI |
| Model selection & cost-quality tradeoffs | | | | AI |
| Eval design (LLM-as-judge, regression suites) | | | | AI |
| RAG / knowledge-base architecture | | | | AI |
| Automation workflow design (Zapier/Make/n8n) | | | | AI |
| AI output quality review (slop detection, fact-checking) | | | | AI |
| Ethical/legal AI use in marketing | | | | AI |

**Marketing-domain lane** — role-specific gaps (populate from the target role; examples below):
| Skill | Current | Target | Gap tier | Domain |
|---|---|---|---|---|
| Conversion rate optimisation & experimentation | | | | Marketing |
| Attribution modeling & GA4 | | | | Marketing |
| Growth modeling (acquisition/retention loops) | | | | Marketing |
| Lifecycle & segmentation design | | | | Marketing |
| Paid acquisition (strategy, not button-pushing) | | | | Marketing |
| SEO (technical + topical authority) | | | | Marketing |
| Product-led growth mechanics | | | | Marketing |
| Data storytelling & exec communication | | | | Marketing |

Fill only the rows relevant to the actual target. Flag any cell as `[verify with a real JD]` if the target requirements are inferred rather than explicit.

**Gap summary:** Surface the top 3–5 gaps with the highest leverage — the ones that gate everything else or appear in every target JD.

---

## Learning Plan (Step 3)

### Length calibration

| Urgency / hours per week | Plan length |
|---|---|
| ≤3 hrs/wk or deadline ≤4 weeks | 4-week sprint — cover only P1 gaps |
| 4–6 hrs/wk | 6–8 week plan |
| 7+ hrs/wk or no hard deadline | 10–12 week plan with a P2 layer |

Never extend a plan past 12 weeks; beyond that it becomes a goal, not a plan. If the gap is larger than 12 weeks of available hours can close, say so and sequence what closes the most valuable gaps first.

### 70/20/10 resource mix

For each priority gap, slot resources into three buckets:

**70% — Learn by doing (projects and experiments)**
- Live experiments using the skill on real work (not toy exercises)
- Shipped deliverables using the new capability (blog post, A/B test, automation workflow)
- Build one small system per AI skill gap (e.g., build a prompt library, write an eval suite)

**20% — Social/peer learning**
- Cohort courses with peer accountability (Maven, Reforge, On Deck)
- Community participation: Growth.Design, Lenny's Slack, Marketing AI Institute community, Indie Hackers
- Find one peer or mentor at the target level and schedule 2 async exchanges per plan

**10% — Formal instruction**
- Docs/courses: Anthropic docs, OpenAI Cookbook, Reforge, CXL, Coursera (Google Analytics cert), Semrush Academy, HubSpot Academy [verify current availability]
- Keep formal instruction ≤2 hrs/wk; it does not compound the same way as doing

### Weekly schedule template

```
Week N  [Theme — e.g., "Prompting fundamentals + first live experiment"]
  Mon–Tue  [Formal / reading — ≤2 hrs]
  Wed–Thu  [Build / experiment — the 70% project]
  Fri      [Reflect + share — post a learning, ask a peer for feedback]
  Milestone: [Specific deliverable to show, not a vague "understand X"]
  Skill tick: [Which gap row this closes or advances]
```

Milestones must be concrete artifacts (a prompt template, a shipped A/B test, a GA4 exploration, a draft automation), not self-assessed knowledge. If the milestone can't be shown to someone, rewrite it.

### Backlog (P2 and P3 gaps)

List the gaps not in the active plan as a sequenced backlog. P2 enters the next iteration; P3 is parking lot. Do not load the current plan with P3 items because enthusiasm is not time.

---

## Output format

**Personal plan (default):** inline in chat. Structured but conversational.

**Team / stakeholder plan:** formatted document saved to `./learning/[slug]-learning-plan.md` where `[slug]` is the user's name or team name. Includes a one-page exec summary, the gap table, the plan, and the backlog. Brand voice applied if brand-brain returned a digest.

**Hiring use case:** when the target role is a role the user is hiring for (not filling themselves), produce a skills matrix and interview scorecard attachment instead of a weekly schedule. Call `marketing-job-description-hiring-scorecard-writer` if installed.

---

## Principles (Non-Negotiable)

- **Honest gaps only.** A gap analysis that flatters instead of diagnoses wastes the plan. Mark `[verify]` on any rating that is assumed, not tested.
- **Doing over reading.** The 70% bucket is not optional filler. If the plan has no shipped artifacts it is not a learning plan, it is a reading list.
- **Sequence matters.** AI foundations (prompting, evaluation) gate almost everything else in the AI lane. Never put agent design before prompt engineering in a plan.
- **Calibrate to real hours.** A plan that needs 15 hrs/week when the user has 4 is not ambitious, it is self-sabotage. Ask and respect the constraint.
- **No tool lists without context.** Recommended resources must map to a specific gap and a specific week. Raw link dumps are not deliverables.
- **Brand-brain first for any stakeholder output.** Do not produce a branded team plan or exec deliverable before brand-brain returns.

---

## What Not to Do

- Do not produce a plan without knowing both the current state and the target — a gap requires two poles.
- Do not recommend tools in a scored comparison — call `ai-tool-evaluator` for that.
- Do not fill the AI lane with marketing-domain items or vice versa; keep lanes distinct.
- Do not extend a plan past 12 weeks — cut scope, don't inflate duration.
- Do not mark a milestone "complete" unless it is a concrete artifact.
- Do not reimplement brand scanning or voice derivation — call `brand-brain`.
- Do not invent course availability, pricing, or certification status — mark `[verify]`.

---

## Quality Checklist (self-review before presenting)

- brand-brain called (or fallback path followed) when a stakeholder deliverable is in scope?
- Both poles known: current level and target level?
- Gap table split into AI lane and marketing-domain lane, rated on Dreyfus 1–4?
- Top 3–5 leverage gaps identified and prioritized?
- Plan length calibrated to hours/week and deadline?
- Each week has a concrete milestone (an artifact, not a feeling)?
- 70% bucket has real shipped work, not just exercises?
- Backlog captures P2/P3 gaps without loading the active plan?
- All course/resource references either verified or marked `[verify]`?
- Team/stakeholder plan saved to `./learning/` with brand voice applied?
