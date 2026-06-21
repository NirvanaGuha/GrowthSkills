---
name: in-app-microcopy-writer-auditor
description: >
  Takes feature/UI context or existing in-app copy and produces ready-to-ship microcopy for
  empty states, tooltips, banners, modals, error messages, success confirmations, and upgrade
  prompts — plus a quality audit on every piece of existing copy: clarity, length, vagueness,
  voice compliance, and activation impact. Writes net-new copy in two modes: a quick single-surface
  pass or a full surface inventory across a feature or flow. Audits existing copy with a structured
  scorecard and prioritized rewrite list. Always brand-voice compliant via brand-brain. Anchors
  copy decisions to the EAST framework (Easy, Attractive, Social, Timely) and the three-register
  model (Instructional / Motivational / Reassuring). Use when the user says "write tooltip copy,"
  "audit our empty states," "clean up the in-app messaging," "what should this error message say,"
  "make the upgrade prompt convert better," "our microcopy is confusing," "write onboarding
  tooltips," or hands over a feature spec and asks for UI copy.
---

# In-App Microcopy Writer & Auditor

Good microcopy does three things in under twelve words: tells users what to do, gives them a reason to do it, and removes the fear of getting it wrong. This skill writes and audits every text surface inside a product — empty states, tooltips, banners, modals, error messages, success messages, and upgrade/upsell prompts — using the brand's real voice, the user's actual awareness stage, and the EAST behavioral framework as the structural spine.

It does not design the UI, restructure information architecture, or touch code. If a surface is unfixable with copy alone, it says so.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads voice adjectives, banned words, ICP, offer mechanics, and real proof. Never writes a single word before the brand digest is in hand.
- **`cta-variant-generator`** — called for any in-app CTA button label within an upgrade prompt or activation nudge rather than duplicating CTA logic here.
- **`icp-persona-builder`** — called when the user's product has no defined persona on file and awareness stage is genuinely unknown; supplies the activation-stage context needed to calibrate register and commitment ceiling.
- **`landing-page-heuristic-live-cro-auditor`** — called if the in-app surface is a modal or full-screen interstitial that functions as a landing page; that skill handles the conversion-heuristic audit; this skill handles the microcopy layer only.
- **`de-slop-humanize-pass`** — called as a final pass on output longer than ~150 words to strip AI-textured filler before delivery.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain skill
Step 1  Classify the job        ──► Write (new) | Audit (existing) | Both
Step 2  Inventory the surfaces  ──► one surface or full flow
Step 3  Apply EAST + register   ──► the framework pass
Step 4  Deliver output          ──► inline (quick) or saved report (full audit)
Step 5  Self-review             ──► quality checklist before presenting
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Use the returned digest — voice adjectives, banned words, offer mechanics and destination URLs, real proof, ICP + awareness tendency — as hard constraints on every surface written. Mark any fact not in the digest as `[verify]`.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user for: (1) voice in three adjectives + banned words/phrases, (2) ICP + their dominant awareness stage in the product, (3) the feature's purpose and the activation milestone it serves. Proceed only after this minimum is in hand.

---

## The EAST framework (the structural spine)

EAST (Behavioural Insights Team) is the operating model for every surface. Each piece of microcopy is evaluated and written against all four levers:

| Lever | In-product microcopy application |
|---|---|
| **Easy** | Reduce cognitive load: one idea per surface, active verbs, no jargon, label the action not the technology |
| **Attractive** | Salience: surface the benefit or the consequence the user cares about; use the exact vocabulary from ICP research, not internal product names |
| **Social** | Normalise the action: "X teams already use this" / "Most users enable this in setup" — with real proof only, else `[verify]` |
| **Timely** | Contextual precision: trigger copy at the moment the user hits the relevant state (empty state, first-use, error, near-limit), not generically |

A surface fails the EAST audit if it scores 0 on any single lever.

---

## The three-register model

Every in-app surface belongs to one of three copy registers. Mixing registers on a single surface is a primary quality defect.

| Register | When to use | Tone signal | Avoid |
|---|---|---|---|
| **Instructional** | User needs direction: empty state, first-run tooltip, setup wizard, error message | Clear, directive, present tense; lead with the verb | Passive voice, "Please," over-explaining the obvious |
| **Motivational** | User is close to an activation or upgrade threshold | Value-forward, specific benefit, low-commitment CTA; use brand voice fully | Hype language, fake urgency, unconfirmed proof |
| **Reassuring** | User is anxious or uncertain: destructive action modal, payment confirmation, permission request, data-sensitivity notice | Calm, brief, honest; acknowledge the concern before redirecting | Minimising real risk, burying cancel/undo options in copy |

---

## Write mode — net-new microcopy

### Quick pass (single surface)

1. **Name the surface type** (empty state / tooltip / banner / modal / error / success / upgrade prompt).
2. **Identify the register** (Instructional / Motivational / Reassuring).
3. **Apply EAST** — score each lever, write to close any gap.
4. **Write the copy:**
   - **Headline / label** — ≤8 words; active verb or clear noun phrase; never "No [thing] yet."
   - **Body / subtext** — ≤25 words; one idea; ICP vocabulary from the brand digest.
   - **CTA** — call `cta-variant-generator` or produce a primary + one alternate inline.
   - **Microcopy / reassurer** — one line; address the most likely hesitation.
