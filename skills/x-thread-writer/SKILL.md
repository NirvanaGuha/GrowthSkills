---
name: x-thread-writer
description: >
  Turns a topic, URL, rough notes, or an approved content brief into a publish-ready X (Twitter)
  thread — 8–15 numbered tweets with a high-retention hook, structured middle tweets that build
  on one idea, and a reply-thread CTA. Two modes: Draft (default) takes source material and
  produces a complete thread; Repurpose takes an existing long-form asset (article, video
  transcript, LinkedIn post, email) and atomizes it into a thread. Every thread is written in the
  brand's real voice and reflects its real proof — brand context comes from the shared brand-brain
  skill, not re-derived here. Applies the Hook-Stack-Punch framework to ensure each tweet earns
  the next read, not just fills space. Use whenever the user says "write a thread," "thread this
  up," "turn this into a thread," "X thread from this article," "tweet thread about X," "repurpose
  this for X," or hands over a URL, notes, or brief and asks for social distribution.
---

# X Thread Writer

Topic, URL, or rough notes in → a high-retention thread out. Every tweet earns the next. Every thread ends with a real call to action — not "follow me for more." Brand voice and proof come from the shared `brand-brain`, so the thread sounds like the brand, not like a content mill.

This skill writes and structures threads. It does not manage editorial calendars (call `editorial-calendar-builder`), repurpose across all platforms at once (call `content-repurposer-atomizer`), or write LinkedIn variants (call `linkedin-post-writer`). If you want all channels from one asset, call `content-repurposer-atomizer` instead — it will invoke this skill internally.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's voice, banned words, ICP, offer, real proof, and positioning. Do not write a single tweet before it returns.
- **`headline-hook-generator`** (optional) — when the hook is weak or multiple hook options are wanted; invoke it, take the strongest result, adapt to tweet format.
- **`cta-variant-generator`** (optional) — for the reply-thread CTA tweet when conversion is the goal; invoke it with placement = "X reply-thread CTA" and the brand's offer.
- **`proof-vault`** (optional) — when the thread needs a data/stat tweet; invoke for verified proof rather than inventing numbers.
- **`de-slop-humanize-pass`** (optional) — run automatically on the full thread when the draft feels AI-templated; replaces filler openings, generic transitions, and passive constructions.

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain (bootstraps on first use)
Step 1  Pick the mode       ──► Draft (default) | Repurpose
Step 2  Apply Hook-Stack-Punch framework
Step 3  Self-review against the checklist
Step 4  Present + offer to persist
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer + destination URLs, real proof, ICP + awareness tendency, positioning. Write nothing until it returns.

Obey voice and banned-words as hard overrides. Use only real proof (mark anything unconfirmed `[verify]`). Match the brand's known CTAs and destinations for the reply-thread tweet.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If neither exists, ask the user to install `brand-brain` (preferred) or answer four quick questions: what the brand does · ICP + awareness stage · offer + destination · 3 voice adjectives + banned words.

### Step 1 — Pick the mode

- **Draft mode (default).** Source material (topic, URL, notes, brief) → full thread. Fetch the URL if given; extract the core argument and the two or three sharpest supporting points.
- **Repurpose mode.** Long-form asset → thread. Extract the single best angle (not a summary; find the *most tweetable* claim), shed the nuance that doesn't survive 280 chars, and restructure.

When unsure, default to Draft and offer Repurpose at the end ("Want me to try a different angle from the same source?").

---

## The Hook-Stack-Punch framework

This is the operating theory behind every thread this skill writes. A thread is not a numbered list — it's a retention machine where each tweet's job is to make the next tweet irresistible.

### Hook tweet (Tweet 1)
The entire thread lives or dies here. The hook has one job: stop the scroll and force a "wait, what?" or a "that's exactly my problem."

