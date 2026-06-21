---
name: sales-asset-reviewer
description: >
  Reviews cold email drafts or sales decks against a structured critique framework — scoring
  personalization, clarity, CTA strength, and generic/unsupported claims, then providing
  line-by-line rewrites for every flagged element. Two modes: Email Review (cold or
  follow-up email draft → scored critique + rewritten version) and Deck Review (slide-by-slide
  audit of a sales deck → slide-level issues + recommended rewrites). Loads active brand context
  via brand-brain so every critique enforces voice, ICP fit, real proof, and offer accuracy.
  Output saves to ./outreach/ (emails) or ./sales-decks/ (decks). Use when the user says
  "review my cold email," "critique this sales deck," "score my outreach," "does this email
  land?," "why isn't my deck converting?," "tighten this pitch," "feedback on my sales copy,"
  or pastes a draft email or deck outline and asks whether it's good.
---

# Sales Asset Reviewer

A cold email or sales deck that goes out unreviewed is a silent revenue leak. This skill runs every sales asset through a structured, senior-eye critique before it touches a prospect — scoring personalization depth, message clarity, CTA fitness, and claim integrity, then rewriting the weak lines rather than just flagging them.

It reviews, rewrites, and coaches. It does not source prospects, build sequences from scratch, or redesign decks visually. If the underlying offer or ICP targeting is the root problem, it names that — it does not paper over strategic gaps with tactical polish.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, ICP, banned words, offer mechanics, and real proof. No critique begins without it.
- **`proof-vault`** — when the asset makes a specific claims that need verification or stronger proof; call to pull real validated proof points.
- **`objection-library-builder`** — when the asset fails to handle foreseeable objections or the reviewer needs to suggest a reframe.
- **`icp-persona-builder`** — when the ICP match is unclear or the email is targeting a vague segment; generates a persona to score against.
- **`cta-variant-generator`** — when the CTA needs a full rewrite rather than a one-line fix; hands off for stronger variants.
- **`account-dossier-builder`** — when an email claims personalization but lacks real account research; surfaces what real personalization would look like.
- **`cold-outreach-sequence-architect`** — when the reviewed asset is part of a longer sequence that needs a full rebuild.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain; get voice, ICP, banned words, offer, proof
Step 1  Detect mode     ──► Email Review | Deck Review (auto-detect or user-stated)
Step 2  Score the asset ──► PICCA scorecard (5 dimensions × 0–10)
Step 3  Critique        ──► Line-by-line flags with severity labels
Step 4  Rewrite         ──► Rewritten version of each flagged element + a clean full draft
Step 5  Save artifact   ──► ./outreach/ or ./sales-decks/ (if reusable)
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, ICP, positioning. **Do not produce any critique until it returns.**

Obey voice and banned-words as hard constraints. Use only returned real proof — any unconfirmed number in the asset under review is flagged `[verify]`. Check ICP fit against the returned ICP definition.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If neither exists, ask the user for brand slug, target ICP, offer mechanics, and 3 voice adjectives + banned words before proceeding.

---

## The PICCA Scorecard

Every asset gets a scored rubric before line-level critique begins. Score each dimension 0–10, show a summary table, then justify each score in one line.

| Dimension | What it measures | A score of 10 looks like |
|---|---|---|
| **P — Personalization** | Research depth; real account/person specifics vs. merge-tag fakery | Recipient's real trigger, genuine pain, or recent event tied to the pitch |
| **I — ICP Fit** | Match between audience and the brand's actual ICP | Role, industry, company stage, and pain map exactly to the ICP definition |
| **C — Clarity** | One clear premise; no buried lede; scannable in 15 sec | Subject/opener states the problem and the offer before sentence three |
| **C — CTA Strength** | Single, low-friction, next-step-sized ask matched to awareness stage | One ask; not a purchase, not a vague "let me know"; easy to say yes to |
| **A — Asset Integrity** | All claims real, specific, and sourceable; no generic superlatives | Every number cited; no "industry-leading," "best-in-class," or phantom proof |

Total: /50. Interpretation: 40–50 = send-ready (minor polish); 30–39 = rewrite flagged sections; below 30 = strategic rethink needed before any polish.

---

## Email Review mode

**Triggered when:** the asset is a cold email, follow-up email, LinkedIn DM, or any short-form outreach message.

### The critique pass

Work through the email in structural order:

1. **Subject line** — curiosity vs. clarity balance; personalization signal; spam-trigger words; length (≤50 chars for mobile preview).
2. **Opening line** — research quality (real trigger vs. template flattery); leads with them or with you?
3. **Problem / relevance frame** — does it name the specific pain this recipient plausibly has, or a generic industry pain? Are ICP signals used correctly?
4. **Offer / value bridge** — one clear "here's what I do and why it's relevant to you" statement; is the offer mechanics correct per brand?
5. **Proof / social proof** — any claim made here must trace back to `brand-brain`'s real proof; flag `[verify]` on anything unconfirmed; suggest `proof-vault` if stronger evidence is needed.
6. **CTA** — one ask; matches awareness stage (early-stage → low-commitment ask like a question or a 15-min call, not a demo with a 45-minute commitment); no double asks.
7. **Tone / voice** — banned words caught; voice adjectives honored; no filler phrases ("Hope this email finds you well," "I wanted to reach out," "Just following up").

