---
name: automation-workflow-designer-debugger
description: >
  Designs and debugs multi-step marketing automations across any tool stack — Zapier, Make
  (Integromat), n8n, HubSpot Workflows, ActiveCampaign, Klaviyo Flows, Brevo, Customer.io,
  Ortto, Iterable, or plain webhook chains. Two modes: Designer takes a trigger + desired
  outcome + tool stack and produces a fully documented multi-step automation spec (field
  mappings, filter logic, delay timing, error branches) plus an importable JSON scaffold
  where the target platform supports it; Debugger takes a broken Zap, scenario, or flow
  (error message, screenshot, or described symptom) and returns a root-cause diagnosis with
  a step-by-step fix. Built around the TDFEM framework (Trigger → Data → Filter → Execute →
  Monitor). Brand voice and naming conventions come from the brand-brain skill. Calls
  esp-map-platform-builder for platform capability decisions and lifecycle-journey-mapper
  to cross-check timing against the customer journey. Use when the user says "build me an
  automation," "design a workflow," "my Zap/scenario keeps failing," "help me map this
  trigger to outcome," "write the logic for this flow," "debug my automation," "my webhook
  isn't firing," "translate this into a Make scenario," "automation spec," or hands over
  a broken automation and asks what went wrong.
---

# Automation Workflow Designer & Debugger

Give it a trigger and an outcome, get a working automation spec. Give it a broken flow, get a root-cause diagnosis with a fix. Every spec ships with field mappings, filter logic, delay rationale, error-branch handling, and — where the platform allows it — an importable JSON scaffold.

This skill designs and debugs. It does not write the email copy inside the flow (call `abandon-flow-writer`, `lead-nurture-drip-builder`, or `push-notification-copy-generator`), define the lifecycle model (call `lifecycle-journey-mapper`), or map platform capability (call `esp-map-platform-builder`). It orchestrates those outputs into a working automation architecture.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, naming conventions, and ICP so segments and flow names are on-brand.
- **`lifecycle-journey-mapper`** (when the automation spans lifecycle stages) — confirms timing and stage logic before committing delay values.
- **`esp-map-platform-builder`** (when the platform is ambiguous or the user is choosing) — determines which automations the target platform can natively support.
- **`abandon-flow-writer`** (when the automation outcome is an abandonment recovery sequence) — writes the message copy that slots into the spec's action nodes.
- **`lead-nurture-drip-builder`** (when the outcome is a nurture sequence) — writes copy for drip nodes.
- **`push-notification-copy-generator`** (when the flow includes push notifications) — writes the notification payload.
- **`data-qa-measurement-gotcha-checker`** (Debugger mode) — checks event/data quality issues that may be the root cause.
- **`usage-triggered-message-sequencer`** (when the trigger is a product event) — supplies the event-to-message mapping this skill wraps in flow logic.

---

## How a run works

