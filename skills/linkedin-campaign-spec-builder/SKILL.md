---
name: linkedin-campaign-spec-builder
description: >
  Takes a B2B offer, ICP job titles/seniority/firmographics, and a budget → produces a complete,
  activation-ready LinkedIn Campaign Manager spec: campaign objective, audience layers (core +
  retargeting + exclusions), bid type and budget allocation, ad format selection with format-fit
  rationale, and a structured copy brief for each format. Built on LinkedIn's own Campaign
  Objectives framework (Awareness → Consideration → Conversions) and the B2B awareness-funnel
  model — every structural choice (objective, format, bid type) derives from the funnel stage,
  not from preference. Composes `brand-brain` for voice and ICP, `icp-persona-builder` for
  audience profile depth, `audience-targeting-spec-writer` for cross-platform parity, and
  `ad-to-landing-page-message-match-auditor` to gate copy on offer alignment. Saves a
  campaign spec file to `./linkedin-campaigns/[slug]-spec.md`. Use when the user says
  "LinkedIn campaign," "set up LinkedIn ads," "LinkedIn targeting spec," "LinkedIn campaign
  brief," "LinkedIn objective/audience/bid," "map out a LinkedIn campaign," or hands over a
  B2B offer and asks how to run it on LinkedIn.
---

# LinkedIn Campaign Spec Builder

Turn a B2B offer and an ICP into an activation-ready LinkedIn Campaign Manager spec — objective, audience, bid, format, and copy brief — before the first dollar is spent. Structural choices are derived from funnel stage and LinkedIn's objective-to-format constraints, not from guessing. Copy brief is locked to the brand's real voice and real offer. Nothing goes into this spec that you'd have to override in Campaign Manager.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, banned words, offer mechanics, ICP, and real proof. This skill does not re-derive brand context.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their offer one-liner, ICP job titles/seniority/company size, and 3 voice adjectives before proceeding.
- **`icp-persona-builder`** (optional) — if the ICP is thin or unvalidated, delegate audience depth here; pass the brand slug so it doesn't re-scan.
- **`audience-targeting-spec-writer`** (optional) — call for cross-platform parity when LinkedIn is one channel in a broader paid spec; avoid duplicating targeting logic.
- **`ad-to-landing-page-message-match-auditor`** (optional, gate) — before finalising the copy brief, pass the proposed headline + destination URL; fail hard on mismatch.
- **`channel-roi-scorecard`** (optional) — call when the user wants a LinkedIn-vs-other-channel ROI comparison to justify budget allocation.
- **`data-qa-measurement-gotcha-checker`** (optional, data gate) — invoke when the user provides conversion data or ROAS inputs to catch attribution-window and (not set) issues before they corrupt the brief.

---

## How a run works

```
Step 0  Load brand           ──► brand-brain → voice, offer, ICP, proof
Step 1  Clarify inputs       ──► offer, ICP, budget, funnel stage, landing URL
Step 2  Set objective        ──► LinkedIn Objective framework → one objective
Step 3  Build audience spec  ──► core targeting + retargeting layers + exclusions
Step 4  Bid & budget alloc   ──► bid type rationale + budget split per layer
Step 5  Format selection     ──► format-fit table → chosen formats with rationale
Step 6  Copy brief           ──► per-format brief: headline, intro text, CTA, offer proof
Step 7  Message-match gate   ──► ad-to-landing-page-message-match-auditor (if URL present)
Step 8  Save & present       ──► ./linkedin-campaigns/[slug]-spec.md
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency. Do not build any part of the spec before it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their offer one-liner, ICP job titles/seniority/company size, and 3 voice adjectives before proceeding.

---

### Step 1 — Clarify inputs (only the gaps)

Use the brand digest to pre-fill what you already know. Ask only for what is missing:

| Required | Usually in brand.md | Usually missing |
|---|---|---|
| Offer one-liner | offer_mechanics | — |
| ICP job titles + seniority | icp | — |
| Company size / industry / geo | icp | often thin |
| Primary conversion goal | positioning | sometimes |
| Landing page URL | offer destinations | often missing |
| Monthly budget (USD or local) | — | almost always ask |
| Funnel stage (cold / warm / hot) | awareness_tendency | clarify if mixed |

Batch missing items into a single question block — never drip one question at a time.

---

### Step 2 — Set the LinkedIn Objective

LinkedIn's Campaign Manager offers six objectives mapped to three funnel stages. Pick **one** per campaign group. For mixed-funnel briefs, specify separate campaign groups.

| Stage | LinkedIn Objective | When to use |
|---|---|---|
| Awareness | **Brand Awareness** | New category, cold ICP, CPM-optimised |
| Consideration | **Website Visits** | Drive traffic, evaluate content consumption |
| Consideration | **Engagement** | Thought leadership, organic amplification |
| Consideration | **Video Views** | Demo or explainer distribution |
| Conversions | **Lead Generation** | In-platform Lead Gen Form (no friction landing page) |
| Conversions | **Website Conversions** | Trial, demo, purchase on the brand's own URL |

Decision rule: default to **Lead Generation** for B2B gated offers (demo, trial, content download) with a known ICP list and CPL under $150 [verify against current LinkedIn benchmarks]. Shift to **Website Conversions** when the landing page is conversion-optimised and the offer is bottom-funnel. Use **Website Visits** for retargeting nurture or content-heavy TOFU.

State the chosen objective and a one-sentence rationale anchored to funnel stage.

---

### Step 3 — Build the audience spec

Use LinkedIn's targeting dimensions in layers. Always produce three layers.

#### Layer A — Core (cold prospecting)

Build from the ICP using **job title OR job function + seniority** (never both job title AND function — overlap deflates reach). Add company size and industry. Add geo.

```
Targeting dimensions (pick the minimum effective set):
  Job Titles         — use exact + variants (plural, abbreviation, function synonym)
  Job Seniority      — Senior, Manager, Director, VP, CXO (per ICP)
  Company Size       — employee range matching ICP firmographics
  Industry           — LinkedIn industry taxonomy (use ≥3 to avoid over-restriction)
  Geography          — country / region
