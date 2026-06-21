---
name: campaign-concept-developer
description: >
  Takes a keyword, theme, or business goal plus the active ICP and target channel mix and
  develops a full campaign concept: the big idea (the unifying creative platform), a hero message,
  and a set of execution angles per channel. Built around the Big-Idea-First framework — every
  downstream asset (ad, email, landing page, social) flows from one central creative tension rather
  than being generated ad hoc. Calls brand-brain for voice/ICP/proof context, then hands off to
  headline-hook-generator and cta-variant-generator for execution-ready copy, and to
  campaign-brief-builder when the output needs to become a paid-media brief. Deliberately stops
  before writing finished body copy — it frames, it does not draft. Use when the user says "develop
  a campaign idea," "come up with a campaign," "what should this campaign be about," "give me a big
  idea," "campaign concept," "campaign platform," "positioning for this campaign," or hands you a
  theme and asks how to run with it across channels.
---

# Campaign Concept Developer

One brief, one big idea, every channel aligned. This skill builds the creative platform a campaign
runs on — the single unifying tension that makes an ad, an email, a landing page, and a social post
feel like one campaign instead of three separate projects. Without a platform, execution fragments.
With one, even junior writers stay on-voice and on-strategy.

Scope: concept and execution angles. This skill does not write finished body copy, produce ad
creative, or author email sequences. When it's done, hand the output to the right downstream skill.

---

## Skills this calls

- **`brand-brain`** (required, always first) — active brand voice, ICP, banned words, offer
  mechanics, proof, positioning.
- **`icp-persona-builder`** *(optional)* — call when the ICP is thin or the user asks for a
  deeper persona cut before concepting.
- **`positioning-messaging-architect`** *(optional)* — call when the brand lacks a clear
  positioning line or when the campaign needs to stake out new territory.
- **`headline-hook-generator`** — pass the big idea + hero message to generate execution-ready
  hooks per channel; do not write headlines yourself.
- **`cta-variant-generator`** — pass the campaign platform to generate aligned CTAs per
  placement; do not write CTAs yourself.
- **`campaign-brief-builder`** — when the concept needs to become a paid-media or launch brief.
- **`ad-copy-variant-generator`** — when the user wants finished ad copy from the angles.
- **`content-brief-builder`** — when a concept angle needs to become a content piece.
- **`a-b-multivariate-test-designer`** — when two concept directions should be tested rather
  than one picked.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain; get voice, ICP, proof, positioning
Step 1  Clarify inputs        ──► keyword/theme, ICP cut, channel mix, campaign goal
Step 2  Develop the platform  ──► apply Big-Idea-First framework (below)
Step 3  Generate angles       ──► per-channel execution directions
Step 4  Self-review           ──► quality checklist before presenting
Step 5  Hand off              ──► recommend downstream skills for execution
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before producing anything.
It returns the active brand digest: voice adjectives, banned words, ICP + awareness tendency,
offer mechanics + destination URLs, real proof, positioning line. Obey voice + banned-words as
hard overrides. Use only real proof; mark everything else `[verify]`.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly; if none exists, ask the user for five things — ICP, offer, proof, voice
adjectives, banned words — then proceed. Prefer the call.

### Step 1 — Clarify inputs (before concepting)

If any of these are missing, ask before proceeding:
- **Theme or keyword** — what the campaign is *about*
- **Goal** — awareness, trial, re-engagement, upsell, or launch
- **ICP cut** — which segment, awareness stage (Problem / Solution / Product-aware)
- **Channel mix** — paid social, organic content, email, push, or a subset
- **Constraint** — launch window, budget tier (scrappy / mid / full-build), or creative format

Do not concept without a goal and at least one channel. Everything else has sensible defaults.

---

## The Big-Idea-First framework

A campaign concept has three layers. Build them in order; each funds the next.

### Layer 1 — The Creative Tension (the Big Idea)

A **big idea** is not a tagline. It is a single, named *tension* the campaign holds: between what
the audience currently believes and what the brand wants them to believe. Find it by running the
**Before/After/Bridge** lens:

| Lens | Question |
|---|---|
| **Before** | What does the ICP believe (or fear) *before* this campaign exists? |
| **After** | What do you want them to believe *after* seeing it? |
| **Bridge** | What is the single most credible path from Before to After? |

Name the tension in one plain sentence (not a headline). This is the creative brief in a box:
*"We want [ICP] to move from believing [X] to believing [Y], using [proof/mechanism] as the
bridge."* If you cannot name it, the concept is not ready.