```
Step 0  Load brand context          ──► brand-brain (naming, voice, ICP)
Step 1  Detect mode                 ──► Designer | Debugger
Step 2  Gather or receive inputs    ──► ask only what's missing
Step 3  Apply the TDFEM framework
Step 4  Produce the deliverable     ──► Spec doc + JSON scaffold | Root-cause report
Step 5  Self-review checklist       ──► validate before presenting
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, naming conventions, banned words, and ICP. Use these to name flows, segments, and tags consistently. Hold output until it returns.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none exists, ask four questions: brand name, ICP, preferred naming style (snake_case / Title Case / kebab-case), and any banned terms. Prefer the call.

### Step 1 — Detect mode

- **Designer** — user describes a trigger + outcome (or a gap in their ops stack they want to close).
- **Debugger** — user describes a broken flow: error message, unexpected behavior, failed step, or "it's not triggering."

When unsure, ask one question: "Are you building something new or debugging something broken?" Do not assume.

---

## Designer mode

### Minimum viable inputs

| Input | If missing |
|---|---|
| Trigger (event, webhook, form fill, schedule, tag applied, etc.) | Ask |
| Desired outcome (message sent, CRM field updated, Slack alert, deal created, etc.) | Ask |
| Tool stack (source system + automation platform + destination) | Ask; or call `esp-map-platform-builder` if undecided |
| Audience / segment scope | Infer from brand-brain ICP, confirm if ambiguous |

Once you have these, do not ask for more — design with what you have and flag assumptions.

### The TDFEM framework (the architecture spine)

Every automation spec follows five layers. Each layer is a section in the output document.

**T — Trigger**
Define the exact event that starts the flow: platform, event name or webhook endpoint, required payload fields, deduplication rule (what prevents the same contact from re-entering), and cooldown period.

**D — Data**
Map every field the flow reads or writes: source field → destination field, data type, transformation (lowercase, concatenate, date-offset), and fallback value when the field is null or missing. This is where most "it's not working" bugs live — document it explicitly.

**F — Filter**
State each branch condition as an explicit if/then rule: the field being tested, the operator, the value, and what each outcome path does. Distinguish hard filters (stop the flow) from soft filters (route to a branch). Flag mutually exclusive conditions that can deadlock.

**E — Execute**
List every action node in sequence: step number, action type (send email, update field, add tag, create deal, fire webhook, wait, split), platform-specific action name, and any required payload or template reference. For message nodes, note the copy source (written inline, or calls a sibling skill). For delays, state the rationale (time-of-day delivery, cooldown, behavioral wait).

**M — Monitor**
Define what "working" looks like: the success metric, the failure metric, the error notification (who gets alerted and how), and how to verify in the platform's history/log. Include the re-enrollment rule if the flow is evergreen.

### Output format — Automation Spec

```
## Automation Spec: [Flow Name]
Brand: [slug] | Platform: [tool] | Mode: Designer
Last updated: [date] | Version: 1.0

### Overview
[One sentence: trigger → outcome. Who it serves, why it exists.]

### Trigger
- Platform + event/trigger name:
- Required payload fields:
- Deduplication rule:
- Cooldown:
- Enrollment filter (who qualifies):

### Data map
| Source field | Destination field | Type | Transform | Fallback |

### Filter logic
| Step | Condition | Pass path | Fail path |

### Execution sequence
| # | Action type | Platform name | Details / template ref | Delay / timing |

### Error handling
- On hard failure: [retry count, alert destination]
- On soft failure (null field, unsubscribe): [branch]

### Monitor
- Success metric + threshold:
- Failure metric + threshold:
- Alert: [channel, owner]
- Verification steps (how to confirm it fired):