Minimum audience floor: 50,000 members (LinkedIn will warn below this)
```

Flag if the combined audience estimate will likely fall below 50K; suggest relaxing one dimension.

#### Layer B — Warm retargeting

Compose from LinkedIn's Matched Audiences + Insight Tag signals:

- Website visitors (last 30/60/90 days by URL path — specify the path, e.g. `/pricing`, `/demo`)
- Video viewers (25%, 50%, 75% completion from prior campaigns)
- Lead Gen Form openers (did not submit)
- Uploaded contact list (CRM export of ICP accounts — CSV match by email or company)

Layer B bids higher and carries the more direct conversion offer.

#### Layer C — Exclusions (always explicit)

List at minimum:
- Existing customers (CRM upload)
- Current trial / active users (email list)
- Employees (company page follower exclusion)
- Competitors' company pages (if the offer is a switching play and audiences are activated)

Do not skip exclusions. Every dollar wasted on existing customers is invisible waste.

---

### Step 4 — Bid type and budget allocation

**Bid type by objective and layer:**

| Objective | Layer A (cold) | Layer B (warm) |
|---|---|---|
| Brand Awareness | Maximum Delivery (CPM) | Enhanced CPC or CPM |
| Website Visits | Enhanced CPC | Maximum Delivery |
| Lead Generation | Maximum Delivery | Manual CPC with bid cap |
| Website Conversions | Maximum Delivery (let LinkedIn optimise) | Manual CPC |

**Budget split rule of thumb** for a single-offer campaign:
- 70% Layer A (scale), 30% Layer B (convert) — adjust toward 50/50 once retargeting pool exceeds 10K members.
- Layer B daily budget floor: $20/day minimum to exit learning phase [verify].

State the monthly budget, daily equivalent, and the A/B split in dollar terms. If the monthly budget is under $3,000, flag that LinkedIn's algorithm needs at minimum $1,500/month per campaign group to exit the learning phase [verify] — below this, consider consolidating.

---

### Step 5 — Format selection

Choose from LinkedIn's active formats. For each campaign group, recommend a **primary format** and one **fallback format** if the primary requires assets not yet available.

| Format | Best fit | Character limits (2024 [verify]) |
|---|---|---|
| **Single Image Ad** | TOFU awareness, MOFU nurture, retargeting | Headline ≤150 chars · Intro ≤150 chars visible (600 max) · CTA button required |
| **Carousel Ad** | Multi-feature offers, step-by-step processes | 2–10 cards · Per card headline ≤45 chars |
| **Video Ad** | Demo, testimonial, explainer | 3 sec–30 min; 15–30s optimal for cold [verify] |
| **Document Ad** | Gated report/whitepaper download (Lead Gen Form) | Up to 10 pages visible; drives Lead Gen Form |
| **Conversation Ad** | BOFU, personalised outreach to warm list | Message tree: 3–5 CTAs; InMail open rate varies |
| **Lead Gen Form + Single Image** | Best CPL for most B2B offers | Prefilled with member data; 3–5 fields optimal |
| **Thought Leader Ad** | Amplifying an employee's organic post | Requires admin access to member's profile |

Selection rule: **Lead Gen Form + Single Image** is the default for conversion campaigns with CPL as the KPI. Move to Document Ad when the offer is content-heavy and the lead magnet is the hook. Add Carousel when the offer has a multi-step proof story.

---

### Step 6 — Copy brief (per format)

Write a structured brief for each chosen format — not final ad copy. The copy writer or `ad-copy-variant-generator` produces the final copy from this brief.

```
## Copy Brief — [Format name]

