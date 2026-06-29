---
name: martech-stack-auditor-mapper
description: >
  Takes a current tools list (with owners, costs, and integrations) and returns a complete
  martech stack audit: a gap/overlap/redundancy report, a consolidation recommendation with
  estimated cost savings, a visual dependency map of data flows between tools, and a
  categorized stack health scorecard. Uses Scott Brinker's Marketing Technology Landscape
  (6 top-level categories) as the organizing framework so every tool gets a canonical home and every
  gap is named, not vague. Calls brand-brain to load the brand's ICP and growth stage before
  assigning priorities, so recommendations match where the team actually is. Saves a reusable
  audit artifact for diff-based refresh audits. Use when the user says "audit my martech,"
  "I'm paying for too many tools," "what should we cut/consolidate," "map our data flows,"
  "we have overlap between X and Y," "tool sprawl," "stack review," "martech rationalization,"
  or hands over a tools list and asks what's missing or redundant.
---

# Martech Stack Auditor & Mapper

Hand over a tools list; get back a structured audit. Every tool placed in the Marketing Technology Landscape categories, every overlap named, every gap assessed against the brand's actual growth stage, and a dependency map that makes the data flows legible. Designed for a single operator making rationalization decisions — not a 40-page consulting deck.

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
Step 2  Place each tool in the Marketing Technology Landscape categories
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
| Category (user-stated) | No | Override with Landscape placement |
| Primary owner | Yes | Marketing / Sales / Product / Ops / IT |
| Monthly/annual cost | Preferred | Mark `[verify]` if unknown |
| Integrations listed | Preferred | "connects to X, Y" |
| Active vs. shelfware | Yes | Ask if unclear |
| Contract end / next renewal | Preferred | |

If costs and renewals are unknown, proceed — but flag them as mandatory inputs before any consolidation decision is final.

---

## Step 2 — Marketing Technology Landscape placement

Map every tool to Scott Brinker's [Marketing Technology Landscape — chiefmartec](https://chiefmartec.com/marketing-technology-landscape/), which organizes martech into six top-level categories, each with many sub-categories. Use the six top-level categories as the spine:

1. **Advertising & Promotion** — paid media, display, social ads, ABM, affiliate
2. **Content & Experience** — CMS, SEO, landing pages, video, personalization, email/push
3. **Social & Relationships** — social listening, community, influencer, reviews
4. **Commerce & Sales** — e-commerce platform, POS, CPQ, sales enablement, chat
5. **Data** — CDP, data warehouse, BI, analytics, tag management, privacy/consent
6. **Management** — project management, collaboration, DAM, attribution, agile

> Source: six top-level categories per Brinker's Marketing Technology Landscape Supergraphic (chiefmartec). The Supergraphic's sub-category lists shift edition to edition, so place tools by *function* rather than against a fixed sub-category count.

### Placement tie-breaker (when a tool spans categories)

Most real tools touch more than one category (HubSpot = CRM + email + CMS; Segment = CDP + tag management). Forcing one home is how audits go vague. Resolve placement in this order, and record both the **primary** and any **secondary** homes:

1. **Spend-weight** — file the tool under the category it's *paid for*. You buy HubSpot for CRM + automation, not its blog CMS; primary = Content & Experience (automation), CRM noted as secondary.
2. **System-of-record test** — if the tool is the source of truth for a data object (contacts, events, orders), its primary home is **Data** regardless of its marketing-facing features.
3. **Owner-of-record** — break remaining ties by which team's budget line it sits on (from Step 1's owner field).

The **secondary home is not noise — it is the overlap seed.** Every secondary placement is a candidate row for Step 3's overlap analysis: two tools sharing a category (primary or secondary) are where redundancy hides.

---

## Step 3 — Gap, overlap, and redundancy scoring

### Gap analysis
For each sub-category a brand at this growth stage needs, check coverage. Rate each sub-category:

- **Covered** — has an active, integrated tool
- **Covered (shelfware)** — tool exists but is unused or disconnected
- **Gap** — no tool; assign severity from the table below
- **Not applicable** — the sub-category genuinely doesn't apply (e.g., no paid team = "paid search platform" is N/A)

**Gap-severity decision table.** Severity is not a vibe; it is the answer to "does revenue or measurement break without this, *at this growth stage*?" Read across:

| Severity | Test it must pass | Examples by stage |
|---|---|---|
| **Critical** | A live acquisition or retention channel runs *blind or manual* without it; or you cannot measure a channel you actively spend on | No analytics with live paid spend; no CRM once Sales > 2 people; no consent tool while running EU pixels |
| **High** | A primary channel works but is materially handicapped — capped scale, heavy manual labor, or attribution holes | No CDP at Series B with 5+ data sources; no marketing automation past ~5k contacts; no attribution while running 3+ paid channels |
| **Nice-to-have** | Improves efficiency or polish but no channel breaks and nothing goes unmeasured | DAM at a 4-person team; dedicated SEO suite when GSC covers current volume |

