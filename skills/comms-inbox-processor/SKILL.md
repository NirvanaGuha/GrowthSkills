---
name: comms-inbox-processor
description: >
  Takes a raw email inbox dump, a forwarded batch of threads, or a pasted set of messages and turns the
  chaos into a clean, actionable triage in one pass. For each message it applies the 4-D method
  (Do / Delegate / Defer / Delete) to determine the right action, assigns a reply-needed flag and an
  urgency tier, drafts on-brand replies for anything that needs a human response, writes TL;DRs for
  long threads so the user can skim before deciding, and produces a single sorted action list the user
  can execute top-to-bottom. Works with email, Slack threads, LinkedIn DMs, or any text-form message
  batch. Brand context is loaded via `brand-brain` so all drafted replies match the active brand's voice
  and avoid banned language. Use when the user says "process my inbox," "triage these emails," "draft
  replies to these," "clear my email backlog," "summarize this thread," "sort my messages by priority,"
  "inbox zero," or pastes a batch of messages and needs to know what to do with them.
---

# Comms Inbox Processor

Inbox chaos is a symptom, not a personality trait. This skill reads a message batch — emails, Slack threads, DMs — and outputs a ranked action list with drafted replies, TL;DRs, and triage decisions, so the user can execute rather than re-read.

One pass in. Clean action list out. Every drafted reply is in the active brand's voice because brand context loads first.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, banned words, and ICP so every drafted reply is on-brand. This skill does NOT implement brand resolution, voice inference, or scanning; that lives in `brand-brain`, once.
  Fallback if `brand-brain` is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [the sender context, reply tone preference (formal/casual/brand name), and any banned phrases to avoid].
- **`escalation-note-change-announcement-drafter`** *(optional)* — delegate to it when a message requires a sensitive internal escalation or a change-management response rather than a standard reply.
- **`loom-async-video-script-writer`** *(optional)* — delegate to it when the best reply is clearly a short async video rather than text.
- **`meeting-agenda-action-item-builder`** *(optional)* — delegate to it when a thread resolves into a meeting that now needs a structured agenda.
- **`doc-note-summarizer`** *(optional)* — delegate long attachment content to it when an email references a document that the user must read before replying.

---

## How a run works

```
Step 0  Load the brand      ──► call `brand-brain` (voice, banned words, ICP)
Step 1  Ingest + parse      ──► read every message; identify sender, subject, body, thread depth
Step 2  Triage each message ──► 4-D classification + urgency tier + reply-needed flag
Step 3  Draft replies       ──► on-brand drafts for every reply-needed item
Step 4  Write TL;DRs        ──► summaries for threads > ~5 exchanges or > ~300 words
Step 5  Output action list  ──► sorted, labeled, ready to execute top-to-bottom
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, offer context, and ICP. Every drafted reply must obey the returned voice and avoid the banned words. Do not draft a single reply before brand context loads.

Fallback if `brand-brain` is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [the sender context, reply tone preference (formal/casual/brand name), and any banned phrases to avoid].

---

## Step 1 — Ingest and parse

Accept any of:
- A raw paste of emails (with From/Subject/Date/Body blocks)
- A forwarded email chain or Slack thread export
- A numbered or bulleted list of message summaries
- A screenshot description or dictated summary of a message batch

For each message, extract: **sender identity** (name, company, role if inferrable), **subject/topic**, **apparent intent** (request, response, FYI, action item, social/networking, spam, internal ops, customer support, sales outreach), and **thread depth** (standalone vs. ongoing chain).

If the input is ambiguous — e.g., a single wall of text with no headers — ask the user to identify message boundaries before proceeding. Don't guess a split that could merge two separate threads.

---

## Step 2 — Triage with the 4-D Method

The 4-D method (Merlin Mann / David Allen lineage): **Do, Delegate, Defer, Delete/Archive**. Apply it to every parsed message.

| 4-D Action | When to apply |
|---|---|
| **Do** | Reply needed now or within 24h; short action (<2 min) |
| **Delegate** | Needs someone else to own; includes a handoff note |
| **Defer** | Needs the user's attention but not today; gets a date |
| **Delete/Archive** | No action required; FYI read, newsletter, spam, CC-only |

**Urgency tiers** (overlay on 4-D):

| Tier | Label | Definition |
|---|---|---|
| 1 | `URGENT` | Time-sensitive; customer/partner/exec; hard deadline at risk |
| 2 | `REPLY-TODAY` | Needs a response within business day; non-urgent but waiting |
| 3 | `THIS-WEEK` | Deferrable; defer flag with a day assigned |
| 4 | `ARCHIVE` | No action; read or discard |

**Reply-needed flag:** `[REPLY]` when the user must author a response. `[FWD]` when the right move is forwarding/delegating. `[NO-REPLY]` otherwise.

---

## Step 3 — Draft replies (for every `[REPLY]` item)

For each message flagged `[REPLY]`:

1. **Read the intent** — what is the sender actually asking for? Reply to the real question, not the surface wording.
2. **Apply voice** — use the active brand's voice adjectives; honor banned words; match formality to the sender relationship (customer vs. investor vs. internal teammate).
3. **Be complete but short** — one paragraph is the default; use bullets only when there are 3+ parallel items. No throat-clearing openers ("Hope you're well").
4. **Include a clear close** — end with the next step: a question, a CTA, a confirmed date, or a simple "Let me know if you need anything else."
5. **Mark anything unconfirmed** — if the reply references facts the user must verify, wrap them in `[verify]` rather than inventing.

Drafts appear directly below their triage entry. Label them clearly: `Draft reply →`.

---

## Step 4 — TL;DR summaries

Write a TL;DR for any thread that is:
- More than ~5 message exchanges, OR
- More than ~300 words in the pasted body, OR
- Explicitly flagged by the user as "summarize this"

TL;DR format:
```
TL;DR: [2–3 sentences: what was discussed, where it stands, what decision or action is pending]
Key action: [one sentence or none if purely FYI]
```

For short, clear messages: skip TL;DR (it adds friction, not value).

---

## Step 5 — Output: the action list

Present output in this order: `URGENT` → `REPLY-TODAY` → `THIS-WEEK` → `ARCHIVE`. Within each tier, group by 4-D action (Do → Delegate → Defer → Delete).

```
## Inbox Triage — [date] | [message count] messages

