---
name: crisis-communications-writer
description: >
  Incident summary + company position + facts → initial holding statement and full public crisis
  statement that acknowledges the issue, demonstrates real action, and protects the brand's long-term
  credibility. Built on the Situational Crisis Communication Theory (SCCT) framework: matches response
  strategy to crisis type and responsibility level so you don't over-apologize, under-apologize, or
  accidentally imply guilt you don't have. Two primary outputs: (1) a holding statement for the first
  60 minutes when facts are incomplete, and (2) a full crisis statement once the position is confirmed.
  Optional add-ons: internal all-hands note, media FAQ, and a post-crisis learnings brief.
  Does NOT implement brand context itself — calls `brand-brain` to load voice, positioning, and proof.
  Does NOT manage press distribution — hands off to `press-release-writer-reviewer` for wire formatting.
  Use when the user says "crisis statement," "holding statement," "incident response," "write our
  apology," "data breach communication," "PR crisis," "how do we respond to this," "write a
  statement about [incident]," or pastes a news article/complaint thread and asks what to say.
---

# Crisis Communications Writer

Every crisis has a clock. The first statement drops before you have all the facts. The full statement lands after lawyers, leadership, and comms align. Both must be right — wrong tone in the first 60 minutes sets the narrative for weeks.

This skill writes both. It does not invent facts, minimize real harm, or paper over accountability gaps with brand voice. It uses the SCCT framework (Coombs, [verify]) to match your response posture to the actual crisis type — because a ransomware attack requires a different stance than a viral customer complaint, and conflating them is how reputations collapse.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, positioning, and proof. No statement before this returns.
- **`press-release-writer-reviewer`** (optional) — if the full statement needs wire formatting or AP-style polish.
- **`escalation-note-change-announcement-drafter`** (optional) — for the internal all-hands version of the same incident.
- **`proof-vault`** (optional) — for citing real past actions or commitments when available.

---

## How a run works

```
Step 0  Load brand context ──► call brand-brain
Step 1  Classify the crisis ──► SCCT cluster + responsibility level
Step 2  Triage the clock ──► holding statement mode vs. full statement mode
Step 3  Draft
Step 4  Self-review: legal-safe, factually bounded, on-voice
Step 5  Present + offer add-ons (internal note, FAQ, post-crisis brief)
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned voice adjectives, banned words, and positioning as hard overrides. Obey them even under pressure — a crisis statement that sounds off-brand adds a second story to the first.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none exists, ask for: brand name + tone descriptors, any banned words or phrases, and the company's public positioning line. Do not draft without at least these.

### Step 1 — Classify the crisis (SCCT)

Before drafting, name the cluster and responsibility level. This determines which response posture is appropriate.

**SCCT Crisis Clusters** (Coombs 2007, updated [verify]):

| Cluster | Examples | Responsibility attribution |
|---|---|---|
| **Victim** | Natural disaster, workplace violence by outsider, product tampering by third party | Low — company is also a victim |
| **Accidental** | Technical failure, product recall from unforeseeable defect, data breach from novel exploit | Low-medium — unintentional |
| **Preventable** | Human error, misconduct, knowingly violated standard, ignored warnings | High — company caused it |

**Response posture by cluster:**

| Cluster | Posture | What it sounds like |
|---|---|---|
| Victim | Bolster + inform | "We are responding to an external incident…" |
| Accidental | Diminish + corrective action | "A failure in [system] caused…; we have taken the following steps" |
| Preventable | Full apology + corrective action + compensation if warranted | "We fell short of the standard we hold ourselves to…" |

**Responsibility escalators** — shift toward Preventable regardless of cluster if: prior similar incidents exist, warnings were ignored, or leadership was aware. Note this and flag to the user — never downplay a known history.

### Step 2 — Triage the clock

Ask (or infer from the input) which output is needed:

- **Holding statement** — facts still incoming; the purpose is to acknowledge and buy time without creating liability. Issued within 60 minutes. Maximum 100 words.
- **Full statement** — position confirmed, facts known, corrective actions identified. Issued when leadership and legal have aligned. 250–500 words is the norm; more only if warranted.
- **Both** — the common sequence: draft holding first, then full.

---

## Holding statement (60-minute output)

The holding statement has exactly four jobs: (1) acknowledge the issue exists, (2) signal you're investigating, (3) show human concern for affected parties, (4) commit to a follow-up time. It does nothing else. No speculation. No root-cause claims. No promises about outcomes.

```
## Holding Statement — [brand] — [incident type]
Issued: [date/time or "pending"]

[1–2 sentences: acknowledgment of the issue, on-voice]
[1 sentence: we are actively investigating / responding]
[1 sentence: concern for affected parties by name if identifiable]
[1 sentence: we will provide an update by [time window — be specific, never "as soon as possible"]]

