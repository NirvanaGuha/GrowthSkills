---
name: landing-product-page-copy-writer
description: >
  Turns an offer, ICP notes, and a feature list into a full, conversion-structured landing or product
  page — hero, problem/agitation, value props with proof, objection handling, social proof, FAQ, and CTA
  — written in the brand's real voice and message-matched to where the traffic came from. Built on a
  named section architecture (the AAA stack: Awareness-matched hero → Argument → Action) over a PAS / 4U
  spine, so a junior produces a senior page. It does NOT manage brand context — it calls `brand-brain` to
  load voice, ICP, offer/pricing + destinations, and proof (bootstrapping on first use), and composes with
  sibling skills (`cta-variant-generator`, `proof-vault`, `objection-library-builder`) instead of redoing
  their work. Use whenever the user says "write a landing page," "product page copy," "build a sales page,"
  "write the hero/above-the-fold," "homepage copy," "rewrite this landing page," "PDP copy," or hands over
  an offer + features and asks for a page that converts. Writes structured page copy with a wireframe and
  rationale — it does not design the page or build the HTML.
---

# Landing & Product Page Copy Writer

Hand it the offer, who it's for, and what it does — get back a complete page laid out in conversion order, section by section, in the brand's voice, using only real proof. Every page is built to one promise, message-matched to its traffic source, and ladders the reader from "what is this" to "I'll click." Brand context (voice, banned words, ICP + awareness, offer mechanics + destination URLs, proof) comes from the shared `brand-brain` skill — never guessed here.

This skill writes copy and the section wireframe. It does not design the page, write HTML/CSS, or pick the visual system. If the offer itself is broken — unclear value exchange, no real proof, wrong audience — it says so plainly instead of dressing a weak offer in confident copy.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's digest (voice, banned words, ICP + awareness tendency, offer/pricing + destinations, real proof, positioning). This skill never re-implements brand scanning, interviewing, or storage.
- **`cta-variant-generator`** (when installed) — for the hero, mid-page, and final CTA blocks. Pass it the page promise + placement; don't hand-roll buttons here.
- **`proof-vault`** (when installed) — to pull tagged, permission-aware proof for the value props and social-proof band. If absent, use only proof returned in the brand digest and mark the rest `[verify]`.
- **`objection-library-builder`** (when installed) — to source the real top objections and their reframes for the objection-handling section / FAQ. If absent, derive likely objections from the ICP but flag them as inferred.
- *(also useful, optional)* `positioning-messaging-architect` for the value-pillar spine and `headline-hook-generator` for hero headline angles — both already feed `brand.md`; synthesize inline if unavailable.

---

## How a run works

