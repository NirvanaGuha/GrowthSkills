---
name: cold-outreach-sequence-architect
description: >
  Designs and writes full multi-touch cold outreach sequences for a target ICP segment — email + LinkedIn,
  personalized per persona, mapped to a stated solution and channel mix. Takes an ICP segment definition,
  your solution, and a channel mix preference; returns a complete, ready-to-send sequence: subject lines,
  email bodies, LinkedIn connection notes, follow-up messages, and the messaging angle behind each touch.
  Sequences are built on the PAVE message-arc (Problem → Aspiration → Validation → Escalation) plus a
  deliverability layer, so every message earns attention with a specific, relevant insight — and actually
  lands in the inbox — rather than relying on generic flattery or product-first pitching. Calls brand-brain for
  voice and proof, account-dossier-builder for account-level research, and objection-library-builder for
  pre-emptive objection handling. Artifacts save to ./outreach/. Use when someone says "write cold emails,"
  "build an outreach sequence," "LinkedIn + email cadence," "cold prospecting copy," "SDR playbook," or
  "personalized outreach for [ICP/segment/persona]."
---

# Cold Outreach Sequence Architect

Build sequences that open conversations, not sequences that get filtered. Every touch earns the next one: a specific insight, a clear connection to a pain the persona actually has, and a single low-friction ask. No spray-and-pray, no "just checking in," no invented flattery.