### Severity labels

- `🔴 BLOCKS SEND` — the email should not go out until fixed (fake personalization, fabricated proof, wrong CTA commitment, ICP mismatch at a fundamental level).
- `🟡 WEAKENS PERFORMANCE` — won't kill the send but measurably hurts reply rate (generic opener, vague CTA, unsupported claim, passive construction).
- `🟢 POLISH` — stylistic improvements; ship as-is or apply before a high-value send.

### Output format

```
## Sales Asset Review — [asset title or first 6 words]
**Mode:** Email Review
**Brand:** [slug, from brand-brain]
**ICP match:** [Confirmed / Partial / Mismatch — one line]

### PICCA Scorecard
| P | I | C | C | A | Total |
|---|---|---|---|---|-------|
| _ | _ | _ | _ | _ | _/50  |

### Critique
[Structural section by section — flag, severity label, one-line rationale]

### Rewrites
[Before → After for every 🔴 and 🟡 flag; grouped by section]

### Clean rewrite
[Full email, clean, no annotations]

### One-line coaching note
[The single biggest lever for this sender's next email]
```

---

## Deck Review mode

**Triggered when:** the asset is a sales deck, pitch deck, leave-behind PDF, or slide-by-slide outline.

### The critique pass

Work slide by slide (or section by section for outlines). For each slide, evaluate:

- **Headline clarity** — does the slide title make a claim or just label a category? Claims convert; labels don't. ("We reduced churn 30%" > "Our Results")
- **Audience relevance** — is this slide earning attention from this ICP, or is it internal self-congratulation?
- **Proof quality** — every metric or customer name checked against real proof from `brand-brain`; unconfirmed items flagged `[verify]`.
- **Talk-track alignment** — the visual and the verbal tell the same story; no slide that needs three minutes of narration to make sense.
- **Flow and momentum** — does the sequence Problem → Stakes → Insight → Solution → Proof → Ask hold? Flag any slide that breaks momentum.

Use the same severity labels (🔴 / 🟡 / 🟢) applied per slide.

### Output format

```
## Sales Asset Review — [deck title]
**Mode:** Deck Review
**Brand:** [slug, from brand-brain]
**ICP match:** [Confirmed / Partial / Mismatch — one line]

### PICCA Scorecard
[Same table; P is scored on how well the deck is tailored, not on merge fields]

### Slide-by-slide critique
[Slide N — title — flag(s) with severity + rationale]

### Rewrites
[Slide headlines, bullet rewrites, proof substitutions for every 🔴 and 🟡]

### Structural note
[One paragraph on flow: is the narrative arc Problem → Stakes → Insight → Solution → Proof → Ask intact?]

### One-line coaching note
[The single highest-leverage change for this deck]
```

Save deck reviews to `./sales-decks/[slug]-[deck-name]-review.md`. Save email reviews to `./outreach/[slug]-[prospect-or-tag]-review.md`. Never save to the skill folder or to `brand.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No critique begins before the active brand is loaded. Its voice, ICP, and proof are the evaluation baseline — not generic "best practices."
- **Rewrite, don't just flag.** Every 🔴 and 🟡 issue gets a concrete rewrite. Flagging without fixing is lazy review.
- **Honest proof only.** If the asset claims something unconfirmed, flag it `[verify]`. Never suggest substituting a different invented number.
- **Severity is binary at the top.** A BLOCKS SEND flag is not a suggestion — the asset does not ship until it's resolved.
- **Personalization fakery is always 🔴.** "I noticed you're in [industry]" is not personalization; it is a merge-tag dressed up. Call it out every time.
- **One CTA per asset.** Multiple asks in a single email or a deck that closes with five next steps both get flagged.
- **Structural issues before stylistic ones.** If the ICP is wrong or the offer is buried, fix that first. Polish is last.

## What Not to Do

- Don't produce a critique before `brand-brain` returns.
- Don't invent alternative proof points — pull from `proof-vault` or mark `[verify]`.
- Don't redesign slides visually or create sequences from scratch — delegate to `cold-outreach-sequence-architect`.
- Don't let generic superlatives ("industry-leading," "world-class," "best-in-class") survive a review without either a specific claim replacing them or a `[verify]` flag.
- Don't confuse template personalization ("Hi {FirstName}, I saw you work in SaaS") with real personalization (a genuine trigger from account research).
- Don't give a pass to a weak CTA because the rest of the email is strong — a muddled ask kills even a well-written email.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any critique began?
- PICCA table complete with a one-line justification per score?
- Every 🔴 and 🟡 flag has a concrete rewrite (not just a comment)?
- Clean full-email rewrite or structural note present (not just annotation)?
- All proof claims checked against brand-brain's real proof; `[verify]` applied to anything unconfirmed?
- Banned words and voice adjectives honored in every rewritten line?
- Artifact saved to correct project path (`./outreach/` or `./sales-decks/`) when a reusable deliverable was produced?
- One-line coaching note present — the single highest-leverage takeaway for this sender?
