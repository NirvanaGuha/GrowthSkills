---
name: linkedin-carousel-builder
description: >
  Turns a key idea, article, data set, or rough brief into a fully written LinkedIn carousel
  deck — slide-by-slide copy, a numbered structure, and a native-document posting strategy.
  Applies the Hook-Bridge-Value-CTA framework (the four-zone architecture every high-reach
  carousel uses) so a junior marketer produces output that reads like a senior's best-performing
  post. Handles any carousel type: educational how-to, data story, personal story arc,
  listicle, counter-intuitive argument, case study snapshot. Calls brand-brain first so
  every slide is on-voice. Optionally calls headline-hook-generator for the title card and
  cta-variant-generator for the follow slide. Saves the deck to ./carousels/[slug].md for
  reuse. Use when the user says "build a LinkedIn carousel," "write carousel slides,"
  "turn this into a carousel," "LinkedIn document post," "swipe content," or hands over
  an article, data point, or idea and asks for a slide deck.
---

# LinkedIn Carousel Builder

Every carousel is a mini-course in 8–12 slides. The reader swipes because each card answers one question completely — and the next card raises a new one. This skill produces that deck: title card that earns the swipe, body cards that each carry one idea, a CTA card that converts the engaged reader. Every word is in the brand's voice, every claim is sourced or flagged `[verify]`.

This skill writes copy. It does not produce images, Canva files, or PDF exports. For visual execution, hand the output to `canva-figma-workflow-accelerator`. For distributing across platforms, hand it to `content-repurposer-atomizer`.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, ICP, proof, and banned words. Do not write a single slide before this returns.
- **`headline-hook-generator`** (optional, on request or when the input hook is weak) — sharpens the title-card headline and the deck's opening hook line.
- **`cta-variant-generator`** (optional, on request or when generating a CTA card) — produces the follow/CTA card copy across angles; synthesize inline when absent.
- **`post-quality-reviewer-voice-auditor`** (optional, after draft) — run a voice audit pass on the full deck before delivery when the user asks for a review.

---

## How a run works

```
Step 0  Load brand        ──► call brand-brain skill
Step 1  Classify the job  ──► pick carousel type + decide slide count
Step 2  Build the spine   ──► one-sentence per slide before writing
Step 3  Write the deck    ──► title / body / CTA cards, HBVC zones
Step 4  Self-review       ──► checklist pass; flag weak hooks and broken swipe logic
Step 5  Deliver + save    ──► inline + save to ./carousels/[slug].md
```

### Step 0 — Brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns: voice adjectives, banned words, real proof, ICP + awareness tendency, offer mechanics, and positioning. Obey voice and banned-words as hard overrides. Every proof point used must come from the brand digest or be marked `[verify]`. Do not write slides before this returns.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer 4 inline questions: what it is, ICP + awareness stage, 3 voice adjectives + banned words, and one real proof point. Prefer the call.

### Step 1 — Classify the job

Pick the carousel type from the input. When unclear, ask one question.

| Type | When to use | Slide shape |
|---|---|---|
| **How-to / process** | Teach a method, tool, or framework | Cover → numbered steps → "The trap to avoid" → CTA |
| **Data story** | Anchor to a surprising stat and explain it | Stat hook → context → implications → what to do → CTA |
| **Counter-intuitive argument** | Challenge a common belief | Bold claim → "Here's why most people get this wrong" → evidence → the real answer → CTA |
| **Listicle** | Curated list (tools, mistakes, rules) | Cover → list items 1–N (one per card) → "The one most people skip" → CTA |
| **Personal story arc** | First-person narrative with a lesson | Hook moment → before state → inciting event → learnings → takeaway → CTA |
| **Case study snapshot** | Evidence-based success story | Result hook → situation → what they did → the result in numbers → transferable lesson → CTA |

Decide the slide count: 8 slides minimum (LinkedIn counts slides in the document; fewer than 8 rarely rewards the algorithm), 12 maximum for most topics. 15 is the hard cap — readability degrades beyond that.

### Step 2 — Build the spine (one sentence per slide)

Before writing full copy, output the spine: slide number + a single sentence describing what that card does. Get quick approval or proceed if no objection in 30 seconds. The spine catches structural problems before they're written into prose.

Example spine (how-to, 9 slides):
```
1. Hook: the counterintuitive claim that earns the swipe
2. The problem: why most people do X wrong
3. Step 1 of the method
4. Step 2
5. Step 3
6. The common mistake that kills the result at step 3
7. The shortcut most people miss
8. Proof: one real outcome from applying this
9. CTA: follow + one micro-action
```

---

## The HBVC Framework (Hook → Bridge → Value → CTA)

Every high-reach LinkedIn carousel uses this four-zone structure. Apply it as the organizing logic of the deck, not as literal labels on cards.

**Hook zone (slides 1–2):** The title card must earn the swipe. LinkedIn shows only the first slide in the feed. If the hook does not create a felt gap — curiosity, recognition, surprise, or a specific promised outcome — the carousel dies at zero opens. The second slide is the bridge: it validates the promise of slide 1 and gives the reader a reason to continue. Together these two slides are the highest-leverage real estate in the deck.

**Bridge zone (slides 3–4):** Establish the framework or problem context before entering the value body. Name the enemy (the wrong belief, the hard constraint, the overlooked mechanism). This gives each subsequent value card somewhere to land.

