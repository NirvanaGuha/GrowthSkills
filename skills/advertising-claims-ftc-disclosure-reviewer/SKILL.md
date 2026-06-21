---
name: advertising-claims-ftc-disclosure-reviewer
description: >
  Reviews marketing copy — ads, landing pages, email, social posts, influencer briefs, product
  listings, push notifications, and press releases — for FTC compliance risk before it ships.
  Flags three distinct risk categories: (1) substantiation gaps on performance/superiority claims
  ("best," "#1," "up to X%," "clinically proven"), (2) missing or inadequate #ad / paid-partnership
  disclosures for influencer and sponsored content, and (3) deceptive-pricing, urgency, and
  endorsement violations under FTC's 2023 Guides. Every flag cites the actual regulation or FTC
  guidance section, not generic advice. Returns a prioritized fix list — Critical (mandatory
  before publish), High (strong risk), and Advisory (best practice) — plus corrected copy where
  appropriate. Does NOT rewrite entire campaigns; it reviews and redlines. Use whenever the user
  says "review my ad copy for FTC," "check these claims," "do we need a disclaimer," "is this
  #1 claim legal," "influencer disclosure review," "check my endorsement," "substantiation check,"
  or hands over any marketing copy and asks if it's legally safe to publish.
---

# Advertising Claims & FTC Disclosure Reviewer

Send copy in, get a prioritized compliance flag list out — with the actual rule cited, a severity rating, and a redline fix. The goal is to catch the three most common FTC exposure vectors before legal sees the campaign, not after.

This skill reviews and redlines. It does not rewrite campaigns, give legal advice (always recommend counsel for material risk), or substitute for a full legal review on regulated categories (supplements, financial products, health claims). It flags, explains the rule, proposes a fix, and hands back a clean version of the copy at issue.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, proof points, banned words, and offer mechanics. Substantiation review depends on knowing what proof actually exists for a claim; brand-brain is the source. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (a) product/service category, (b) any existing proof or studies for the claims in the copy, and (c) any known brand restrictions on claim types, before proceeding.
- **`proof-vault`** *(if installed)* — cross-references claims against documented proof points. Synthesize inline from brand-brain's returned proof if absent.
- **`consent-privacy-compliance-auditor`** *(if installed)* — for copy with data-collection or cookie-consent angles; run in parallel when the copy includes a lead capture element.
- **`email-compliance-auditor-gdpr-can-spam`** *(if installed)* — for email copy; invoke to cover CAN-SPAM/GDPR alongside FTC claims in one pass.

---

## How a run works