5. **Deliver** inline. No file unless the user asks.

### Full flow pass (feature or multi-surface)

Produce a surface inventory table first, then write each surface:

```
| Surface | Type | Register | EAST gaps | Copy (H / Body / CTA / Reassurer) |
```

Save to `./microcopy/[feature-slug]-copy.md` when the inventory exceeds five surfaces.

---

## Audit mode — existing copy

For each piece of copy handed over, run the **Microcopy Scorecard**:

| Dimension | Pass criteria | Common failure |
|---|---|---|
| **Clarity** | A first-time user understands the required action without help-doc support | Jargon, internal names, passive constructions |
| **Length** | Headline ≤8 words; body ≤25 words; CTA ≤5 words | Over-explanation, restating what the UI already shows |
| **Register fit** | Matches the surface's functional moment | Motivational copy on an error; Reassuring tone on an empty state |
| **EAST completeness** | All four levers addressed or consciously omitted with reason | Missing Timely (generic copy not tied to the triggering state) |
| **Voice compliance** | No banned words; voice adjectives detectable in the writing | Brand-neutral filler ("Oops!", "Uh oh!", "Looks like…") unless the brand allows it |
| **Activation alignment** | Moves the user one step closer to the activation metric | Copy that names the feature but not the user benefit |

**Output format (per surface):**

```
Surface: [name]
Current copy: [verbatim]
Scorecard: Clarity [P/F] · Length [P/F] · Register [P/F] · EAST [E/A/S/T each P/F] · Voice [P/F] · Activation [P/F]
Priority: [Critical / High / Medium] — [one-line rationale]
Rewrite: [revised headline / body / CTA / reassurer]
```

Save full audits to `./microcopy/[feature-slug]-audit.md`. Inline for ≤3 surfaces.

---

## Surface-specific craft notes

- **Empty states:** the most under-invested surface. Always include a motivational reason to take the first action, not just a label for the absence. "You haven't added any campaigns yet" → "Run your first campaign — results show up here within 24 hours." [Verify timing claim against real product behavior.]
- **Tooltips:** triggered by hover or (?); must answer "why does this exist, and what should I do with it" in ≤2 sentences. Never restate the field label.
- **Error messages:** name what went wrong, why (if non-obvious), and the exact remediation step. Never blame the user. "Something went wrong" is a scorecard failure.
- **Success/confirmation messages:** brief and concrete. State what changed and what happens next. "Saved" alone is insufficient; "Campaign saved — it goes live at 9 AM your time" is the floor.
- **Upgrade/upsell prompts:** Motivational register. Anchor to the specific capability the user just tried to access, not to the plan name. Call `cta-variant-generator` for the CTA.
- **Permission requests (notifications, location, camera):** Reassuring register. Lead with the user benefit, not the permission ask. Explain what you will and will not do with the data.

---

## Principles

- **Brand-brain first, always.** No copy written before the brand digest is loaded. Voice adjectives and banned words are hard overrides, not suggestions.
- **EAST is the audit spine.** A surface that fails any single lever gets a rewrite, not a pass.
- **Register before vocabulary.** Getting the register wrong produces copy that confuses or irritates regardless of word choice.
- **One idea per surface.** Two ideas per surface means one surface too few was designed, not one copy block too long.
- **Real proof or `[verify]`.** No invented social proof numbers; if the product team hasn't confirmed a claim, flag it.
- **Activation alignment is the success metric.** Copy that describes the product without moving users toward the activation milestone has failed its job, regardless of voice compliance.

## What Not to Do

- Don't write copy before `brand-brain` returns the active brand.
- Don't mix registers on a single surface (motivational headline + error-level reassurer = confused user).
- Don't use "Oops!", "Uh oh!", "Heads up!", or "Just a reminder" unless the brand voice explicitly includes them — these are filler defaults, not brand choices.
- Don't restate what the UI already communicates visually (a disabled button + "This feature is unavailable" in a tooltip is redundant).
- Don't invent social proof ("Trusted by 10,000 teams") without a `[verify]` marker.
- Don't redesign information architecture — if the copy problem is actually a UI structure problem, name it and stop; don't paper over it.
- Don't audit without rewriting. A scorecard with no rewrite is incomplete output.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy was written?
- Voice adjectives detectable; no banned words appear anywhere in output?
- Every surface assigned a register; no register mixing on a single surface?
- All four EAST levers addressed or consciously noted as omitted?
- Headlines ≤8 words; body copy ≤25 words; CTAs ≤5 words (or deviation noted with reason)?
- Every social proof claim is confirmed from the brand digest or flagged `[verify]`?
- Audit output: scorecard + priority + rewrite for every surface examined?
- Full flow or full audit (>3 surfaces) saved to `./microcopy/[feature-slug]-copy.md` or `-audit.md`?
- `de-slop-humanize-pass` called if any block of prose exceeds ~150 words?
