---
name: cta-variant-generator
description: >
  Writes high-converting calls to action for any conversion point — landing-page hero, pricing page,
  email button, push notification, blog-post end, in-app prompt, or ad. Two modes: by default it takes
  in copy and returns a single recommended CTA, or takes an existing CTA and returns a stronger one;
  on request it produces a full battery of variants across persuasion angles with an A/B recommendation.
  It does NOT manage brand context itself — it calls the `brand-brain` skill to load the active brand's
  voice, ICP, offer, and proof (and to bootstrap one on first use), so every CTA is on-voice and
  multi-brand aware. Use whenever the user says "write a CTA," "improve this CTA/button," "make this
  convert," "CTA variants/options," "A/B test the CTA," "what should the button say," "rewrite my call
  to action," or hands over copy/an offer and asks for the action prompt. Writes and recommends CTAs
  only — it does not redesign the page.
---

# CTA Variant Generator

Give it copy, get the right call to action. Give it a weak CTA, get a stronger one. Ask for options, get a tested-shaped battery with a clear pick. Every CTA is written in the brand's real voice, using the brand's real offer and real proof — because brand context comes from the shared `brand-brain` skill, not from guessing or re-deriving it here.

This skill generates and recommends. It does not redesign the page, restructure the offer, or rewrite the surrounding body copy. If the offer itself is the problem, it says so — it does not paper over a weak offer with a clever button.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's context. CTA does not implement brand scanning, interviewing, or storage; that lives in `brand-brain`, once.
- *(optional, when installed)* `proof-vault` for proof microcopy, `headline-hook-generator` for eyebrow/headline lines in full CTA blocks. Synthesize inline when absent.

---

## How a run works

```
Step 0  Load the brand  ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Pick the mode   ──► Quick (default) | Battery (on request)
Step 2  Do the work
Step 3  Self-review against the brand, then present
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request (and any named brand). It returns the active brand's digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it (scan + a few questions) before returning; **do not write any CTA until it returns**.

Obey the returned voice and banned-words as hard overrides, use only the returned real proof (mark anything else `[verify]`), and anchor message-match to the brand's known CTAs/destinations.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer mechanics + destination · 3 voice adjectives + banned words), then proceed. Always prefer the call.

### Step 1 — Pick the mode

- **Quick mode (default).** The everyday job: copy/offer/existing-CTA in → *the* CTA out (not a menu). One recommended CTA (+ 1–2 alternates).
- **Battery mode.** Triggered by "variants," "options," "give me 10," "A/B," "test," or an explicit count → the full angle-spread table + A/B recommendation.

When unsure, default to Quick and offer Battery at the end.

---

## Quick mode (default)

### (a) Copy in → CTA out
1. **Read the copy** for the promise it makes and the single next action it implies. The CTA pays off *that* promise (message match) — don't introduce a new one.
2. **Fix the awareness + commitment ceiling** from the brand's ICP + the placement (see *CTA craft*). Don't ask for more than the reader is ready to give.
3. **Write one recommended CTA**: button label (lead with the value or the verb) + a friction-reducer microcopy line, using the brand's real offer + proof.
4. **Add 1–2 alternates** on *different* angles (not synonyms) so there's something to test.
5. **One line of why** it fits this audience + placement.

### (b) CTA in → better CTA out
1. **Diagnose in one line** — generic verb, vague value, wrong commitment for the stage, no risk-reducer, broken message match, banned word, over the limit.
2. **Rewrite** — one stronger primary + one different-angle alternate.
3. **Show before → after** and name what changed and why. Keep any genuine constraint intact.

Quick-mode output stays short: the recommended CTA, alternates, the one-line rationale, microcopy. No big table unless asked.

---

## Battery mode (on request)

1. **Pressure-test the offer (gate).** A CTA can't fix an unclear value exchange, a commitment mismatch, or a missing risk-reducer. Name it and offer a fix — don't generate lipstick.
2. **Set the awareness ceiling.**
3. **Generate across angles, not synonyms** — ≥6 distinct motivations, a mix of first/second person (always ≥2 first-person), every label action-led, specific, inside the char limit.
4. **Pair friction-reducer microcopy** (4–6 lines) from the brand's real offer + proof.
5. **Adapt to placement / channel** (limits + tone).
6. **Build 2–3 full CTA blocks** (eyebrow / headline / button / microcopy).
7. **Recommend an A/B pair** — a primary and a deliberately *different-angle* Variant B, each with a one-line rationale and the hypothesis the test resolves.

```
## CTA options — [what / placement]
Context: [brand · audience · awareness stage · funnel stage · channel · destination]
Brand: [slug, via brand-brain]
[⚠ offer gate note, if any]
### Button labels
| # | Label | Angle | Commitment | Voice | Chars |
### Friction-reducer microcopy
### Full CTA blocks
### Recommendation  (Primary #_ · Variant B #_ · You'll learn: … · Message-match note)
```

Save battery output to `./cta/[slug]-ctas.md` if asked; Quick-mode output is inline.

---

## CTA craft (shared by both modes)

**Awareness ladder → commitment ceiling (Schwartz).** Match the ask to where the reader's head is; never exceed it.

| Awareness | Right commitment | Example |
|---|---|---|
| Unaware | zero / curiosity | "See what you're missing" |
| Problem-aware | low — educate | "Show me how it works" |
| Solution-aware | medium — evaluate | "Compare plans" / "See it in action" |
| Product-aware | high — trial/buy | "Start my free trial" |
| Most-aware | highest — transact | "Get started — free" |

**Angles to vary (not vocabulary):** value/benefit · specificity/number · low-commitment · honest urgency · curiosity · social proof · risk reversal · transformation/identity · loss aversion.

**Voice:** always include first-person variants ("Start *my* …"). **Microcopy closes:** risk-reducers, real proof, expectation-setters; unconfirmed proof is `[verify]`. **Placement limits:** hero/pricing → one primary + ≤1 low-commitment secondary; email → ≤~25 chars + a text-link variant; push → the line *is* the promise; in-app → contextual; blog-end → soft, low ceiling; exit/sticky → higher-contrast, often an incentive.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No CTA before `brand-brain` returns. Its voice + banned-words override everything here.
- **One job per CTA.** Each asks for exactly one action.
- **Vary motivation, not vocabulary.** Different reasons to click, not ten ways to say "Sign up."
- **Action over abstraction.** Lead with the verb or value; never "Submit"/"Click here"/"Learn more" as the only option.
- **Match the ask to the awareness.** Never exceed the stage's commitment ceiling.
- **Honest urgency and honest proof only.** No fake countdowns; unconfirmed proof is `[verify]`.
- **Message match is sacred.** Keep the promise the copy/ad/subject line made.

## What Not to Do

- Don't write CTAs before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage here — call `brand-brain`.
- Don't redesign the page or rewrite body copy — flag, don't fix.
- Don't stack competing primary CTAs; don't exceed a char limit; don't invent proof/differentiators.
- Don't use emojis or exclamation marks unless the brand allows them.
- Don't hand back near-identical labels — if you can't find distinct angles, say the offer is too thin.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any CTA?
- Voice + banned-words honored; only real proof used (rest `[verify]`)?
- Quick: one recommendation + 1–2 different-angle alternates + a one-line why? (Improve: before→after + what changed?)
- Battery: ≥6 angles, ≥2 first-person, ≥1 low-commitment; primary + different-angle Variant B with a stated hypothesis?
- No variant exceeds the awareness ceiling or char limit; message match addressed; offer weakness flagged not hidden?
