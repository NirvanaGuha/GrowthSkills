---
name: onboarding-enablement-knowledge-pack-builder
description: >
  Role description + SOPs + tool links + brain-dump notes → structured onboarding, enablement,
  or knowledge-transfer pack that gets a new hire, contractor, or successor productive in days
  rather than weeks. Takes raw inputs in any shape — job description, messy Google Docs, Notion
  exports, Slack thread dumps, tool stack lists, runbooks — and assembles them into a Role Brief,
  a Day-1/Week-1/Month-1 milestone map, an annotated tool access checklist, a task inventory with
  tribal-knowledge callouts, a "how we work" norms section, and an FAQ. Calls brand-brain for voice
  alignment and doc-note-summarizer + sop-builder-reviewer for input processing. Outputs a single
  portable Markdown pack saved to ./onboarding/ plus an optional Notion/Google Doc handoff via
  publishing-integration-hub. Use when the user says "onboard a new hire," "build an onboarding
  doc," "knowledge transfer," "contractor brief," "offboarding runbook," "train my replacement,"
  "document the role," "ramp checklist," "enablement pack," or "what does this role do."
---

# Onboarding & Enablement Knowledge Pack Builder

Raw inputs → a structured pack a new person can open on Day 1 and actually follow. Not a wiki dump. Not a list of links. A real ramp document with sequenced milestones, annotated tools, task ownership, and the tribal knowledge that would have taken three months of Slack archaeology to discover.

The output is opinionated: it follows **CORE** (Context → Orientation → Ramp → Execute), our house working model — four phases that mirror how a competent person actually gets productive, not how most companies think they do.

---

## Skills this calls

