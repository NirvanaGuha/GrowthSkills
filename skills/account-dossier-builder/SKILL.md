---
name: account-dossier-builder
description: >
  Company name + URL → structured account dossier: business model, revenue signals, tech stack,
  org chart hints, recent news, strategic priorities, and pain hypotheses tied to your offer.
  The raw intelligence feed that makes every downstream ABM task (outreach, cold email, meeting
  prep, battlecard, proposal) actually personalized instead of template-flavored. Builds one
  dossier per run; designed to feed cold-outreach-sequence-architect, meeting-prep-follow-up-pack,
  and competitive-intelligence-dossier. Uses the OSINT + Signal Triage framework to separate
  high-signal facts from inferred hypotheses — so a sales rep or growth marketer knows exactly
  which claims to verify before using them in a conversation. Never fabricates customers, numbers,
  or quotes. Use whenever the user says "research this account," "build a dossier," "prep me on
  this company," "who is [Company]," "pull intel on," "account brief," "before my call with,"
  or pastes a company name + URL and needs a research packet rather than a sequence or pitch.
---

# Account Dossier Builder

Company name + URL in. Structured intelligence packet out. Every claim sourced or marked `[infer]`. The downstream skills — outreach, meeting prep, battlecard — consume this; they don't redo the research.

This skill researches and synthesizes. It does not write the outreach email, design the deck, or generate the pitch. When those are next, it hands off cleanly to the right sibling.

---

## Skills this calls

- **`brand-brain`** (required, first) — loads your brand's ICP, positioning, offer mechanics, and proof so the dossier's pain hypotheses map to your actual solution, not a generic one.
- **`competitive-intelligence-dossier`** (optional) — call when the account is also a potential competitor or when competitive overlap is strategically relevant to the deal.
- **`icp-persona-builder`** (optional) — call to match the account's key buyer personas against your ICP before scoring fit.
- **`meeting-prep-follow-up-pack`** — downstream consumer: pass this dossier as input.
- **`cold-outreach-sequence-architect`** — downstream consumer: uses pain hypotheses + tech stack signals to personalize sequences.
- **`objection-library-builder`** — downstream consumer: cross-references the account's likely objections against the dossier's context signals.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (ICP, offer, proof)
Step 1  Gather signals      ──► fetch + scan the target account across all five signal layers
Step 2  Triage signals      ──► separate FACT from INFER; mark confidence
Step 3  Synthesize dossier  ──► build the structured eight-section output
Step 4  Score fit           ──► ICP tier + deal-context flags
Step 5  Hand off            ──► surface the right downstream skill for the next job
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns your ICP definition, offer mechanics, positioning, and proof. Pain hypotheses in the dossier must map to your real offer, not abstract pain. If `brand-brain` is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If neither exists, ask: what does your product do, who is the ICP, and what pain does it solve — then proceed.

### Step 1 — Gather signals across the five layers

Work through every layer present. Skip silently if a source is unavailable.

| Layer | Sources | What to extract |
|---|---|---|
| **1. Public web presence** | Homepage, About, /pricing, /customers, /blog, /careers | Business model, product lines, stated positioning, customer logos, open roles (signals: growth areas, tech debt, team size) |
| **2. Tech stack** | BuiltWith, Wappalyzer public data, footer/source hints, integration pages, job descriptions | CMS, e-commerce platform, analytics, ad stack, CRM signals, infra/cloud hints |
| **3. News & signals** | Press releases, TechCrunch/Crunchbase, LinkedIn company posts (last 90 days), funding rounds, acquisitions, product launches | Recent strategic moves, budget signals, leadership changes, new GTM |
| **4. People & org** | LinkedIn (company page + key roles), job posts | Relevant buyer personas, reporting lines, hiring signals pointing to priority investments |
| **5. Social proof & voice** | G2/Capterra reviews, Twitter/X, customer testimonials on their site | How customers describe them, recurring pain in their reviews (= your angle in), competitive context |

Fetch what you can via WebFetch/WebSearch. If a layer is blocked or empty, note it in the dossier; don't invent to fill the gap.

### Step 2 — Signal Triage (the OSINT discipline)

Every claim gets a confidence tag before it enters the dossier:

