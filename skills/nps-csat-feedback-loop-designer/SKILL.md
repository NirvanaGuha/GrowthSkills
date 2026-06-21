---
name: nps-csat-feedback-loop-designer
description: >
  Turns a raw NPS or CSAT export — plus a description of the touchpoint that triggered it — into a
  closed-loop feedback system: a categorized response table that reveals the real drivers behind the
  score, a routing matrix that sends each respondent to the right owner and action, and on-brand
  follow-up sequences for detractors (recovery + churn save) and promoters (testimonial + referral
  amplification). Built on the Inner Feedback Loop framework (Collect → Categorize → Route → Act →
  Close) with Satmetrix/Bain NPS mechanics and Kano Model category logic baked in. Brand voice and
  proof come from brand-brain; follow-up copy is composed from existing library skills rather than
  written from scratch. Use when someone says "what do I do with my NPS results," "close the loop
  on CSAT," "follow up with detractors," "respond to survey respondents," "NPS action plan,"
  "automate survey response routing," or hands you a feedback CSV and asks what to do next.
---

# NPS & CSAT Feedback Loop Designer

Survey data is not insight until you route it, act on it, and tell the respondent what happened. This skill closes that gap — turning a flat export into a categorized action table, a routing matrix, and on-brand follow-up sequences that make detractors feel heard and turn promoters into advocates.

It does not conduct the survey, choose the platform, or set up integrations. It takes what already exists and makes the feedback actionable.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, ICP, proof, banned words, and offer mechanics before any copy is produced. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP job-to-be-done, 3 voice adjectives, key product proof points, and the offer/CTA destination before proceeding.
- **`churn-save-sequence-writer`** — compose for Detractor recovery sequences rather than write from scratch; pass the categorized detractor themes and brand digest.
- **`review-testimonial-solicitation-sequence`** — compose for Promoter testimonial/advocacy sequences; pass the promoter segment definition and brand digest.
- **`cta-variant-generator`** — used inside follow-up email CTAs to ensure each recovery or amplification message has a correctly staged call to action.
- **`lifecycle-email-push-copy-reviewer`** — passes the drafted sequences through a voice + compliance review before presenting to the user.
- **`sentiment-shift-detector`** *(optional)* — if two time-period exports are provided, call this skill first to surface themes that got better or worse before routing.
- **`jtbd-customer-interview-suite`** *(optional)* — when verbatim comments are rich enough to yield JTBD insight, delegate transcript analysis to this skill.

---

## How a run works

```
Step 0  Load brand            ──► call brand-brain
Step 1  Ingest & assess       ──► parse the export, identify gaps, confirm touchpoint
Step 2  Categorize            ──► Inner Feedback Loop + Kano bucketing
Step 3  Score & prioritize    ──► NPS / CSAT mechanics + impact weighting
Step 4  Route                 ──► routing matrix per segment × category × owner
Step 5  Sequence              ──► compose follow-up via churn-save + testimonial skills
Step 6  Close-the-loop note   ──► product / ops update template
Step 7  Quality check → present
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, and ICP. Do not draft a single line of copy before it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP job-to-be-done, 3 voice adjectives, key product proof points, and the offer/CTA destination before proceeding.

---

### Step 1 — Ingest and assess

Accept any of:
- CSV / spreadsheet export (NPS, CSAT, or CES) with score + verbatim columns
- A pasted table or JSON
- A summary ("27% Detractors, top complaint: slow onboarding")

**Confirm before proceeding:**
- Touchpoint that triggered the survey (onboarding, post-support, in-product milestone, renewal?)
- Time window and approximate N
- Which segments are known (plan tier, cohort, industry, CSM owner)
- What the user wants: full system, routing-only, sequences-only, or close-the-loop note

If verbatims are absent, flag it — categories will be lower confidence and marked `[low-signal]`.

---

### Step 2 — Categorize (Inner Feedback Loop + Kano)

**The Inner Feedback Loop framework** structures every closed-loop system around five stages:
Collect → Categorize → Route → Act → Close. This skill owns stages 2–5; stage 1 (Collect) is upstream; stage 5 (Close) produces the product-update template.

**Categorize verbatims** using two orthogonal lenses:

**Lens A — Theme taxonomy** (tag each verbatim to one primary, one secondary):

| Theme | Examples |
|---|---|
| Onboarding & setup | "took too long to configure," "unclear first steps" |
| Core product value | "does exactly what I need," "missing X feature" |
| Support & responsiveness | "rep was great," "waited 3 days" |
| Pricing & value perception | "too expensive for what it does," "worth every cent" |
| Reliability & performance | "keeps crashing," "always fast" |
| Integrations & ecosystem | "can't connect to our CRM" |
| Relationship & trust | "feel like a number," "our CSM is amazing" |

**Lens B — Kano category** (shapes routing priority):

| Kano | Meaning | Routing implication |
|---|---|---|
| Must-be (Basic) | Its absence causes extreme dissatisfaction; presence is invisible | Fix immediately; P0 for product/ops |
| Performance | Satisfaction scales linearly with quality | Prioritize in roadmap; signal for upgrade angles |
| Delighter | Surprise value; creates promoters | Amplify in marketing; proof-vault candidate |
| Indifferent | Customers don't care either way | De-prioritize; may be internal effort sink |
| Reverse | Some hate it; some love it | Segment before acting; do not over-correct |

Build the categorized response table:

```
## Categorized Response Table
Touchpoint: [name]  |  N = [count]  |  Period: [dates]  |  Brand: [slug]

