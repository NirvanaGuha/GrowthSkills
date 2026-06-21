---
name: event-recap-blog-post-writer
description: >
  Turns event raw material — agenda, speaker quotes, poll results, live Q&A, chat highlights,
  and attendance stats — into a 600–900 word recap blog post that earns long-tail search traffic,
  re-engages attendees, and converts non-attendees into pipeline. Applies the SOAR Recap framework
  (Scene → Outcomes → Aha moments → Reader payoff) so the post reads like a curated editorial
  piece, not a schedule dump. Calls brand-brain for voice and proof; calls on-page-seo-optimizer
  for keyword targeting; calls content-repurposer-atomizer when you also need social/email cuts
  from the same raw material. Output is a complete, publish-ready draft saved to ./events/ with
  an SEO metadata block. Use when the user says "write the event recap," "turn this webinar into
  a blog post," "write up last week's event," "I have the transcript / slide deck / Q&A,"
  "publish a recap," or hands over any raw post-event notes and asks for content.
---

# Event Recap Blog Post Writer

A great recap is not a transcript. It is a curated editorial piece that makes the reader feel they missed something worth knowing — and gives them enough of it that they benefit from reading. It has a search angle (people look up topics, not event names), real quotes that prove humans said real things, and a payoff the non-attendee gets without attending.

This skill applies the **SOAR Recap framework** — Scene, Outcomes, Aha moments, Reader payoff — to produce a 600–900 word post that extends the event's shelf-life into search, re-engages attendees for future events, and seeds nurture flows.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — voice, banned words, proof, ICP, offer + destination URLs. No draft before this returns.
- **`on-page-seo-optimizer`** (Step 1) — keyword targeting, title tag, meta description, internal linking plan. The recap earns traffic only if it targets a real search demand.
- **`headline-hook-generator`** (Step 2, optional) — if headline variants are needed before the draft begins.
- **`content-repurposer-atomizer`** (post-draft, on request) — slice the finished recap into LinkedIn post, email newsletter teaser, and social pull-quotes.
- **`cta-variant-generator`** (post-draft) — recommend the end-of-post CTA aligned to funnel stage and brand offer.
- **`de-slop-humanize-pass`** (final pass) — strip AI tells, flatten generic phrasing, sharpen voice.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain
Step 1  Establish search angle  ──► call on-page-seo-optimizer (or synthesize if absent)
Step 2  Inventory raw inputs    ──► triage what the user supplied; surface gaps
Step 3  Build the SOAR scaffold ──► structure before drafting
Step 4  Draft (600–900 words)   ──► write once, senior voice, SOAR sequence
Step 5  Polish pass             ──► call de-slop-humanize-pass; lock CTA via cta-variant-generator
Step 6  Write SEO metadata block
Step 7  Save to ./events/; surface repurposing offer
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Wait for the active brand's digest — voice adjectives, banned words, offer mechanics, real proof, ICP. Obey voice and banned-words as hard overrides throughout. Mark any proof not returned by `brand-brain` as `[verify]`.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If none exists, ask four questions: what the brand does · ICP + awareness stage of typical event attendee · offer + primary CTA destination · 3 voice adjectives + banned words. Proceed once answered.

---

## Step 1 — Establish the search angle (before drafting)

Call `on-page-seo-optimizer` with the event topic and brand context. It returns: a primary keyword the recap should target, a recommended H1/title tag formula, meta description, and any internal linking targets. If the skill is absent, ask the user for the target keyword or derive the most obvious long-tail term from the event topic (e.g. "how to reduce cart abandonment" for an eCommerce retention webinar). Do NOT title the post "[Event Name] Recap — [Date]" as the primary H1 — that earns zero search traffic.

---

## Step 2 — Inventory the raw inputs

Before writing, map what the user provided to SOAR slots. Missing inputs determine where you synthesize vs. quote:

| Input | SOAR slot |
|---|---|
| Event name, date, attendee count, format | Scene |
| Key takeaways, session outcomes, speaker conclusions | Outcomes |
| Verbatim quotes, poll results, chat highlights, surprising data | Aha moments |
| Replay link, related resource, next event, offer | Reader payoff |

If a SOAR slot is empty, ask one targeted question to fill it. Do not silently invent quotes, stats, or speakers.

---

## Step 3 — Build the SOAR scaffold

Map inputs to a concrete outline before writing a single sentence:

1. **Scene** (50–80 words): event name, format, date, who showed up, why it mattered right now. One grounding sentence about the problem or trend that made this event worth attending.
2. **Outcomes** (150–200 words): 2–3 main takeaways or session conclusions. Structured clearly — named, not buried in prose. These are the search-worthy assertions the post wants to rank for.
3. **Aha moments** (250–350 words): the most quotable, surprising, or actionable moments. Exact speaker quotes when available; attribute them by name + title. Poll results or data from the event if provided. At least one moment that makes a non-attendee wish they had been there.
4. **Reader payoff** (80–120 words): what the reader can do with this right now — a downloadable resource, replay link, next event registration, or the brand's related offer. One CTA, from `cta-variant-generator`, matched to the ICP's awareness stage.

