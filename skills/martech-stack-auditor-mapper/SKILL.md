---
name: martech-stack-auditor-mapper
description: >
  Takes a current tools list (with owners, costs, and integrations) and returns a complete
  martech stack audit: a gap/overlap/redundancy report, a consolidation recommendation with
  estimated cost savings, a visual dependency map of data flows between tools, and a
  categorized stack health scorecard. Uses the Stackie.io taxonomy (6 categories, 27 sub-
  categories) as the organizing framework so every tool gets a canonical home and every
  gap is named, not vague. Calls brand-brain to load the brand's ICP and growth stage before
  assigning priorities, so recommendations match where the team actually is. Saves a reusable
  audit artifact for diff-based refresh audits. Use when the user says "audit my martech,"
  "I'm paying for too many tools," "what should we cut/consolidate," "map our data flows,"
  "we have overlap between X and Y," "tool sprawl," "stack review," "martech rationalization,"
  or hands over a tools list and asks what's missing or redundant.
---

# Martech Stack Auditor & Mapper

Hand over a tools list; get back a structured audit. Every tool placed in the Stackie taxonomy, every overlap named, every gap assessed against the brand's actual growth stage, and a dependency map that makes the data flows legible. Designed for a single operator making rationalization decisions — not a 40-page consulting deck.

This skill audits and maps. It does not run vendor negotiations, build GTM tags, or write campaign copy. If a specific tool setup is needed post-audit, it hands off to the right sibling skill.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, growth stage, channel mix, and team size. Stack priorities are meaningless without knowing whether the brand is a 3-person bootstrapped shop or a 50-person mid-market team. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for growth stage, primary channels, team size, and approximate annual martech budget before proceeding.
- **`tracking-plan-taxonomy-builder-auditor`** — called when the audit surfaces data-collection gaps or event-tracking inconsistencies; produces a governed event taxonomy and tracking plan.
- **`data-qa-measurement-gotcha-checker`** — called as a quality gate when the dependency map reveals data handoffs between tools; flags broken data flows, attribution blind spots, and double-counting risks.
- **`utm-parameter-bulk-builder`** — called when the audit flags inconsistent or missing UTM conventions across the stack.
- **`consent-privacy-compliance-auditor`** — called when any tool in the stack handles PII, consent, or pixel tracking; reviews for GDPR/CCPA exposure and flags dark patterns.
- **`gtm-tag-builder-server-side-conversion-setup`** — downstream: called when the audit recommends consolidating tracking through a tag manager.
- **`vendor-cost-comparison-renewal-brief`** — downstream: called when the user wants a side-by-side cost model for a consolidation option or a renewal decision.

---

## How a run works

