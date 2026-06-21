---
name: tracking-plan-taxonomy-builder-auditor
description: >
  Turns a product spec, PRD, GA4 event export, or raw "we fire these events" brain-dump into
  two things in one run: (1) a governed event taxonomy — a canonical registry of every event name,
  property, value set, trigger moment, and business purpose, in a format that survives team growth
  and tool churn; and (2) a live-data audit — comparing what the spec says should be firing against
  what is actually in GA4 (or the export you paste), then returning a prioritised remediation table
  with effort/impact scoring so fixes land in the right sprint. In audit-only mode (just paste an
  event export, no spec) it reverse-engineers a working taxonomy from live data and flags gaps.
  In build-only mode (spec only, no data) it produces the governed taxonomy and a validation
  checklist. Encodes the five GA4/measurement gotchas (attribution windows, sampling, SRM,
  (not set), self-referral) plus naming-convention anti-patterns (camelCase pollution, over-generic
  events, missing entity context). Calls `data-qa-measurement-gotcha-checker` as the data-quality
  gate before finalising. Produces a usable spec, not advice. Use when the user says "write a
  tracking plan," "audit our events," "our GA4 events are a mess," "build event taxonomy," "what
  events should we fire," "tracking plan template," "event naming convention," "clean up our
  analytics," "GA4 event schema," or hands over a product spec or event dump and asks what to do
  with it.
---

# Tracking Plan & Taxonomy Builder / Auditor

Most analytics problems are taxonomy problems. Events fired without a plan accumulate into an unmaintainable swamp — duplicate events with different names, missing properties that make segmentation impossible, no record of why anything was added. This skill builds the plan that prevents the swamp, or audits your existing data and shows you the path out.

Two modes, often run together: **Build** (spec → governed taxonomy) and **Audit** (live event export → gap report). Both produce something you can act on this sprint — not a slide deck about alignment.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand voice, ICP, and product context so event names and business-purpose fields map to real product language, not generic placeholders. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the product name, core user actions (top 5–8), and the primary analytics tool in use before proceeding.
- **`data-qa-measurement-gotcha-checker`** (required gate, Step 4) — runs the canonical GA4/measurement gotcha checks on the output taxonomy or audit before finalisation; never skip this gate even when the user is in a hurry.
- **`utm-campaign-naming-enforcer`** (optional compose) — when the tracking plan is being built alongside a campaign launch, call this to validate UTM conventions match the taxonomy's `source`/`medium`/`campaign` properties and avoid attribution collisions.
- **`analytics-report-reviewer`** (optional compose) — if the user also needs a narrative summary of the audit findings for a stakeholder deck, hand the remediation table to this skill rather than rebuilding the reviewer logic here.
- **`kpi-tree-builder`** (optional compose) — after the taxonomy is confirmed, call to map events to north-star and leading-indicator KPIs so the plan has a business rationale column, not just a technical spec.

---

## How a run works

