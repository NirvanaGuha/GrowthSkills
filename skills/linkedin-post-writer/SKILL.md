---
name: linkedin-post-writer
description: >
  Turns a topic, URL, rough notes, or any existing asset into a publish-ready LinkedIn prose post —
  hook, body, and CTA — written in the brand's real voice for the brand's real ICP. Loads brand
  context from brand-brain first (voice, banned words, ICP, proof, positioning, offer) so no post
  is generic or off-brand. Operates in two modes: Quick (default, single polished post from a brief)
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
Step 2  Classify the post type and build the structural spine
Step 3  Draft — hook → body → CTA
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

## LinkedIn craft framework: The PESO-Hook spine

Every post is built on a four-part spine derived from the Platform-Engineered Scroll Arrest + Offer model:

| Part | Job | Length guidance |
|---|---|---|
| **Hook** (line 1–2) | Arrest the scroll. One sentence max before the "see more" cut-off (~210 chars). Written to feel personal, specific, or counterintuitive — never a thesis. | 1–2 lines |
| **Body** | Deliver the substance — one idea, well-developed. Story, insight, data point, or opinion. No heading structure; white space between short paragraphs. | 150–300 words total |
| **Bridge** | One line that earns the CTA — a micro-reframe, a question, a handoff. Keeps momentum without feeling abrupt. | 1 line |
| **CTA** | One clear ask matched to the post's awareness level. Soft (comment, reaction, share, save) or direct (click, trial) — never stacked. Call `cta-variant-generator` when a real CTA is needed. | 1–2 lines |

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
2. **Match the post type** (table above). Pick the one that serves the idea best.
3. **Draft using the PESO-Hook spine.** Hook first, always. One idea only in the body. No bullet points or headers — LinkedIn prose only.
4. **Embed real proof** from the brand-brain digest or proof-vault if the claim needs it. Mark unconfirmed numbers `[verify]`.
5. **Apply brand voice + banned words** as hard overrides. No exceptions.
6. **Call `de-slop-humanize-pass`** — strip AI cadence, hollow openers ("In today's landscape…"), sycophantic transitions, and repetitive rhythm before presenting.
7. **Present the post** with: the plain-prose post ready to paste, the hook strength rationale (one sentence), the post type label, and a character count (optimal LinkedIn range: 900–1,900 chars).
8. Offer to save to `./social/`.

---

## Battery mode (on request)

Produce 3 variants — each on a **different angle**, not a synonym of the same idea.

**Angle taxonomy — vary these, not the wording:**

| Angle | What it does | Hook archetype |
|---|---|---|
| Authority / insight | Positions the brand/person as knowing something others don't | "Most [X] don't know…" / "After [N] [experiences]…" |
| Social proof / story | Uses a real outcome or customer moment | "We [did X] and [result]." |
| Contrarian / reframe | Challenges a dominant assumption | "[Common belief] is wrong." / "Stop [X]." |
| Problem-first | Opens with the audience's pain before any solution | "You're [doing X] and it's costing you…" |
| Curiosity / data | Leads with a specific stat or surprising fact | "[Number] [surprising fact]." |
| First-person transformation | "I used to / I learned / I was wrong about…" — personal arc | Builds parasocial trust, high engagement |

Battery output format:

```
## LinkedIn Post Battery — [topic/brand]
Brand: [slug, via brand-brain] · ICP: [one-line description] · Awareness: [stage]

### Variant A — [Angle label]
[Full post — plain prose, ready to paste]
Hook strength: [one sentence] | Type: [label] | Chars: [N]

### Variant B — [Angle label]
[Full post — plain prose, ready to paste]
Hook strength: [one sentence] | Type: [label] | Chars: [N]

### Variant C — [Angle label]
[Full post — plain prose, ready to paste]
Hook strength: [one sentence] | Type: [label] | Chars: [N]

### Recommendation
Primary: Variant [X] — [one-line rationale tied to ICP awareness stage]
A/B pair: Variant [X] vs. Variant [Y] — Hypothesis: [what this test resolves]
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
- **One idea, fully owned.** A LinkedIn post that tries to say three things says nothing. Cut to the single strongest idea.
- **Hook is the whole game.** The scroll stops or it doesn't. Rewrite the hook last, after the body confirms what the payoff actually is.
- **De-slop is mandatory.** AI-cadence copy gets skipped on LinkedIn. Every post goes through `de-slop-humanize-pass` before presenting.
- **Prose only, always.** Plain sentences. No markdown. Buyer sees what you send.
- **Real proof or `[verify]`.** Never invent stats, customer names, or outcomes.
- **One CTA.** Stacking two asks (comment AND click the link AND share) performs worse than one clear soft ask.

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
- Hook fits in ~210 characters and could stand alone as a reason to read on?
- Body is one idea, developed fully, in short paragraphs with white space?
- No markdown, no headers, no bold in the final post text?
- Link moved to first-comment note if the input contained a URL?
- `de-slop-humanize-pass` called — no hollow openers, no AI rhythm?
- Only real proof used; unconfirmed stats marked `[verify]`?
- Quick: single post + hook rationale + type label + char count?
- Battery: 3 genuinely different angles, A/B recommendation with hypothesis, timing note?
- Offer to save to `./social/` made?
