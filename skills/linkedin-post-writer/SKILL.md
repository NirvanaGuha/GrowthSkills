---
name: linkedin-post-writer
description: >
  Turns a topic, URL, rough notes, or any existing asset into a publish-ready LinkedIn prose post —
  hook, body, and CTA — written in the brand's real voice for the brand's real ICP. Loads brand
  context from brand-brain first (voice, banned words, ICP, proof, positioning, offer) so no post
  is generic or off-brand. Diagnoses the reader's awareness stage (Schwartz's five stages) to pick
  the hook archetype and cap the CTA, then scores every hook on a 4-axis rubric before keeping it.
  Operates in two modes: Quick (default, single polished post from a brief)
  and Battery (3 angle-diverse variants with an A/B recommendation and posting-time guidance). Calls
  de-slop-humanize-pass before finalizing copy to strip AI cadence, and cta-variant-generator for
  the closing CTA when one is needed. Optionally calls headline-hook-generator to max the opening
  line and proof-vault for verified social proof. Outputs plain prose only — no markdown — because
  LinkedIn renders asterisks, hashes, and backticks as literal characters. Offer to save the final
  post (or battery) to ./social/ for reuse. Use whenever the user says "write a LinkedIn post,"
  "turn this into a LinkedIn post," "post about [topic]," "LinkedIn draft," "make this LinkedIn-ready,"
  "repurpose this for LinkedIn," or hands over a URL / article / notes and asks for a social version.
---

# LinkedIn Post Writer

Topic, URL, or rough notes in — publish-ready LinkedIn prose out. Every post is on-voice for the active brand, written for its real ICP, and cleared through de-slop before it lands in your hands. This skill writes. It does not advise on LinkedIn strategy, manage your schedule, or build content calendars — those live in `social-content-calendar-builder` and `editorial-calendar-builder`.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's voice, banned words, ICP + awareness tendency, real proof, offer, and positioning. Never write before it returns.
- **`de-slop-humanize-pass`** (required) — strips AI rhythm, generic openers, and hollow filler before the post is presented.
- **`cta-variant-generator`** (conditional) — called when the post needs a real closing CTA (not just a soft sign-off), or when the user asks for CTA options.
- **`headline-hook-generator`** (optional) — called in Battery mode or when the hook is the explicit constraint. Synthesize inline if absent.
- **`proof-vault`** (optional) — called when the post needs social proof and the brand brain's digest doesn't have enough. Mark any unconfirmed stat `[verify]`.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain; never write until it returns
Step 1  Pick the mode   ──► Quick (default) | Battery (on request)
Step 2  Diagnose awareness stage (Schwartz) → pick hook archetype + CTA ceiling; classify post type
Step 3  Draft body → write 3 hooks → score on the rubric (keep ≥6/8)
Step 4  De-slop pass   ──► call de-slop-humanize-pass
Step 5  Self-review (checklist below), then present
Step 6  Offer to save  ──► ./social/<slug>-linkedin-<date>.md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest: voice adjectives, banned words, ICP + awareness tendency, real proof, offer mechanics + destinations, positioning. If no brand exists yet, brand-brain bootstraps it. Do not write a single sentence of post copy before this returns.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to run brand-brain first, or answer a 4-question inline setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words) — then proceed.

### Step 1 — Pick the mode

- **Quick (default):** one polished, publish-ready post. The standard job for most inputs.
- **Battery (on request):** 3 angle-diverse variants + A/B recommendation + optimal-post-time guidance. Triggered by "variants," "options," "give me options," "A/B," "multiple angles," or an explicit count.

When unsure, default to Quick and offer Battery at the end.

---

## The Hook-Body-Bridge-CTA spine (house structure)

Every post is built on a four-part spine. This is this skill's own working structure — not a borrowed framework — and the order is fixed: the reader meets the hook before anything else, and the CTA is the last thing they read.

| Part | Job | Length guidance |
|---|---|---|
| **Hook** (line 1–2) | Arrest the scroll. One sentence max before the "see more" cut-off (~210 chars on desktop, ~140 on mobile feed). Personal, specific, or counterintuitive — never a thesis statement. | 1–2 lines |
| **Body** | Deliver the substance — one idea, well-developed. Story, insight, data point, or opinion. No heading structure; white space between short paragraphs. | 150–300 words total |
| **Bridge** | One line that earns the CTA — a micro-reframe, a question, a handoff. Keeps momentum without feeling abrupt. | 1 line |
| **CTA** | One clear ask, matched to the reader's awareness stage (see the ladder below). Soft (comment, reaction, share, save) or direct (click, trial) — never stacked. Call `cta-variant-generator` when a real CTA is needed. | 1–2 lines |

---

## The framework that drives the spine: Schwartz's five stages of awareness

