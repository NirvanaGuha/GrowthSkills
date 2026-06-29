---
name: sop-builder-reviewer
description: >
  Turns raw process knowledge into a polished, actionable Standard Operating Procedure — and
  reviews any existing SOP for gaps, ambiguity, missing owners, and failure modes. Input can
  be anything: a messy Slack thread, a Loom transcript, a working prompt, a brain dump, bullet
  notes, a voice memo, a copy-pasted runbook, or an existing SOP draft. Output is a human-
  readable SOP in a consistent house format, plus a structured Gap & Ambiguity Review that flags
  every missing step, orphaned decision, uncleaned edge case, and un-owned task. Works for any
  repeatable growth or content-marketing process: content production, campaign launch, weekly
  reporting, onboarding, partnership outreach, or ad-hoc ops workflows. Applies the SIPOC
  frame: Suppliers → Inputs → Process → Outputs → Customers, overlaid with a severity-sorted
  gap & failure-mode audit to surface what breaks and who is accountable. Saves reusable SOPs to ./ops/ by
  default so the team can version and link them. Use when the user says "write an SOP," "turn
  this into a process doc," "document how we do X," "review this runbook," "what's missing from
  this process," "standardize this workflow," "who owns what step," or pastes a messy thread
  and asks to clean it up.
---

# SOP Builder & Reviewer

Raw process knowledge is everywhere — in someone's head, a Slack thread, a recording, or a
half-finished doc. This skill extracts it, structures it, and stress-tests it. Hand over
anything messy; get back a clean, owner-assigned, edge-case-aware SOP and a gap review that
tells you exactly what still needs answering before this process runs reliably.

Two modes: **Build** (raw input → new SOP) and **Review** (existing SOP → gap/ambiguity audit).
On most inputs both run in sequence — build first, then immediately self-audit the output.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice so any SOP written for
  external-facing use or shareable deliverables honors tone and banned words. Light-touch for
  internal process docs; still load it.
- **`meeting-prep-follow-up-pack`** — when input is a meeting transcript or post-meeting notes,
  call this first to extract decisions and action items before structuring the SOP.
- **`pre-mortem-post-mortem-generator`** — for mature SOPs, call after building to layer in a
  structured failure-mode analysis (the gap audit in Step 4 uses its output).
- **`prioritization-framework-suite`** — when multiple SOP candidates come in at once and the
  user needs to decide which to document first.
- **`campaign-brief-builder`** — when the SOP being built is for a campaign launch workflow;
  call to fill the campaign spec section rather than reinventing it inline.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (voice + banned words for any shareable output)
Step 1  Classify the input   ──► raw source (Build) | existing SOP (Review) | both
Step 2  Extract & structure  ──► SIPOC scaffold → step-by-step SOP draft
Step 3  Gap & ambiguity audit ──► severity-sorted gap/ambiguity/owner audit on the draft
Step 4  Resolve or flag      ──► ask for gaps it cannot infer; mark unresolvable ones [clarify]
Step 5  Write & save         ──► final SOP + Gap Review to ./ops/<slug>-sop.md
```

---

## Step 0 — Brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned voice
adjectives and banned words when writing any SOP section that will be read by external
parties or published (e.g., partner onboarding, customer-facing runbooks). For purely
internal process docs, honor voice at a light touch — don't force brand messaging into
step titles, but do avoid language the brand bans.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's
`brand.md` directly. If none exists, proceed with a neutral professional register and note
at the top of the SOP: `[brand-brain not loaded — verify tone before distributing externally]`.

---

## Step 1 — Classify the input

| Input type | Mode |
|---|---|
| Slack thread, brain dump, Loom transcript, bullet notes, voice memo | **Build** → extract then structure |
| Existing SOP, runbook, process doc, checklist | **Review** → audit then recommend fixes |
| Partial draft or rough outline | **Build + Review** → complete, then audit |

When the input is ambiguous, default to Build + Review and say so in one line.

---

## Step 2 — SIPOC scaffold (the extraction pass)

Before writing a single step, map the process to the SIPOC frame. This forces completeness
upstream and exposes the owner question early.

| SIPOC element | What to capture |
|---|---|
| **Suppliers** | Who or what provides the inputs to this process? (person, tool, upstream process) |
| **Inputs** | What must exist / be ready before Step 1 can begin? (data, assets, approvals, credentials) |
| **Process** | The ordered steps — numbered, imperative, one action each |
| **Outputs** | What does a completed run produce? (artifact, state change, notification, published asset) |
| **Customers** | Who receives or depends on the output? (internal team, external partner, end user) |

Extract from the raw input what can be inferred. Mark anything unresolvable as `[clarify]`.
Do not invent owners, SLAs, or systems — use `[clarify: who owns this?]` or `[clarify: which
tool?]` inline.

---

## Step 3 — SOP format (the writing pass)

Write every SOP in this structure. Adjust depth to complexity: simple ops = shorter; multi-
team or high-stakes = full.

```markdown
# SOP: [Process Name]

