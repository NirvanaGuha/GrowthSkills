---
name: ad-to-landing-page-message-match-auditor
description: >
  Takes live ad copy (headline, body, CTA, visual description) plus a landing page URL or pasted
  page copy and returns a structured message-match audit scored across four dimensions: headline
  continuity, offer/value-prop alignment, CTA chain coherence, and visual/tonal congruence.
  Surfaces every gap where the ad makes a promise the page breaks — and every friction point where
  a mismatched frame causes a visitor to question whether they landed in the right place. Produces
  an overall match score (0–100), a ranked fix list with effort/impact ratings, and a rewritten
  headline pair showing the corrected ad↔page hook if the score is below threshold. Built on the
  established CRO concept of message match (popularized primarily by Unbounce): scanned promise →
  page confirmation → action alignment, scored through our own four-pillar working rubric. Composes
  `cta-variant-generator` for CTA fixes, `landing-page-heuristic-live-cro-auditor`
  for full-page heuristic issues, `analytics-report-reviewer` when performance data is attached,
  and `data-qa-measurement-gotcha-checker` as a data-quality gate when the user pastes metrics.
  Invoke when the user says "ad doesn't match landing page," "message match audit," "ad to LP
  alignment," "why is my ad CTR high but conversion low," "check my landing page vs ad," "match
  my ad to my page," "scent trail audit," or pastes ad copy alongside a URL and asks what's wrong.
---

# Ad-to-Landing Page Message-Match Auditor

High click-through, low conversion is almost always a broken promise. The visitor clicked because the ad made a claim — specific headline, specific offer, specific tone — and the landing page either buried it, changed it, or forgot it entirely. This audit finds every break in that scent trail and tells you exactly what to fix, in what order.

Message match is an established CRO concept (popularized primarily by Unbounce): the promise a visitor clicked must be confirmed on the page they land on. Our scoring engine, the **Message Match Matrix**, is a house working rubric that operationalizes that concept — every ad fires four implicit promises (the headline hook, the offer/value-prop, the CTA commitment level, and the visual/tonal frame), and each must be confirmed on the page within the first viewport. Miss one and you're paying for curiosity you're not converting.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, offer, ICP, banned words, and real proof before the audit begins. The auditor does not implement brand resolution, scanning, or interviewing; that lives in `brand-brain`, once. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the brand name, target ICP, primary offer mechanics, and known voice adjectives before proceeding.
- **`cta-variant-generator`** — called when the audit finds a broken CTA chain; generates corrected ad CTA ↔ page CTA pairs rather than rewriting them inline.
- **`landing-page-heuristic-live-cro-auditor`** — call (or surface as next step) when the page has structural CRO issues beyond message match; this skill audits the ad↔page seam only.
- **`data-qa-measurement-gotcha-checker`** — call before interpreting any attached performance data (CTR, CVR, ROAS) to gate against GA4 attribution, sampling, and (not set) issues.
- **`analytics-report-reviewer`** — call when the user attaches a performance report alongside the audit, to gate weak assumptions before acting on the numbers.

---

## How a run works

