---
name: haro-source-response-writer
description: >
  Turns a journalist query and raw spokesperson facts into a tight, on-deadline source
  response — credentials paragraph, direct answer to the query, and a standalone quotable
  soundbite — ready to paste into HARO / Connectively, Qwoted, SourceBottle, or a direct
  reporter email. Loads brand context (voice, proof, positioning, spokesperson bio) via
  brand-brain before writing a single word, so every pitch is on-brand and factually
  grounded, never generic. Applies the three-part CAB formula (Credentials → Answer →
  Bite) that editors and journalists expect. Also includes a self-review gate against
  common journalist pet-peeves: over-length, off-topic, unsubstantiated superlatives,
  and embargo-blind pitches. Saves approved responses to a log file so you can track
  placements and build a reusable source library. Use when the user says "write a HARO
  response," "answer this media query," "help me pitch this journalist," "write my source
  pitch," "draft a quote for press," "HARO / Connectively / Qwoted response," or pastes
  a journalist query and asks for help replying.
---

# HARO / Source Response Writer

Journalist query in, pitch-ready response out — credentials, direct answer, quotable bite. No fluff, no generic "thought leadership," no missed deadlines.

Journalists scanning 200+ HARO responses need to see three things instantly: that you are the right person, that your answer is actually useful, and that they can quote you without editing. This skill is built around that reality, not around the marketer's desire to "get coverage."

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, proof points, positioning, and any existing spokesperson bios. Do not write the response until it returns.
- *(optional, when installed)* `proof-vault` — pulls verified stats and data for the answer body. Synthesize inline from brand-brain's proof digest when absent.
- *(optional, when installed)* `press-release-writer-reviewer` — consult for AP-style conventions when the response is destined for a formal wire or outlet with strict style rules.

---

## How a run works

```
Step 0  Load brand context ──► call brand-brain
Step 1  Parse the query    ──► extract outlet, beat, deadline, specific question, word limit
Step 2  Assess fit         ──► gate: is this actually on-topic for the brand/spokesperson?
Step 3  Draft (CAB)        ──► Credentials → Answer → Bite
Step 4  Self-review        ──► journalist pet-peeve checklist
Step 5  Present + log      ──► output + optional save to ./pr/haro-log.md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, proof points, positioning, and ICP. Also look for a `personas.md` or `style-guide.md` companion; if a spokesperson bio is recorded there, use it — do not invent credentials.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user for: (1) brand name + one-sentence positioning, (2) spokesperson name + title + 2–3 real credentials, (3) 3 voice adjectives + banned words. Always prefer the skill call.

### Step 1 — Parse the query

Extract and surface explicitly:
- **Outlet / journalist** (if provided)
- **Deadline** — flag if less than 4 hours away
- **Beat / topic** (the actual subject of the story)
- **Specific question asked** — quote it verbatim; this is what the answer must address
- **Word or format constraints** (e.g. "under 150 words," "one quote only")
- **Embargo or exclusivity note**, if any

If any of these are missing and they affect the response (especially deadline), ask before drafting.

### Step 2 — Fit gate

Before writing, assess fit in one sentence:
- Does the spokesperson have **genuine authority** on this specific topic? (Not adjacent, not aspirational — actual.)
- Is the brand's story **actually relevant** to what the journalist is reporting on?

If fit is weak, say so clearly and offer to either sharpen the angle or suggest a different spokesperson. Do not paper over poor fit with credential inflation.

---

## The CAB Formula (core framework)

Every source response is three parts, in this order. No preamble.

### C — Credentials (2–3 sentences, ~40–60 words)

Answer the journalist's implicit first question: "Why should I trust this person on this topic?"

- Name, title, company, and the **specific experience or credential** that qualifies them for *this query* — not a generic bio
- One data point that proves the credential (years, scale, outcomes) using real proof from brand-brain; mark anything unverified `[verify]`
- No "passionate about," no "serial entrepreneur," no buzzwords the journalist will delete

```
[Name] is [title] at [Company], where [they/she/he] [specific relevant credential — e.g., "leads email retention for 25,000+ ecommerce stores"]. [Supporting proof point — e.g., "The platform has processed over X billion push notifications [verify]."]
```

### A — Answer (3–6 sentences, ~80–120 words)

This is the substance. Answer the **exact question asked** — not the question you wish they'd asked.

- Open with the direct answer (the lede), not a hedge
- Support with 1–2 specific data points, examples, or mechanisms — real or `[verify]`
- Stay on the outlet's beat and the story's angle; don't pivot to your product
- If the brand is relevant to the answer, earn the mention through the answer's substance — don't announce it
- Use the brand's voice (adjectives from brand-brain), honor banned words

### B — Bite (1–2 sentences, ~25–40 words)

A standalone, quotable sentence the journalist can drop into the story without editing.

- Self-contained: reads naturally without knowing the question
- Specific, not abstract — a number, a contrast, or a concrete image lands better than a generality
- Opinionated: journalists want a point of view, not a hedge
- No company name required (often better without); no calls to action
- No jargon that would need a footnote

---

## Full output format

```
**Source response — [outlet/query topic] — [date]**