```
Step 0   Load brand context  ──► brand-brain (bootstraps on first use)
Step 1   Detect mode         ──► Build | Audit | Both
Step 2   Ingest inputs       ──► spec / PRD / event export / brain-dump
Step 3   Build or audit      ──► taxonomy table + gap/remediation table
Step 4   Data-quality gate   ──► data-qa-measurement-gotcha-checker
Step 5   Deliver outputs     ──► save to ./tracking-plan/
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest including product context, ICP, and real feature/action vocabulary. Use product language from brand.md for event names and purpose descriptions — never use generic placeholder names when a real product term is known.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the product name, core user actions (top 5–8), and the primary analytics tool in use before proceeding.

---

## Step 1 — Detect the mode

| Input present | Mode |
|---|---|
| Spec / PRD / feature list only | **Build** — produce taxonomy, skip audit section |
| GA4 event export / raw event names only | **Audit** — reverse-engineer taxonomy from data, produce gap report |
| Both spec and export | **Full** — build taxonomy, run audit against it, show delta |
| Neither | Ask for one of the above before proceeding |

Ask one clarifying question only: which analytics tool is the target (GA4, Amplitude, Mixpanel, Segment, other)? Assume GA4 unless told otherwise.

---

## Step 2 — Ingest and normalise

**From a spec / PRD:** extract every user action, state transition, and conversion moment. Identify the entity involved (user, session, product, order), the triggering condition, and the business question it answers. Group into logical domains (acquisition, activation, engagement, monetisation, retention — AARRR is the explicit framework).

**From an event export:** deduplicate event names, count property coverage gaps (events with no properties or with only `page_location`/`page_title`), identify camelCase/snake_case/mixed-case pollution, flag events that look like duplicates (same semantic, different casing or `_v2` suffix), and note events that fire in production but appear nowhere in any spec.

---

## Step 3a — Build: the governed event taxonomy

The taxonomy table is the canonical registry. For every event:

```
| event_name | trigger_moment | entity | properties (key: type: required/optional) | value_sets (enum or free-text) | business_question_answered | AARRR_stage | owner | added | status |
```

**Naming convention (enforce strictly — GA4 snake_case standard):**
- `snake_case` only; no camelCase, no spaces, no dots
- Structure: `[entity]_[action]_[context]` where context is optional
  - Good: `checkout_started`, `feature_x_activated`, `plan_upgraded`
  - Bad: `buttonClick`, `event1`, `checkout` (too generic, misses action)
- Maximum 40 characters
- No PII in event names or property values; flag any properties that could carry PII for masking/exclusion
- Use a consistent `page_` prefix for all page-view variants; never mix `pageview`, `page_view`, `screen_view` for the same surface

**Required properties for every event (the minimum viable context set):**
- `user_id` (string, required when authenticated) — use the same field name across every event
- `session_id` (string, required) — prevent hit-level / session-level conflation
- `timestamp_utc` (string/epoch, required) — data-warehouse joins depend on this
- `source_event_version` (string, optional) — schema migration guard; default `"1.0"`

**The AARRR property rule:** every event must have at least one property that answers *which user*, *what action*, and *what outcome* — bare events with no properties are rejected at the quality gate.

---

## Step 3b — Audit: the gap and remediation table

Compare the live event export against the canonical taxonomy (or, in audit-only mode, against what a well-governed taxonomy for this product *should* contain). Output three tables:

**Table 1 — Coverage map**

```
| event_name | in_spec | in_live_data | verdict |
| checkout_started | ✓ | ✓ | OK |
| cart_item_added  | ✓ | ✗ | MISSING — not firing |
| buttonClick      | ✗ | ✓ | GHOST — in data, not in spec |
```

**Table 2 — Property-completeness audit**

For every event that fires in live data, check each required property. Flag: missing required property / wrong type (string sent as int) / null rate >20% (unreliable) / PII risk.

**Table 3 — Remediation priority**

Score each issue: **Impact** (how many downstream reports / segments / audiences break without this?) × **Effort** (dev-hours to fix: Low = 1, Med = 3, High = 8). Sort by Impact/Effort descending.

```
| issue | event(s) affected | impact | effort | priority | recommended_fix | sprint |
```

Sprint column: Now (this sprint) / Next / Backlog.

---

## Step 4 — Data-quality gate

Before presenting output, invoke the `data-qa-measurement-gotcha-checker` skill. Pass the taxonomy and/or audit table as context. It enforces the five canonical GA4 gotchas:

1. **Attribution-window mismatch** — are conversion events within GA4's default 30-day lookback or have you implicitly assumed a longer window? Flag cross-channel attribution gaps.
2. **Sampling threshold** — does the property volume exceed GA4's ~500K session unsampled threshold for standard reports? If the plan includes high-frequency events, note that Explorations may sample.
3. **SRM (Sample Ratio Mismatch)** — if any events feed an A/B test, flag the need for SRM checks before interpreting results.
4. **(not set) exposure** — any event missing `user_id` when the user is authenticated will inflate `(not set)` in user-scoped dimensions. Flag as a blocking issue if >5% of conversion events are in scope.
5. **Self-referral / cross-domain leakage** — if properties are collected from multiple domains or subdomains and `linker` / `cross_domain` is not configured, session attribution will break. Flag if the spec includes events from >1 domain.

If the gotcha-checker surfaces any P0 issues, surface them as blockers before delivering the final output. Do not bury them in the appendix.

---

## Step 5 — Deliver outputs

Save all outputs to `./tracking-plan/` relative to the user's CWD. Never overwrite `brand.md`.

```
./tracking-plan/
  taxonomy.md          ← canonical event registry (Step 3a table)
  audit-gap-report.md  ← coverage map + property audit + remediation table (Step 3b)
  qa-gate-notes.md     ← data-quality gotcha findings from Step 4
  CHANGELOG.md         ← append-only version log; create on first run
