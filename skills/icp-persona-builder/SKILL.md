---
name: icp-persona-builder
description: >
  Turns CRM exports, sales-call notes, win/loss data, and voice-of-customer inputs into a sharp ICP
  definition and named buyer personas — who to chase, who to qualify out, and the exact language each
  persona uses. It is a brand-brain COMPONENT (callable standalone or invoked by brand-brain during
  bootstrap/refresh): it owns the `ICP(s)` section of brand.md and an optional deep `personas.md`
  companion file, and it does not touch any other brand section. Built on Jobs-To-Be-Done, a
  firmographic qualification scorecard, the Schwartz awareness ladder, and full persona cards (goals,
  pains, triggers, objections, watering holes, language patterns). Use whenever the user says "build
  our ICP," "ideal customer profile," "who is our buyer," "create personas," "qualify accounts,"
  "who should we be targeting," or hands over customer data and asks who to sell to. It defines and
  segments the buyer — it does not write campaigns or pick channels.
---

# ICP & Persona Builder

Feed it the messy truth — closed-won deals, lost deals, support transcripts, sales calls, survey verbatims — and get back a defensible ICP and a small set of named personas a junior rep could qualify against on a cold call. Every persona is grounded in the brand's real positioning, offer, and proof, because that context comes from the shared `brand-brain` skill, not from guessing here.