| Respondent ID | Score | Segment | Theme (primary) | Theme (secondary) | Kano | Confidence | Verbatim (excerpt) |
```

---

### Step 3 — Score and prioritize

**NPS mechanics (Bain/Satmetrix):**
- Promoters: 9–10. Passives: 7–8. Detractors: 0–6.
- NPS = % Promoters − % Detractors. Industry SaaS median: ~31 [verify for current year].
- NPS is directional, not precise at small N (<50). Flag this.

**CSAT mechanics:**
- CSAT % = (satisfied responses / total) × 100. "Satisfied" = top 1–2 of scale.
- Benchmark: B2B SaaS ~77–80% [verify].

**Impact weighting.** For each theme, compute:
- Frequency (% of respondents mentioning it)
- Severity (weighted by score band — Detractor mentions count 2×, Passive 1×, Promoter 0.5× for problem themes)
- Kano urgency (Must-be → highest; Delighter → separate track)

Output a **Priority Stack**: top 3 fix themes (Detractors/Must-be), top 3 amplify themes (Promoters/Delighter), and any Reverse themes that need segmentation before action.

---

### Step 4 — Route

Build the **Routing Matrix** — the operational core of the closed-loop system:

```
## Routing Matrix

| Segment | Score Band | Theme | Kano | Owner | Action | SLA | Trigger |
|---|---|---|---|---|---|---|---|
| Enterprise / Onboarding | Detractor (0–6) | Onboarding & setup | Must-be | CSM + Product | 1:1 call + product ticket | 24h | Auto-tag in CRM |
| SMB / Support | Passive (7–8) | Support & responsiveness | Performance | Support lead | Follow-up email + CSAT re-survey in 30d | 48h | ESP automation |
| Promoter | Any | Relationship & trust | Delighter | Marketing | Testimonial ask + referral invite | 72h | Triggered sequence |
```

Rules:
- Every Detractor gets a human touchpoint (call or email with a named sender), not a bulk send.
- Passives are the highest-leverage segment for NPS movement — route them to education + value-realization nudges, not generic thank-yous.
- Promoters: testimonial ask first, referral second (never both in the same email; sequence them 5–7 days apart).
- Any Must-be issue in Detractor zone → escalate to product/ops within 24h regardless of segment.

---

### Step 5 — Compose follow-up sequences

Do not write these from scratch. Compose:

**Detractor recovery sequence** → invoke `churn-save-sequence-writer`, passing:
- The top 2 Detractor themes with Kano category
- Segment definitions (plan, cohort)
- Brand digest from Step 0
- Instruction: 2–3 email sequence, opener from a named human, no promotional copy in email 1

**Promoter amplification sequence** → invoke `review-testimonial-solicitation-sequence`, passing:
- Promoter segment definition
- Target platforms (G2, Capterra, Google, or brand-specific)
- Brand digest from Step 0
- Instruction: 2-touch sequence (testimonial ask day 0, referral invite day 5–7 if no response to first)

After both skills return, pass the full draft sequences through `lifecycle-email-push-copy-reviewer` for voice + clarity + character-limit compliance.

For every CTA button in those sequences, call `cta-variant-generator` with the awareness stage (Detractor = problem-aware, Promoter = most-aware) and the brand digest; replace any generic "Reply here" CTAs with the returned recommendation.

---

### Step 6 — Close-the-loop product update template

The loop is only closed when the respondent knows their feedback changed something. Produce:

```
## Close-the-Loop Update Template

