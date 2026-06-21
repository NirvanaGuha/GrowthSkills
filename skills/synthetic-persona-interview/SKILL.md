---
name: synthetic-persona-interview
description: >
  Runs a simulated focus group against one or more ICP personas — synthetic respondents built from
  real brand + segment context — to pressure-test messaging, positioning, landing-page copy, email
  subjects, pricing angles, and objection handling before real spend or real launch. The personas
  are not flat demographic archetypes; each is constructed from the brand's confirmed ICP, awareness
  stage, known objections, and job-to-be-done, so every response is grounded in plausible buyer
  psychology rather than fiction. Outputs: a structured transcript, a findings summary (what landed,
  what confused, what triggered objections), and a priority recommendation list the team can act on
  immediately. Use when the user says "simulate how my ICP would react," "pressure-test this
  messaging," "run this past a persona," "synthetic focus group," "pretend you're my customer,"
  "what objections would this raise," "do a persona interview," or hands over copy/positioning and
  asks what a real buyer would think.
---

# Synthetic Persona Interview

Pressure-test messaging, copy, or positioning against your ICP — before spending money finding out what was wrong.

This skill builds synthetic respondents from the brand's confirmed ICP context (loaded via `brand-brain`), then runs them through a structured interview or focus-group simulation using the **Pre-Suasion + Mental Contrasting (WOOP) framework**: each persona first states what they Want, then their Outcome expectation, then surfaces Obstacles, then reacts to the Plan (the stimulus being tested). That four-move structure ensures every response surfaces both the hopeful read and the realistic friction — which is where real research earns its money.

This skill simulates; it does not replace talking to real customers. When it flags a blind spot, book the real conversation.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, offer, proof, and voice. Do not construct personas before `brand-brain` returns.
- **`icp-persona-builder`** (call when the brand has no confirmed persona yet, or when the user wants richer demographic/psychographic depth beyond what `brand-brain` carries).
- **`objection-library-builder`** (call or reference when the brand has an existing objections library — synthetic responses should be calibrated against real documented objections, not invented wholesale).
- **`positioning-messaging-architect`** (optional — if the stimulus being tested is a positioning draft, having its author artifact available improves response specificity).
- **`voice-of-customer-mining-pipeline`** (optional — if real VOC data exists, surface it as a calibration layer so synthetic responses skew toward confirmed language, not generic SaaS-speak).

---

## How a run works

```
Step 0  Load brand context ──► call brand-brain (always first)
Step 1  Build the panel     ──► construct 2–4 synthetic respondents from ICP + segment
Step 2  Prime the stimulus  ──► what exactly is being tested, in what channel/context?
Step 3  Run the WOOP passes ──► each persona goes through Want → Outcome → Obstacle → Plan
Step 4  Cross-panel summary ──► where did consensus cluster? where did it fracture?
Step 5  Priority actions     ──► ranked list of fixes, each scoped to one change
```

Never skip Step 0. A persona built without the brand's ICP data is cosplay, not research.

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest — ICP definition, awareness tendency, positioning, offer mechanics, real proof, banned words — as the scaffolding for every persona. If `brand-brain` returns a `personas.md` companion file, load it; the existing personas are the starting point, not the construction from scratch.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user: (1) Who is the ICP — role, company stage, trigger event? (2) What awareness stage are they typically in when first encountering this brand? (3) What are the top 1–2 objections you hear on sales calls? Proceed only with those answers.

---

### Step 1 — Build the panel

Construct 2–4 synthetic respondents. Each persona needs:

| Field | Where it comes from | Example |
|---|---|---|
| **Name + archetype label** | invented but plausible | "Priya — The Skeptical Scaler" |
| **Role + company stage** | ICP definition from brand-brain | VP Marketing, Series B SaaS, 40 person team |
| **Awareness stage** (Schwartz) | brand-brain ICP awareness tendency | Solution-aware: knows the category, evaluating options |
| **Primary JTBD** | ICP job-to-be-done | Reduce cart-abandonment rate before Q4 peak |
| **Dominant objection archetype** | objection library or brand-brain proof gaps | Price/ROI skeptic · Trust/proof skeptic · Status-quo defender · Technical gatekeeper |
| **Decision power** | varies across panel | Champion / Economic buyer / Technical validator |
| **Emotional register** | drives tone of simulated voice | Cautious and data-driven · Impatient and ROI-focused |

Aim for contrast, not clones. At minimum: one champion-type persona + one skeptic/obstacle-holder. If the brand serves multiple segments, represent the highest-value and highest-friction segments.

If the user names a specific persona, build only that one. If they say "the usual suspects," build 3: champion, skeptic, and economic buyer.

Announce the panel to the user before running the simulation. Offer to adjust before proceeding.

---

### Step 2 — Prime the stimulus

Before running, state explicitly:

1. **What is being tested** — messaging headline, full landing page copy, pricing page, email subject + preview text, positioning one-liner, CTA battery, objection-handling script.
2. **The assumed channel/context** — cold ad at top-of-funnel, mid-funnel landing page, bottom-of-funnel demo request page, sales deck slide. This sets the persona's priming state (what they knew before seeing it).
3. **The awareness stage at point of contact** — pulled from the persona definition.

If the user hands over a stimulus without context, ask: "What channel will this appear in, and what did the reader just do before seeing it?" One question, not five.

---

### Step 3 — Run the WOOP passes

For each persona, run the four-move WOOP sequence. Write each persona's voice distinctly — the economic buyer uses finance framing; the champion uses operational framing; the skeptic challenges the proof.