Funnel stage: [cold / warm / hot]
Objective: [LinkedIn objective]
Audience: [Layer A or B descriptor]
Landing destination: [URL or Lead Gen Form]

Headline (≤[limit] chars):
  Job to be done: [what the headline must accomplish]
  Key proof / hook: [from brand.md proof_points — real only, else [verify]]
  Angle: [value | specificity | risk-reversal | curiosity | social proof]
  Banned words / phrases: [from brand.md]

Intro text (≤150 visible chars):
  Open with: [pain statement | stat | question — pick one]
  Voice note: [from brand.md voice adjectives]

CTA button: [choose from LinkedIn's button options: Download / Learn More / Sign Up / Register / Apply / Request Demo]

Offer proof microcopy (Lead Gen Form thank-you screen or intro):
  [one real proof point from brand.md; mark [verify] if unconfirmed]
```

Produce one brief block per format × per layer combination in scope.

---

### Step 7 — Message-match gate

If a landing page URL was provided: invoke `ad-to-landing-page-message-match-auditor` (Skill tool) with the headline + URL. If the audit returns a mismatch flag, surface it inline and pause the spec — do not present a mismatched brief as ready to activate.

If no URL was provided: note in the spec that message-match validation is pending and must be run before activation.

---

### Step 8 — Save and present

Save the completed spec to `./linkedin-campaigns/[brand-slug]-[campaign-slug]-spec.md`. Create the folder if absent. Confirm the path. Present the spec inline as well.

---

## The B2B Funnel × LinkedIn Objective matrix (decision reference)

The foundational framework for every structural choice in this skill is the intersection of **buyer awareness stage** (Schwartz / solution-aware model) with **LinkedIn's objective constraints**:

- A buyer who does not know your solution exists needs CPM-bought impressions (Awareness objective), not a Lead Gen Form they will never fill.
- A buyer who knows the category but not your brand needs education and consideration content (Website Visits or Video Views), not a "Book a demo" hard ask.
- A buyer who has visited your pricing page is hot — warm retargeting with Lead Gen Form or Website Conversions and a direct offer is right; brand awareness spend here is waste.

This matrix is why the audience layers, bid types, and format choices are not interchangeable between layers. Always anchor every recommendation back to a buyer-stage rationale, not a platform feature.

---

## Principles

- **One objective per campaign group.** Mixing objectives in a single group confuses LinkedIn's delivery algorithm and prevents clean reporting.
- **Minimum effective audience, not maximum.** Broader is not better on LinkedIn; CPMs are high. Tight ICP targeting with a strong offer beats spray.
- **Exclusions are non-negotiable.** Never launch without excluding current customers and employees.
- **CPL benchmarks vary.** LinkedIn B2B CPL benchmarks range $50–$300+ [verify] depending on offer, industry, and seniority. Do not invent a CPL target; surface the benchmark range and let the user set the target.
- **Brand voice from brand-brain only.** Do not write copy adjectives or proof from memory — use the returned digest, mark anything unconfirmed `[verify]`.
- **Copy brief ≠ finished ad copy.** This skill produces the brief and the spec; `ad-copy-variant-generator` produces the final variants.

---

## What not to do

- Do not produce the spec before `brand-brain` returns the brand context.
- Do not combine Job Title + Job Function targeting — it collapses audience size unpredictably.
- Do not recommend Conversation Ads for cold audiences — they require warm signals to avoid spam-flag rates.
- Do not set a specific CPL target from memory — benchmark ranges only, marked `[verify]`.
- Do not skip the exclusions layer to save time.
- Do not launch a copy brief with a mismatched landing page — gate on Step 7.
- Do not produce "final ad copy" here — produce the brief; let a copy skill do the writing.

---

## Quality checklist

- `brand-brain` called and digest returned before any spec work began?
- Input gaps resolved in a single question batch (not drip-questioned)?
- Exactly one LinkedIn objective selected with a buyer-stage rationale?
- Three audience layers (core, warm retargeting, exclusions) fully specified?
- Bid type matched to objective AND layer — not a blanket recommendation?
- Budget allocated with daily equivalent and learning-phase floor noted?
- Format selected with a primary + fallback and character limits stated?
- Copy brief produced per format × layer, using only real proof from brand.md (rest `[verify]`)?
- Message-match gate run (or flagged as pending)?
- Spec saved to `./linkedin-campaigns/[slug]-spec.md`?
