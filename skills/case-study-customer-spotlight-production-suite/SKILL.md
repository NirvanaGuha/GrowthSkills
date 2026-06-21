---
name: case-study-customer-spotlight-production-suite
description: >
  Turns a customer interview transcript or a set of use-case bullets into a complete case study
  production kit: outreach email to solicit/approve the story, a structured Challenge / Solution /
  Outcome draft built on the STAR framework (Situation → Task → Action → Result), a polished
  customer spotlight post for the blog or sales deck, and an SEO-optimized title + meta
  description ready for the CMS. Two input modes: (a) raw transcript/notes — extract, structure,
  enrich, draft; (b) just the use-case bullets / CRM won-deal notes — fill the STAR scaffolding
  from what's there and flag the verified gaps. Calls brand-brain for voice and proof discipline,
  proof-vault to bank the result as reusable evidence, content-qa-reviewer before final hand-off,
  and on-page-seo-optimizer for the SEO pass. Saves to ./case-studies/<slug>/ so Sales, Content,
  and Customer Success all pull from the same file. Use when the user says "write a case study,"
  "customer spotlight," "customer success story," "turn this interview into a case study,"
  "we need proof content," "case study for the sales deck," or hands over a transcript/won-deal
  note and asks for the story.
---

# Case Study & Customer Spotlight Production Suite

Customer evidence is the hardest-working asset in B2B marketing — it closes deals, anchors SEO, seeds social, and feeds every downstream proof reference. This skill takes the rawest input (a 45-minute interview recording transcript, CRM won-deal bullets, even a one-paragraph Slack message from Customer Success) and ships a complete production kit: the approval email, the structured narrative, the polished spotlight copy, and a search-optimized head. Every claim is verified against what the customer actually said; nothing is invented.

The controlling framework is **STAR with a Revenue Wrapper**: Situation → Task → Action → Result, with the Result anchored to a quantified business outcome. Generic "we love this product" testimonials fail sales. STAR + numbers pass the "so what?" test every time.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, banned words, ICP, offer/pricing, positioning, and real proof. Obey voice + banned-words as hard overrides.
- **`proof-vault`** — after the draft is approved, bank the verified result stats, customer quote, and logo attribution as reusable proof points so every other skill can reference them.
- **`content-qa-reviewer`** — runs a final pass before hand-off: factual consistency, brand voice, claim verification, formatting.
- **`on-page-seo-optimizer`** — produces the SEO title, meta description, and H1 recommendations for the published piece.
- *(optional)* `headline-hook-generator` for the spotlight headline when you want a tested battery of options; `content-repurposer-atomizer` when the brief calls for social cuts, a slide pull-quote, or email snippet alongside the full piece; `de-slop-humanize-pass` when the draft needs a final polish pass to remove AI-flat prose.

---

## How a run works