**Version:** 1.0  |  **Owner:** [name or role]  |  **Last updated:** [date]
**Trigger:** [What event or schedule starts this process?]
**Scope:** [What this covers and explicitly what it does not cover]
**Frequency:** [one-time | per-campaign | weekly | on-trigger | ad-hoc]

---

## Inputs required before starting
- [ ] [Input 1 — source/supplier]
- [ ] [Input 2 — source/supplier]

## Steps

### 1. [Verb-first step title]
**Owner:** [role]  |  **Tool:** [name]  |  **SLA:** [time or condition]

[One clear sentence of what to do. If there is a decision branch, show it explicitly.]

> Decision: If [condition A] → go to Step X. If [condition B] → go to Step Y.

### 2. …

## Output / Done state
[What does "complete" look like? What artifact exists, what system is updated, who is notified?]

## Edge cases & escalation
| Situation | Response | Escalate to |
|---|---|---|
| [common failure or exception] | [what to do] | [person/role] |

## Related docs / links
- [Link to upstream or downstream processes]
```

Every step is:
- **Verb-first, imperative.** "Export the CSV" not "The CSV needs to be exported."
- **Single action.** Split compound steps. Never "Do X and Y and check Z."
- **Owner-assigned.** If a step has no clear owner, write `[clarify: owner]`.
- **Tool-named.** If a tool is used, name it. Not "the platform" — "HubSpot / GA4 / Notion."

---

## Step 4 — Gap & ambiguity review

After building (or when reviewing an existing SOP), run a structured audit before presenting.
This is the review half of the skill — apply it to your own output, not just to inputs. The
audit borrows the failure-mode mindset of FMEA but ranks gaps on a single severity axis (not
FMEA's full Severity × Occurrence × Detection score).

**The seven gap categories:**

| Category | What to check |
|---|---|
| **Missing steps** | Is there a logical gap between two adjacent steps? |
| **Orphaned decisions** | Decision branches with no defined path for one outcome |
| **Unowned steps** | Steps with no assigned role or a vague owner ("the team") |
| **Missing inputs** | A step requires something that no prior step produces or sources |
| **Undefined done-state** | No clear signal that a step is complete and it is safe to proceed |
| **Untested edge cases** | Common failure modes (tool down, missing data, late approval) not addressed |
| **Stale references** | Tool names, URLs, or role titles that may change and create confusion |

Output the Gap Review as a table immediately after the SOP, sorted by severity (High /
Medium / Low). Each row: gap type | step # | description | recommended fix or `[clarify]`.

If any High-severity gaps exist, surface them before finalizing and ask the user to resolve
them. Do not silently produce a broken SOP.

---

## Step 5 — Save the output

Save to `./ops/<process-slug>-sop.md` by default (the user's working directory, not the skill
folder). Confirm the path in one line. If the user specifies a different location, use it.
On updates, bump the version number and `last updated` date; do not silently overwrite.

For long or multi-team SOPs, offer to split the Gap Review into a separate
`./ops/<process-slug>-gap-review.md` so the SOP stays clean for daily use.

---

## Principles (Non-Negotiable)

- **SIPOC before steps.** Map suppliers, inputs, outputs, and customers first — never write
  steps into a vacuum; missing boundaries are the most common SOP failure.
- **One action per step.** Compound steps are the second most common failure. Split them.
- **Owners, not "the team."** Every step must have a named role. "The team" is not an owner.
- **Clarify, don't invent.** Missing information gets `[clarify]`, not a plausible guess.
  A SOP with a wrong step is more dangerous than one with a flagged gap.
- **The gap audit is non-negotiable.** Every SOP the skill produces gets a gap audit — even the
  ones that look simple. Edge cases live in the "simple" ones.
- **Brand-brain first for anything external.** Light-touch is fine for internal docs; hard
  override for anything that leaves the team.

---

## What Not to Do

- Do not write steps without first completing the SIPOC scaffold — it produces structurally
  incomplete SOPs.
- Do not assign owners by guessing — use `[clarify: owner]` and surface it in the gap review.
- Do not suppress the gap review because the SOP "looks complete" — always run it.
- Do not silently overwrite an existing `./ops/<slug>-sop.md` — bump the version.
- Do not reimplement brand resolution — call `brand-brain`.
- Do not use passive voice in steps. "The file is exported" → "Export the file."
- Do not produce a SOP with High-severity gaps without surfacing them to the user first.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and voice/banned-words loaded (fallback noted if absent)?
- SIPOC scaffold completed before steps were written?
- Every step is verb-first, single-action, owner-assigned, and tool-named (or `[clarify]`)?
- Done-state defined for the overall process and for any branching steps?
- Edge cases and escalation paths present?
- Gap Review run; High-severity gaps surfaced to user before finalizing?
- Output saved to `./ops/<process-slug>-sop.md` with version and date?
- No invented owners, invented tools, or invented SLAs — real or `[clarify]`?
