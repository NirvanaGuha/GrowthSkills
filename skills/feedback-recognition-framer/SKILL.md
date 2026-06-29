---
name: feedback-recognition-framer
description: >
  Turns a raw observation, incident note, or deliverable review into one of three structured
  artifacts: (1) SBI-formatted constructive feedback ready to deliver in a 1:1 or async note,
  (2) a performance-review paragraph or full peer-review response, or (3) a genuine public
  shoutout for Slack, email, or LinkedIn. Uses the Situation–Behavior–Impact (SBI) framework
  as the spine for all three modes so the output is specific, non-judgmental, and
  action-oriented — not vague praise or hard-to-hear bluntness. Brand voice from brand-brain
  sets tone and channel conventions; quick-tone-softener-diplomacy-pass is called on any
  constructive draft before hand-off. Use whenever the user says "write feedback for X,"
  "help me say this to my colleague," "write a performance review," "shoutout for Slack,"
  "recognition message," "I want to flag this," "how do I frame this," or hands you notes
  about someone's work and asks how to communicate it.
---

# Feedback & Recognition Framer

Raw observation in, polished communication out — specific enough to be useful, structured enough
to survive being read cold. Every draft uses SBI so the person receiving it understands exactly
what happened, what they actually did or said, and why it mattered — not a verdict.

Three modes, one framework:

1. **Constructive** — SBI feedback for 1:1 delivery or async note, with a forward-looking
   "what I'd like to see" close.
2. **Review** — performance-review paragraph or peer-review response shaped by SBI evidence
   with a balanced strengths + growth structure.
3. **Recognition** — genuine shoutout for any channel (Slack, email, LinkedIn, all-hands)
   where specificity is what makes praise land.

---

## Skills this calls

- **`brand-brain`** (required) — loads brand/team voice, tone adjectives, banned words, and
  channel conventions. Feedback to a team member at a startup sounds different than at an
  enterprise; brand-brain carries that context. Do not write a draft before it returns.
- **`quick-tone-softener-diplomacy-pass`** (required on constructive drafts) — runs the
  diplomacy pass after drafting; flags anything that reads as a personal attack or
  over-hedged to uselessness. Call it before presenting the final output.
- **`loom-async-video-script-writer`** *(optional)* — if the user wants to deliver feedback
  as a Loom rather than text, hand the SBI draft to this skill for a spoken script.
- **`escalation-note-change-announcement-drafter`** *(optional)* — if the situation has
  escalated beyond a peer exchange (e.g. a manager-up or HR paper trail), route there instead.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain
Step 1  Classify the mode   ──► Constructive | Review | Recognition
Step 2  Extract SBI anchors from the raw input
Step 3  Draft in the right format
Step 4  Constructive only: call quick-tone-softener-diplomacy-pass
Step 5  Present + offer alternates or delivery coaching
```

### Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned voice adjectives,
banned words, and channel conventions to calibrate tone. A company that values directness gets
a tighter SBI close; one that values warmth gets more acknowledgment before the impact
statement.

**Fallback:** if brand-brain is absent, read `~/.brandbrain/brands/.active` + that brand's
`brand.md`. If neither exists, ask the user two questions — (a) "How direct is this team's
communication culture (1 = very diplomatic, 5 = very direct)?" and (b) "What channel is this
for?" — and proceed inline. Prefer the call.

### Step 1 — Classify the mode

| Signal words / inputs | Mode |
|---|---|
| "feedback," "I need to tell them," "how do I frame," behavior notes | Constructive |
| "performance review," "peer review," "360," "self-review," review cycle | Review |
| "shoutout," "recognize," "appreciation," "kudos," "great job" context | Recognition |

When ambiguous: ask one question ("Is this positive, constructive, or for a formal review?").
Default to Constructive when the input describes something that went wrong.

---

## The SBI spine (shared by all modes)

**Situation — Behavior — Impact** (Center for Creative Leadership, CCL [verify exact wording]).

| Element | What it answers | Common failure |
|---|---|---|
| **Situation** | When/where? Specific context, not "you always" | Too vague ("last week") or too broad ("in general") |
| **Behavior** | What did they do/say, observable, not interpreted? | Attributing motive ("you clearly didn't care") |
| **Impact** | What was the effect on the team, project, or outcome? | Skipping this — impact is what makes feedback worth giving |

**The forward close (Constructive only):** after Impact, add a specific, concrete "what I'd
like to see" — not "be better at X" but "next time, doing Y would help because Z." This is
a forward-looking close we add to SBI here (our working extension); the underlying feed-forward idea is well established, but the "SBI+1" label is ours, not an established CCL term.

**For Recognition:** SBI proves the praise. Without Situation + Behavior, a shoutout is just
noise. Impact tells the audience *why it mattered*, which is what makes public recognition
stick.

---

## Mode 1 — Constructive feedback

### Inputs needed
- Observable behavior (not hearsay, not interpreted intent)
- The situation it happened in (project, meeting, date range)
- The impact (on the team, work, or outcome — not on your emotions alone)
- Delivery channel: async note, 1:1 spoken, or Loom

### Output format
```
Situation:  [1–2 sentences, specific context]
Behavior:   [observable action or words, no motive attribution]
Impact:     [effect on the work/team/outcome]
What I'd like to see:  [concrete, forward-looking, one action]