```
Step 0  Load the brand    ──► call brand-brain (loads voice, proof, ICP, offer, positioning)
Step 1  Assess the input  ──► transcript mode vs. bullets mode; gap inventory
Step 2  Sourcing email    ──► approval/quote-sign-off email (if story isn't yet approved)
Step 3  STAR extraction   ──► structure the narrative; surface quantified outcomes
Step 4  Draft the pieces  ──► Challenge/Solution/Outcome long-form + Spotlight post
Step 5  SEO pass          ──► call on-page-seo-optimizer; deliver title + meta
Step 6  QA pass           ──► call content-qa-reviewer; apply flags
Step 7  Bank the proof    ──► call proof-vault to store verified stats + quote
Step 8  Save artifacts    ──► ./case-studies/<customer-slug>/
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing any copy. It returns the active brand's voice adjectives, banned words, ICP + awareness tendency, offer mechanics and destination URLs, real proof points, and positioning line. Use those as hard overrides throughout — never invent product claims, never attribute outcomes the brand has not verified.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none exists, ask the user to install `brand-brain` or answer a 4-field mini-setup (what it is · ICP + awareness · offer · 3 voice adjectives + banned words), then proceed. Always prefer the call.

---

## Step 1 — Assess the input

**Transcript mode** (preferred): a call recording transcript, detailed interview notes, or a Slack-to-CS thread. Extract the STAR components directly from what the customer said. Prefer the customer's own words for the pull-quote.

**Bullets mode**: won-deal notes, CRM close data, or a short "they reduced X by Y%" writeup. Fill in STAR from what's given; flag every inferred component as `[verify with customer]` so nothing fabricated reaches the final draft.

Before drafting, produce a brief **gap inventory**:
- Quantified result present? If not, flag it — the story will be weak without it.
- Company name, title, use-case confirmed? Any restrictions on logo/name use?
- Customer quote available? (Pull-quote is essential for the spotlight post.)
- Approval status: story approved for publication, or does the outreach email go first?

---

## Step 2 — Sourcing / Approval email

When approval is still needed (common in bullets mode, sometimes in transcript mode), produce:

- **Subject:** Concise; references the outcome so the customer feels proud, not asked for a favor.
- **Body (≤150 words):** One sentence of context → one sentence on the outcome you want to highlight → the specific ask (quote sign-off, stat confirmation, or full story approval) → a "happy to send a draft first" close.
- **Stat / quote confirmation section** (inline or appended): exact claims you need them to confirm, numbered for easy reply.

Save to `./case-studies/<customer-slug>/00-approval-email.md`.

---

## Step 3 — STAR extraction (the structural core)

Build the STAR scaffold with a Revenue Wrapper:

| Component | What to pull | Quality bar |
|---|---|---|
| **Situation** | Who is the customer, what was their context, what was the scale of the problem | ≥1 concrete detail (team size, volume, industry) |
| **Task** | What they needed to accomplish; their constraint or deadline | Specific goal, not just "grow revenue" |
| **Action** | How they used the product/service; specific features or workflows engaged | Feature-level specificity where possible |
| **Result** | Quantified outcome + timeline | A real number or `[verify]`; never invented |
| **Revenue wrapper** | The business-impact frame: retention, CAC reduction, revenue, time saved, churn avoided | Tie to the ICP's core value lens from brand-brain |

Rules on numbers: use exactly what the customer said. If they said "roughly 30%," write "~30% `[verify]`." If no stat is available, present the qualitative result honestly and note the gap — do not pad.

---

## Step 4 — Draft the case study pieces

### A. Challenge / Solution / Outcome long-form (600–900 words)

The canonical long-form asset for the website, sales deck, and SEO:

1. **Headline** — outcome-first ("How [Company] Cut Churn by 28% in 90 Days"). Draft 2–3 options; mark a recommended primary.
2. **Challenge section** (~150 words): Situation + Task in narrative prose. Start with the customer's world, not the brand's product. Use the customer's own language where possible.
3. **Solution section** (~250 words): The Action. Walk the reader through exactly what the customer deployed and why. Name specific features or workflows — not "used our platform." Avoid superlatives that aren't in the customer's words.
4. **Outcome section** (~200 words): The Result + Revenue Wrapper. Lead with the hardest number. Tie it to the ICP pain lens from brand-brain. End with the pull-quote (customer's exact words, attributed: Name, Title, Company).
5. **CTA block**: brand-aligned next step — use the brand's offer mechanics and destination URL from brand-brain. Flag if cta-variant-generator is needed for a full battery.

### B. Customer spotlight post (300–400 words)

A tighter version for the blog "Spotlight" series, social amplification, or the sales deck "Customers" slide. Same structure, higher compression. Lead with the pull-quote. One H2 section per STAR component. End with a one-line summary stat ("Result in 90 days: 28% churn reduction").

Save both to `./case-studies/<customer-slug>/01-case-study-draft.md` and `./case-studies/<customer-slug>/02-spotlight-post.md`.

---

## Step 5 — SEO pass

Call `on-page-seo-optimizer` with the long-form draft, the target keyword (if the user supplied one; else propose one based on the outcome + industry), and the brand context. It returns the SEO title, meta description, and H1 recommendation. If it's not installed, produce them inline:

- **SEO title** (≤60 chars): outcome keyword + brand or customer name.
- **Meta description** (≤155 chars): the result + a soft CTA.
- **H1**: may differ from the headline; should match the primary keyword intent.

Save to `./case-studies/<customer-slug>/03-seo-meta.md`.

---

## Step 6 — QA pass

Call `content-qa-reviewer` on the long-form draft. Apply every flag before final hand-off. At minimum, check: every stat is attributed or marked `[verify]`; no banned words; voice matches the brand-brain adjectives; no fabricated social proof; product claims are accurate to what the customer described; the pull-quote is verbatim (or marked lightly edited for clarity).

---

## Step 7 — Bank the proof

Call `proof-vault` with the verified stats, pull-quote, company name, use case, and result timeline. This makes the outcome available to every other skill in the library — so ad copy, cold outreach, landing pages, and email sequences can reference the same proof without hunting for it.

---

## Step 8 — Save artifacts

```
./case-studies/<customer-slug>/
  00-approval-email.md       (when approval was needed)
  01-case-study-draft.md     (long-form Challenge / Solution / Outcome)
  02-spotlight-post.md       (300–400 word tighter version)
  03-seo-meta.md             (SEO title, meta, H1)
  04-qa-notes.md             (content-qa-reviewer flags + resolutions)