### Layer 2 — The Hero Message

One sentence that could run above the fold on the landing page, in the email subject line, or as
the opening hook of any ad — and make the creative tension tangible. It is not a promise list; it
is the single clearest articulation of the bridge. Characteristics:
- Speaks to awareness stage (never exceeds the ICP's commitment ceiling)
- Leads with the Before-state or the proof, not the product features
- Uses the brand's real voice (from brand-brain); banned words are absolute

Write one primary hero message and one alternative on a different angle.

### Layer 3 — Execution Angles (per channel)

For each channel in scope, produce one execution direction: the specific *angle* (not finished
copy) this channel should take on the big idea, plus the format, tone register, and the job it
does in the campaign arc (awareness / consideration / conversion).

Angles should diverge in approach while sharing the same creative tension. An email angle and a
paid social angle should feel like the same campaign to the reader, not the same sentence
copy-pasted.

**Standard channel directions:**

| Channel | Angle format |
|---|---|
| Paid social (feed) | Hook type + creative format (static/video/carousel) + awareness job |
| Email | Subject-line strategy + tone register + conversion job |
| Push notification | Trigger moment + value framing |
| Organic content / SEO | Content angle + search intent alignment |
| Landing page | Above-fold frame + proof hierarchy |

Only include channels in scope. For each, name the hook angle in 1–2 sentences. Do not write
the body copy — flag `→ headline-hook-generator` and `→ cta-variant-generator` for execution.

---

## Output format

```
## Campaign Concept — [Theme / Goal]

Brand: [slug, from brand-brain]
ICP cut: [segment + awareness stage]
Goal: [awareness | trial | re-engagement | upsell | launch]
Channels in scope: [list]

### The Big Idea
[Creative tension: one sentence naming the Before/After/Bridge]

### Hero Message
Primary: [hero message]
Alternate: [different-angle version]

### Execution Angles

**[Channel 1]**
Angle: [1–2 sentences]
Format: [format note]
Job: [awareness / consideration / conversion]
→ next: headline-hook-generator / cta-variant-generator

**[Channel 2]**
...

### Downstream handoffs
- [skill] for [specific output needed]
```

Save to `./campaigns/[slug]-concept.md` when the user asks to persist it.

---

## Principles (Non-Negotiable)

- **brand-brain first.** No concept before it returns. Voice + banned-words override everything.
- **One tension, all channels.** If the channels don't share a creative tension, it is not a
  campaign — it is a content calendar. Flag this and fix the platform before generating angles.
- **Awareness ceiling.** Hero message and angle framing never ask for more commitment than the
  ICP's awareness stage allows. A Problem-aware audience does not get a "Buy now" hero message.
- **Real proof or `[verify]`.** The big idea can be built on a claim only if brand-brain returned
  it as confirmed proof. Unconfirmed claims get `[verify]`.
- **Name, don't write.** This skill frames execution angles; it does not write body copy. Call the
  right downstream skill for that. Staying at concept altitude keeps the platform coherent.
- **Weak input, honest flag.** If the theme is too broad, the goal is missing, or the offer
  doesn't support the tension, say so. Do not paper over a thin brief with a confident-sounding
  concept.

---

## What not to do

- Don't produce a concept before brand-brain returns the active brand.
- Don't write finished ad copy, email body copy, or CTAs — hand off to the named skills.
- Don't generate five big ideas and let the user pick; develop one strong direction, offer a
  second only when the first has a real weakness or the user explicitly asks for options.
- Don't use the same angle for every channel — diverge in approach while sharing the tension.
- Don't invent proof, competitor claims, or results to make the creative tension land.
- Don't conflate a tagline with a big idea; the creative tension is an internal brief tool, not
  necessarily a consumer-facing line.

---

## Quality checklist (self-review before presenting)

- [ ] brand-brain called and active brand loaded (or fallback documented)?
- [ ] Voice + banned-words honored throughout; only real proof used (rest `[verify]`)?
- [ ] Creative tension named as a Before/After/Bridge sentence — not just a tagline?
- [ ] Hero message speaks to the ICP's awareness stage; commitment ceiling respected?
- [ ] Each angle diverges in approach while sharing the same platform?
- [ ] Channel angles match the channels actually in scope — no phantom channels added?
- [ ] Finished copy deferred to headline-hook-generator / cta-variant-generator — not written here?
- [ ] Downstream handoffs named with specific outputs needed?
- [ ] Thin brief or unsupportable claim flagged rather than papered over?