The spine is *structure*; the engine that fills it is **Eugene Schwartz's five stages of awareness** (*Breakthrough Advertising*, 1966) — the same real framework the sibling `cta-variant-generator` uses for commitment ceilings, applied here to two earlier decisions: **which hook archetype opens the post**, and **how hard the closing CTA is allowed to push**. A LinkedIn feed is cold by default, so most posts sit one or two stages colder than a landing page for the same brand. Diagnose the stage first; let it pick the hook and cap the CTA.

| Awareness stage | What the reader knows | Hook archetype that fits | CTA ceiling |
|---|---|---|---|
| **Unaware** | Doesn't know they have the problem | Pattern interrupt, surprising stat, or a story that *names* a pain they hadn't articulated | Soft only — reaction or "follow for more"; never a click |
| **Problem-aware** | Feels the pain, no vocabulary for it | "You're [doing X] and it's quietly costing you…" — name and frame the problem | Soft — comment prompt, save; a free resource at most |
| **Solution-aware** | Knows solutions exist, comparing approaches | Contrarian reframe — "Most [X] reach for [common fix]. Here's why it stalls." | Medium — "DM me," lead-magnet link in comment one |
| **Product-aware** | Knows your category/brand, not yet convinced | Proof or outcome hook — a real number, a customer moment | Direct — trial, demo, or a link (in comment one) |
| **Most-aware** | Ready, needs a reason now | Announcement or offer hook — what changed, why now | Direct + time-bound, if the urgency is honest |

The mapping is a default, not a cage: a deliberately contrarian post to a most-aware crowd can still open cold. But never *exceed* the ceiling — a click CTA on an unaware-stage post is the single most common reason a LinkedIn post dies in the feed.

### Score the hook before you keep it

The hook is the whole game, so it earns its own pass. Draft three openers, then score each 0–2 on four axes; keep the highest, rewrite anything that scores 0 on any axis:

| Axis | 0 | 1 | 2 |
|---|---|---|---|
| **Specificity** | Abstract noun ("growth," "success") | One concrete detail | A number, name, or moment you can picture |
| **Tension** | States a fact | Implies a gap | Opens a loop the reader needs closed |
| **Stage fit** | Wrong awareness stage | Adjacent stage | Lands exactly on the reader's stage (table above) |
| **Survives the cut** | Best part is below "see more" | Payoff teased pre-cut | Complete enough to stop the scroll before the cut-off |

A hook scoring below 6/8 goes back. Rewrite the hook *last*, after the body confirms what the payoff actually is — you can't promise what you haven't yet written.

**Post types and their dominant structures:**

| Type | Best structural pattern | When to use |
|---|---|---|
| Insight / opinion | Contrarian hook → evidence → implication → soft CTA | Thought leadership, positioning |
| Story / case | Moment or outcome hook → context → turning point → lesson → CTA | Proof, trust-building |
| How-to / list | Problem-led hook → numbered or spaced steps → caveat or nuance → CTA | Educational, saves |
| Announcement | Outcome hook → what/why/who → detail → direct CTA | Product launches, milestones |
| Repurposed asset | Best single insight from the source → tease the depth → link CTA | Driving traffic to content |

---

## Quick mode (default)

1. **Parse the input.** What's the post's single idea? If the input is a URL or article, fetch the strongest insight — not the whole summary.
2. **Locate the awareness stage** (Schwartz table above) from the brand's ICP tendency + this post's job. The stage picks the hook archetype and caps the CTA.
3. **Match the post type** (table below). Pick the one that serves the idea best.
4. **Draft using the Hook-Body-Bridge-CTA spine.** Body first to find the payoff, then write 3 hooks and run the scoring rubric. One idea only in the body. No bullet points or headers — LinkedIn prose only.
5. **Embed real proof** from the brand-brain digest or proof-vault if the claim needs it. Mark unconfirmed numbers `[verify]`.
6. **Apply brand voice + banned words** as hard overrides. No exceptions.
7. **Call `de-slop-humanize-pass`** — strip AI cadence, hollow openers ("In today's landscape…"), sycophantic transitions, and repetitive rhythm before presenting.
8. **Present the post** with: the plain-prose post ready to paste, the hook score (X/8) and one-line rationale, the awareness stage + post type label, and a character count (optimal LinkedIn range: 900–1,900 chars).
9. Offer to save to `./social/`.

---

## Battery mode (on request)

Produce 3 variants — each on a **different angle**, not a synonym of the same idea.

**Angle taxonomy — vary these, not the wording.** Each angle has a natural awareness stage; for 3 genuinely different variants, spread them across at least two stages.

| Angle | What it does | Hook archetype | Natural stage |
|---|---|---|---|
| Authority / insight | Positions the brand/person as knowing something others don't | "Most [X] don't know…" / "After [N] [experiences]…" | Solution-aware |
| Social proof / story | Uses a real outcome or customer moment | "We [did X] and [result]." | Product-aware |
| Contrarian / reframe | Challenges a dominant assumption | "[Common belief] is wrong." / "Stop [X]." | Solution-aware |
| Problem-first | Opens with the audience's pain before any solution | "You're [doing X] and it's costing you…" | Problem-aware |
| Curiosity / data | Leads with a specific stat or surprising fact | "[Number] [surprising fact]." | Unaware / problem-aware |
| First-person transformation | "I used to / I learned / I was wrong about…" — personal arc | Builds parasocial trust, high engagement | Any stage |

