---
name: quick-tone-softener-diplomacy-pass
description: >
  Takes a blunt, terse, or potentially inflammatory Slack message, email, or internal memo draft and
  returns a diplomatically rephrased version that lands well without burying the core message — plus
  optional memo formatting when the output needs to travel as a formal document. Built around the
  SBI (Situation–Behavior–Impact) diplomatic communication framework combined with the Assertiveness
  Ladder (direct → firm → diplomatic → collaborative), so the output is never mealy-mouthed — it
  stays assertive and honest while removing the friction that causes defensive reactions. Works for
  difficult feedback, escalations, pushback to leadership, cross-functional coordination friction,
  and any draft that made the user pause before hitting Send. Use when the user says "soften this,"
  "make this less blunt," "diplomacy pass," "rewrite this so it doesn't sound aggressive," "how do
  I say this without starting a war," "tone check," "help me phrase this professionally," or pastes
  a draft with a note that it feels off.
---

# Quick Tone-Softener & Diplomacy Pass

Paste the draft; get back a version people can actually hear. Not watered-down — assertive, honest, and stripped of the unnecessary friction that turns a fair message into a defensive reaction. Every pass is grounded in SBI framing and calibrated to the relationship register the sender names.

This skill edits for tone and landing, not for substance. It does not hide bad news, walk back legitimate criticism, or manufacture warmth that isn't there. If the underlying message is unreasonable, it says so.

---

## Skills this calls

- **`brand-brain`** (light-touch, required) — loads the user's preferred voice adjectives and any banned words or formality conventions so shareable outputs (forwarded emails, memos) honor the brand register. For purely internal, no-forward messages, use only the voice adjectives and skip brand positioning. If brand-brain is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md`, or ask the user for three voice adjectives and a formality target.
- **`de-slop-humanize-pass`** — call after drafting when the rewrite risks sounding like AI-polished corporate speak; that skill strips the filler.
- **`meeting-prep-follow-up-pack`** — if the context is post-meeting fallout or a follow-up that escalated, pull the meeting notes from there first.
- **`quote-polisher`** — when the user needs one specific line (an exec quote, a one-sentence position) rewritten rather than a full draft.

---

## How a run works

```
Step 0  Load brand voice  ──► call brand-brain (voice adjectives + banned words only)
Step 1  Diagnose the draft ──► name exactly what is spiking the tone
Step 2  Set the register   ──► confirm relationship level + desired outcome
Step 3  Rewrite via SBI    ──► structure + soften without removing the point
Step 4  Present + explain  ──► show before/after + what changed and why
Step 5  Optional: memo fmt ──► if forwarding as a formal document, apply structure
```

### Step 0 — Load brand voice (always first)

Invoke the `brand-brain` skill to get voice adjectives and banned words. Apply them to any output that might be shared externally or forwarded. For a quick internal Slack message, use only the voice register (direct/warm/formal/etc.) and skip brand positioning — do not inject product messaging into an interpersonal note.

**Fallback:** read `~/.brandbrain/brands/.active` + `brand.md`; if absent, ask: "Three words that describe how you want to sound professionally?" and proceed.

---

## Step 1 — Diagnose the draft

Read the draft and name in one line what is causing the tone problem. Use the taxonomy below — be specific, not generic ("this sounds harsh").

| Signal | What it looks like |
|---|---|
| **Blunt accusation** | "You dropped the ball on this." / "This was handled wrong." |
| **Passive-aggressive hedge** | "Not sure if you saw my message from Tuesday…" |
| **Unexplained ultimatum** | "Fix this by EOD or I'm escalating." |
| **Venting embedded in a request** | "I'm exhausted dealing with this every sprint." |
| **Condescension framing** | "As I mentioned…" / "Per my last email…" |
| **Missing acknowledgment** | Pure criticism or pure demand with zero context or empathy signal |
| **Overly deferential** | So hedged the ask disappears: "…if it's not too much trouble, maybe?" |

Name the signal(s). Do not guess at intent — describe the effect on the reader.

---

## Step 2 — Set the register

Before rewriting, confirm or infer from context:

1. **Relationship level** — peer / direct report / skip-level / external partner / executive above sender
2. **Channel** — Slack (informal, fast) / email (more considered) / formal memo (document of record)
3. **Desired outcome** — inform · request action · give feedback · escalate · push back · repair relationship
4. **Urgency** — urgent (keep short) / non-urgent (can invest in warmth)

If the user provides context, use it. If not, state your inference and flag it: *"Reading this as peer-level email, action request, non-urgent — correct me if not."*

---

## Step 3 — Rewrite via the SBI Assertiveness Ladder

### The SBI structure (core technique)

Every diplomatic message that carries a real point works on three rails:

- **Situation** — neutral, shared-reality framing of the context ("In the Q3 launch review…")
- **Behavior** — specific, observable, non-attributing description of what happened ("the deliverable arrived two days after the agreed date…")
- **Impact** — concrete consequence on the work, not a character judgment ("which meant we had to compress QA and we shipped three bugs that hit customers")

