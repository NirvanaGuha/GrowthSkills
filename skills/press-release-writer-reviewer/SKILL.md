---
name: press-release-writer-reviewer
description: >
  Writes and peer-reviews press releases for any B2B or B2C announcement — product launches,
  funding, partnerships, executive hires, milestones, crisis updates, or market data drops.
  Two modes: WRITE produces a publication-ready release from an announcement brief (quotes,
  boilerplate, facts); REVIEW takes a drafted release and returns an AP-style critique with
  line-level edits, a wire-formatting pass, and a distribution readiness score. Both modes
  load brand context from the `brand-brain` skill first and compose with the
  `press-release-social-blog-amplification-pack` skill for downstream amplification.
  Built on the AP Stylebook plus the standard inverted-pyramid wire-release structure so a junior
  marketer produces a release an editor would accept. Trigger phrases: "write a press
  release," "draft a PR," "review / critique this press release," "AP style pass,"
  "wire distribution formatting," "announcement copy," "format for PR Newswire / Business
  Wire / GlobeNewswire," "format the boilerplate," "check my press release."
---

# Press Release Writer & Reviewer

Turn an announcement brief into wire-ready copy. Bring a draft, get an AP-clean critique. Either way, the release is built on real facts, brand voice, and the structural conventions editors expect — not a bloated template filled with superlatives.

This skill writes and reviews. It does not manage amplification (social, email, blog) — that handoff lives in `press-release-social-blog-amplification-pack`.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, positioning, boilerplate, proof points, banned words. Never reinvent brand scanning here.
- **`press-release-social-blog-amplification-pack`** (optional, on request) — hands the finished release off for social/email/blog amplification after WRITE mode completes.
- **`proof-vault`** (optional) — surfaces verified proof points for stats, customer quotes, and differentiator claims if installed.
- **`competitive-intelligence-dossier`** (optional) — referenced for market context claims if available.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (always first)
Step 1  Pick the mode           ──► WRITE (default) | REVIEW
Step 2  Do the work (framework below)
Step 3  Self-review checklist
Step 4  Offer amplification handoff (WRITE mode only)
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (`skill: brand-brain`). It returns: voice adjectives, banned words, positioning, ICP, real proof, standard boilerplate (About paragraph), and the path to `brand.md`. Do not write a single word of the release before it returns.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none, ask for (1) company name and one-line positioning, (2) the boilerplate About paragraph, (3) three voice adjectives and banned words, then proceed.

---

## WRITE mode (default)

Triggered when the user provides an announcement brief — or asks to "write" / "draft" the release.

### Minimum inputs to collect before writing

| Required | If missing |
|---|---|
| Announcement type (launch, funding, hire, partnership, data, milestone) | Ask |
| Headline subject + the core news (one sentence) | Ask |
| Embargo date / publish date | Assume IMMEDIATE if not given |
| At least one attributed executive or spokesperson quote | Ask; flag as `[quote needed]` if not provided |
| Key fact(s): number, date, product name, deal size, customer name, stat | Ask; `[verify]` anything unconfirmed |
| Target wire(s): PR Newswire, Business Wire, GlobeNewswire, or none | Ask if distribution is mentioned |

### The AP/Industry-Standard Press Release Structure

Every release follows this skeleton — no exceptions, no creative reordering:

```
FOR IMMEDIATE RELEASE   (or: EMBARGOED UNTIL [Date, Time TZ])

[CITY, State, Month DD, YYYY] —

HEADLINE: Active verb · brand name · core news · ≤ 170 chars
SUBHEADLINE (optional): supporting detail · ≤ 120 chars

LEAD PARAGRAPH (≤ 60 words)
  Who, what, when, where, why — the entire news in one paragraph.
  No adjectives. No "excited to announce." The lead is the story.

BODY (2–3 paragraphs)
  ¶2 — Why it matters: market context, the problem it solves, one
        proof point (stat, customer name, metric). One source per claim.
  ¶3 — How it works / product detail / deal terms.
  ¶4 (optional) — Social proof: customer quote OR analyst/partner quote.

EXECUTIVE QUOTE (separate paragraph)
  Format: "Quote text," said [First Last], [Title], [Company].
  Quote must add new information — not restate the lead.
  One quote minimum; two maximum (one internal, one external/customer).

ABOUT [COMPANY]  (from brand-brain boilerplate, verbatim or lightly trimmed)

###  (signals end of release)

Media Contact:
[Name]
[Title]
[Email]
[Phone]  (optional)
```

### Craft rules for WRITE mode

**Headline:** Start with brand name or product name. Active verb. No hype words ("revolutionary," "best-in-class," "game-changing"). ≤ 170 characters for wire SEO. Sub-headline is optional but raises searchability.

**Lead:** The five Ws in ≤ 60 words. Never open with "Company X today announced that it is excited to…" — open with the news.

**Body paragraphs:** Each paragraph has one job. Market context → product detail → proof. Do not pile jobs. Every number or stat gets a source attribution or `[verify]` marker.

**Quotes:** AP quote format — comma inside closing mark, attribution after. Executive quote adds information not in the lead. Customer/partner quote amplifies the impact. Never fabricate a quote; use `[quote needed: suggested angle — ]` as a placeholder if none provided.