- **FACT** — directly stated on the company's own properties or a reliable third-party source (Crunchbase, press release, job listing). Cite the source in brackets.
- **[infer]** — logical inference from one or more FACT signals. Clearly labeled. Buyer must validate before using in a live call.
- **[verify]** — a number, customer name, or claim that appeared online but needs direct confirmation (often a secondary source or a stale figure).

Never upgrade an `[infer]` to FACT without a second source.

---

## The dossier (eight sections)

Output the dossier in this structure. Omit a section only if no signal exists after a genuine search attempt — never omit and never pad.

```
# Account Dossier — [Company Name]
Date: [today]  |  Researcher: Account Dossier Builder  |  Brand context: [brand slug]

## 1. Snapshot
One-paragraph company profile: what they do, who they serve, business model, scale signals
(employees [infer from LinkedIn], funding stage, revenue [verify if public]).

## 2. Business model & revenue signals
How they make money (subscription / transactional / marketplace / services).
Pricing page observations. Customer segments. Growth-stage signals.

## 3. Tech stack
| Category | Tool/Platform | Confidence | Signal source |
e.g. CMS, e-commerce, analytics, CRM, CDP, ads, email/push, infra

## 4. Strategic moves (last 90 days)
Bullet list. Each item: [DATE or approximate] — event — what it signals.
New hires, launches, funding, partnerships, content pivots, job postings.

## 5. Org & buying center
Key personas likely relevant to your deal:
| Role | Name (if found) | LinkedIn hint | Likely priority |
Note decision-making structure if inferable.

## 6. Pain hypotheses
Ranked list (most → least likely based on signals). For each:
- Hypothesis: [specific pain]
- Signal(s) that support it: [cite]
- Confidence: FACT / [infer] / [verify]
- How your offer addresses it: [map to brand's real capability — from brand-brain]

DO NOT fabricate pain. If signals don't support a hypothesis, don't list it.

## 7. ICP fit score
Tier: 1 (strong fit) / 2 (partial fit) / 3 (thin fit) / Not a fit
Rationale: 2–3 sentences mapping this account against your ICP from brand-brain.
Flags: [any deal-context flags — e.g., "contract renewal likely Q3 [infer from job posts]",
"competitor installed [verify]", "expansion-stage budget signals"]

## 8. Recommended next step
Which downstream skill to invoke and with what inputs from this dossier.
e.g. → cold-outreach-sequence-architect (use sections 5 + 6 as personalization context)
     → meeting-prep-follow-up-pack (pass full dossier as context)
     → competitive-intelligence-dossier (their tech stack shows competitive overlap)
```

Save to `./outreach/[account-slug]-dossier.md`. Confirm the path on save.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Pain hypotheses that don't map to your real offer are noise. Load brand context before building the dossier.
- **FACT / [infer] / [verify] on every claim.** The discipline that separates a useful dossier from a hallucinated one. A rep who walks into a call repeating a fabricated fact loses trust instantly.
- **Research depth over speed.** Work through all five layers before synthesizing. Thin dossiers built from only the homepage are not worth downstream personalization.
- **Pain hypotheses are hypotheses.** Rank them, source them, map them to your offer — but never present them to a buyer as confirmed fact.
- **One account per run.** Scope is tight so depth is real. For list-scale enrichment, hand to `account-list-builder-icp-scorer`.
- **No fabricated social proof.** Never invent customer names, quotes, numbers, or competitive wins. Real proof only, or `[verify]`.

---

## What not to do

- Don't write the outreach email here — that's `cold-outreach-sequence-architect`.
- Don't build the competitive analysis here — call `competitive-intelligence-dossier` for that.
- Don't pad sections with generic industry boilerplate when signals are thin — note the gap.
- Don't upgrade `[infer]` to FACT to make the dossier look more authoritative.
- Don't skip the ICP fit score — the whole point is knowing whether to invest sales cycles in this account.
- Don't save inside the skill folder — always `./outreach/[account-slug]-dossier.md`.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded — pain hypotheses map to your real offer?
- All five signal layers attempted; gaps noted (not padded)?
- Every claim tagged FACT, `[infer]`, or `[verify]` — nothing untagged?
- Tech stack section populated (even if partial) — it drives personalization downstream?
- Pain hypotheses ranked and sourced; each one mapped to a brand capability?
- ICP fit score assigned with rationale — not skipped?
- Recommended next step names the exact downstream skill and the inputs to pass it?
- Saved to `./outreach/[account-slug]-dossier.md` and path confirmed?