### Assumptions & open questions
[List anything inferred; flag decisions the user should confirm before activating]
```

### JSON scaffold

After the spec, if the platform supports export/import (Make, n8n, Zapier Transfer, HubSpot export), produce a minimal importable JSON with:
- Correct module/node types for that platform
- Field mapping stubs (keys present, values as `"{{placeholder}}"`)
- Filter conditions as configured objects, not strings
- A `_meta` block with spec version, brand slug, and date

Label it clearly as a scaffold — the user must configure credentials and test before activating. If the platform does not support import (e.g., Klaviyo Flows, ActiveCampaign), produce a step-by-step setup checklist instead.

---

## Debugger mode

### Triage order

Work down this checklist before diagnosing. Most failures are in the top three.

1. **Data quality** — is the trigger event actually firing with the expected payload? Check source logs first. If unclear, flag and call `data-qa-measurement-gotcha-checker`.
2. **Filter deadlock** — does a filter condition catch *all* contacts before they reach the broken step? Check for a never-true condition, a typo in a field name, or an AND chain where one clause is always false.
3. **Field mapping null** — is a required field empty or mis-typed, causing a downstream step to fail silently or error?
4. **Rate limit / throttle** — is the platform silently dropping events at volume (Zapier task limits, Make operation caps, API rate limits on the destination)?
5. **Credential / scope expiry** — did an OAuth token or API key expire? Platform connection test first.
6. **Timing race condition** — is the trigger firing before a dependent record is written (e.g., webhook fires before CRM contact is created)?
7. **Platform-specific gotchas** — see the gotcha table below.

### Platform gotchas (high-signal, verified)

| Platform | Common failure |
|---|---|
| Zapier | Zap pauses after 3 consecutive errors — check Task History, not just the Zap editor |
| Make | Incomplete bundles (missing optional fields) silently stop a module — enable "Continue even if Make returns no items" for optional lookups |
| n8n | Expression syntax errors evaluate to `undefined` at runtime but show no editor error — test with the Expression editor's live-data mode |
| HubSpot Workflows | Re-enrollment must be explicitly enabled per trigger; contacts already in the workflow skip it silently |
| Klaviyo | Flow analytics lag up to 24 hours — use "Preview & Test" send logs, not the analytics tab, for same-session debugging |
| ActiveCampaign | Automations with the same entry trigger can create duplicate entries if the contact matches the condition again — check "Allow multiple entries" settings |
| Customer.io | Server-side and client-side `identify` calls can create duplicate profiles if the `id` type differs (string vs. integer) |

### Debugger output format

```
## Debug Report: [Flow Name / Description]
Brand: [slug] | Platform: [tool] | Mode: Debugger
Reported symptom: [quote the user's description]

### Root cause
[One sentence verdict. Be specific: the field name, the step number, the condition.]

### Evidence
[What in the error message / log / structure points to this cause]

### Fix
[Step-by-step remediation. Platform-specific field names and menu paths where possible.]

### Verification
[How to confirm the fix worked, and what to watch in logs]

### Secondary issues (if any)
[Other fragility spotted during diagnosis — flag, don't fix without asking]
```

---

## Principles

- **TDFEM every time.** Every designed flow has all five layers documented, even if some are one-liners. An undocumented assumption is a future incident.
- **Data map before execution.** Null fields and type mismatches are the leading cause of silent failures. Always surface them.
- **Error branches are not optional.** Every flow has a defined behavior for the failure case. "It will probably work" is not a spec.
- **Brand-brain owns naming.** Flow names, tag conventions, and segment labels must match the brand's naming style — inconsistent naming breaks downstream reporting.
- **Import/export where possible.** A JSON scaffold the user can import beats a screenshot to re-click. Always produce one for platforms that support it.
- **Diagnose, then fix.** In Debugger mode, state the root cause before prescribing a fix. Do not prescribe a fix for a symptom while the real cause is upstream.
- **Flag assumptions explicitly.** Any design decision made without explicit user input is an assumption — list it and ask for confirmation before the user activates.

## What Not to Do

- Don't write message copy inside the spec — call the appropriate copy skill and reference the output.
- Don't design a flow before `brand-brain` returns — naming and segment logic depend on it.
- Don't diagnose in Debugger mode before checking the data layer — 80% of failures are data issues upstream of the step that visibly errored.
- Don't produce a JSON scaffold for platforms that don't support import — produce a setup checklist instead.
- Don't mark a flow spec "ready to activate" without completing the Monitor section — an unmonitored automation is a liability.
- Don't invent platform-specific field names or API parameters — use `[verify field name in platform]` if uncertain.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and naming conventions applied to flow name, tags, and segment labels?
- Designer: all five TDFEM layers present with explicit field mappings and filter conditions?
- Designer: error branches and a Monitor section included?
- Designer: JSON scaffold produced (or setup checklist where JSON is not supported)?
- Designer: assumptions list complete with at least one confirmation ask?
- Debugger: root cause stated before the fix; evidence cited from the error/log/structure?
- Debugger: fix is step-by-step with platform-specific paths, not generic advice?
- No invented platform field names — all uncertain names flagged `[verify]`?
- Copy nodes reference a sibling skill or a placeholder — no copy written inline without delegating?
- Output saved to `./ops/[brand-slug]-[flow-name]-spec.md` (Designer) or `./ops/[brand-slug]-[flow-name]-debug.md` (Debugger)?