```

Always save before presenting the inline summary. Tell the user the path in one line.

The inline summary is: taxonomy row count, event domains covered, audit verdict counts (OK / MISSING / GHOST), top 3 remediation priorities, and any P0 blockers from the data-quality gate. Keep it to ≤10 lines.

---

## The framework: AARRR × entity-action-context taxonomy

This skill uses the **AARRR funnel** (Acquisition, Activation, Retention, Referral, Revenue — Dave McClure) as the business-purpose axis for every event, crossed with a three-part naming convention (**entity_action_context**) for the technical axis. The intersection of the two gives each event a location on the product map and a business owner — which is the minimum viable governance layer.

The property schema follows the **Segment Spec** pattern (identify, track, page, group calls with standardised property names) even for GA4 implementations, because it produces taxonomy that is portable across analytics tools without a full re-instrumentation.

---

## Principles

- **Taxonomy is a contract, not a list.** Every event has a named owner and an explicit business question it answers; events without owners drift.
- **Build mode and audit mode must agree.** If the live data contradicts the spec, the spec is wrong until proven otherwise — don't assume implementation is the only failure mode.
- **Properties before events.** A well-named event with no properties is nearly useless; an over-named event with rich properties can be queried into the shape you need.
- **Reject PII at the schema layer.** Flag any property that could carry email, name, or device ID for hash/exclusion before it reaches the data warehouse.
- **Never skip the data-quality gate.** Tracking plans that ignore GA4 gotchas produce dashboards that mislead rather than inform.
- **Persistence.** Save to `./tracking-plan/`; the CHANGELOG.md is an append-only log — never overwrite history.

---

## What not to do

- Don't produce a tracking plan that is just a list of event names — every event must have trigger moment, properties, and business question.
- Don't invent event names that don't map to real product actions visible in the spec or brand context.
- Don't mark issues `[verify]` to avoid making a call — if the data shows a ghost event, say so; only use `[verify]` for numbers you cannot confirm.
- Don't skip the data-quality gate even when the user is only asking for a quick audit.
- Don't store outputs inside the skill folder; always write to `./tracking-plan/` in the user's CWD.
- Don't conflate GA4 event-scoped and user-scoped dimensions — note which properties are session-level vs. user-level and which reports they can and cannot appear in.
- Don't suggest rebuilding `data-qa-measurement-gotcha-checker`'s logic inline; call the skill.

---

## Quality checklist (self-review before presenting)

- [ ] `brand-brain` called first; product vocabulary from brand.md used in event names and purpose column?
- [ ] Mode correctly detected (Build / Audit / Full); clarifying question asked only if analytics tool is ambiguous?
- [ ] Every taxonomy row has: event name in snake_case ≤40 chars, trigger moment, entity, required properties, business question, AARRR stage, owner?
- [ ] Audit tables present: coverage map, property-completeness, remediation priority (sorted by Impact/Effort)?
- [ ] `data-qa-measurement-gotcha-checker` invoked; all five gotchas checked; P0 blockers surfaced prominently?
- [ ] PII-risk properties flagged for masking/exclusion?
- [ ] Files saved to `./tracking-plan/` before inline summary presented?
- [ ] CHANGELOG.md entry appended (never overwritten)?
- [ ] Inline summary ≤10 lines covering row count, domains, audit verdict counts, top-3 remediations, P0 blockers?