**Value zone (slides 5–N-1):** The body cards. One idea per card. One card cannot carry two unrelated ideas. Each card earns the next swipe with a micro-resolution + micro-open pattern: it answers the question the previous card raised, then raises a new one. Never pad — a card with nothing to say is cut, not filled with generic filler.

**CTA zone (final card):** Not a soft "hope this helped." The CTA card has a primary action (follow, save, share, reply) and a specific micro-action that proves engagement (e.g., "Drop your biggest bottleneck in the comments and I'll give you the fix"). Call `cta-variant-generator` when multiple CTA options are needed; synthesize inline otherwise.

---

## Slide copywriting spec (LinkedIn native document format)

LinkedIn carousels post as PDF documents rendered full-bleed per slide. The real estate is square (1:1) or portrait (4:5). Treat each card as a billboard, not a page.

**Title card:** Headline (max 12 words — strong verb + specific promise/claim), a subline that adds specificity or names the ICP (1 sentence), and the slide count ("8 slides"). No logo on slide 1 — it signals ad and kills organic reach. The brand name belongs on slide 2 or the footer.

**Body cards:** One headline per card (6–10 words), 2–3 lines of supporting copy (40–70 words maximum per card). If a concept needs more than 70 words it gets split into two cards or belongs in an article. Use parallel structure across cards in the same zone for scannability.

**Proof and stat cards:** Lead with the number or outcome in large display text. Follow with one sentence of context. Source it or mark `[verify]`.

**The "cliffhanger" card (optional, mid-deck):** At the natural halfway point (slide 4–5), one card can end on an unresolved question rather than a micro-resolution — forces completion. Use sparingly; once per deck.

**CTA card:** Primary action (bold, imperative) + one specific micro-engagement ask + brand name/handle. Keep it to 3 lines.

---

## Output format

Present the full deck as numbered slides with labeled zones. Show character count per card when over 70 words.

```
## LinkedIn Carousel — [Title] ([N] slides)
Brand: [slug, via brand-brain]
Type: [how-to / data story / etc.]
ICP: [from brand digest]
Awareness stage: [from brand digest]

---
### Slide 1 — Hook [HOOK ZONE]
**Headline:** ...
Subline: ...
"[N] slides"

### Slide 2 — Bridge [HOOK ZONE]
...

### Slide 3 — [label] [BRIDGE ZONE]
...

[continue to final slide]

---
### Slide [N] — CTA [CTA ZONE]
Primary: ...
Micro-action: ...
Handle/Brand: ...

---
**Spine recap:** [1-line description per slide]
**Swipe logic check:** [single paragraph assessing whether each card earns the next one]
**Post copy:** [3–5 line native post text to accompany the document upload — not a caption; this is the LinkedIn text field above the carousel]
```

Save to `./carousels/[slug].md`. The slug is a kebab-case version of the title.

---

## Post copy (the text field above the upload)

The carousel document is attached, but LinkedIn renders the first ~210 characters of the post text before "see more." Those characters do double duty: they pre-sell the deck for people who won't swipe and give algorithm context for distribution. Write them as a standalone hook — not "In this carousel I share..." but a punchy opening line or bold claim that works even without the carousel. End with one sentence inviting the swipe.

---

## Principles (Non-Negotiable)

- **Brand-brain first, always.** No slides before the brand context loads. Voice + banned-words are hard overrides.
- **One idea per card.** If you can't say it in 40–70 words with a 6–10 word headline, it's two cards.
- **Earn every swipe.** Each card ends with a micro-resolution and raises a new micro-question. Dead ends kill completion rate.
- **Hook zone is the highest-leverage real estate.** Slides 1–2 determine whether the deck gets read at all. Do not rush them.
- **Real proof or `[verify]`.** Never invent stats, outcomes, or case-study numbers. If the brand digest has no proof point for a claim, mark it.
- **Slide count is a craft decision.** 8–12 is right for most topics. Don't inflate to 15 to seem thorough; don't compress to 5 to seem efficient.
- **The CTA card is not a footnote.** It is a conversion surface. A soft "hope this helped" wastes the reader who got to the end.

## What Not to Do

- Don't write slides before `brand-brain` returns.
- Don't use generic "learn more / stay tuned / hope this was helpful" language on the CTA card — call `cta-variant-generator` or write a specific action.
- Don't put two ideas on one card and title the card with both.
- Don't add emoji unless the brand voice explicitly allows it.
- Don't confuse the post text (the LinkedIn text field) with the slide copy — write both separately.
- Don't produce a "listicle" when the input is actually a process or data story — classify correctly before writing.
- Don't skip the spine step on a 10+ slide deck; structural errors are much harder to fix in full prose.
- Don't fabricate proof points to fill a proof card; flag `[verify]` and note what real evidence would look like.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback completed) before first slide?
- Voice + banned-words honored throughout?
- Spine reviewed (explicitly or internally) before full copy written?
- Slide 1 headline: strong verb + specific promise, 12 words or under?
- Slide 2 earns the continuation — validates the promise and raises a question?
- Body cards: one idea each, 40–70 words, 6–10 word headline?
- HBVC zones all present and in the right order?
- Proof points: real (from brand digest) or marked `[verify]`?
- CTA card has a primary action + a specific micro-engagement ask?
- Post text written separately, opening 210 chars work as standalone hook?
- Output saved to `./carousels/[slug].md`?