```
Step 0  Load the brand   ──► call `brand-brain` (bootstraps on first use)
Step 1  Frame the page   ──► page type · ONE promise · ONE audience · awareness stage · traffic source
Step 2  Gather the parts ──► pull proof / objections / CTA from sibling skills (or the digest)
Step 3  Write the stack  ──► section by section, in conversion order (the AAA stack, below)
Step 4  Self-review + present ──► wireframe + copy + rationale; offer to persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the active brand's digest and the `brand.md` path; on a new brand it bootstraps first. **Write no page copy until it returns.** Obey the returned voice + banned-words as hard overrides, anchor every CTA to a real destination URL, and use only real proof — mark anything unconfirmed `[verify]`.

**Fallback if `brand-brain` isn't installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, ask the user to install `brand-brain` (preferred) or run a 5-question mini-setup (what it is · ICP + awareness · offer mechanics + destination URL · 3 voice adjectives + banned words · top 2 proof points). Always prefer the call.

### Step 1 — Frame before you write (the gate)

A page that tries to do two jobs converts at neither. Lock these five before writing a word — ask the user only what the brand digest doesn't already answer:

| Frame variable | Why it gates the whole page |
|---|---|
| **Page type** | Landing (one offer, one action) vs. product/PDP (catalog item, spec-heavy) vs. homepage (multi-path) vs. long-form sales page. Sets length + section set. |
| **The ONE promise** | The single transformation this page sells. Every section serves it; competing promises kill conversion. |
| **The ONE audience** | One persona/segment from the ICP. Pages written "for everyone" speak to no one. |
| **Awareness stage** | Schwartz: unaware → most-aware. Sets where the page *starts* (see hero rules) and how much you educate before you sell. |
| **Traffic source / message-match** | The ad, email, or query they arrived from. The hero must pay off *that exact promise* — mismatch is the #1 silent conversion killer. |

If two of these conflict (e.g., a cold-traffic ad pointed at a most-aware "buy now" page), surface it and recommend a split — don't average them into a mushy page.

---

## The AAA stack — the section architecture

Build every page as three movements: **Awareness-matched hook → Argument → Action**, expanded into the conversion-ordered section stack below. Order is load-bearing: each section earns the scroll to the next. Cut sections for short/warm pages; never reorder.

| # | Section | Job | Built on | Length |
|---|---|---|---|---|
| 1 | **Hero (above the fold)** | In 5 seconds: what it is, who it's for, why it's better, what to do next. Message-match the source. | 4U headline (Useful · Urgent · Unique · Ultra-specific) + sub-head that names the mechanism | Headline + sub-head + primary CTA + 1 proof cue |
| 2 | **Problem → Agitation** | Make the reader feel the cost of the status quo in *their* words. (Skip for most-aware/PDP.) | PAS (Problem-Agitate) using ICP pains + VOC language | 1 short block |
| 3 | **The turn / mechanism** | Introduce the solution as the answer to that pain — the "how it works" in 3 steps. | PAS (Solution) + positioning's unique mechanism | 3 steps or 1 block |
| 4 | **Value props** | 3 outcome-led benefits, each = benefit headline → 1-line explainer → proof point. Features serve benefits, never lead. | FAB (Feature→Advantage→**Benefit**), benefit-first; value pillars from positioning | 3 blocks |
| 5 | **Social proof band** | Borrow credibility: logos, a metric, 1–2 quotes tagged to the matching use case. | `proof-vault` (real, permissioned only) | Logos + 1 stat + 1–2 quotes |
| 6 | **Objection handling / FAQ** | Disarm the specific reasons *this* buyer stalls; each answer attaches a proof point. | `objection-library-builder` (price/trust/fit/timing/effort) | 4–7 Q&A |
| 7 | **Offer recap + risk reversal** | Restate the value exchange, the price frame, and the guarantee/trial/no-card mechanic. | `offer-pricing-brain` mechanics from the digest | 1 block |
| 8 | **Final CTA** | Repeat the primary action with the promise restated; one ask, message-matched. | `cta-variant-generator` | 1 CTA block |

**Length discipline.** Cold/long sales page → full stack. Warm/most-aware → hero + value props + proof + CTA (cut 2, 3, 6). PDP → hero + spec-flavored value props + proof + FAQ + buy CTA; lead with the buy action higher. A short page that converts beats a long page nobody finishes.

---

## Copywriting craft (applies to every section)

- **Message match is sacred.** The hero headline echoes the promise of the ad/email/query that sent the click. If you don't know the source, write to the awareness stage and flag the assumption.
- **Benefit-led, feature-backed.** Lead each value prop with the outcome the reader gets; name the feature as the *reason to believe* it. Run every feature through FAB and lead with the B.
- **One reader, second person.** "You" / "your," one persona, their literal vocabulary from the ICP and VOC. No "businesses," "users," "customers" in the abstract.
- **Specifics over adjectives.** Numbers, timeframes, named outcomes beat "powerful," "seamless," "best-in-class." Ban vague intensifiers; the brand's banned-words list is a hard filter on top.
- **Proof under every claim.** Each benefit and objection answer carries a proof point — a stat, quote, or logo. No proof on hand → soften the claim or mark `[verify]`; never invent it.
- **Scannable rhythm.** Short paragraphs, descriptive sub-heads that tell the story when read alone, one idea per block. The page must make sense skim-read.
- **One job per page.** A single primary CTA repeated; at most one low-commitment secondary (e.g. "See pricing"). Never stack competing primary actions.

---

## Output format

```
## Landing page copy — [page name / offer]
Frame: [page type · the ONE promise · audience · awareness stage · traffic source]
Brand: [slug, via brand-brain]
[⚠ offer / message-match gate note, if any]

### Wireframe
[numbered section stack actually used, with a 3–6 word label each — the skim map]

### Copy
[Section by section, in order: sub-head label + the written copy. CTAs from cta-variant-generator.
 Proof lines tagged real vs. [verify]. Microcopy under CTAs.]

### Rationale (brief)
[Awareness stage → why the page starts where it does · the message-match line · which sections
 were cut and why · any offer weakness flagged.]
```

---

## Persistence

If the user wants to keep it, offer to save to `./pages/[slug]-[page-name].md` (or a path they give) — **never** inside the skill folder and **never** to `brand.md`. Save the wireframe + copy + rationale together so a designer or the next writer has the full intent, not just the words.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No page copy before `brand-brain` returns. Its voice + banned-words override everything here.
- **One promise, one audience, one action.** Frame before writing; refuse to average two conflicting jobs into one page.
- **Message match is the hero's first duty.** Pay off the exact promise that sent the click.
- **Benefits lead, features support, proof backs every claim.** Real proof or `[verify]` — never invented stats, customers, or differentiators.
- **Compose, don't duplicate.** Pull CTAs, proof, and objections from the sibling skills; don't re-derive them here.
- **Order is the conversion mechanism.** Sections earn the scroll in sequence; cut for length, never scramble.
- **Honesty over polish.** If the offer is weak, name it. Don't paper over a broken value exchange with clever copy.

## What Not to Do

- Don't write before `brand-brain` returns the active brand.
- Don't design the page, write HTML/CSS, or specify the visual system — copy + wireframe only.
- Don't lead with features, spec dumps, or "we"-centric company talk.
- Don't invent proof, testimonials, metrics, customers, or differentiators; don't use banned words or unapproved emojis/exclamations.
- Don't stack competing primary CTAs or point a CTA at a destination not in the brand's offer.
- Don't write a page "for everyone," and don't reorder the stack to hide a weak section.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any copy?
- The five frame variables locked, and the page serves exactly ONE promise / audience / action?
- Hero message-matches the traffic source and clears the 5-second test (what · who · why better · what next)?
- Every value prop benefit-led (FAB), every claim carrying real proof — rest marked `[verify]`?
- CTAs sourced from `cta-variant-generator` (or on-voice), each pointed at a real destination URL?
- Objections/FAQ pulled from `objection-library-builder` (or flagged inferred), each answer attaching proof?
- Voice + banned-words honored; specifics over adjectives; scannable when skim-read?
- Wireframe + rationale included; offered to persist outside the skill folder?
