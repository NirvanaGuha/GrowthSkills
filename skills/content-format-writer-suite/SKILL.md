---
name: content-format-writer-suite
description: >
  Takes a brief and a declared format type — comparison, listicle, how-to, case study, or
  glossary/FAQ — and produces a fully written, publication-ready piece in that format.
  Each format has its own structural framework, reader-intent model, and quality gates,
  so the output matches what the format actually demands rather than applying generic
  blog conventions to every job. Brand voice, ICP awareness stage, proof points, and
  banned words come from `brand-brain` — this skill does not derive them.
  Upstream inputs (brief, angle, target keyword) can arrive from `content-brief-builder`
  or `topic-cluster-pillar-architect`. Output can be passed directly to `publishing-integration-hub`
  for WordPress/Webflow publish or to `content-qa-reviewer` for a quality gate pass.
  Invoke when the user says "write a comparison post," "draft a how-to," "build a listicle,"
  "write the case study," "produce the FAQ page," "write this piece," or hands over a brief
  and names a format. This skill writes the piece — it does not build the editorial calendar,
  choose the topic, or publish.
---

# Content Format Writer Suite

A brief plus a format type — that is the contract. This skill knows how each format works at a structural level, not just cosmetically, and produces a piece that actually fits the format's job: a comparison aids a deliberating buyer; a how-to reduces time-to-success; a listicle is scannable and shareable; a case study closes the proof gap; a glossary/FAQ earns featured snippets and captures decision-stage traffic.

Every piece is written on-brand and in-voice using the active brand's context from `brand-brain`, composed with sibling skills where relevant, and sized correctly (not padded).

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, ICP, positioning, offer, proof, and banned words. This skill does not scan, interview, or store brand data. **Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for voice, ICP, and banned words inline (or to run `brand-brain-bootstrapper` first) — never draft on undefined brand context.
- **`content-brief-builder`** (optional) — call when the user has a topic but not yet a structured brief; use its output as input here.
- **`topic-cluster-pillar-architect`** (optional) — when the piece slots into a cluster, call to confirm angle and internal-link targets before drafting.
- **`headline-hook-generator`** (optional) — call to generate and pressure-test the H1 + intro hook before committing.
- **`proof-vault`** (optional) — call to retrieve confirmed proof points for case studies and comparison pages; mark unconfirmed stats `[verify]`.
- **`cta-variant-generator`** (optional) — call to write the end-of-piece CTA and any in-line conversion CTAs.
- **`content-qa-reviewer`** (optional) — call after writing for the voice-audit/quality-gate pass before handing off to publishing.
- **`publishing-integration-hub`** (optional) — call to push the finished piece to WordPress/Webflow/Notion with SEO fields set explicitly.

---

## How a run works

```
Step 0  Load the brand     ──► call brand-brain; never draft before it returns
Step 1  Lock the format    ──► identify one of the five formats below (or ask)
Step 2  Validate the brief ──► check for the minimum inputs per format; surface gaps before writing
Step 3  Draft              ──► apply the format's structural framework + voice from brand-brain
Step 4  Self-review        ──► run the quality checklist; fix before presenting
Step 5  Offer next step    ──► content-qa-reviewer, cta-variant-generator, or publishing-integration-hub
```

**Minimum brief inputs (any format):** target keyword or topic, intended reader + awareness stage, word-count target or scope signal (short/medium/long), and any explicit "must include" requirements. Missing inputs: surface them up-front, don't guess them into the draft.

---

## The Five Formats

### 1. Comparison

**Reader intent:** "Help me choose between X and Y." The reader is at solution-aware to product-aware; the piece must earn trust by being genuinely balanced before it tilts toward the recommendation.

**Framework — weighted multi-criteria decision analysis (MCDA / weighted scoring model), rendered as prose.** MCDA is a real decision-science method: define criteria, weight each by how much it matters to *this* decision, score every option per criterion, and let the weighted totals carry the verdict. A comparison post is that model made readable — the weights are implicit in which criteria you lead with and how much room you give them.