```

Confirm save paths and tell the user which files need customer approval before publication.

---

## Principles

- **STAR with a Revenue Wrapper, always.** A story without a quantified result is a testimonial. A testimonial without a business outcome is a compliment. This skill ships proof, not compliments.
- **Customer's words over your polish.** The pull-quote is verbatim. The challenge section borrows their language. Rewriting the customer's voice into marketing prose breaks authenticity.
- **`[verify]` before you publish.** Every number, timeline, and stat that wasn't confirmed in the transcript or brief gets flagged. Nothing unconfirmed reaches the final draft silently.
- **Brand-brain first, always.** No copy before the brand is loaded. Voice and banned-words are hard overrides.
- **Bank the proof immediately.** A case study that isn't in proof-vault only earns its ROI once. Banked, it earns it every time another skill references it.
- **Compose, don't duplicate.** SEO pass goes to on-page-seo-optimizer; QA goes to content-qa-reviewer; proof goes to proof-vault. Don't reimplement those jobs here.

---

## What not to do

- Don't invent statistics, outcomes, timelines, or customer roles — not even illustratively. Mark gaps `[verify]` and note them in the gap inventory.
- Don't write copy before `brand-brain` returns.
- Don't rewrite the pull-quote into polished marketing prose — it must sound like the customer. Mark minor clarity edits "(lightly edited)."
- Don't skip the SEO and QA passes because the draft "looks good" — the checklist is the gate.
- Don't let a case study ship without knowing whether customer approval is required. Default is: assume it is, generate the approval email first.
- Don't save anything into the skill folder or into brand.md.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; voice + banned-words honored throughout?
- Gap inventory produced; every missing stat or unconfirmed claim marked `[verify]`?
- Approval email produced when status is unclear?
- STAR scaffold complete: Situation, Task, Action, Result all populated (or explicitly flagged)?
- At least one quantified result present, or absence explicitly noted?
- Pull-quote verbatim and attributed (Name, Title, Company)?
- Long-form (600–900 w) and spotlight (300–400 w) both drafted?
- `on-page-seo-optimizer` called; SEO title (≤60 chars), meta (≤155 chars), H1 delivered?
- `content-qa-reviewer` called; all flags resolved or documented?
- `proof-vault` called; verified stats + quote banked?
- Artifacts saved to `./case-studies/<customer-slug>/` with correct filenames?
- Customer approval requirement confirmed and communicated?