```
Step 0  Load the brand (always first)
Step 1  Ingest & normalize the stack inventory
Step 2  Place each tool in the Stackie taxonomy
Step 3  Score: gaps, overlaps, redundancies, integration health
Step 4  Build the dependency map
Step 5  Produce the audit report + consolidation recs
Step 6  Save the artifact; offer downstream handoffs
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Pull growth stage, ICP, primary acquisition channels, team size, and any named tech-stack context from the returned digest. These determine the "right-size" tier for the audit: a seed-stage company running three tools needs different guidance than a Series B team with 40+. Do not score gaps until brand context is loaded.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for growth stage, primary channels, team size, and approximate annual martech budget before proceeding.

---

## Step 1 — Ingest & normalize the inventory

Accept the stack in any format: a bullet list, a spreadsheet paste, a CSV, or a plain description. For each tool, extract or ask for:

| Field | Required | Notes |
|---|---|---|
| Tool name | Yes | |
| Category (user-stated) | No | Override with Stackie placement |
| Primary owner | Yes | Marketing / Sales / Product / Ops / IT |
| Monthly/annual cost | Preferred | Mark `[verify]` if unknown |
| Integrations listed | Preferred | "connects to X, Y" |
| Active vs. shelfware | Yes | Ask if unclear |
| Contract end / next renewal | Preferred | |

If costs and renewals are unknown, proceed — but flag them as mandatory inputs before any consolidation decision is final.

---

## Step 2 — Stackie taxonomy placement

Map every tool to the [Stackie.io Marketing Technology Landscape](https://chiefmartec.com) taxonomy. Use the 2024 edition's six top-level categories:

1. **Advertising & Promotion** — paid media, display, social ads, ABM, affiliate
2. **Content & Experience** — CMS, SEO, landing pages, video, personalization, email/push
3. **Social & Relationships** — social listening, community, influencer, reviews
4. **Commerce & Sales** — e-commerce platform, POS, CPQ, sales enablement, chat
5. **Data** — CDP, data warehouse, BI, analytics, tag management, privacy/consent
6. **Management** — project management, collaboration, DAM, attribution, agile

Place each tool in its primary sub-category. If a tool spans two sub-categories (e.g., HubSpot = CRM + email automation), note the overlap — that is where redundancy analysis starts.

---

## Step 3 — Gap, overlap, and redundancy scoring

### Gap analysis
For each sub-category a brand at this growth stage needs, check coverage. Rate each sub-category:

- **Covered** — has an active, integrated tool
- **Covered (shelfware)** — tool exists but is unused or disconnected
- **Gap** — no tool; assess severity: Critical / High / Nice-to-have based on growth stage + channels
- **Not applicable** — the sub-category genuinely doesn't apply (e.g., no paid team = "paid search platform" is N/A)

### Overlap and redundancy analysis
Any two tools covering the same primary function are an overlap candidate. Classify:

| Type | Definition | Action signal |
|---|---|---|
| **Redundancy** | Both tools do the same job; one is disposable | Consolidate; one is waste |
| **Intentional overlap** | Different teams use different tools for the same function for valid reasons (e.g., Sales uses one CRM module, Marketing uses another) | Document; may still be worth rationalizing |
| **Capability overlap** | One tool has the feature but the team uses a separate tool because of habit/integration gaps | Candidate for consolidation pending integration check |

### Integration health
Score the stack's integration fabric: what percentage of tools are connected to the system of record (CRM or CDP)? Isolated tools that handle customer data but don't sync are the primary source of attribution failure and data quality risk. Flag all isolated tools as integration debt.

---

## Step 4 — Dependency map

Produce a text-format dependency map that shows data flows:

```
[Source / collection layer]
  └── Website analytics (GA4)  ──► [Data store]
  └── Forms (HubSpot)           ──► CRM (HubSpot) ──► Email platform ──► [Activation]
  └── Push SDK (PushEngage)     ──► (isolated — no CRM sync)  ← flag

[Data store]  CRM (HubSpot)
  └── Syncs to: Email, Ads (via Audience), Attribution
  └── Does NOT sync to: Push platform, Chat, BI tool  ← integration gaps