1. **Setup paragraph** — name the decision, who it is for, and what the article settles (no fake suspense; readers know a vendor wrote this).
2. **Criteria selection (the weighting step)** — choose 4–7 dimensions tied to the reader's job-to-be-done, and order them by weight: the criterion that decides the purchase goes first and gets the most room. State the weight logic in one line ("For a 5-person team, price matters less than onboarding speed — so that's where we start"). This is what separates a real comparison from a feature dump.
3. **Per-product section** — for each alternative: one-paragraph honest description, a scored row against the criteria, 1–2 limitations stated plainly.
4. **Head-to-head scoring table** — criteria (rows, in weight order) × products (columns). Cells carry a graded value, not a raw checkmark — use the rubric below so "Yes" and "Yes, but capped at 3 seats" don't collapse into the same tick.
5. **Who should choose X / Who should choose Y** — segment-specific verdicts, one or two sentences each. Different readers weight the criteria differently; this section is where you let the weights flip. Earns trust; does not dodge the recommendation.
6. **Verdict + CTA** — one paragraph; the brand's product wins on the *highest-weighted* criteria for the target reader, named honestly. CTA from `cta-variant-generator` matched to the awareness stage.

**Scoring rubric (use in the head-to-head table — no bare checkmarks):**

| Cell value | Means | Example |
|---|---|---|
| **Full** | meets the criterion with no caveat | "Unlimited seats, all plans" |
| **Qualified** | meets it with a stated limit | "Yes — capped at 10k contacts" |
| **Partial** | present but materially weaker | "Email only; no SMS" |
| **Add-on** | available only at extra cost | "$49/mo add-on" |
| **None** | not offered | "Not available" |

**Calibration rules:** do not trash competitors; cite limitations factually; the weight order must reflect the *reader's* priorities, not the brand's strengths (if your product only wins on a low-weight criterion, say so — the verdict is for a different segment). Any benchmark number not from the brand's own published data → `[verify]`. Length: 1,000–2,000 words typical (scales with number of alternatives).

> Note: this is a weighted-scoring structure, **not** a Gartner Magic Quadrant — there are no two axes and no quadrant placement. If the brief calls for a genuine two-axis quadrant visual (e.g., "ability to execute × completeness of vision"), that is a different deliverable; build it explicitly or hand off to `infographic-data-viz-spec-writer`.

---

### 2. Listicle

**Reader intent:** "Give me the best options/tips/examples fast; I'm scanning, not reading."