Battery output format:

```
## LinkedIn Post Battery — [topic/brand]
Brand: [slug, via brand-brain] · ICP: [one-line description] · Awareness: [stage]

### Variant A — [Angle label · stage]
[Full post — plain prose, ready to paste]
Hook: [score]/8 — [one sentence] | Type: [label] | Chars: [N]

### Variant B — [Angle label · stage]
[Full post — plain prose, ready to paste]
Hook: [score]/8 — [one sentence] | Type: [label] | Chars: [N]

### Variant C — [Angle label · stage]
[Full post — plain prose, ready to paste]
Hook: [score]/8 — [one sentence] | Type: [label] | Chars: [N]

### Recommendation
Primary: Variant [X] — [one-line rationale tied to the reader's awareness stage]
A/B pair: Variant [X] vs. Variant [Y] — different stages or angles — Hypothesis: [what this test resolves]
Post timing: [Tue/Wed/Thu, 7–9 AM or 12–1 PM local for best reach — [verify] for your specific audience]
```

---

## LinkedIn format rules (non-negotiable)

- **Plain prose only.** No `**bold**`, no `# headings`, no backticks. LinkedIn renders these as raw characters.
- **Short paragraphs.** 1–3 lines. Single blank line between each. White space is structure.
- **Emojis:** only if the brand voice explicitly allows them. Keep functional (→ ✓) not decorative.
- **No clickbait without substance.** The hook must be paid off in the body.
- **Links:** LinkedIn suppresses reach on posts with links in the body. Put a URL in the first comment, not the post — flag this to the user when a link is needed.
- **Character range:** 900–1,900 characters performs best organically for most B2B audiences [verify for your account]. Under 300 is often a missed opportunity. Over 3,000 risks "see more" abandonment.
- **Hashtags:** 0–3 max, placed at the end if used. The brand's default hashtag stance lives in brand.md; follow it.
- **No title + body format.** LinkedIn is not a blog. No bolded title on line 1 followed by a body — hook as a sentence, always.

---

## Principles

- **Brand-brain first.** No post before it returns. Voice and banned words override everything written here.
- **Diagnose the stage, then write.** A cold feed reader is rarely product-aware. Schwartz's stage picks the hook and caps the CTA — guessing the stage is guessing the whole post.
- **One idea, fully owned.** A LinkedIn post that tries to say three things says nothing. Cut to the single strongest idea.
- **Hook is the whole game, and it's earned, not guessed.** Draft three, score them on the rubric, keep nothing under 6/8. Rewrite the hook last, after the body confirms what the payoff actually is.
- **De-slop is mandatory.** AI-cadence copy gets skipped on LinkedIn. Every post goes through `de-slop-humanize-pass` before presenting.
- **Prose only, always.** Plain sentences. No markdown. Buyer sees what you send.
- **Real proof or `[verify]`.** Never invent stats, customer names, or outcomes.
- **Never exceed the awareness ceiling.** One CTA, and one no harder than the stage allows. A click ask on an unaware-stage post is the most common reason a post dies in the feed.

## What not to do

- Don't write before `brand-brain` returns.
- Don't use markdown in the final post — it will render as literal characters on LinkedIn.
- Don't put a link in the post body — flag that it belongs in the first comment.
- Don't produce near-identical Battery variants — if you can't find 3 distinct angles, say so and offer 2.
- Don't open with "I" as the literal first character — LinkedIn's algorithm has historically deprioritized this; start with the hook concept or a number.
- Don't stack two primary CTAs in a single post.
- Don't use banned words from the brand digest under any framing.
- Don't skip the de-slop pass and present raw draft output.
- Don't invent proof points or customer outcomes.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded before any copy was written?
- Voice adjectives honored throughout; banned words absent?
- Awareness stage diagnosed (Schwartz), and the hook archetype + CTA ceiling chosen from it?
- Hook scored on the rubric (≥6/8), fits in ~210 chars, and stands alone as a reason to read on?
- CTA does not exceed the stage's ceiling, and there's only one of them?
- Body is one idea, developed fully, in short paragraphs with white space?
- No markdown, no headers, no bold in the final post text?
- Link moved to first-comment note if the input contained a URL?
- `de-slop-humanize-pass` called — no hollow openers, no AI rhythm?
- Only real proof used; unconfirmed stats marked `[verify]`?
- Quick: single post + hook score + stage/type label + char count?
- Battery: 3 genuinely different angles across ≥2 stages, A/B recommendation with hypothesis, timing note?
- Offer to save to `./social/` made?