**Boilerplate:** Use the brand-brain boilerplate verbatim; trim only for length. Never invent company history or claim count/revenue `[verify]`.

**Wire SEO (when targeting a wire):** Headline under 170 chars, sub-headline under 120, embed one target keyword naturally in lead, include the full product/company name in the first sentence. PR Newswire requires word count ≥ 400; Business Wire accepts 400–800.

**Length:** 400–600 words body (not counting boilerplate + contacts). Anything longer needs explicit justification.

### WRITE output format

```
---
PRESS RELEASE — [Brand] — [Date]
Mode: WRITE | Wire target: [wire or N/A] | Word count: [n]
---

[Full release text, formatted to spec]

---
Distribution note: [any wire-specific formatting flags]
Amplification: type /press-release-social-blog-amplification-pack to generate social, email, and blog versions.
```

Save to `./press-releases/[brand-slug]-[YYYY-MM-DD]-[slug].md` when the user confirms.

---

## REVIEW mode

Triggered when the user pastes or references an existing draft and asks for a review, critique, AP style pass, or wire check.

### The AP-Style + Editorial Review Framework (4 gates)

**Gate 1 — Structure integrity**
Does every required section exist in the right order? Flag missing sections (no sub-headline is fine; no lead paragraph is not). Score: Pass / Fix required.

**Gate 2 — Lead quality**
Does the lead contain all five Ws in ≤ 60 words? Does it open with the news or with "excited to announce"? Rewrite the lead if it fails.

**Gate 3 — AP style pass (line level)**
Check and correct:
- Spell out numbers one through nine; numerals for 10+
- State abbreviations: use AP postal codes (Calif., N.Y.) on first reference
- Job titles: lowercase after a name ("Jane Smith, chief executive officer"), capitalize before ("Chief Executive Officer Jane Smith")
- Dates: Month DD, YYYY — no "th/st/nd"
- Percent: always spell as % after numerals ("12%")
- Oxford comma: AP does NOT use it
- Quotation marks: double, comma/period inside
- Company names: no ™ or ® in body text
- Superlatives ("leading," "#1," "first"): flag each — require a verifiable source or delete

**Gate 4 — Wire distribution readiness**
If a wire is named: confirm word count meets minimum (PR Newswire ≥ 400, Business Wire 400–800); flag missing media contact block; check headline character count; note multimedia attachment opportunities (logo, product image, video) that boost pickup.

### REVIEW output format

```
---
PRESS RELEASE REVIEW — [Brand] — [Date]
---

DISTRIBUTION READINESS SCORE: [0–10]
  Structure: [Pass/Fix] | Lead quality: [Pass/Rewrite] | AP style: [x issues] | Wire-ready: [Yes/No/Partial]

STRUCTURAL NOTES
[numbered findings]

LEAD REWRITE (if needed)
Before: …
After:  …

AP STYLE CORRECTIONS
[table: Location | Issue | Correction]

WIRE-SPECIFIC FLAGS
[bulleted, per wire named]

REVISED EXCERPT
[show the corrected version of the most-edited paragraph(s)]
```

---

## Principles

- **Brand-brain first.** No draft until `brand-brain` returns. Its voice and banned words are hard overrides; its boilerplate is used verbatim.
- **News leads.** The first sentence is the story. If the lead is not the news, rewrite it — do not work around it.
- **AP style is the floor, not a suggestion.** Editors, wires, and journalists expect it; deviations signal amateur copy.
- **Honest proof only.** Every stat, ranking, and claim needs a source or `[verify]`. No invented proof, no unsubstantiated superlatives.
- **Quotes add information.** A quote that restates the lead wastes the quote slot and signals the spokesperson had nothing to say.
- **Compose, don't duplicate.** Amplification is `press-release-social-blog-amplification-pack`'s job; hand off, never rebuild.

## What not to do

- Do not write the release before `brand-brain` returns.
- Do not open the lead with "excited to announce," "proud to share," or any emotion-first construction.
- Do not invent executive quotes — placeholder with `[quote needed]` and the suggested angle.
- Do not use `™`, `®`, or `©` in body text (wire convention; fine in boilerplate).
- Do not exceed 600 words body without a reason; do not pad with market-size paragraphs that add no news.
- Do not skip the media contact block when targeting a wire.
- Do not produce review edits without showing before/after for every rewrite suggestion.

## Quality checklist (self-review before presenting)

- `brand-brain` called; voice, banned words, and boilerplate loaded?
- WRITE: all five Ws in the lead (≤ 60 words); at least one attributed quote; `[verify]` on every unconfirmed stat?
- WRITE: structure matches AP skeleton exactly (FOR IMMEDIATE RELEASE → dateline → headline → lead → body → quote → About → ### → contact)?
- REVIEW: all four gates evaluated; distribution readiness score stated; line-level AP corrections shown in table form?
- Wire-specific checks run if a wire was named?
- Boilerplate used verbatim from brand-brain (not rewritten)?
- Amplification handoff offered at the end of WRITE mode?
