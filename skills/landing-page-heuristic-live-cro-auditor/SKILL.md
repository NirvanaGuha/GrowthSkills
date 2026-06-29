---
name: landing-page-heuristic-live-cro-auditor
description: >
  Landing page URL or screenshot → fetched, scored, evidence-backed conversion audit mapped to
  heuristics (clarity, trust, friction, urgency) with prioritized fixes. Pulls the live page (or
  reads a screenshot/paste), scores it against a seven-dimension heuristic framework, surfaces
  the highest-leverage conversion leaks with supporting evidence, and outputs a ranked fix list
  you can hand straight to a developer or copywriter. Calls brand-brain to anchor audit findings
  against the brand's real ICP, proof, and positioning — so gaps are called out in context, not
  as generic best-practice lectures. Use when the user says "audit my landing page," "why isn't
  my page converting," "CRO review," "critique this page," "find conversion leaks," "score my
  lander," "what should I fix first," "pre-launch page review," or pastes a URL and asks what
  to improve.
---

# Landing Page Heuristic & Live CRO Auditor

Hand it a URL or a screenshot — it fetches, reads, scores, and tells you exactly what is
killing your conversion rate and in what order to fix it. Every finding is tied to evidence on
the page, not generic CRO folklore.

This skill audits and prioritizes. It does not rewrite the page — if you need revised copy from
a finding, call `landing-product-page-copy-writer` or `cta-variant-generator` for that section.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, voice, offer mechanics, real
  proof, and positioning so every finding is judged against the right audience and message, not
  a generic visitor.
- **`cta-variant-generator`** — called on request when a CTA failure is the top finding and the
  user wants a drop-in fix, not just a diagnosis.
- **`landing-product-page-copy-writer`** — call when the audit reveals structural copy rewrites
  are needed beyond a single CTA or headline.
- **`proof-vault`** — call when trust/proof is the primary leak and the brand needs stronger
  social-proof assets surfaced.
- **`a-b-multivariate-test-designer`** — call to formalize any high-priority finding into a
  testable experiment brief.

---

## How a run works

```
Step 0  Load the brand    ──► call brand-brain; get ICP, offer, proof, voice, positioning
Step 1  Fetch the page    ──► URL: WebFetch + screenshot; image: read directly; paste: accept
Step 2  Score             ──► seven heuristic dimensions, 0–10 each, evidence-cited per score
Step 3  Rank the leaks    ──► P1/P2/P3 by conversion impact × effort; 3–5 actionable fixes
Step 4  Present + offer   ──► scorecard + ranked fix list; offer to save report or chain tools
```

### Step 0 — Brand context (always first)

**Invoke `brand-brain`** before reading a single element. It returns the ICP (awareness stage,
pain vocabulary, commitment ceiling), the brand's real proof points, the offer mechanics and
primary destination URLs, and voice/banned-words. Every heuristic score is calibrated against
*this* audience, not a generic B2B or B2C baseline. If brand-brain is not installed, read
`~/.brandbrain/brands/.active` and that brand's `brand.md`; if none, ask the user for ICP,
offer, and key proof before proceeding.

### Step 1 — Fetch the page

- **URL provided:** use WebFetch to pull the live HTML; take a screenshot if the tool supports
  it. Note above-the-fold content, hero, nav, CTA placement, social proof, form, footer.
- **Screenshot/image provided:** read it directly — annotate visible sections.
- **Pasted HTML/copy:** accept and parse.

Always note the URL and timestamp in the audit header so the report is replayable.

### Step 2 — Score against the seven heuristics