This skill **defines and segments the buyer**. It does not write the campaign, pick the channel, or build the offer. It tells the rest of the library *who* — voice, CTAs, headlines, and ads read the ICP it writes and aim at it.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode) — resolves and loads the active brand so the ICP is anchored to the real positioning, offer, and proof. This skill does not implement brand scanning, interviewing, or storage; that lives in `brand-brain`, once.
- *(optional, when installed)* `competitive-intelligence-dossier` for "vs. the alternative they use today," `objection-library-builder` to deepen the objection rows. Synthesize inline when absent.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED (brand-brain passed context) | STANDALONE (resolve via brand-brain)
Step 1  Ingest the inputs ──► CRM / calls / VoC / win-loss → evidence, not vibes
Step 2  Do the work       ──► ICP definition + qualification scorecard + persona cards
Step 3  Self-review, then RETURN (called) or PERSIST (standalone)
```

### Step 0 — Detect the mode (always first)

- **CALLED BY brand-brain.** If the invocation passes a brand slug + current `brand.md` content + scanned raw inputs, you are in called mode. Use the passed context. Do the work. **RETURN your `ICP(s)` section content** for brand-brain to fold in (and flag if a deep `personas.md` is warranted). **Do not call `brand-brain` back** — no recursion.
- **STANDALONE.** No brand context passed → **invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. It bootstraps on first use; wait for it. Then do the work and **PERSIST** (Step 3).

**Fallback if `brand-brain` is not installed (standalone only):** read `<root>/brands/.active` and that brand's `brand.md` directly (`<root>` = `./.brandbrain/` if present, else `~/.brandbrain/`). If none exists, ask the user to install `brand-brain` (preferred) or answer a 4-line mini-setup (what it is · what it replaces today · primary conversion action · who's bought so far), then proceed. Always prefer the call.

### Step 1 — Ingest the inputs

Pull from whatever the user has; rank by signal. **Closed-won + closed-lost beats opinion. Recorded calls beat survey scales. A direct quote beats a paraphrase.**

| Input | What you mine it for |
|---|---|
| CRM export (won/lost, deal size, cycle, source) | Firmographics that actually convert; disqualifiers; cycle length |
| Sales-call notes / recordings | Triggers, the words they use, objections in their own voice |
| Win/loss interviews | Why they chose you, what they almost chose instead |
| Support / onboarding transcripts | Real jobs, friction, the "aha" that retains |
| VoC: reviews, surveys, NPS verbatims | Language patterns, watering holes, emotional stakes |
| The brand.md (offer, proof, positioning) | What you can credibly promise this buyer |

If inputs are thin, say so and build a **hypothesis ICP** marked `[hypothesis]` — never present a guess as data. Note the evidence (or its absence) per claim.

---

## Craft 1 — ICP definition (the account-level filter)

The ICP describes the **company you should chase**, not the person. Keep it tight enough to disqualify. A good ICP is a knife: it cuts accounts out.

- **Firmographics:** industry/vertical · company size (employees / revenue) · business model · geography · tech stack or platform (e.g. "on Shopify Plus").
- **Trigger / timing:** the event that makes them in-market now (new funding, a hire, a migration, a compliance deadline, hitting a scale ceiling).
- **The job they're hiring for (JTBD):** the progress they're trying to make — "When ___, I want to ___, so I can ___." Anchor on the job, not the demographic.
- **Anti-ICP (who to qualify OUT):** the segments that look adjacent but churn, haggle, or never convert. This is the highest-leverage section — write it explicitly.

### Qualification scorecard (so a rep can score an account in 60 seconds)

| Signal | Weight | Disqualifier? |
|---|---|---|
| In target vertical / model | high | — |
| Above size / scale floor | high | below floor = DQ |
| On the required platform / stack | high | wrong stack = DQ |
| Active trigger present | medium | — |
| Owns the metric your offer moves | medium | — |
| Budget authority reachable | medium | — |

Pick a small set of weighted signals (typically 5–7), set explicit DQ conditions, and define **tiers** (Tier 1 must-pursue / Tier 2 / Tier 3 nurture). Tie tiers to the awareness ladder so downstream skills know the default CTA commitment. Use real conversion patterns from the CRM to set thresholds; mark invented thresholds `[verify]`.

---

## Craft 2 — Buyer personas (the human inside the account)

Accounts don't buy — people do. Build **2–4 named personas max** (more is dilution, not precision). Cover the buying roles that actually decide: typically an **economic buyer** (signs), a **champion** (feels the pain), and where relevant a **blocker/skeptic** (security, finance, legal). Name them for memorability ("Retention-Lead Rachel"), but the name is a handle, not a caricature — every field must be defensible from the inputs.

### Persona card

| Field | What goes here |
|---|---|
| **Name + role** | Memorable handle · real title(s) · buying role (economic / champion / blocker) |
| **JTBD** | The functional + emotional + social job: "When ___, help me ___, so I ___" |
| **Goals / metrics owned** | What they're measured on — the number that gets them promoted or fired |
| **Pains / frustrations** | The status-quo cost, in their words |
| **Triggers** | The event that puts them in-market; the moment they start searching |
| **Awareness stage** | Schwartz ladder position (below) → default CTA commitment |
| **Objections** | Top 3–5, each paired with the truthful counter from brand proof (`[verify]` if unproven) |
| **Watering holes** | Where they actually learn/lurk (specific communities, newsletters, podcasts, search terms) — not "LinkedIn" |
| **Language patterns** | Verbatim phrases they use; words they'd never use; how they describe the problem unprompted |
| **What they're replacing** | The incumbent / DIY / "do nothing" they're switching from |

### Awareness ladder → messaging entry point (Schwartz)

Where the persona sits dictates where copy must *start* — meet them there, don't skip rungs.

| Awareness | Persona's headspace | Copy enters at | Default CTA commitment |
|---|---|---|---|
| Unaware | doesn't know the problem | name the problem / status-quo cost | zero — curiosity |
| Problem-aware | feels the pain, no solution category | educate on the category | low — "show me how" |
| Solution-aware | knows the category, not you | differentiate vs. alternatives | medium — compare/evaluate |
| Product-aware | knows you, weighing it | proof, objection handling, risk reversal | high — trial/demo |
| Most-aware | ready, needs a nudge | the deal, the urgency, the ask | highest — buy/start |

Record each persona's **dominant** awareness stage (and note if a segment splits). This is the single field downstream skills lean on most — get it right.

---

## What you OWN and how you persist

You own **exactly** the `ICP(s)` section of `brand.md` and an optional companion `personas.md`. Nothing else.

- **`ICP(s)` section of brand.md** — the tight summary: per ICP → Who · Pains/triggers · Awareness tendency (→ default CTA commitment) · Metrics they own. Keep it to what other skills need at a glance; depth lives in the companion.
- **`brands/<slug>/personas.md`** (optional companion) — the full persona cards, the qualification scorecard, the anti-ICP. Write this when there's enough evidence to justify deep cards; otherwise keep everything in the brand.md section. Cross-link: the `ICP(s)` section ends with "→ full cards: `personas.md`" when the companion exists.

**Called mode:** RETURN the `ICP(s)` section content (and a one-line "deep personas.md recommended: yes/no"). Do not write files — brand-brain folds it in and owns the write.

**Standalone mode — PERSIST after self-review:**
1. Read the current `brand.md`. Replace **only** the `## ICP(s)` block with your new content; leave every other section byte-for-byte. Bump `updated` to today and append yourself to `sources` (e.g. `icp-persona-builder YYYY-MM-DD` + the input files used). Never touch `status` unless promoting a `[hypothesis]` ICP the user has confirmed.
2. If you built deep cards, write `<root>/brands/<slug>/personas.md`.
3. Confirm in one line where you saved: *"Updated the ICP(s) section in `<root>/brands/<slug>/brand.md`" (+ "and full persona cards in `…/personas.md`")*.