**Spokesperson:** [Name], [Title], [Company]
**Deadline:** [deadline or "not specified"]
**Fit note:** [one line — why this spokesperson is the right source]

---

[Credentials paragraph]

[Answer paragraph]

**Quotable:**
"[Bite]"

---

**Contact:** [name] | [email if user provides] | [company URL]
```

Keep the total body (credentials + answer) under 200 words unless the query explicitly asks for more. The bite is always set apart.

---

## Logging (optional, on request)

When the user says "save this" or "log it," append to `./pr/haro-log.md`:

```markdown
## [Date] — [Outlet / Query topic]
- **Query:** [paste or one-line summary]
- **Spokesperson:** [name + title]
- **Deadline:** [deadline]
- **Sent:** [yes/pending]
- **Outcome:** [placement / no response / follow-up needed]
- **Response:**
[full response body]
---
```

This builds a reusable source library: patterns that get placements, credentials that resonate, bites worth recycling.

---

## Journalist pet-peeve checklist (self-review gate — run before presenting)

| Check | Pass condition |
|---|---|
| Answers the exact question | The lede of the Answer section addresses the specific query, not an adjacent topic |
| Credentials are specific | Credential ties directly to *this* query's topic, not a generic bio |
| Under 200 words (body) | Credentials + Answer together ≤ 200 words, or query explicitly allows more |
| Bite is standalone | Quotable reads naturally without context; no dangling pronouns |
| No unsubstantiated superlatives | No "leading," "best-in-class," "#1," "revolutionary" without proof; rest `[verify]` |
| No PR preamble | Response opens with credentials, not "Hi [journalist], I loved your article on…" |
| Deadline flag | If deadline < 4 hours, flagged prominently before the draft |
| Brand voice honored | Voice adjectives from brand-brain used; no banned words |
| Proof is real | Every number is sourced from brand-brain's proof digest or marked `[verify]` |

---

## Principles

- **Answer the question, not your pitch.** Journalists will use the response or they won't. A response that answers their question gets used; one that pivots to product features does not.
- **Credentials qualify, they do not sell.** The credential section exists to establish authority on *this topic*, not to market the brand.
- **One bite, one point.** A quotable with two claims gets edited to one by the journalist — usually the weaker one. Make the choice yourself.
- **Real proof or `[verify]`.** Never invent statistics. If a number isn't in brand-brain's proof digest, mark it `[verify]` and tell the user to confirm before sending.
- **Brand-brain first.** No response before it returns. Its voice + banned words override everything here.
- **Fit gate is non-negotiable.** A mediocre response from the right spokesperson beats a polished response from the wrong one.

---

## What not to do

- Don't open with "Hi [journalist name]" or any pleasantry — journalists read responses stripped of email salutations.
- Don't include a product pitch, demo request, or link to a landing page in the response body.
- Don't fabricate credentials or inflate titles — a journalist who fact-checks will ghost the source permanently.
- Don't write a quote that requires context to make sense ("It's a game-changer" — for what?).
- Don't skip the fit gate and write for a query where the spokesperson has no genuine authority.
- Don't reimplement brand resolution here — call `brand-brain`.
- Don't exceed the outlet's stated word limit; if none stated, stay under 200 words body.

---

## Quality checklist

- [ ] `brand-brain` called and returned before drafting?
- [ ] Query parsed: outlet, deadline, exact question, constraints all noted?
- [ ] Fit gate passed (genuine spokesperson authority on this specific topic)?
- [ ] CAB structure followed: Credentials → Answer → Bite, no preamble?
- [ ] Credentials specific to *this* query, not generic?
- [ ] Answer opens with a direct response to the exact question asked?
- [ ] Bite is standalone, opinionated, under 40 words, no jargon?
- [ ] Body (credentials + answer) under 200 words (or query allows more)?
- [ ] All proof sourced from brand-brain; unverified items marked `[verify]`?
- [ ] Voice adjectives used; banned words absent?
- [ ] Deadline flagged if under 4 hours?
- [ ] Log appended to `./pr/haro-log.md` if user requested?