— [Brand name] [Team / spokesperson if known]
```

**Hard limits for holding statements:**
- Never speculate on cause
- Never state the scope of impact until confirmed
- Never name internal systems, vendors, or individuals unless legally required
- Never use passive voice to obscure agency ("mistakes were made" → never)
- Do not commit to compensation until the position is confirmed

---

## Full crisis statement

Structure follows the SCCT posture for the classified cluster, plus the Coombs 4-part corrective action arc when responsibility is accidental or preventable:

**1. What happened** (factual, bounded — only what is confirmed)
**2. Impact acknowledgment** (who is affected, what they experienced — specific, not generic)
**3. Corrective action** (what has already been done + what is being done now + what will change)
**4. Commitment** (a specific, time-bound follow-up commitment — not a vague pledge)

For Preventable cluster, add an accountability statement before corrective action: one sentence owning the failure explicitly, without deflection. Legal-safe does not mean absent of accountability.

```
## Full Crisis Statement — [brand]
[Date]

[Opening: acknowledgment in brand voice, 1–2 sentences]

[What happened: 2–4 sentences, facts only, bounded claims]

[Impact acknowledgment: who, what they experienced, show you understand the real harm]

[Corrective action — past: what you have already done since discovery]
[Corrective action — present: what is happening right now]
[Corrective action — future: what structural change prevents recurrence]

[Commitment: specific follow-up by a named date or milestone]

[If Preventable cluster: accountability line here, before or after corrective action]

[Closing: one sentence on company values/commitment — only if it doesn't read as deflection]

— [Name, Title] / [Brand name]
```

**Always mark unconfirmed facts `[verify]`.** Never assert impact scale, root cause, or affected-party count without a confirmed source.

---

## Optional add-ons (offer at end of primary output)

**Internal all-hands note** — same facts, different tone: staff need more context than the public, earlier. Delegates to `escalation-note-change-announcement-drafter` if installed; writes inline otherwise. Typical additions: what employees should say if asked by customers/media (redirect script), who the internal point of contact is, and what is NOT yet confirmed.

**Media FAQ** — 6–10 anticipated journalist questions with approved answers. Each answer is bounded: what can be confirmed, what is still under investigation, what cannot be disclosed and why. Mark speculative answers clearly.

**Post-crisis learnings brief** — for use 2–4 weeks after resolution. Structured: timeline of events → initial response assessment → what landed well → what didn't → recommended SOP changes. Save to `./crisis/[slug]-[incident-slug]-postmortem.md`.

Save primary outputs to `./crisis/[slug]-[incident-slug]-statement.md` when the user wants persistence.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Voice + banned words override everything, including legal-speak defaults.
- **SCCT posture before a word is written.** The wrong posture (apologizing when you're the victim; diminishing when you caused it) is worse than a delayed statement.
- **Factual bounds are absolute.** Only confirmed facts in the statement. Everything else is `[verify]` or explicitly noted as "under investigation."
- **No passive-voice accountability laundering.** "Mistakes were made," "it was determined that" — never. Own the verb.
- **Specificity over sincerity theater.** Vague regret is noise. Specific corrective action is signal. The public wants the latter.
- **The holding statement buys time, not goodwill.** Its only job is to close the information vacuum; do not try to spin it.
- **History escalates posture.** A prior incident of the same type shifts the cluster toward Preventable. Note it, don't hide it.

---

## What Not to Do

- Do not draft before `brand-brain` returns the active brand.
- Do not speculate on cause, scope, or attribution in the holding statement.
- Do not minimize real harm with qualifiers ("minor inconvenience," "small number of users").
- Do not use the full statement to announce the crisis is over until it is confirmed over.
- Do not produce generic "we take your concerns seriously" filler — if you can't write a specific corrective action, say so and ask the user for it.
- Do not advise legal strategy, confirm or deny legal liability, or suggest the company is not liable — that is outside scope; flag and note.
- Do not reimplement brand scanning, interviewing, or storage — call `brand-brain`.
- Do not hand off to `press-release-writer-reviewer` without checking that the statement is factually bounded first.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice + banned-words honored throughout?
- SCCT cluster and responsibility level named; posture matches?
- Holding statement: under 100 words, no speculation, specific follow-up time given?
- Full statement: all four SCCT corrective-action arc parts present (what happened / impact / corrective action / commitment)?
- Preventable cluster: explicit accountability line present, not deflected?
- Zero unconfirmed facts stated as confirmed; all uncertain items marked `[verify]`?
- No passive-voice accountability laundering; no minimizing language?
- Add-ons offered (internal note, media FAQ, postmortem)?