This skill architects the sequence strategy and writes every message. It does not source the prospect list (that is `account-list-builder-icp-scorer`) and does not decide whether the channel is right for the funnel stage (that is the human's call — but it will flag if the ask exceeds the trust level the channel can support).

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — voice, banned words, offer mechanics, proof, positioning, ICP. No message written before this returns.
- **`account-dossier-builder`** (required per target account, Step 1) — company profile, tech stack, pain hypotheses, recent news. Personalization anchors come from here, never fabricated.
- **`objection-library-builder`** (Step 2) — pre-emptive objection handling woven into middle and late touches.
- **`icp-persona-builder`** (Step 2, if personas are not already in brand.md) — confirms role-level pains, language, and awareness stage per persona.
- **`cta-variant-generator`** (Step 5) — generates the sequence's CTA options (meeting ask, content offer, reply CTA) at the right commitment ceiling.
- **`subject-line-preview-text-optimizer`** (Step 5) — stress-tests subject lines on open-rate mechanics and spam-trigger words.

---

## How a run works

```
Step 0  Brand context      ──► call brand-brain (voice, proof, offer, ICP)
Step 1  Account research   ──► call account-dossier-builder per target company
Step 2  Persona + objects  ──► call icp-persona-builder (if absent) + objection-library-builder
Step 3  Sequence strategy  ──► PAVE arc: map touches, angles, channel, segment cadence
Step 4  Deliverability      ──► warm-up state, sending identity, spam-trigger pass
Step 5  Write every asset  ──► call cta-variant-generator + subject-line-preview-text-optimizer
Step 6  Self-review        ──► quality checklist; save to ./outreach/
```

---

## Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP. Do not write a single message before it returns.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If neither exists, ask the user for: solution + differentiator · ICP role/company-type · 3 voice adjectives + banned words · pricing/offer mechanic · 1–2 real proof points.

---

## Step 1 — Account research

For every named target company, invoke `account-dossier-builder` (Skill tool). It returns: business model, tech stack, recent news, funding, headcount signals, and hypothesized pains. This is the **only** source of personalization facts. Never fabricate specifics — if research returns nothing concrete, use segment-level insight (role + industry pattern) and mark it as such, not as account-specific.

If the user provides a list, process the top account in full and note the pattern for scaling.

---

## Step 2 — Persona and objections

Confirm the target persona's role-level pain, vocabulary, and awareness stage. If `personas.md` is in the brand's companion files, read it. If not, invoke `icp-persona-builder`.

Invoke `objection-library-builder` to get the top 3–5 objections for this ICP + solution pairing. These feed into the mid-sequence and penultimate touches.

---

## Step 3 — Sequence strategy: the PAVE message-arc

PAVE is this skill's proprietary message-arc — four angles that cycle across touches so the sequence moves a prospect from "why is this in my inbox" to "let's talk" without ever pitching before it has earned the right. It is named for the arc itself (Problem → Aspiration → Validation → Escalation), not borrowed from any vendor methodology. Map the touch order before writing a word:

| Letter | Angle | What the message does |
|---|---|---|
| **P** — Problem | Opens with the pain, not the product | Earns relevance by naming something real the persona feels |
| **A** — Aspiration | Connects to the outcome they want | Moves from pain to possibility without pitching yet |
| **V** — Validation | Social proof, data, or a customer parallel | Builds credibility; uses only real proof (else `[verify]`) |
| **E** — Escalation | Addresses the objection and sharpens the ask | Clears the last barrier; one specific, low-friction CTA |

**Default structure (5-touch, 10–14 days):** alternating email/LinkedIn is a *starting* cadence, not a law. The right shape is set by the segment (see below), then by intent signals.

| Touch | Channel | Day | Angle | CTA commitment |
|---|---|---|---|---|
| T1 | Email | Day 1 | P — specific insight + pain | Soft: reply / resource |
| T2 | LinkedIn | Day 3 | A — connection note, shared context | Connection only |
| T3 | Email | Day 6 | V — proof point / customer parallel | Medium: 15-min call |
| T4 | LinkedIn | Day 9 | E — objection reframe + nudge | Reply CTA |
| T5 | Email | Day 13 | P (new angle) — pattern interrupt / breakup | Ultra-soft or close |

**Segment-specific cadences — pick the row, don't hand-wave the default:**

| Segment / signal | Touches | Window | Shape that actually works |
|---|---|---|---|
| **Enterprise / 6-fig deal** | 7–9 | 4–6 weeks | Slower, more space between touches. Add a second V (a second proof angle, e.g. peer-logo then ROI data) and a second E. Multi-thread: same arc, sent to 2–3 personas in the buying group with role-specific Problem framing. Never compress — enterprise buyers read async over weeks. |
| **SMB / founder-led** | 3–4 | 5–7 days | Collapse to P → V → E. Founders decide fast and tolerate directness; skip the Aspiration touch and put the proof point in T2. One channel is fine (email *or* LinkedIn, wherever they live). |
| **Warm intent** (site visit, content download, G2 view, job post matching your wedge) | 3 | 4–5 days | Lead T1 with the signal itself ("saw [team] is hiring 3 SDRs — usually means [pain]"), jump straight to V in T2, E in T3. The cold opener is wasted on someone already half-aware. |
| **Inbound-assisted** (replied once, attended webinar, MQL) | 2–3 | 3–5 days | Skip the cold P opener entirely; enter at A or V referencing the prior interaction. Treat as nurture, not cold. |
| **Channel-constrained** (email-only or LinkedIn-only) | keep 5 | 10–14 days | Run the full PAVE arc inside one channel; convert the LinkedIn touches to short email "bumps" (or vice versa), each carrying its assigned angle. Do not drop angles to fit the channel. |

---

## Step 4 — Deliverability: the copy is worthless in spam

A flawless sequence in the spam folder has a 0% reply rate. Before writing, confirm the sending setup; while writing, respect the rules that keep mail in the primary inbox. This is craft the copy itself cannot fix — surface it in the deliverable as a pre-flight block the operator must clear.

**Sending identity (set up once, verify every run).**
- Send cold from a **dedicated domain**, never the primary brand domain — typically a lookalike (`getbrand.com`, `brand-mail.com`). A blocklisting hits the cold domain, not the domain your invoices and customer mail ride on.
- **SPF, DKIM, and DMARC** must all be configured and passing on the sending domain. Missing or failing auth is the single fastest route to spam. If the user can't confirm all three, flag it as a blocker, not a nice-to-have.
- One mailbox should send **no more than ~30–50 cold emails/day**. Past that, mailbox providers read you as a blasting machine. Scale volume by adding mailboxes/domains, not by pushing one harder.

**Warm-up (the gate before any send).**
- A brand-new domain or mailbox must be **warmed for 3–4 weeks** before it touches a real prospect — ramp from a handful of sends/day, with automated warm-up tools generating opens and replies, so the provider builds a positive reputation. Cold-launching a fresh domain at volume torches it on day one.
- Keep the warm-up running underneath live sending; don't switch it off the moment campaigns start.
- In the deliverable's pre-flight block, state `Warm-up status: [yes / no — hold send]`. If "no," the sequence is written but **not cleared to send**.

**Spam-trigger discipline (applies to every line you write).**
- **No links or images in T1.** A bare-text first touch from an unknown sender with a link reads as bulk. Earn the click in a later touch, once a reply or engagement exists.
- Avoid spam-flag vocabulary: *free, guarantee, act now, limited time, risk-free, $$$, 100%, click here*, ALL-CAPS subject lines, and exclamation stacks. Push spammy phrasing through `subject-line-preview-text-optimizer`.
- One link maximum once links are allowed; never attach files in cold mail.
- Plain text > heavy HTML. No tracking-pixel-laden templates, no rented-list footers.
- Personalize the merge so every send is slightly different — identical mass-merged bodies trip bulk filters; the `[SLOT:]` variation in Step 5 is a deliverability feature, not just a relevance one.

**Why the body caps protect inboxing.** The ≤150 / ≤120-word caps below aren't only a readability choice. Short, link-light, low-image messages match the statistical profile of real 1:1 human email — which is exactly what spam filters are tuned to *pass*. Long, link-heavy, image-stuffed bodies match the profile of bulk marketing, which filters are tuned to catch. Tight copy is a deliverability lever as much as a persuasion one.

---

## Step 5 — Write every asset

For each touch, produce:

**Email touches:**
- Subject line (A/B pair) → optimized via `subject-line-preview-text-optimizer`
- Preview text (≤90 chars)
- Body (≤150 words for T1; ≤120 for follow-ups; mobile-first — short paragraphs, no walls of text). No link/image in T1; one link max thereafter.
- CTA → via `cta-variant-generator` at the right awareness/commitment ceiling
- Personalization slots labeled `[SLOT: company_name]`, `[SLOT: recent_news]`, etc. for mail-merge (vary them — see deliverability)

**LinkedIn touches:**
- Connection request note (≤300 chars — personalized, no pitch, finds genuine common ground)
- Follow-up message if connection accepted (≤500 chars)

**Per-touch annotation:**
- Angle and why it fits this persona at this point in the sequence
- Objection addressed (if any)
- What a positive signal looks like (reply type, click, connect) and how to route it

---

## Craft rules: what separates good sequences from noise

**The specificity test.** Every T1 must pass: "Could this have been written without knowing anything about this company or person?" If yes, rewrite it. One specific insight (a product change, a funding round, a job post, a stated priority) beats five generic value props.

**Pain before product.** T1 names a problem; it does not pitch. The product enters at T3 at the earliest, via a proof point, not a feature list.

**Commitment ladder.** T1–2 asks only for attention or a connection. T3 asks for 15 minutes. T5 offers a door to exit gracefully. Never ask for more than the channel + trust level can support.

**LinkedIn is not email.** Connection notes get one genuine shared-context line plus an implicit reason to connect — no CTA, no pitch. The message after acceptance is still conversational.

**The pattern interrupt — how to actually write one.** By the final touch, a prospect's brain has filed your earlier emails under "sales sequence" and auto-archives on sight. A pattern interrupt breaks that prediction so the message gets *read fresh*. Three moves that work, in order of strength:
1. **Change the shape.** If every prior touch was a structured pitch, send a one-line email: "Hi [name] — should I close the loop on this, or is the timing just off?" The brevity itself violates the expected pattern.
2. **Name the dynamic out loud.** Acknowledge the sequence: "This is my last note — I know I've landed in your inbox a few times." Stating the obvious is disarming because no template does it.
3. **Flip the ask direction.** Instead of requesting their time, hand them an easy exit: "A one-word 'not now' is a totally fine reply." Removing pressure is itself the interrupt.
What makes it work is the *contrast* with your own earlier touches — so write the interrupt only after the rest of the sequence exists, and make sure it doesn't read like just another follow-up. A pattern interrupt that looks like every other email isn't one.

**Breakup messages earn replies.** T5 pairs the pattern interrupt with explicit permission to say no. That psychological release — combined with the broken pattern — is why breakups have the highest reply rate in the sequence. Write it honestly: "If this isn't relevant, say the word and I'll stop."

**Voice and proof are non-negotiable.** Every message is in the brand's voice (from brand-brain). Proof is real or `[verify]`. No invented customers, invented results, or invented flattery.

---

## Deliverable format

Save to `./outreach/<brand-slug>-<icp-slug>-sequence.md`:

```
# Cold Outreach Sequence — [Brand] × [ICP Segment]
Generated: [date]  ·  Brand: [slug]  ·  Persona: [role/title]  ·  Company type: [segment]
Sequence type: [segment cadence used]  ·  Channel mix: [email + LinkedIn | email-only | etc.]

## Deliverability pre-flight (clear before sending)
Sending domain: [dedicated cold domain — not the primary brand domain]
SPF / DKIM / DMARC: [all passing? if not, BLOCKER]
Warm-up status: [yes / no — hold send]
Daily send cap per mailbox: [~30–50]

## Sequence strategy
[PAVE message-arc map — one table row per touch; note which segment cadence was applied]

## Assets

### T1 — Email (Day 1)  ·  Angle: Problem
Subject A: ...
Subject B: ...
Preview: ...
Body:
---
[body text with [SLOT:] labels]
---
CTA: ...
Personalization slots: [list]
Route if replies: [what a positive signal looks like]

### T2 — LinkedIn (Day 3)  ·  Angle: Aspiration
Connection note: ...
Post-connect message: ...

[... T3–T5 in same format ...]

## Objections addressed
[Which objections from objection-library-builder appear in which touches]

## Scaling notes
[How to apply this sequence to the rest of the account list; where to fork for a second persona]
```

---

## Principles

The non-negotiables, stated once. Everything in *Craft rules* serves these:
- **Research before writing, brand-brain first.** No personalization without a real fact; no message before the brand digest loads. Voice + banned words override everything.
- **Pain before product, one CTA per message.** The sequence earns the right to pitch; it never opens with it, and never gives a prospect two things to do.
- **Honest proof only.** Every statistic, customer name, and result is real or marked `[verify]` — invented social proof erodes trust the moment it's checked.
- **Deliverability is part of the craft.** The best copy in the spam folder converts no one. Sending identity, warm-up, and spam-trigger discipline ship with the sequence, not after it.

## What Not to Do

- Do not write T1 before `brand-brain` returns and `account-dossier-builder` has run for the target.
- Do not personalize with invented facts ("I saw your company was recently in the news about X" when you don't know).
- Do not pitch the product in T1 or T2 — earn the conversation first.
- Do not write more than 150 words per email body, and never put a link or image in T1 — both length and links cut deliverability.
- Do not use the same angle across back-to-back touches — vary PAVE deliberately.
- Do not clear a sequence to send on a cold-launched, un-warmed, or auth-failing (SPF/DKIM/DMARC) domain.
- Do not run the default 5-touch cadence on an enterprise or warm-intent segment without adjusting it — pick the right row.
- Do not write the breakup as just another follow-up; it must carry a real pattern interrupt and explicit permission to opt out.

## Quality Checklist

- `brand-brain` + `account-dossier-builder` ran first; every personalization slot is a real fact, not fabricated?
- The segment cadence was chosen deliberately (not the default 5-touch by inertia)?
- PAVE angles mapped and each touch advances the arc; no repeated angle back-to-back?
- T1 passes the specificity test (cannot be generic-copy-pasted to any company)?
- Commitment ladder honored: soft → medium → escalation, channel-appropriate?
- Deliverability pre-flight filled: dedicated domain, SPF/DKIM/DMARC passing, warm-up done, ~30–50/day cap?
- T1 is link- and image-free; spam-trigger words scrubbed across all subjects and bodies?
- T5 carries a genuine pattern interrupt (changed shape / named dynamic / flipped ask) + opt-out?
- LinkedIn notes ≤300 chars, no pitch; CTAs via `cta-variant-generator`; subjects via `subject-line-preview-text-optimizer`?
- Proof is real or `[verify]`; objections from `objection-library-builder` woven into the V/E touches?
- Saved to `./outreach/<brand-slug>-<icp-slug>-sequence.md` with all slots labeled?