---

## The SOAR Recap framework — craft notes

**Scene** earns the click. It must signal the topic (keyword) and the stakes, not just the logistics. Bad: "We held our quarterly webinar on June 15." Good: "Retention is harder than it looks — here is what 400 eCommerce operators said about it live."

**Outcomes** are the section skimmable-readers skim. Use a subheading per takeaway. Write each takeaway as a complete, standalone assertion, not a teaser ("Attendees should personalize" is useless; "Segmenting by purchase recency before sending a win-back reduced unsubscribes by 18% [verify]" is scannable and rankable).

**Aha moments** are why people share. The most effective ones are specific, surprising, or counter-intuitive. A real number beats three paragraphs of narrative. A real quote beats a paraphrase. If a poll produced a result that surprised the presenter, that's the lede of this section.

**Reader payoff** is where the brand's business interest and the reader's interest converge. It should feel like a natural next step, not a sales intrusion. The CTA tone must match the ICP's awareness stage (problem-aware → educate; product-aware → trial/demo).

**Length:** 600–900 words. Under 600 and the post lacks enough substance to rank; over 900 and it becomes a transcript. If input is thin, ask for more rather than padding.

---

## Principles

- **Search angle first.** Every recap competes with zero other recaps for that event, but competes with hundreds of posts for the topic keyword. Title and H1 optimize for the topic, not the event name.
- **Quotes are load-bearing.** At least two verbatim speaker quotes, attributed by name and title. They prove humans said real things and are the most shareable sentences in the post.
- **Summarize sessions; quote insight.** The agenda is not content. A surprising claim, a poll result, a moment of honest disagreement — that is content.
- **One CTA at the end.** No CTA scatter throughout the body. The post earns trust first; the CTA at the bottom converts it.
- **Mark what you cannot confirm.** Attendee counts, conversion numbers, speaker credentials from memory — all `[verify]` until sourced.
- **Voice over vocabulary.** The brand's voice adjectives govern register, not synonym choice. A casual brand's recap sounds warm; a technical brand's recap sounds precise. Neither sounds like a press release.

---

## What not to do

- Do not title the post "[Event Name] Recap — [Date]". This earns no search traffic and communicates nothing to a cold reader.
- Do not write a schedule dump ("First, speaker A presented. Then, speaker B presented.") — this is not a recap, it is a minute-of-meeting.
- Do not invent quotes, attendance numbers, or poll results. If the user did not provide them, ask or omit.
- Do not write a post over 900 words without an explicit reason. Padding kills engagement and signals to Google that the post is thin on ideas, not rich in them.
- Do not produce the draft before `brand-brain` returns — voice mismatches compound through editing.
- Do not let the CTA interrupt the editorial content — one end-of-post CTA, on-brand, aligned to awareness stage.

---

## Quality checklist (self-review before presenting)

- [ ] `brand-brain` called and active brand loaded before any draft written?
- [ ] `on-page-seo-optimizer` called (or keyword synthesized); H1 and title tag target a real search term, not the event name?
- [ ] Post hits 600–900 words?
- [ ] SOAR structure is legible: Scene → Outcomes → Aha moments → Reader payoff?
- [ ] At least two verbatim speaker quotes, attributed by full name and title?
- [ ] At least one poll result, stat, or data point from the event (or slot marked for user to fill)?
- [ ] All unconfirmed numbers/claims marked `[verify]`?
- [ ] Voice adjectives and banned-words from brand-brain honored throughout?
- [ ] One CTA at the end, from `cta-variant-generator`, matched to ICP awareness stage?
- [ ] SEO metadata block (title tag, meta description, focus keyword, suggested slug) included in output?
- [ ] Draft saved to `./events/[slug]-recap.md`?
- [ ] `de-slop-humanize-pass` offered (or run) before final delivery?

---

## Output format

Deliver in this order:

1. **SEO metadata block** (title tag ≤60 chars · meta description ≤155 chars · focus keyword · slug · suggested featured image alt text).
2. **Draft post** (H1 + body, SOAR sequence, 600–900 words).
3. **Editor's note** (1–3 flagged `[verify]` items for the user to confirm before publishing; notes on any missing quotes or data substituted with placeholders).
4. **Repurposing prompt** — one line offering to run `content-repurposer-atomizer` to extract a LinkedIn post, email teaser, and pull-quote social card from this draft.

Save the draft to `./events/[event-slug]-recap.md`. Do not save inside the skill folder.