Subject: [First name], you told us [X] — here's what we did

Body:
- Acknowledge the specific theme they raised (personalization token)
- State the concrete action taken or in-progress: [product fix / policy change / new resource]
  (Use brand proof only; mark anything unconfirmed [verify])
- One soft re-engagement CTA aligned to their plan tier
- No upsell in the same send as the acknowledgment

Send trigger: when the roadmap item ships, or at 90-day max even if in-progress
Owner: [CSM / Product Marketing / VP Product — per routing matrix]
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before the brand digest loads. Voice and banned words override everything here.
- **Segment before you act.** Never run Detractor recovery and Promoter amplification to the same list.
- **Must-be issues are P0.** A theme that causes Detractor scores and maps to Kano Must-be goes to product/ops within 24 hours — not into a nurture sequence.
- **Passives are the leverage point.** They are one good experience away from Promoter. Route them to value-realization, not generic thank-yous.
- **Compose, don't rewrite.** Follow-up sequences go through `churn-save-sequence-writer` and `review-testimonial-solicitation-sequence` — not written inline here.
- **Close the loop in product time, not marketing time.** The follow-up email goes out when something changes, not just to acknowledge the survey.
- **Truth only.** If the product issue mentioned hasn't been fixed, say "we're working on it" — not "we fixed it." Invented commitments destroy the trust the loop is trying to rebuild.

---

## What Not to Do

- Don't treat NPS as statistically precise at N < 50 — flag it; use directional language.
- Don't send the same recovery email to a Detractor about pricing and a Detractor about reliability — they need different copy.
- Don't bulk-promote to Passives. It accelerates churn, not upgrade.
- Don't ask for a testimonial and a referral in the same email — it reads as extractive; sequence them.
- Don't skip the close-the-loop update. A survey with no follow-up destroys response rates next cycle.
- Don't reimplement brand resolution — call `brand-brain`.
- Don't redraft sequences that `churn-save-sequence-writer` or `review-testimonial-solicitation-sequence` already own.
- Don't mark an NPS score as "good" or "bad" without the industry benchmark and the brand's own trend — a 40 in SaaS is above median; a 40 in consumer hardware is not.

---

## Frameworks used (quick reference)

**Inner Feedback Loop** — Collect → Categorize → Route → Act → Close. The operational spine; every action maps to a stage.

**Bain/Satmetrix NPS** — Promoter (9–10) / Passive (7–8) / Detractor (0–6); NPS = %P − %D. The score is a leading indicator of revenue retention, not a satisfaction score.

**Kano Model** — Must-be / Performance / Delighter / Indifferent / Reverse. Maps verbatim themes to routing urgency. Must-be issues get P0 escalation regardless of segment; Delighters become marketing proof; Reverse themes require segmentation before any action.

---

## Artifacts produced

All artifacts save to the project root (never inside the skill folder):

| Artifact | Path | Purpose |
|---|---|---|
| Categorized response table | `./feedback/[slug]-categorized-[date].md` | Tagging + evidence base |
| Priority stack | inline in session | Top themes to fix + amplify |
| Routing matrix | `./feedback/[slug]-routing-matrix.md` | Operational owner × action × SLA |
| Detractor recovery sequence | `./feedback/[slug]-detractor-sequence.md` | From churn-save-sequence-writer |
| Promoter amplification sequence | `./feedback/[slug]-promoter-sequence.md` | From review-testimonial-solicitation-sequence |
| Close-the-loop template | `./feedback/[slug]-close-the-loop.md` | Product update email |

---

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called and digest loaded before any copy produced; voice + banned words honored throughout?
- [ ] Touchpoint, N, and time window confirmed before categorizing?
- [ ] Every verbatim tagged to one primary theme and one Kano category; low-signal rows flagged?
- [ ] Priority stack produced with frequency + severity weighting; Must-be issues escalated to P0?
- [ ] Routing matrix assigns a human touchpoint to every Detractor with a named owner and SLA?
- [ ] Detractor and Promoter sequences composed via sibling skills, not written from scratch here?
- [ ] Sequences reviewed by `lifecycle-email-push-copy-reviewer`; CTAs replaced via `cta-variant-generator`?
- [ ] Close-the-loop template includes a personalization token, a concrete action statement, and no upsell in the same send?
- [ ] No invented commitments; unconfirmed fixes marked `[verify]`?
- [ ] Artifacts saved to `./feedback/` with brand slug and date in filename?