Optional delivery note: [how to open the conversation if spoken]
```

### Craft rules
- **One behavior per note.** Don't stack three issues into one SBI; dilution hides the
  most important one.
- **Observable only.** "You said X in front of the client" not "you were dismissive." If
  the user gives interpreted language, extract the underlying observable and flag the swap.
- **Impact without drama.** The team missed a deadline / the client asked a follow-up
  question / the metric shifted. Not "it was devastating." Calibrate to the brand-brain
  directness setting.
- After drafting: invoke `quick-tone-softener-diplomacy-pass` and apply its suggestions
  before presenting.

---

## Mode 2 — Performance / peer review

### Inputs needed
- Review format: peer (360), self-review support, or manager writing about direct report
- Review questions or competencies (if the review tool has set prompts)
- Observations, incidents, or deliverables to draw on (can be messy notes)
- Tone: honest/direct vs. formal HR register

### Output format
For each competency or open-ended question:
```
[Strength paragraph]: SBI evidence of what they did well → impact
[Growth paragraph]:   SBI-grounded observation → what better looks like
```
Or if writing a narrative performance summary:
```
Overall framing sentence (sets fair-witness tone)
2–3 strength evidence paragraphs (each SBI-grounded)
1–2 growth areas (SBI + what better looks like)
Closing forward statement
```

### Craft rules
- **Evidence first, verdict last.** The paragraph earns its conclusion with behavior + impact
  before it names the competency level.
- **Balance is not enforced arithmetic.** If the person had a genuinely strong quarter, write
  that. Don't manufacture growth areas to seem balanced.
- **Ambiguous language trap.** "Great attitude," "culture add" are not SBI-anchored and
  won't survive HR review. Replace with observable behavior.
- HR-register mode: passive constructions are acceptable to reduce personalness; direct mode:
  active voice throughout.

---

## Mode 3 — Recognition / shoutout

### Inputs needed
- What the person did (the behavior, even briefly)
- When/what project (the situation)
- Why it mattered (the impact)
- Channel: Slack (public channel), email, LinkedIn, all-hands, peer nomination form

### Output format
```
[Opening — name + what they did, no generic opener]
[Situation — the context]
[Behavior — what specifically they did]
[Impact — why it mattered to the team or outcome]
[Close — human, not sycophantic]
```

### Channel calibrations
| Channel | Length | Tone | Notes |
|---|---|---|---|
| Slack #kudos | 3–5 sentences | Warm, specific | Tag the person; no emojis unless brand allows |
| Email (manager to team) | 1 short para | Professional warmth | Spell out the impact for readers who weren't there |
| LinkedIn | 3–5 paras | Public, professional, concrete | Call out their name, title, org; no hollow superlatives |
| Peer nomination form | Match word limit | Formal-ish, SBI-grounded | Will be read cold by reviewers who don't know the work |
| All-hands / verbal | Talking points only | Conversational | 3 bullet SBI; don't read a paragraph aloud |

---

## Persistence

Save outputs when the user asks, or automatically for review-cycle work:
- Constructive notes: `./feedback/[person-slug]-[date].md`
- Review drafts: `./reviews/[person-slug]-[cycle].md`
- Recognition: inline by default; save on request to `./recognition/[person-slug]-[date].md`

Never overwrite without prompting.

---

## Principles (Non-Negotiable)

- **SBI before output.** Every draft extracts Situation, Behavior, Impact from the raw input
  before writing. If the user's input doesn't have all three, ask for the gaps — or note what
  you inferred and let them correct it.
- **Observable behavior only.** Do not attribute motive, personality, or intent. If the user's
  input does, extract the observable and flag the swap.
- **Brand-brain first.** Tone, directness level, channel conventions, and banned words come
  from brand-brain, not from defaults.
- **Diplomacy pass on constructive.** Never skip `quick-tone-softener-diplomacy-pass` on a
  constructive draft — it costs nothing and prevents the note from landing as an attack.
- **No evidence, no praise.** Generic kudos ("great job," "amazing work") with no SBI anchors
  is noise. Refuse to write it; ask for the behavior instead.
- **One behavior per note.** Stacking issues hides the most important one and puts the
  recipient on defense.

## What Not to Do

- Don't write a constructive feedback draft without running the diplomacy pass.
- Don't invent behavior or impact — if the user's input is too thin, ask.
- Don't write "you always" or "you never" — these are not SBI-grounded and trigger defensiveness.
- Don't produce hollow recognition copy ("You're such a rockstar!") without SBI evidence.
- Don't conflate modes — a shoutout is not a soft performance review; keep recognition
  unambiguously positive.
- Don't escalate to HR framing (formal documentation language) unless the user requests it
  or the input signals a serious incident; route those to `escalation-note-change-announcement-drafter`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice + banned-words honored; channel conventions applied?
- Mode correctly classified from user input; ambiguity resolved before drafting?
- All three SBI elements present and observable (no motive attribution)?
- Constructive: forward-looking close written; `quick-tone-softener-diplomacy-pass` run?
- Review: evidence precedes verdict; ambiguous language replaced with SBI observation?
- Recognition: specific behavior + impact present; channel calibration applied; no hollow superlatives?
- One behavior per note (constructive); no stacking?
- Output saved to the right relative path if this is a review-cycle or user asked to save?
