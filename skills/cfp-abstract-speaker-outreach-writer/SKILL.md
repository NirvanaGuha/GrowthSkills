---
name: cfp-abstract-speaker-outreach-writer
description: >
  Speaker topic, conference, bio notes → polished abstract, session title, formatted bio,
  and personalized recruiter outreach with follow-up. Two modes: Submit (you're applying to
  a CFP) and Recruit (you're sourcing speakers for your own event). Applies the Kirby Ferguson
  "constraint as creative engine" framework — the tightest, most specific angle always beats
  the broad one in programme committee review. Loads brand context via brand-brain so every
  abstract and outreach lands in the right voice and proves the right proof points. Use
  whenever the user says "write my CFP abstract," "submit a conference talk," "speaker
  outreach," "find speakers," "CFP application," "recruit a speaker," "write a conference
  bio," "my talk proposal," or "conference abstract."
---

# CFP Abstract & Speaker Outreach Writer

Conference slots are decided by committee members reviewing 200+ submissions in 90-second bursts. Broad topics die; narrow, specific, proof-backed abstracts win. This skill writes the words that pass that filter — whether you're submitting a talk or recruiting a speaker to fill a slot.

Two modes, one framework: the abstract, the bio, the title, and the outreach are a coordinated package. Getting one right while leaving the others vague is how good speakers lose slots they deserved.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, proof points, ICP, positioning, and banned words before any copy is written. This skill does not derive brand context itself.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for: speaker topic, ICP/audience, one concrete proof point or case study, and three voice adjectives + any banned words.
- **`icp-persona-builder`** — called in Submit mode to confirm the attendee persona the abstract should speak to, if not already in the brand digest.
- **`proof-vault`** — retrieves verified proof points (stats, case studies, customer quotes) to embed in the abstract. Synthesize inline when absent; mark anything unconfirmed `[verify]`.
- **`cold-outreach-sequence-architect`** — called in Recruit mode to structure the multi-touch outreach and follow-up cadence for speaker sourcing.
- **`headline-hook-generator`** — called to pressure-test session title hooks against click-and-submit rate patterns from programme committee review psychology.
- **`campaign-brief-builder`** — called when the user wants the CFP effort wrapped into a broader event pipeline (e.g., an anchor talk for a product launch).

---

## How a run works

```
Step 0  Load the brand     ──► call brand-brain
Step 1  Determine mode     ──► Submit (applying) | Recruit (sourcing)
Step 2  Gather inputs      ──► structured intake (see below); ask only what's missing
Step 3  Do the work        ──► title → abstract → bio | outreach → follow-up
Step 4  Self-review        ──► quality checklist before presenting
Step 5  Persist artifacts  ──► save to ./cfp/[conference-slug]-[year].md on request
```

### Step 0 — Load brand (always first)

Invoke `brand-brain` (Skill tool) before producing any copy. Use the returned voice adjectives, banned words, proof points, and ICP as hard constraints. Do not write any abstract, title, bio, or outreach until it returns.

### Step 1 — Determine mode

- **Submit mode** — the user (or their speaker) is applying to speak at someone else's event.
- **Recruit mode** — the user is running an event and wants to attract or pitch specific speakers.

When unclear, ask one question: "Are you submitting a talk or inviting a speaker?"

---

## Submit mode — CFP application package

### Intake (ask only what's missing)

| Field | Why it matters |
|---|---|
| Conference name + URL | Tones the abstract to the audience; pulls the stated CFP criteria |
| Track / format | Solo talk, panel, workshop, lightning — word limits differ |
| Core argument (one sentence) | The thesis; if the user can't state it in one sentence, the abstract will be vague |
| Proof anchor | The single stat, case study, or experiment that makes the claim credible |
| Target attendee | Job title + awareness level (practitioner vs. executive vs. newcomer) |
| Speaker name + current role | Bio inputs |
| Speaker's past talks / brand mentions | Credibility signals for the bio |

Do not ask for all seven upfront. Check what the brand digest already supplies, infer conference audience from the URL if provided, and ask only the remaining gaps in one batch.

### The framework: Angle Sharpening (after Ferguson / rhetorical narrowing)

1. **Identify the thesis.** A single falsifiable claim the talk proves — not a topic. "How we cut onboarding drop-off 40% by removing a field" is a thesis; "onboarding best practices" is a topic.
2. **Sharpen the angle.** Apply one sharpening pass: Who specifically? What exact constraint or counterintuition? What proof makes it non-obvious? The sharpest version wins the committee slot.
3. **Stress-test the title.** A programme committee member should be able to explain what they'll learn in one sentence from the title alone. Run `headline-hook-generator` on the title shortlist; pick the one that is specific, benefit-forward, and not a question (questions test poorly in CFP review data [verify]).

### Deliverables

**1. Session title** — one primary + two alternates. Each: ≤10 words, specific claim or vivid outcome, no vague superlatives ("transformative," "game-changing").

**2. Abstract (public-facing)** — 150–250 words unless the CFP states otherwise.

Structure (Problem → Insight → Method → Proof → Takeaway):
- **Hook sentence**: the counterintuitive observation or specific problem, not "In today's world…"
- **Setup (2–3 sentences)**: the real stakes for *this* attendee persona
- **Thesis/method (2–3 sentences)**: what the speaker actually did, the specific framework or decision, named if possible
- **Proof anchor (1–2 sentences)**: the concrete result — number, company name or anonymized tier, timeframe. Mark unconfirmed figures `[verify]`.
- **Takeaways (3 bullets)**: what attendees leave able to do, starting with a verb ("Diagnose," "Implement," "Avoid")
- **Closing hook**: one sentence that makes the reader want to be in the room

Voice, banned words, and proof: honor the brand digest exactly.

**3. Bio (speaker-facing, 100–150 words)**

- Third-person, present tense
- Lead with the proof credential most relevant to the talk (not the most impressive one overall)
- One line of personal texture (not mandatory; skip if the brand voice is austere)
- Follows brand voice adjectives; removes any term on the banned-words list

**4. Reviewer-facing abstract / programme note (optional, 50–75 words)**

A tighter version written for the committee, not the attendee — emphasizes fit for the track, uniqueness vs. typical submissions in that category, and the speaker's credibility for this specific claim. Offer this proactively for competitive conferences.

---

## Recruit mode — speaker sourcing package

### Intake

| Field | Why it matters |
|---|---|
| Event name, date, format | Frame the ask; set the value prop |
| Session slot details | Length, track, audience size |
| Target speaker (name + company, or profile description) | Personalization anchor |
| Why this speaker specifically | The hook for the outreach |
| What you offer | Speaking fee / travel / exposure / audience access — be honest |
| Follow-up tolerance | 1 touch or 2-touch with 7-day gap |

### Deliverables

**1. Personalized outreach (email or LinkedIn DM)**

Structure: compliment-free opener → the specific why-you → the specific opportunity → concrete ask (call, yes/no) → one friction-reducer.

Keep to ≤150 words. The first sentence must not be "I hope this finds you well," "We're huge fans," or any generic opener. It should reference something specific about the speaker's recent work or stated interest.

Call `cold-outreach-sequence-architect` to validate the structure and add the follow-up timing logic.

**2. Follow-up (7–10 days, if no reply)**

One paragraph. Adds new information (updated lineup, confirmed sponsor, relevant stat about the audience) rather than just bumping. Never "just checking in."

**3. Speaker brief / invitation packet (on request)**

- Event overview and audience profile (from brand digest + event details)
- What's expected (prep time, A/V, exclusivity)
- What's provided (travel, fee, recording rights)
- Timeline and decision deadline

---

## Principles

- **Specific beats broad.** Every vague word in a title or abstract is a vote for the submission above yours. Name the method, the audience, the result.
- **Brand-brain first.** No copy before the brand digest returns. Voice and proof are hard constraints.
- **Proof or silence.** Real numbers and named outcomes only; everything else is `[verify]` or removed.
- **The committee is the first reader.** The public abstract sells attendees; but first it has to sell a committee member reading submission #174 at 11 PM. Write for both.
- **Outreach earns attention, it doesn't demand it.** The recruiter is asking for a significant ask (speaker prep = 20+ hours). Treat it accordingly; never be casual about the time cost.
- **One thesis per abstract.** If the talk makes two claims, the abstract tries to do too much. Flag it; help the user choose.

---

## What not to do

- Don't write the abstract before `brand-brain` returns the brand context.
- Don't open the abstract with "In today's [adjective] landscape," "I'm excited to share," or a rhetorical question — these are programme committee red flags.
- Don't invent case study numbers or company names to fill the proof anchor — use `[verify]` or ask for the real data.
- Don't reimplement brand scanning or voice derivation here — call `brand-brain`.
- Don't write a bio that leads with the speaker's most prestigious credential if it's irrelevant to the talk's thesis.
- Don't write recruiter outreach that opens with "We're huge fans" or "I hope this finds you well."
- Don't omit the follow-up plan in Recruit mode — one-touch speaker sourcing underperforms significantly [verify].
- Don't pad the abstract to hit a word count — every sentence must earn its place.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and brand digest loaded before any copy was written?
- Voice adjectives honored; banned words absent from all deliverables?
- All proof numbers real and attributed, or marked `[verify]`?
- Session title: ≤10 words, specific outcome stated, no vague superlatives?
- Abstract: hook is specific (not a landscape statement); proof anchor present; takeaways start with verbs; within word limit?
- Bio: third-person, present tense; leads with the credential most relevant to the talk's thesis?
- Submit mode: reviewer-facing note offered for competitive conferences?
- Recruit mode: first sentence of outreach is specific to the speaker; follow-up adds new information rather than bumping?
- Artifacts saved to `./cfp/[conference-slug]-[year].md` if the user asked for persistence?