Never write brand data inside the skill folder. Get `<root>` from brand-brain (standalone) or the caller (called).

---

## Principles (Non-Negotiable)

- **Brand-brain first (standalone).** No ICP before brand-brain returns; in called mode, never call it back.
- **Own your lane.** Write only `ICP(s)` + the optional `personas.md`. Never overwrite another section or file. Where the system says you "feed" positioning or voice, you *suggest* — you don't write their sections.
- **Evidence over vibes.** Every firmographic, trigger, and quote traces to an input. No input → mark it `[hypothesis]`, don't smuggle a guess in as fact.
- **The anti-ICP is mandatory.** Who you qualify OUT is as load-bearing as who you chase. A definition that excludes nothing is useless.
- **Real language only.** Persona quotes and language patterns are lifted from the inputs, not invented. Never fabricate a customer name or a stat — `[verify]` it.
- **Fewer, sharper personas.** 2–4 named buyers, not eight demographic clusters. Precision beats coverage.
- **Awareness stage is a deliverable.** Every persona gets a Schwartz rung; it's what downstream copy keys off.

## What Not to Do

- Don't produce an ICP before brand-brain returns the active brand (standalone), and don't recurse into brand-brain in called mode.
- Don't write or edit any brand.md section other than `ICP(s)`, or any file other than `personas.md`.
- Don't invent firmographics, stats, customer names, or quotes — use real inputs or mark `[hypothesis]`/`[verify]`.
- Don't build demographic personas with no buying job, no trigger, and no awareness stage — that's a census, not a persona.
- Don't skip the anti-ICP or collapse it into a footnote.
- Don't write campaigns, pick channels, or set messaging — flag what's needed and hand off; that's other skills' work.
- Don't over-segment: if two personas share JTBD, trigger, and objections, they're one.

## Quality Checklist (self-review before returning/persisting)

- Mode detected correctly; brand-brain called (standalone) or passed context used (called) — no recursion?
- ICP is firmographic + JTBD + trigger, tight enough to disqualify, with an explicit **anti-ICP**?
- Qualification scorecard has weighted signals, explicit DQ conditions, and tiers tied to awareness?
- 2–4 named personas, each with JTBD, owned metric, triggers, top objections + truthful counters, watering holes, real language patterns, and a Schwartz awareness stage?
- Every claim traces to an input; thin spots marked `[hypothesis]`; unconfirmed numbers/names `[verify]`?
- Persisted to the right place: only the `ICP(s)` block replaced, `updated` bumped, `sources` appended, companion written if warranted — and confirmed where?