- **`brand-brain`** (required first) — loads voice, tone, and terminology so the pack reads like the company wrote it, not like a consulting deliverable. Also checks for banned words and style conventions.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the brand's name, company voice adjectives (3), key banned words/phrases, and primary tool stack].
- **`doc-note-summarizer`** — processes raw brain-dump inputs (Google Docs, Notion exports, Slack dumps, long PDFs) into structured facts before assembly. Call once per major input blob; skip if inputs are already structured.
- **`sop-builder-reviewer`** — converts any procedural inputs into numbered, review-ready SOPs that slot into the pack's task-inventory section. Call when the user provides runbooks or process notes.
- **`product-feature-knowledge-base-curator`** *(optional)* — if the role is product-adjacent, call to pull a clean feature/tool glossary into the pack's tool section.
- **`loom-async-video-script-writer`** *(optional)* — if the user wants a manager-recorded Loom walkthrough script alongside the written pack, call to produce it.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (voice + terminology)
Step 1  Intake + triage      ──► classify inputs; call doc-note-summarizer / sop-builder-reviewer as needed
Step 2  Extract              ──► pull facts into the CORE framework slots
Step 3  Draft the pack       ──► assemble all six sections
Step 4  Flag gaps            ──► surface every [MISSING] item before saving
Step 5  Save + optionally publish
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns: voice adjectives, banned words, company/product terminology, ICP framing (useful for understanding the role's external audience). Obey the returned voice throughout — every section header, every task description, and every FAQ entry must match the brand's register.

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the brand's name, company voice adjectives (3), key banned words/phrases, and primary tool stack].

---

## Step 1 — Intake and triage

Accept inputs in any shape. Classify each:

| Input type | Route |
|---|---|
| Long unstructured doc / Notion export / Slack dump | → `doc-note-summarizer` |
| Runbook / step-by-step process notes | → `sop-builder-reviewer` |
| Already-structured doc (table, outline, checklist) | → extract directly |
| Oral / conversational notes | → ask 3 clarifying questions (below) before proceeding |

**If inputs are sparse or missing, ask these before proceeding** (batch into one message):
1. What is the role, and what does success look like at 30/60/90 days?
2. What tools does this person need access to, and who grants each?
3. What are the 3–5 recurring tasks that make up 80% of the job?

Never invent tribal knowledge — if a critical piece is missing, mark it `[MISSING — add before sharing]`.

---

## Step 2 — Extract into CORE slots

Map every fact from all inputs into the four CORE phases:

**C — Context.** Why this role exists. What the team/org does. How success for this person maps to the team's north-star metric. Key stakeholders with one-line descriptions. Internal vocabulary / terminology specific to the company or product.

**O — Orientation.** Day-1 logistics: accounts, accesses, equipment, meetings. The "first-week survival kit": who to Slack for what, where to find X, meeting norms, how decisions get made.

**R — Ramp.** Week-by-week task progression leading to full ownership. Milestone gates (what "good" looks like at each). Annotated task inventory — every recurring task with: frequency, estimated time, upstream inputs, downstream consumers, and one tribal-knowledge callout (the thing not in the official doc).

**E — Execute.** The steady-state operating rhythm. Quarterly projects or OKR contributions this role typically owns. How to escalate, how to flag blockers, when to decide vs. ask. Links to SOPs, dashboards, and reference docs.

---

## Step 3 — Draft the six sections

### 1. Role Brief (1 page)
Role title · team · reports to · mission sentence · three 30/60/90-day success criteria · north-star metric this role moves · stakeholders (name · role · what you need from/for them).

### 2. Day-1 / Week-1 / Month-1 Milestone Map
Structured as a sequenced checklist per phase. Each item has: action verb · who owns it · due · done-when definition. No orphan to-dos. Flag any item that requires manager action to unblock.

### 3. Tool Access Checklist
Table: Tool · Purpose · Access type (owner-granted / self-serve / SSO) · Who to ask · Link · Notes (e.g., "free tier is fine for contractors, paid for FTEs"). Include only tools confirmed in the inputs; mark any suspected but unconfirmed tool `[verify]`.

### 4. Recurring Task Inventory
For each task: name · frequency · estimated time · inputs required · output / where it goes · tribal-knowledge callout (the undocumented truth). Flag tasks with only one person who knows how to run them — these are bus-factor risks.

### 5. "How We Work" Norms
Communication defaults (async vs. sync · meeting norms · Slack vs. email rules), decision-making protocol (who decides what without asking), feedback loop (how the manager gives it, how to request it), and 3–5 cultural norms the org lives by but rarely writes down.

### 6. FAQ
10–15 questions a smart new hire would ask in week 1 that are not answered by the formal docs. Derive from the inputs. When the answer isn't in the inputs, write `[MISSING — answer before sharing]`.

---

## The CORE framework (opinionated rationale)

Most onboarding docs fail at the same place: they front-load logistics (O) and skip Context entirely, producing a person who knows where the Zoom link is but not why the job exists. The CORE sequence fixes this:

- **C before O** — understanding why the role exists changes how a person interprets every procedure they learn after.
- **R as a skill curve, not a task list** — Ramp is sequenced by cognitive load, not by calendar. Early tasks build pattern recognition for later ones; tribal-knowledge callouts surface what the official docs don't say.
- **E as a reference, not reading material** — Execute is the steady-state operating manual they'll return to in month 3, not the thing they read on day 1.

The six-section output maps exactly onto CORE: Role Brief (C), Milestone Map (O first-week), Tool Access (O logistics), Task Inventory (R), Norms (O culture + E operating rhythm), FAQ (R gaps).

---

## Saving and publishing

Default: save to `./onboarding/<role-slug>-onboarding-pack.md` in the user's CWD.

If the user wants a Notion or Google Docs version, invoke `publishing-integration-hub` with the assembled Markdown. Never write to `brand.md` or any brand-brain file.

---

## Principles

- **Brand-brain first.** No pack before voice and terminology are loaded; the company should sound like itself in its own onboarding docs.
- **Tribal knowledge is the product.** The six sections are scaffolding. The real value is the callouts that weren't in any official doc.
- **Bus-factor visibility.** If only one person knows how a task works, say so explicitly — that's a risk the manager needs to see.
- **No invented facts.** Real information from real inputs, or `[MISSING]`. Hallucinated SOPs are worse than blank ones.
- **Sequence for cognitive load.** The Ramp section runs easy-to-hard, not alphabetical or by-system. A new person's brain is the constraint.
- **Portable by default.** Output is a single Markdown file that can live in Notion, a Google Doc, a git repo, or a shared drive without breaking.

## What Not to Do

- Don't produce the pack before brand-brain returns — even a single section.
- Don't reimplement brand scanning or voice derivation here; call brand-brain.
- Don't write more than one primary output file per run — the pack is one document.
- Don't fill in `[MISSING]` items with plausible-sounding guesses; surface them as gaps.
- Don't treat "link to the Confluence page" as a task — if the procedure matters, summarize it here; don't offshore it to a dead link.
- Don't use onboarding jargon ("synergize," "hit the ground running," "take ownership") unless the brand explicitly uses it.

## Quality Checklist

- `brand-brain` called and voice + terminology loaded before drafting?
- All six sections present: Role Brief, Milestone Map, Tool Access, Task Inventory, Norms, FAQ?
- Every task in the inventory has: frequency, time estimate, inputs, output, and a tribal-knowledge callout?
- Every tool entry confirmed from inputs (not assumed) — unconfirmed ones marked `[verify]`?
- Bus-factor risks flagged in the Task Inventory?
- All gaps marked `[MISSING — add before sharing]`, not filled with guesses?
- Pack saved to `./onboarding/<role-slug>-onboarding-pack.md`?
- Milestone Map items each have: action verb, owner, due, done-when definition?