Four high-retention hook patterns (use one; pick to the brand's voice):

| Pattern | Template | Works because |
|---|---|---|
| **Counterintuitive claim** | "Most [X] advice is wrong. Here's what actually works:" | Creates cognitive dissonance — must resolve it |
| **Specific outcome promise** | "I [did X] and got [specific result] in [time]. Here's the exact process:" | Credibility via specificity; reader wants the formula |
| **Relatable pain + implicit promise** | "[Painful situation most people know]. This changed everything:" | Emotional hook; self-recognition triggers read |
| **Bold declarative + payoff tease** | "[Provocative statement]. A thread on why." | Works when the statement is genuinely surprising |

Rules for the hook tweet:
- No more than two sentences. If it runs to three, cut the weakest.
- Never start with "I'm going to share" or "In this thread." State the thing.
- End with a line break + "🧵" or "(thread)" — marks it as a thread, not a single post.
- If the hook is weak, invoke `headline-hook-generator` before continuing.

### Stack tweets (Tweets 2–13)
Each tweet builds one idea, proves it, or illustrates it. Never just restates the previous tweet with different words.

**The three tweet types to rotate through:**

| Type | Job | Format signal |
|---|---|---|
| **Claim tweet** | Advance the argument | Lead with the assertion; no hedging |
| **Evidence tweet** | Prove a claim | Stat, example, case, quote — real proof only; `[verify]` anything unconfirmed |
| **Contrast tweet** | Sharpen the idea | Before/after, myth vs. reality, what most people do vs. what works |

**Stack rules:**
- Each tweet must be able to stand alone and still be shareable.
- End most stack tweets with one line of blank space + the next hook (a question or incomplete thought that pulls the reader to the next tweet). Exception: evidence tweets — let the number land clean.
- Aim for 200–260 characters per tweet. Below 120 feels thin unless it's a deliberate one-liner punch.
- Never use filler openers: "So,", "Now,", "Next,", "This means that", "In other words."
- Numbered tweet labels (`2/`, `3/` etc.) are optional — use them when the thread is instructional; omit them when it's narrative/opinion.
- A thread of 8–10 tweets is often better than 14. Cut any tweet whose removal doesn't weaken the argument.

### Punch tweet (second-to-last)
The payoff tweet before the CTA. Delivers the sharpest insight, the most surprising conclusion, or the hardest-earned advice. This is the tweet people screenshot and quote. If the whole thread were one tweet, this is it.

Rules:
- Make it one to three sentences max. Compression is the craft.
- It should feel like a conclusion even without the CTA that follows.

### CTA tweet (final)
A reply-thread CTA, not a self-promotional sign-off.

Three CTA patterns ranked by performance:

| Pattern | When to use | Example |
|---|---|---|
| **Offer the next step** | When the brand has a relevant product/trial/demo | "If you want [outcome], [brand] does exactly this. [link] — [one-line offer]" |
| **Conversation prompt** | When growing reach matters more than direct conversion | "Which of these surprised you most? Reply below — I read every one." |
| **Amplification ask** | When the thread is reference-quality | "If this was useful, repost to someone building [X]. It takes 2 seconds." |

Only one CTA per thread. Never stack all three. For conversion-goal threads, invoke `cta-variant-generator` with placement = "X reply-thread CTA" and use the returned recommendation; it will anchor to the brand's real offer and destination URL.

---

## Draft mode output format

```
## Thread — [Topic / angle in one clause]
Brand: [slug, via brand-brain]
Mode: Draft | Repurpose
Awareness target: [e.g. Problem-aware → Solution-aware]
Estimated read time: [X tweets · ~Y seconds]

---
Tweet 1 (Hook)
[text]

Tweet 2
[text]

…

Tweet N−1 (Punch)
[text]

Tweet N (CTA)
[text]

---
Notes:
• Hook pattern used: [name]
• CTA type: [Offer / Conversation / Amplification]
• Proof used: [source] | [verify flags if any]
• Voice flags: [any banned words avoided, any notable voice choices]
```

---

## Repurpose mode: finding the best angle

When repurposing a long-form asset, don't summarize it — extract the single most tweetable claim. Run this filter:

1. **Most counterintuitive point** — what would most people in the ICP find surprising?
2. **Most specific result** — what number, outcome, or before/after stands out?
3. **Most actionable takeaway** — what can someone do differently today?

Pick one. The other two become alternates to offer ("Want a thread on the [X] angle instead?").

Repurpose mode also checks whether the source asset has already been threaded. If it has, it finds a *different* angle and flags the overlap.

---

## Character count and X format rules

- **Tweet limit:** 280 characters (standard). Threads under a paid Blue/X Premium account allow up to 4,000 characters per tweet — ask the user if unsure; default to 280.
- **Links eat 23 characters** regardless of actual URL length (t.co wrapping).
- **Media tweets** (image/GIF/video) allow slightly shorter copy — the visual does work.
- **Reply-thread format:** the CTA tweet should be posted as a reply to Tweet 1 (not part of the numbered sequence) when the thread is instructional and you want the CTA to feel separate. Flag this in Notes if relevant.
- **Emoji policy:** obey the brand's voice; if brand-brain returns a voice that doesn't use emoji, write emoji-free. If the brand uses emoji, one per tweet max — at the end, not as punctuation mid-sentence.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No tweet written before brand-brain returns. Voice and banned-words override everything here.
- **Hook-Stack-Punch, not a listicle.** This framework is the difference between a thread people read and one they abandon after tweet 2.
- **One idea per tweet.** If a tweet covers two ideas, split it or cut one.
- **Real proof or [verify].** Never invent a stat, case, or differentiator to fill a tweet.
- **The punch tweet must earn it.** If the second-to-last tweet doesn't make someone want to screenshot it, rewrite it.
- **CTA is singular and honest.** One ask per thread. No fake urgency.

## What Not to Do

- Don't write the thread before brand-brain returns the active brand.
- Don't reimplement brand voice/ICP derivation here — that lives in brand-brain.
- Don't start any tweet with "So,", "Now,", "Next,", "In conclusion," or "In this thread."
- Don't pad to hit a tweet count target — cut any tweet that doesn't earn its place.
- Don't invent proof, stats, or customer names; mark anything unconfirmed `[verify]`.
- Don't stack a promotional CTA and a conversation prompt — pick one.
- Don't repurpose an asset with the same angle that was already threaded — find a new one.

## Quality Checklist (self-review before presenting)

- [ ] brand-brain called and active brand loaded (or bootstrapped) before any tweet was written?
- [ ] Voice + banned-words honored; no invented proof (all unconfirmed items marked `[verify]`)?
- [ ] Hook tweet: ≤2 sentences, no "In this thread," ends with thread marker, stops the scroll?
- [ ] Stack tweets: each advances or proves one idea; no filler openers; rotates claim/evidence/contrast?
- [ ] Punch tweet: the sharpest insight, ≤3 sentences, feels like a conclusion?
- [ ] CTA tweet: one pattern only, anchored to a real destination URL from brand-brain?
- [ ] Thread is 8–15 tweets; any tweet whose removal doesn't weaken the argument has been cut?
- [ ] Character counts within limits (default 280)?
- [ ] de-slop pass run if the draft felt templated?
- [ ] Offer made to save the thread to `./social/[brand-slug]-threads/[topic-slug].md`?