```
Step 0  Load the brand + proof  ──► call brand-brain; note real proof points returned
Step 1  Classify the input       ──► single asset | batch | influencer brief | full campaign
Step 2  Run the three-track scan ──► Substantiation · Disclosure · Deceptive-Practice
Step 3  Rate + cite + redline    ──► Critical / High / Advisory; regulation cite; fix
Step 4  Self-check               ──► no invented rules; every flag has a citation
Step 5  Output the review        ──► flag table + redlines + publish decision
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest — real proof points, offer mechanics, positioning, voice adjectives, banned words, ICP. The proof-point list is essential: a claim that has documented backing in brand-brain is treated differently from one that doesn't.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (a) product/service category, (b) any existing proof or studies for the claims in the copy, and (c) any known brand restrictions on claim types, before proceeding.

Do not produce any compliance output before brand context is loaded.

### Step 1 — Classify the input

| Input type | Notes |
|---|---|
| Single asset (email, ad, LP, post) | Run all three tracks; output one flag table |
| Batch (multiple assets or a campaign) | Run per-asset; share a rollup severity summary |
| Influencer brief | Focus on Track 2 (Disclosure) and Track 1 (any performance claims seeded in the brief) |
| Full campaign | Run tracks per asset; produce a campaign-level risk summary at the end |

---

## The Three-Track Scan

### Track 1 — Substantiation (FTC's "Reasonable Basis" Standard)

**The rule:** The FTC requires that objective claims — express or implied — be substantiated by *competent and reliable evidence* before the claim is made. For health/safety claims, that standard is generally "competent and reliable scientific evidence." For performance claims in other categories, it's the evidence a reasonable expert in the field would require. *FTC Policy Statement on Deception (1983); FTC Policy Statement Regarding Advertising Substantiation (1984); FTC Act § 5.*

**Flag these claim patterns:**

| Pattern | What to check |
|---|---|
| Superlatives: "best," "#1," "the only," "world's leading" | Is there an independently verified basis? A survey? A third-party ranking? If self-declared or no source, flag Critical unless clearly labeled as opinion. |
| Quantified performance: "up to X%," "saves $Y," "X× faster," "increases revenue by Z" | Is there a real study or internal data behind it? "Up to" framing reduces (but does not eliminate) risk. Flag if no source or if the "up to" represents an outlier. |
| "Proven," "clinically proven," "scientifically shown," "doctor-recommended" | Requires actual clinical or scientific study for the specific product. Flag Critical for health/supplement categories; High elsewhere. |
| Testimonial-based claims: "I lost 20 lbs in 30 days" | FTC's 2023 Guides update (16 C.F.R. Part 255, effective Sept 2023): atypical results must be accompanied by a clear-and-conspicuous disclosure of typical results OR proof that the testimonial is representative. "Results may vary" alone is no longer sufficient. |
| Free trial / "no risk" | FTC's 2023 Negative Option Rule update: requires unambiguous disclosure of all recurring charges, cancellation terms, and method — before the consumer incurs the obligation. |
| Comparison claims: "better than [competitor]" | Requires substantiation; puffery defense weakens when specific competitor or metric is named. |

**Output format per Track 1 flag:**
```
CLAIM: [exact text]
SEVERITY: Critical | High | Advisory
RULE: [FTC citation or guidance section]
RISK: [one-sentence plain-English risk description]
FIX: [redlined copy or disclosure addition]
```

---

### Track 2 — Influencer & Sponsored Content Disclosures

**The rule:** FTC's Endorsement Guides (16 C.F.R. Part 255), revised substantially in 2023 — the most significant update since 2009. Key requirements:

- Any **material connection** between the endorser and the brand must be **clearly and conspicuously** disclosed. Material connections include payment, free product, family relationships, employment, and equity.
- **"Clear and conspicuous"** means (a) unavoidable — not buried in a sea of hashtags; (b) platform-appropriate — not hidden in "more" on Instagram; and (c) presented *before* engagement (the reader/viewer should see the disclosure before the substantive claims).
- **Social media specifics:** `#ad` or `#sponsored` placed as the first or second hashtag satisfies the requirement on most platforms. `#sp`, `#collab`, `#partner`, `#thanks` alone do *not*. A disclosure inside a story tap-through or after 5 other hashtags is not "clear and conspicuous."
- **Video:** a verbal disclosure at the start of the video AND an on-screen superimposed text disclosure are both recommended; verbal-only is risky. YouTube's built-in "paid promotion" toggle does *not* eliminate the need for in-content disclosure.
- **Reviews/endorsements in ads:** FTC now explicitly prohibits creating or disseminating fake reviews, incentivizing positive reviews without disclosure, and suppressing negative reviews. *FTC Rule on the Use of Consumer Reviews (effective Aug 2024).*
- **AI-generated endorsements:** FTC guidance (May 2023 blog + ongoing enforcement) warns that AI-generated testimonials or fake personas are deceptive if presented as real consumers.

**Flag these patterns:**

| Pattern | Flag |
|---|---|
| Influencer copy/brief with no disclosure instruction | Critical — add required disclosure language to the brief |
| Disclosure present but buried (≥3rd hashtag, after "more," inside caption body after long text) | High — must be first or second hashtag or precede the body text |
| `#sp`, `#collab`, `#partner`, `#thanks` only | High — add `#ad` or `#sponsored` |
| Review-generation campaign with incentive and no disclosure instruction | Critical — FTC Aug 2024 Rule |
| Testimonial quotes in ad copy presented as organic without a disclosure | High |

---

### Track 3 — Deceptive Pricing, Urgency & Dark Patterns

**The rule:** FTC Act § 5 (unfair or deceptive acts); FTC's Enforcement Policy Statement on Deceptively Formatted Advertisements; FTC's 2022 "Bringing Dark Patterns to Light" report.

| Pattern | Flag level | Rule |
|---|---|---|
| Fake urgency ("Only 3 left!" when inventory is not actually limited; countdown timer that resets) | Critical | FTC Act § 5; 2022 dark patterns report |
| Strikethrough "was" price that was never the actual selling price | Critical | FTC's Guides Against Deceptive Pricing (16 C.F.R. Part 233) |
| Drip pricing (headline price excludes mandatory fees revealed only at checkout) | Critical | FTC's ongoing enforcement; FTC Act § 5 |
| Free trial framing that obscures auto-renewal | Critical | FTC's Negative Option Rule (2023 update) |
| Pre-checked opt-ins for recurring subscriptions | Critical | Negative Option Rule |
| "Award-winning" or "as seen in [publication]" without the award/publication being real | High | FTC Act § 5 substantiation |
| Social proof numbers that are not current or are inflated ("trusted by 10,000+ businesses" when actual number is lower) | High | FTC Act § 5 |