Two rules keep this honest: **(1)** a gap is only Critical/High for a channel the brand *actually runs* — no paid team means "paid search platform" is N/A, not a gap; **(2)** a sub-category that the system-of-record already covers natively is **Covered**, not a gap (don't flag "no standalone email tool" when HubSpot sends the email).

### Overlap and redundancy analysis
Any two tools covering the same primary function are an overlap candidate. Classify:

| Type | Definition | Action signal |
|---|---|---|
| **Redundancy** | Both tools do the same job; one is disposable | Consolidate; one is waste |
| **Intentional overlap** | Different teams use different tools for the same function for valid reasons (e.g., Sales uses one CRM module, Marketing uses another) | Document; may still be worth rationalizing |
| **Capability overlap** | One tool has the feature but the team uses a separate tool because of habit/integration gaps | Candidate for consolidation pending integration check |

### Integration health — the Integration Fabric Score (house model)

The integration fabric is the real story of a stack, so score it with a repeatable rule instead of a gut percentage. This is a **house model** (our coinage, not a published standard); state it as such if asked.

First, identify the **system of record (SoR)** — the CRM or CDP every tool should orbit. Then weight each tool by what it carries, because not all silos cost the same:

| Tool carries... | Weight |
|---|---|
| Customer/event/PII data (must reach the SoR) | 3 |
| Campaign or content data (nice to sync) | 2 |
| Standalone utility, no shared data (e.g., a design tool) | 1 — *excluded from the score* |

Score each weighted tool's connection: **Live sync = full credit, Manual export = half credit, Missing = zero.** Then:

```
Integration Fabric Score (0–100) =
  Σ (tool weight × connection credit)  ÷  Σ (tool weight)  × 100

  where connection credit:  Live = 1.0,  Manual = 0.5,  Missing = 0
  and standalone utilities (weight 1, no shared data) are excluded from both sums
```

Map the score to the 1–5 scorecard dimension in Step 5: **90–100 → 5, 75–89 → 4, 60–74 → 3, 40–59 → 2, <40 → 1.** A stack can have full coverage and still score a 2 here — that gap between coverage and fabric is the headline finding to surface.

Any **weight-3 tool that is Missing** (customer/event/PII data not reaching the SoR) is **integration debt** and a mandatory flag, regardless of the overall score — it is the single largest source of attribution failure and CAC inflation.

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

#### Cut order — the consolidation ladder

Don't hand over a flat list of cuts; sequence them, because a stack is a graph and the wrong first cut breaks a downstream tool. Order the recommendations down this ladder — cut from the top first:

1. **Shelfware, zero downstream** — unused tool nothing else reads from. Pure win; cut now, no migration.
2. **Redundancy, leaf node** — true duplicate where the cut tool feeds nothing else (no other tool depends on its data). Cut after confirming no manual export quietly depends on it.
3. **Redundancy, feeds others** — duplicate that is a data source for another tool. Cut only after the survivor is wired to replace that feed; otherwise you trade a redundancy for a Missing weight-3 silo.
4. **Capability overlap** — keep the tool the team actually uses; the consolidation here is *behavioral* (move the workflow into the tool you keep), so gate it on the integration being live, not just present.
5. **Intentional overlap** — do **not** auto-cut. Flag the process change required and the team that owns it; recommend a decision, not an execution.

Rule of thumb: **never cut a node before its dependents are re-parented.** Cross-check every proposed cut against the Step 4 dependency map — if the cut tool has an outbound arrow, it is at best a rung-3 cut, never a rung-1.

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
- **Marketing Technology Landscape as the organizing spine.** Every tool gets a canonical home; every gap has a name. Vague "you're missing a tool in this area" is not an audit.
- **Costs or `[verify]`.** Never invent pricing. Use the user's inputs; flag everything unconfirmed.
- **Distinguish redundancy from intentional overlap.** Cutting a tool that Sales intentionally uses separately from Marketing will break process; document before recommending.
- **Integration fabric is the real story.** Tools that don't talk to each other are data silos; that's where attribution breaks and CAC inflates. Score it with the weighted Integration Fabric formula, not a gut percentage, and surface the coverage-vs-fabric gap prominently.
- **Sequence the cuts; never cut a node before its dependents are re-parented.** Consolidation runs down the ladder (shelfware → leaf redundancy → feeding redundancy → capability → intentional), cross-checked against the dependency map. A cut that orphans a downstream tool trades one problem for a worse one.
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
- Every tool placed in a Marketing Technology Landscape category, with primary + secondary home recorded (tie-breaker: spend-weight → system-of-record → owner)?
- Every sub-category needed for this growth stage assessed (Covered / Gap / N/A), with each Gap given a severity (Critical / High / Nice-to-have) via the channel-breakage test — not a vibe?
- Overlap classified as Redundancy / Intentional / Capability — not just "these two tools are similar"?
- Integration Fabric Score computed with the weighted formula (not a gut %), mapped to the 1–5 scorecard; any weight-3 tool with a Missing sync flagged as integration debt?
- Dependency map shows live / manual / missing integrations?
- Consolidation recs sequenced down the cut ladder (no node cut before its dependents are re-parented), and each includes estimated saving (or `[verify]`), migration effort, and what gap opens on cut?
- Artifact saved to `./martech-audit/[brand-slug]-stack-audit-[YYYY-MM-DD].md`?
- Downstream handoffs offered where relevant (tracking-plan, data-qa, consent, UTM, GTM, vendor-cost)?