#### W — Want
The persona states what they are hoping this stimulus will do for them. This surfaces the **expectation gap**: does the copy meet the reader where their hope actually lives, or does it pitch something adjacent?

> *[Persona name] reads/sees the stimulus. Internal monologue:*
> "What I actually need right now is ______. My first scan tells me this [does / doesn't / partially] promise that because ______."

#### O — Outcome expectation
The persona plays out the imagined result of clicking / buying / engaging. This surfaces **credibility gaps and value-proposition clarity**.

> "If I follow through on this, I expect to get ______. But I'm not sure that's actually what it delivers because ______."

#### O — Obstacle
The persona names the single biggest thing stopping them from taking the next action. This is the **friction heat-map**: price, trust, complexity, internal politics, missing proof, unclear next step.

> "The thing that's stopping me is ______. Specifically: ______."

Keep this honest. A persona that never objects is a useless one.

#### P — Plan (reaction to the stimulus as the plan)
The persona evaluates whether the stimulus adequately addresses or routes around the obstacle they named.

> "Does this copy/page/message resolve that obstacle? [Yes / Partially / No] — because ______."
> "What would I need to see to move forward? ______."

**Format the transcript as a clean table or block-quote per persona.** The user should be able to scan it in 90 seconds and immediately feel the fracture lines.

---

### Step 4 — Cross-panel summary

After all personas complete their WOOP passes:

**Consensus zones** — what did every persona agree on (positive or negative)?

**Fracture lines** — where did personas diverge? Which objection was segment-specific vs. universal?

**Highest-friction moment** — the single point in the stimulus where the most personas stalled or rejected.

**Biggest positive signal** — the claim or proof point that moved the most skeptics.

**Blind spots** — things the copy assumed the reader knew that none of the personas actually knew going in.

---

### Step 5 — Priority actions

Ranked list of 3–5 changes, each written as a concrete editorial or structural action. Not "improve the headline" — write the fix or write the brief for the fix.

Format:

```
## Priority Actions — [stimulus label]

1. [CHANGE TYPE: Rewrite / Add / Remove / Reorder] — [specific element]
   Why: [which persona / consensus insight drove this]
   Fix: [one concrete suggestion or new line]

2. ...
```

If the changes are material enough, flag which sibling skill handles the rewrite: `cta-variant-generator` for CTAs, `positioning-messaging-architect` for a positioning pivot, `objection-library-builder` to document the new objections surfaced.

Save the full output to `./research/persona-interview-[stimulus-slug]-[date].md` when the user asks to keep it. Inline otherwise.

---

## WOOP framework: why it works here

WOOP (Wish → Outcome → Obstacle → Plan) is a mental-contrasting framework from Gabriele Oettingen's implementation-intention research. Applied to synthetic persona interviewing, it forces each simulated respondent to move through the full psychological arc a real buyer traverses: aspiration → expectation → friction → decision. Without the Obstacle move, simulated responses are just positive paraphrasing of the copy. Without the Plan move, there is no verdict — only description.

Schwartz's Awareness Ladder calibrates the Wish and Outcome moves: a Solution-aware persona wishes for differentiation, not education; a Product-aware persona wishes for de-risking, not discovery. Mis-match between the copy's assumed awareness level and the persona's actual level is the single most common reason campaigns underperform — this structure surfaces it explicitly.

---

## Principles

- **Brand-brain first.** No persona is constructed before `brand-brain` returns. A persona without the confirmed ICP is fiction.
- **Contrast over consensus.** The most useful panel includes a skeptic. A focus group of champions tells you nothing you didn't know.
- **Voice stays distinct.** The economic buyer does not sound like the champion. Distinct vocabulary, distinct framing, distinct objection type.
- **Obstacles are mandatory.** Every persona must surface at least one blocker. If a persona has no objection, go back and make them harder.
- **The fix is concrete.** Priority actions are editorial briefs, not abstract "improve clarity" notes.
- **Honest limits.** When a synthetic finding could materially affect budget or strategy, say so: "This should be validated with 3–5 real discovery calls before acting."

---

## What not to do

- Do not build personas before `brand-brain` loads — demographic fiction without ICP grounding is worse than nothing.
- Do not produce a panel of clones — same awareness stage, same objection, same role. Contrast is the entire point.
- Do not simulate generic SaaS buyers. Anchor every persona to the brand's confirmed ICP and the specific stimulus context.
- Do not bury the priority actions in narrative. A practitioner needs to paste them into Jira in two minutes.
- Do not claim synthetic simulation replaces real customer interviews. Flag it when stakes are high.
- Do not invent proof points for personas to find convincing. Only use proof from `brand-brain`'s returned digest.

---

## Quality checklist

- `brand-brain` called and active brand loaded (or bootstrapped) before any persona was constructed?
- Panel has 2–4 distinct personas: at minimum one champion + one skeptic, with different awareness stages?
- Stimulus primed: channel, context, and assumed awareness state stated before running?
- Every persona completed all four WOOP moves (Want → Outcome → Obstacle → Plan)?
- Obstacle move surfaced a real, specific blocker — not a generic one?
- Cross-panel summary identifies consensus zones, fracture lines, highest-friction moment, and blind spots?
- Priority actions are concrete, ranked, and each points to a specific element of the stimulus?
- If VOC data or an objection library exists, were synthetic responses calibrated against it?
- High-stakes findings flagged for real-customer validation?