```

Format this as an annotated ASCII/Markdown diagram appropriate for pasting into Notion or a doc. If the user asks for a visual, note that `infographic-data-viz-spec-writer` can produce a Figma-ready spec from this structure.

Mark each integration as:
- **Live** — confirmed bidirectional or one-way sync
- **Manual** — export/import workflow; reliability risk
- **Missing** — data silo; flag for remediation priority

---

## Step 5 — Audit report and consolidation recommendations

### Stack Health Scorecard

| Dimension | Score (1–5) | Notes |
|---|---|---|
| Coverage vs. growth stage | | Sub-categories filled vs. needed |
| Redundancy / waste | | Cost and cognitive |
| Integration fabric | | % of tools connected to system of record |
| Data quality risk | | Isolated tools handling PII/events |
| Contract hygiene | | Renewals tracked; shelfware flagged |
| **Overall** | | |

### Consolidation recommendations

For each redundancy or shelfware tool:
1. **Name the overlap** (Tool A vs. Tool B; shared function)
2. **Recommend: Keep A, cut B / evaluate migration** with a one-line rationale
3. **Estimated annual saving** (use provided costs; mark `[verify]` if estimated)
4. **Migration effort**: Low / Medium / High (based on data volume + integration complexity)
5. **Suggested replacement** if a gap would open post-cut

### Gap recommendations

For each Critical or High gap:
1. **Name the gap** (sub-category, use case)
2. **Recommended tool category** (not a vendor prescription unless the user asks)
3. **Integration requirement** (must sync with: X, Y)
4. **Priority** anchored to growth stage: what breaks first without it?

---

## Step 6 — Save artifact and offer handoffs

Save the full audit to `./martech-audit/[brand-slug]-stack-audit-[YYYY-MM-DD].md`.

After delivering the audit, proactively offer the relevant downstream handoffs:

- Integration gaps flagged → call `tracking-plan-taxonomy-builder-auditor` and `data-qa-measurement-gotcha-checker`
- Inconsistent UTMs found → call `utm-parameter-bulk-builder`
- PII/consent-handling tools present → call `consent-privacy-compliance-auditor`
- User wants renewal cost model → call `vendor-cost-comparison-renewal-brief`
- User wants to consolidate to GTM-based tracking → call `gtm-tag-builder-server-side-conversion-setup`

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Growth stage and channel mix determine what's a gap vs. what's premature. Never score before loading brand context.
- **Stackie taxonomy as the organizing spine.** Every tool gets a canonical home; every gap has a name. Vague "you're missing a tool in this area" is not an audit.
- **Costs or `[verify]`.** Never invent pricing. Use the user's inputs; flag everything unconfirmed.
- **Distinguish redundancy from intentional overlap.** Cutting a tool that Sales intentionally uses separately from Marketing will break process; document before recommending.
- **Integration fabric is the real story.** Tools that don't talk to each other are data silos; that's where attribution breaks and CAC inflates. Surface this prominently.
- **Audit, don't redesign.** The deliverable is a decision-ready report, not a vendor pitch or a stack rebuild proposal. Scope creep here costs trust.
- **Compose, don't duplicate.** Data-quality gates go through `data-qa-measurement-gotcha-checker`; compliance reviews go through `consent-privacy-compliance-auditor`. Don't reimplement their logic here.

## What Not to Do

- Don't score gaps or produce recommendations before `brand-brain` returns the brand's growth stage and context.
- Don't invent costs or vendor pricing; mark all unconfirmed figures `[verify]`.
- Don't recommend cutting a tool without checking whether it's a data source for another tool in the stack.
- Don't collapse intentional overlap (different-team usage) into redundancy without flagging the process change required.
- Don't produce vendor-specific recommendations unless the user explicitly asks; recommend categories and capabilities first.
- Don't run a compliance review inline — call `consent-privacy-compliance-auditor` for anything touching PII or consent surfaces.
- Don't overwrite a previously saved audit file; append a new dated file and diff if the user wants a refresh.

## Quality Checklist (self-review before delivering)

- `brand-brain` called and growth stage / channel mix loaded (or fallback invoked) before any scoring?
- Every tool placed in a Stackie taxonomy sub-category?
- Every sub-category needed for this growth stage assessed (Covered / Gap / N/A)?
- Overlap classified as Redundancy / Intentional / Capability — not just "these two tools are similar"?
- Integration fabric scored; isolated tools (especially those handling events or PII) flagged explicitly?
- Dependency map shows live / manual / missing integrations?
- Consolidation recs include estimated saving (or `[verify]`), migration effort, and what gap opens on cut?
- Artifact saved to `./martech-audit/[brand-slug]-stack-audit-[YYYY-MM-DD].md`?
- Downstream handoffs offered where relevant (tracking-plan, data-qa, consent, UTM, GTM, vendor-cost)?