Score each dimension 0–10. Every score requires **one piece of on-page evidence** — quote the
actual headline, describe the actual CTA button, note what proof is (or isn't) visible. No
score without evidence. Unverified claims or inferred off-page context → `[verify]`.

### Step 3 — Rank conversion leaks

After scoring, identify the 3–5 highest-leverage issues. Rank by **impact × ease**: a score of
≤5 on a high-weight dimension beats a score of 3 on a low-weight one. Assign each a priority
tier: P1 (fix this week), P2 (next sprint), P3 (nice-to-have).

---

## The seven heuristics (our Clarity-Trust-Friction-Urgency working model, extended)

This skill uses an opinionated, named-here checklist rather than a generic one — it is our own
house synthesis, not an externally established framework. The seven dimensions draw directionally
on the MECLABS Conversion Heuristic, Flint McGlaughlin's message-match model, and BJ Fogg's
Behavior Model (motivation × ability). Weights reflect their typical share of conversion impact.

| # | Dimension | Weight | What it tests |
|---|---|---|---|
| 1 | **Message match** | ×2 | Does the hero headline mirror the ad/email/search intent that sent the visitor here? Does above-the-fold copy resolve the question "am I in the right place?" within 5 seconds? |
| 2 | **Value clarity** | ×2 | Is the primary value proposition specific, differentiated, and immediately legible — or vague/generic? Can a stranger name what they get and why it beats alternatives? |
| 3 | **CTA strength & placement** | ×1.5 | Primary CTA: visible above the fold? Action-led, specific, matched to the awareness stage? Commitment appropriate for a cold visitor? Secondary CTAs competing? |
| 4 | **Trust & proof** | ×1.5 | Social proof (logos, reviews, case-study numbers) present and specific? Proof calibrated to the ICP (enterprise logos ≠ SMB testimonials)? Risk-reducers (guarantee, no-card, cancel-any-time) near the CTA? |
| 5 | **Friction & form** | ×1 | Form field count appropriate for the offer's commitment level? Load speed / visual noise / nav distractions pulling the visitor off the conversion path? |
| 6 | **Urgency & relevance** | ×1 | Any legitimate scarcity or time relevance? Is it honest or manufactured? Offer feels current vs. stale? |
| 7 | **Mobile & accessibility** | ×0.5 | Primary CTA tappable above fold on mobile? Critical copy legible without zoom? (Structural — not a full WCAG audit.) |

**Weighted score formula:** `(Σ score × weight) / (Σ weights)` → expressed as a score out of 10.

---

## Scorecard output format

```
## Landing Page CRO Audit — [URL or description]
Audited: [date] | Brand: [slug] | ICP: [one-line from brand-brain]

### Dimension scores
| Dimension          | Score | Evidence (quoted from page)                          |
|--------------------|-------|------------------------------------------------------|
| Message match      |  /10  |                                                      |
| Value clarity      |  /10  |                                                      |
| CTA strength       |  /10  |                                                      |
| Trust & proof      |  /10  |                                                      |
| Friction & form    |  /10  |                                                      |
| Urgency            |  /10  |                                                      |
| Mobile/access.     |  /10  |                                                      |

**Weighted overall: X.X / 10**

---

### Ranked conversion leaks

**P1 — [Dimension: short title]**
Finding: [what the page does vs. what it should do for this ICP]
Evidence: "[exact quote or visible element]"
Why it matters: [mechanism — awareness mismatch / no proof / CTA too high-commitment / etc.]
Fix: [specific, actionable — not "improve your CTA"]
→ Chain: [cta-variant-generator / landing-product-page-copy-writer / a-b-multivariate-test-designer if applicable]

**P2 — ...**
**P3 — ...**

---

### What the page does well
[2–3 genuine strengths — every page has some. Do not omit.]
```

---

## Principles

- **Brand-brain first.** No score before the ICP and offer are loaded. A hero headline is
  "unclear" relative to a specific audience — not in the abstract.
- **Evidence per score.** Quote the page. Never score from memory of what landing pages
  "usually" do; score what is actually there.
- **Weighted severity beats count.** Three P1 message-match failures outrank ten P3 cosmetic
  issues. Rank by conversion impact, not by how easy it is to list things.
- **Real proof only.** If the page claims "trusted by 10,000 teams" and brand-brain does not
  confirm it, flag as `[verify]` — do not echo it as a strength.
- **Honest urgency, honest friction.** Call out fake countdown timers and dark patterns — they
  hurt brand trust and long-term conversion, not just ethics.
- **Strengths are required.** A credible audit names what is working. All-negative reports lose
  stakeholder trust and miss genuine anchors for the redesign.
- **Chain, don't duplicate.** When a fix requires new copy or a test plan, invoke the right
  sibling skill — do not rewrite the whole page inline.

---

## What not to do

- Don't score without page evidence — a number without a quote is an opinion.
- Don't apply B2C norms to a B2B page or vice versa; calibrate to the ICP from brand-brain.
- Don't generate 15 findings — depth on 3–5 high-impact issues beats a shallow checklist of 15.
- Don't call a low score on "urgency" a critical fix if the offer has none by design (SaaS
  free trial with no seat cap needs zero fake urgency).
- Don't run a full WCAG accessibility pass — that is out of scope; flag structural mobile issues
  only.
- Don't rewrite the page or sections inline unless the user explicitly asks and the fix is a
  single element; chain to `landing-product-page-copy-writer` otherwise.
- Don't manufacture proof or invent differentiators to fill the "strengths" section.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and ICP + offer loaded before any score?
- Every dimension has a score AND on-page evidence — no unsubstantiated number?
- Weighted overall score calculated correctly?
- Findings ranked P1/P2/P3 by conversion impact, not by volume?
- Each fix is specific and actionable (not "improve trust")?
- Strengths section present with genuine examples?
- Sibling skills chained for copy/test execution rather than duplicated inline?
- `[verify]` applied to any claim not confirmed by brand-brain or the page itself?
- Report offered to save to `./reports/cro-audit-[slug]-[date].md`?