SBI removes the accusation without removing the fact. It replaces "you dropped the ball" with a sequence of agreed facts that leads the reader to the same conclusion without triggering defensiveness.

### The Assertiveness Ladder (calibrate to register)

Choose the rung that matches relationship level and desired outcome — never go below the minimum needed to be heard:

| Rung | When to use | Example move |
|---|---|---|
| **Collaborative** | Repair, ambiguity, peers you want to stay close to | "Can we align on…" / "I want to make sure we're set up to…" |
| **Diplomatic** | Standard professional ask, feedback, most cross-functional | "I noticed…" / "I want to flag…" / "It would help me if…" |
| **Firm** | Clear ask with stakes, escalation path named | "I need X by Y so that Z doesn't happen." |
| **Direct** | Urgent, executive-level, formal record | Plain declarative: "This is blocked. I need a decision by Friday." |

Never let diplomacy drop the message below Firm when the situation genuinely requires it. Diplomatic ≠ toothless.

### Rewrite rules

- Open with a neutral Situation or an acknowledgment; never open with the complaint.
- Replace attribution ("you did X wrong") with behavioral observation ("X happened" / "the result was X").
- Make the ask explicit and single — one request per message.
- Add one genuine acknowledgment of the other person's context or workload where it is warranted — never performative.
- Cut softeners that undermine the ask: "just," "sorry to bother you," "I might be wrong but," "if that's okay."
- Keep hedges that signal openness: "Let me know if I've missed something," "happy to talk through it."
- Match length to channel: Slack messages should be shorter after rewriting, not longer.

---

## Step 4 — Present before / after

Always show:

```
ORIGINAL
[paste]

WHAT'S CAUSING FRICTION
[1–2 bullets from the diagnosis taxonomy]

REWRITE — [register label, e.g. "Diplomatic / peer-level email"]
[rewritten version]

WHAT CHANGED
[2–3 lines: specifically what moved and why it lands better]
```

If the original had multiple problems, call each one by name.

### Optional: alternate rung

If the diagnosed rung is Diplomatic but the user might need to go Firm (e.g., pattern of ignored requests), offer a Firm alternate in a collapsible note: *"If this has happened before and you need a harder edge, here's the Firm version: …"*

---

## Step 5 — Optional memo formatting

When the user signals this is going up the chain, needs a paper trail, or will be forwarded:

```
To:
From:
Date:
Re:

[Opening: context in one sentence]
[Body: SBI + ask, max 3 paragraphs]
[Closing: next step + your availability]
```

Apply brand voice from `brand-brain` to tone. Keep it short — a memo that needs scrolling loses authority.

Save memo output to `./ops/[slug]-memo-[YYYY-MM-DD].md` if the user says "save" or "document this."

---

## Principles

- **Keep the substance.** Softening the tone is not the same as walking back the point. If the original criticism is fair, it stays — reframed, not removed.
- **Assertive, not obsequious.** Diplomatic does not mean deferential. The Assertiveness Ladder keeps a floor on directness.
- **SBI over attribution.** Specific observable behavior + impact beats character judgment every time. It's harder to argue with facts.
- **One ask per message.** Multiple requests in one message split attention and make it easy to action one and ignore the rest.
- **Match the channel.** Slack is not email. Email is not a memo. Length and warmth signals are calibrated per channel.
- **Brand-brain voice honored on shared outputs.** Internal Slack DMs are exempt; anything that travels gets the brand register applied.
- **Flag if the message itself is the problem.** If the content is unreasonable — an impossible deadline, an unfair accusation, a policy violation — say so rather than giving it a diplomatic coat of paint.

---

## What not to do

- Do not remove legitimate criticism to avoid discomfort — that defeats the purpose.
- Do not manufacture warmth: "Hope this finds you well" on an escalation reads as mockery.
- Do not pad — longer is not more diplomatic; it is harder to read and often more passive-aggressive.
- Do not use "per my last email" or "as previously discussed" — flag them in the diagnosis and cut them.
- Do not rewrite the original as a question if it should be a statement ("Could we maybe think about potentially…").
- Do not apply brand positioning to interpersonal messages — brand-brain voice adjectives only, no product copy.
- Do not guess at intent behind the original author's bluntness and editorialize it into the rewrite.

---

## Quality checklist

- `brand-brain` called; voice adjectives + banned words applied to any shareable output?
- Draft diagnosed with a specific signal from the taxonomy, not a generic "this sounds harsh"?
- Register (relationship level, channel, outcome, urgency) confirmed or stated as an inference?
- Rewrite uses SBI structure: neutral Situation → observable Behavior → concrete Impact?
- Assertiveness Ladder rung chosen and calibrated to register — not softer than needed?
- Before/after shown with named changes?
- No legitimate criticism removed; single explicit ask preserved?
- Memo format applied and saved if requested?
- `de-slop-humanize-pass` called if the rewrite risks sounding AI-polished?