**Framework — inverted pyramid per item, ordered by the serial-position effect.** Two real structures do the work. The **inverted pyramid** (journalism's standard: most important information first, supporting detail after) governs *each item* — a scanner who reads only the first line of an item should still get the payoff. The **serial-position effect** (Ebbinghaus; demonstrated experimentally by Murdock, 1962) — readers recall the first and last items in a series best, the middle worst — governs the *ordering of the whole list*.

1. **H1 + intro** (3–5 sentences max) — promise the list, state the selection criterion, name the intended reader. No lengthy preamble.
2. **Numbered items** — each item: `### N. [Action verb or noun phrase]` → 1–3 short paragraphs, inverted-pyramid order: **what it is** (the payoff line), then **why it qualifies**, then **one concrete example or proof point**. The first sentence must stand alone for a scanner who reads nothing else.
3. **Order by serial position** — the two strongest items go in positions **1 and last**; the weakest sit in the middle, where they're least scrutinized. "Decreasing-energy order" (best first, fading out) is a myth — it wastes the recency slot and lets the list die at the bottom.
4. **Quick-reference table** (optional for 10+ items) — name, category, one-line use case; a scannable summary that respects the middle-of-list recall dip.
5. **Closing CTA** — one sentence + button copy from `cta-variant-generator`.

**Ordering rule of thumb (serial position):**

| List position | Recall strength | Put here |
|---|---|---|
| #1 (primacy) | highest | strongest / most surprising item |
| Last (recency) | high | second-strongest; the one you want remembered |
| Middle | lowest | solid-but-expected items, evenly spaced |

**Calibration rules:** every item must actually belong (no filler to hit a round number); if an item needs more than ~120 words it belongs in a how-to or comparison, not here. Keep depth consistent across items — uneven depth signals padding. Length: 800–1,500 words typical.

---

### 3. How-To

**Reader intent:** "Walk me through this step-by-step; I want to finish the task, not read an essay."

**Framework — Cognitive Load Theory applied to instructional design (Sweller):**
1. **Problem + payoff** (1 paragraph) — name the task, who it is for, and what the reader will be able to do by the end. Do not bury this.
2. **Prerequisites** — bullet list of what the reader needs before step 1. Include skill assumptions (e.g., "basic WP admin access") and software versions where relevant.
3. **Numbered steps** — each step is `### Step N: [imperative verb phrase]`. Structure within each step: **Action** (what to do, present tense imperative) → **Why** (one sentence, only when non-obvious) → **Expected result** (what they should see/have after completing the step). Screenshots or code blocks can be flagged as `[add screenshot: X]`.
4. **Troubleshooting section** (optional, recommended for technical topics) — 3–5 common failure modes as `**Problem:** ... **Fix:** ...` pairs.
5. **Next steps + CTA** — where does the reader go from here; one on-brand CTA.

**Calibration rules:** never combine two actions in one step; never skip the expected result — that is what reduces support tickets; no step should exceed ~100 words. Length: 800–2,500 words depending on complexity; go longer if it means steps are complete, not if it means padding.

---

### 4. Case Study

**Reader intent:** "Show me this worked for someone like me." The reader is product-aware to most-aware; they need proof, specificity, and identification.

**Framework — Before-After-Bridge (BAB) scaled to narrative.** BAB is the classic copywriting structure — establish the *Before* state (the pain), paint the *After* state (the resolved world), then show the *Bridge* that got from one to the other. It is an anonymous, widely-taught formula with no single verified author — so credit it as the BAB formula, not to any named copywriter. A case study is BAB stretched across a real customer's timeline: the three beats map directly onto the eight sections below.

| BAB beat | Job | Maps to sections |
|---|---|---|
| **Before** | make the reader recognize their own pain | Hero summary (the problem) · Context |
| **After** | show the resolved state, quantified | Hero summary (the result) · Results · Quote |
| **Bridge** | make the path credible and repeatable | Decision · Implementation · What made it work |

1. **Hero summary** (above the fold) — customer name, their industry/size, the problem solved, and the headline result. This single line carries both the Before and the After; readers share it, so make it quotable.
2. **Context: the before** — customer profile (company type, relevant scale signals), the specific pain or blocker, what they had tried before. Sourced from a real interview or published public quote; `[verify]` any number not confirmed.
3. **The decision** (Bridge begins) — why they chose this solution (not a puff paragraph; include the real reason — ease, price, integration, a peer's recommendation).
4. **Implementation** (Bridge) — what they actually did; timeline if available; any friction. Specific over vague — the friction is what makes the path believable.
5. **The after: results** — quantified outcomes first (% change, $ impact, time saved), qualitative second (team sentiment, strategic unlock). Real numbers or `[verify]`; never invent outcomes.
6. **Quote** — pull a real customer quote that names the result. Attribute with name + title + company.
7. **What made it work** (Bridge closes) — 2–3 factors the customer credits; this is what makes the After look repeatable rather than lucky, and keeps the study from reading like a one-sided ad.
8. **CTA** — match the awareness stage: "Read more customer stories" or "See if [product] fits your stack" — not "Sign up now" from a cold-traffic case study.

**Calibration rules:** the Before pain and the After result must be the *same customer's* — no composite "before from one account, after from another." Do not write a case study from scratch without real customer input; if no confirmed data is available, write a `[PLACEHOLDER — needs customer data]` draft structure and note what must be verified before publish. Length: 600–1,200 words.

---

### 5. Glossary / FAQ

**Reader intent:** "Define this for me" (glossary) or "Answer my specific question fast" (FAQ). These pages exist to earn featured snippets and anchor cluster internal links — they are SEO infrastructure, not just content.

**Framework — Entity-based SEO + People Also Ask (PAA) architecture:**

**Glossary:**
1. **Page intro** (2–3 sentences) — what the glossary covers and who it is for; include the primary keyword.
2. **Alphabetical or thematic sections** — each term: `### [Term]` → **Definition** (one crisp sentence, <35 words; this is the featured-snippet target) → **Expanded explanation** (2–4 sentences adding context, use cases, or common misconceptions) → **Related terms** (2–3 inline links to sibling entries).
3. **See-also section** — link to pillar pages and how-tos that use the terms in practice.

**FAQ:**
1. **Page intro** (1–2 sentences) — confirms what the page answers and for whom.
2. **Questions as `### H3` headings** — one direct-answer sentence first (featured-snippet target), then 2–4 sentences of fuller context. Each answer stands alone (Google may surface it without the surrounding page).
3. **Group questions** by theme under `## H2` sections if there are >8 questions; scanners use headings as navigation.
4. **CTA** — light conversion ask at the bottom; FAQ readers are often early-funnel.

**Calibration rules:** the one-sentence definition/answer is the unit of value — spend editing time there. Every internal link earns a PageRank transfer; use `internal-linking-planner` output when available. Do not pad definitions with the brand backstory.

---

## Principles

- **Format first, voice second.** Obey the structural framework before reaching for creative flourishes. A clever how-to intro that buries the prerequisites creates support tickets.
- **Brand-brain is non-negotiable.** No draft before it returns. Its voice and banned words override everything in this skill.
- **Right length is earned length.** Expand to cover the job; do not pad to hit a word count. Call out if a brief's scope implies a length the format cannot support well.
- **Real proof only.** Every number, claim, and outcome from a source the brand controls or can confirm. Unconfirmed: `[verify]`. Never invent a customer result.
- **Framework names are attributions, not decorations.** Each format above names a real, verifiable framework (MCDA, serial-position effect, inverted pyramid, BAB, Cognitive Load Theory / Sweller, entity-based SEO + PAA). Apply the actual mechanic it names — do not paste a credibility tag (a famous name, a "quadrant," a "principle") onto a structure that doesn't use it. If a structure is a house pattern, label it a house pattern.
- **One H1, one primary keyword, one intent.** Do not try to serve two incompatible reader intents in one piece.
- **Compose, do not duplicate.** When a sibling skill covers upstream (brief, keyword angle) or downstream (QA, publish, CTA), call it. Do not reimplement.

---

## What Not to Do

- Do not write any piece before `brand-brain` returns the active brand's context.
- Do not apply one generic structure to all five formats — each has a distinct framework above; use it.
- Do not attach a famous-sounding label to a structure that doesn't earn it — no "Gartner quadrant" on a plain comparison table, no named author on the anonymous BAB formula, no invented "principle." Name the real mechanic or call it a house pattern.
- Do not invent customer outcomes in a case study or invent product capabilities in a comparison.
- Do not pad a listicle to reach a round number; do not split a natural paragraph into two steps to inflate a how-to.
- Do not treat a glossary as a blog post — definitions first, prose second.
- Do not produce a FAQ page that answers questions the target reader is not actually asking; check PAA data or use `jtbd-customer-interview-suite` output if available.
- Do not surface a finished draft without running the quality checklist.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and returned active brand context before drafting?
- Voice adjectives honored; banned words absent; proof claims sourced or `[verify]`-tagged?
- Correct structural framework applied for the declared format (not generic blog structure)?
- H1 contains the target keyword; intro serves the reader's intent within the first 50 words?
- For comparisons: balanced tone, criteria ordered by weight, head-to-head scoring table uses the graded rubric (no bare checkmarks), segment-specific verdicts included?
- For listicles: every item earns its place; consistent depth; items ordered by serial position (strongest at #1 and last, not best-first-fading-out); each item front-loads its payoff (inverted pyramid)?
- For how-tos: prerequisites listed; every step has action + expected result; no two actions merged?
- For case studies: headline result in hero summary; all numbers `[verify]`-tagged unless confirmed; real customer quote included or placeholder flagged?
- For glossary/FAQ: one-sentence definition/answer present per entry; related/internal links included?
- End-of-piece CTA present and matched to the reader's awareness stage?
- Piece saved to `./content/[slug]-[format].md` when the user requests a saved draft?