### URGENT — Do now
───────────────────────────────
[#] From: [Name / Company]
    Subject: [subject]
    Intent: [one-line classification]
    4-D: DO | [REPLY]
    TL;DR: [if applicable]
    Draft reply →
    [drafted reply text]

### REPLY-TODAY — Reply within business day
[same structure]

### THIS-WEEK — Defer to [day]
[same structure; no draft needed unless user requests one]

### ARCHIVE — No action
[condensed: sender + subject + reason in one line per message]
```

Save the output to `./inbox/triage-[YYYY-MM-DD].md` if the user asks to save it, or if there are more than 10 messages. Inline otherwise.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No drafted reply before brand context loads. Voice and banned words are hard overrides.
- **Triage every message.** No message left un-classified. If it's unclear, mark it `THIS-WEEK` and note the ambiguity.
- **Draft the reply, don't describe it.** "I'd suggest something like..." is not a draft. Write the actual words.
- **Short by default.** Drafts should be shorter than the email they're responding to, unless the message explicitly requires a detailed answer.
- **One action per message.** If a thread requires two separate actions (e.g., a reply AND a task creation), surface both, but keep them distinct.
- **No invented facts.** If a reply requires a number, date, or claim the user hasn't provided, mark it `[verify]`.
- **Honest urgency.** Don't escalate tier to look thorough. Archive truly means archive.

---

## What Not to Do

- Don't draft replies before `brand-brain` returns brand context.
- Don't write warm openers ("Hope this finds you well") — they add length, not warmth.
- Don't classify everything as `URGENT` to seem diligent; triage has no value if everything is Tier 1.
- Don't write a TL;DR for a two-sentence email — that's more words than the original.
- Don't delegate to `escalation-note-change-announcement-drafter` for routine replies — only trigger it for genuinely sensitive escalations or org-wide announcements.
- Don't invent names, dates, or commitments that aren't in the source messages.
- Don't reopen a thread that the user has marked Archive just because there's a surface-level action implied in the wording.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback path taken) before any reply drafted?
- Every message in the batch has a 4-D classification, urgency tier, and reply-needed flag?
- Every `[REPLY]` item has a complete draft reply — not a description, but the actual words?
- Every reply honors the brand's voice adjectives and avoids banned words; unconfirmed facts marked `[verify]`?
- TL;DRs written only for threads that genuinely need them (length threshold honored)?
- Output sorted correctly: URGENT → REPLY-TODAY → THIS-WEEK → ARCHIVE?
- Saved to `./inbox/triage-[YYYY-MM-DD].md` if batch size > 10 or user requested save?