---

## Output format

For each reviewed asset, produce:

```
### [Asset name / type]

PUBLISH DECISION: ✓ Publish as-is | ⚠ Publish with fixes | ✗ Do not publish until Critical items resolved

| # | Track | Severity | Claim / Element | Rule | Risk | Fix |
|---|---|---|---|---|---|---|
| 1 | Substantiation | Critical | "…" | FTC Act §5 / 1984 Substantiation Policy | … | … |
| 2 | Disclosure | High | "#sp" only | 16 C.F.R. § 255.5 (2023) | … | … |
…

REDLINES:
[Original text] → [Fixed text]
…

NOTES FOR COUNSEL:
[Any item requiring regulated-category legal review; regulated categories: health, supplements,
 financial products, children's advertising (COPPA), environmental claims ("green," "sustainable")]
```

Save to `./compliance/[slug]-ftc-review.md` when the user requests a record or the batch is ≥3 assets.

---

## The framework: FTC's Three-Part Deception Test

Every flag in this skill traces to one of three legs of the FTC's own deception standard (*FTC Policy Statement on Deception, 1983*):

1. **Representation, omission, or practice** — is there a claim (explicit or implied) or a missing disclosure?
2. **Likely to mislead a reasonable consumer** — would a typical member of the target audience be misled?
3. **Material** — would the claim or omission affect a purchase decision?

When all three legs are present, the deception test is met and the risk is Critical or High. When leg 2 or 3 is ambiguous (e.g., obvious puffery, clearly satirical), flag as Advisory.

---

## Regulated categories requiring elevated caution

These categories carry mandatory heightened substantiation or additional regulatory layers beyond the FTC. Always add a "Notes for Counsel" flag:

- **Health / dietary supplements / medical devices** — FTC Act + FDA (21 C.F.R.); "clinically proven" requires RCTs or equivalent; structure/function claims must have a reasonable basis.
- **Financial products / investments** — FTC + SEC / CFPB; "guaranteed returns," "risk-free" are almost always problematic.
- **Children's advertising** — COPPA (16 C.F.R. Part 312); heightened substantiation; no data collection from under-13 without verifiable parental consent.
- **Environmental / sustainability claims** — FTC Green Guides (16 C.F.R. Part 260, 2024 update in progress [verify status]); "eco-friendly," "carbon neutral," "sustainable" require specific factual bases.
- **Real estate / franchise / biz-opp earnings claims** — FTC's Business Opportunity Rule (16 C.F.R. Part 437).

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Proof already in the brand's brain is treated as substantiation; claims with no documented proof are flagged, not assumed valid.
- **Cite the actual rule.** Never say "this might violate FTC rules." Name the regulation, part, or guidance document.
- **Severity is not opinion.** Critical = a well-documented FTC enforcement basis exists. High = clear rule violation, lower but real enforcement probability. Advisory = best practice or emerging enforcement trend.
- **Truth discipline.** If the FTC guidance is uncertain or the rule is actively evolving (e.g., AI endorsements, environmental claims), say so and flag `[verify current enforcement posture]`.
- **Not a lawyer.** This skill is not legal advice. Regulated categories and Critical findings should always go to counsel.
- **Do not suppress.** Flag everything that meets the deception test. Never soften a Critical to an Advisory because the copy is otherwise on-brand.

---

## What Not to Do

- Don't invent FTC regulations — every flag must cite a real rule, guidance, or enforcement action.
- Don't clear a claim as safe just because it's common in the industry (industry practice is not an FTC defense).
- Don't treat brand-brain proof points as automatically sufficient substantiation — check whether the proof matches the specific claim being made.
- Don't rewrite the entire campaign — flag and redline the specific language at issue; leave the rest.
- Don't skip the "Notes for Counsel" block for regulated categories, even if no Critical flags are present.
- Don't confuse puffery (non-actionable opinion: "the best coffee you'll ever taste") with an objective superiority claim ("rated #1 by coffee drinkers") — flag the latter, skip the former.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand's proof points loaded before any review?
- Every flag cites a specific FTC rule, guidance document, or enforcement action — no vague "may violate FTC"?
- All three tracks run: Substantiation, Disclosure, Deceptive Practice?
- Severity assigned per the deception-test framework, not subjectively?
- Testimonial and influencer copy checked against 2023 Guides update (atypical results + material connection)?
- Regulated categories (health, financial, children's, environmental) flagged for counsel even if no Critical items?
- Redline fixes provided for every Critical and High item?
- Publish decision (✓ / ⚠ / ✗) stated at the top of each asset's review?
- Output saved to `./compliance/` if batch ≥3 or user requested a record?