```
Step 0  Load the brand         ──► brand-brain (voice, offer, ICP, real proof)
Step 1  Ingest inputs          ──► ad copy + landing page (URL or paste)
Step 2  Score the four pillars ──► Message Match Matrix
Step 3  Rank the fix list      ──► effort × impact
Step 4  Rewrite if needed      ──► headline pair + CTA fix (via cta-variant-generator)
Step 5  Output the audit card
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics, real proof, ICP, and awareness tendency. Do not begin scoring until it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the brand name, target ICP, primary offer mechanics, and known voice adjectives before proceeding.

Use the brand digest to calibrate: banned words in the ad are a separate flag; offer claims that contradict the canonical offer get a `[verify]` tag; ICP awareness stage informs whether a mismatch is a stage-jump or just a copy gap.

### Step 1 — Ingest inputs

Collect:
- **Ad copy**: headline(s), body, CTA button text, and any visual description or image URL
- **Landing page**: URL (fetch and read) or pasted copy — capture: H1, hero subhead, primary CTA button text, above-the-fold offer statement, and trust elements

If the user provides performance data (CTR, CVR, ROAS), invoke `data-qa-measurement-gotcha-checker` before factoring any numbers into findings. Tainted data gets a `[data-gate: see gotcha-checker output]` flag.

### Step 2 — Score the four pillars

Score each pillar 0–25 (integer). Sum = overall match score (0–100). A score below 70 is a conversion risk; below 50 is a leak.

---

## The Message Match Matrix

### Pillar 1 — Headline Continuity (0–25)

The ad headline makes a specific claim. The page H1 (or first bold statement in viewport) must confirm — not echo verbatim, but land the same specific promise at the same specificity level.

| Signal | Points off |
|---|---|
| H1 confirms the ad's core claim at equal or greater specificity | 0 |
| H1 confirms the category but loses the specific angle | −5 |
| H1 is brand/generic — claim buried below the fold | −10 |
| H1 contradicts or reframes the ad's angle entirely | −20 |
| No H1 present or dynamically empty | −25 |

**Common failure**: ad says "Cut cart abandonment by 30%" → page says "The #1 Push Notification Platform." The claim demoted to a bullet three scrolls down is not a confirmed headline.

### Pillar 2 — Offer / Value-Prop Alignment (0–25)

Every specific offer element named in the ad (free trial, price, guarantee, feature name, discount %) must appear on the page in the same framing, equal or better. A stronger page offer is fine; a weaker or absent one is a break.

| Signal | Points off |
|---|---|
| Offer matches on specifics, prominently placed above fold | 0 |
| Offer present but less specific (e.g., "free" vs. "free 14-day trial, no card") | −5 |
| Offer present but below fold, requiring scroll to confirm | −8 |
| Offer absent from page (visitor must hunt) | −15 |
| Offer contradicts the ad (different price, expired promo, different terms) | −25 |

### Pillar 3 — CTA Chain Coherence (0–25)

The ad CTA sets a commitment level. The page's primary button must honor that level — same action, same or lower friction, no bait-and-switch.

| Signal | Points off |
|---|---|
| Page CTA mirrors ad CTA in action and commitment | 0 |
| Page CTA is a synonym (fine) but loses specificity ("Get started" vs. "Start free trial") | −5 |
| Page CTA asks for higher commitment than ad promised ("Book a demo" when ad said "Try free") | −15 |
| Page CTA is generic or missing entirely | −20 |

When Pillar 3 flags a gap, compose `cta-variant-generator` to produce a corrected pair rather than rewriting inline.

### Pillar 4 — Visual / Tonal Congruence (0–25)

Tone, visual style, and urgency frame must be consistent. A punchy, urgency-driven ad that lands on a calm, enterprise-formal page creates cognitive dissonance even when the copy technically matches. Score based on user's description or a fetched screenshot.

| Signal | Points off |
|---|---|
| Tone, urgency, and visual style consistent between ad and page | 0 |
| Minor tone shift (slightly more formal on page — acceptable) | −3 |
| Noticeable urgency drop (ad drives scarcity, page is evergreen) | −8 |
| Complete mismatch: brand voice, color register, or emotional register flips | −15 |
| Evidence of a generic template page serving multiple ad campaigns | −20 |

---

## Step 3 — Ranked fix list

For each gap identified, produce a fix entry:

```
Fix [n]: [pillar] — [what's broken in one line]
  Impact: High / Medium / Low  (conversion impact)
  Effort: Low / Medium / High  (implementation effort)
  Fix: [specific action — rewrite H1 to X, add offer statement above fold, etc.]
```

Sort by Impact desc, then Effort asc (high impact + low effort first). Fixes requiring a full page rebuild go last.

## Step 4 — Rewrite if score < 70

When the overall score is below 70, produce:

1. **Corrected headline pair**: revised ad headline → revised page H1 (same specific angle, confirmed)
2. **CTA chain fix**: invoke `cta-variant-generator` for the corrected ad CTA ↔ page button pair

Mark any rewritten claims that go beyond the brand's confirmed proof as `[verify before publishing]`.

## Step 5 — Audit card output

```
## Message Match Audit — [Brand] — [Ad Name / Campaign]

Overall score: [0–100] ([pass ≥70 / risk 50–69 / leak <50])
Brand: [slug, via brand-brain]
Ad: [headline summary]
Page: [URL or title]

### Pillar scores
| Pillar | Score | Status |
|---|---|---|
| Headline Continuity | /25 | ✓ / ⚠ / ✗ |
| Offer / Value-Prop Alignment | /25 | |
| CTA Chain Coherence | /25 | |
| Visual / Tonal Congruence | /25 | |
| **Total** | **/100** | |

### Ranked fix list
[Fix entries, sorted effort × impact]

### Rewrites (if score < 70)
[Headline pair + CTA fix]

### Data quality note (if metrics attached)
[data-qa-measurement-gotcha-checker output summary or flag]

### Next steps
- [if structural CRO issues found]: invoke `landing-page-heuristic-live-cro-auditor` for full page analysis
- [if offer unclear]: invoke `offer-pricing-brain` to confirm canonical offer
- [if brand voice flags]: invoke `brand-brain refresh` to update the brand digest
```

Save audit output to `./audits/message-match-[brand-slug]-[YYYY-MM-DD].md` when the user asks to save or when running in batch.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit begins until `brand-brain` returns the active brand digest. Voice, offer mechanics, and ICP awareness stage all calibrate the scoring.
- **Scent trail, not semantic match.** The goal is the visitor's felt continuity — not keyword repetition. (The "scent trail" idea comes from information-foraging research — Pirolli & Card — and was carried into conversion work by Bryan Eisenberg.) Same promise at same specificity; verbatim copying is not required.
- **Score granularly, fix precisely.** A vague "headline mismatch" is useless. Name the specific claim that broke and the specific fix that restores it.
- **Compose, don't duplicate.** CTA fixes go through `cta-variant-generator`. Full-page CRO issues surface `landing-page-heuristic-live-cro-auditor`. Data quality gates go through `data-qa-measurement-gotcha-checker`. Don't rebuild those here.
- **Truth discipline.** Never invent proof or performance claims. Rewritten copy uses only brand's confirmed proof; anything else is `[verify]`.
- **Effort × impact ordering.** A fix that takes one hour and lifts CVR 20% must rank above a fix that takes a week and lifts 5%. Always sort the fix list.

## What Not to Do

- Don't audit the full landing page for general CRO — that's `landing-page-heuristic-live-cro-auditor`. Scope to the ad↔page seam only.
- Don't reimplement brand resolution, voice scanning, or offer confirmation — call `brand-brain` and `offer-pricing-brain`.
- Don't generate CTA variants inline when Pillar 3 fails — compose `cta-variant-generator`.
- Don't interpret attached performance data without running `data-qa-measurement-gotcha-checker` first.
- Don't award partial credit when the confirmed offer is genuinely missing above the fold — that is a full Pillar 2 failure regardless of where it appears on the page.
- Don't suggest "use dynamic keyword insertion" as the default fix — it is one option; structural copy rewrites often have higher lift.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and digest loaded before any scoring?
- All four pillars scored with specific evidence (not just vibes)?
- Each fix entry names the specific gap and specific action (not "improve headline")?
- Fix list sorted by impact desc, effort asc?
- Rewrites (if score < 70) present: corrected headline pair + CTA chain fix via `cta-variant-generator`?
- Any attached metrics gated through `data-qa-measurement-gotcha-checker` before factoring in?
- Claims in rewrites limited to confirmed brand proof or tagged `[verify]`?
- Audit card saved to `./audits/` if user requested or batch mode?
