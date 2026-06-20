---
name: objection-library-builder
description: >
  Mines the real reasons buyers hesitate, stall, and walk — from sales calls, support tickets, churn
  surveys, and lost-deal notes — and turns them into a tagged, reusable objection library: every
  objection categorized (price, trust, fit, timing, competitor, status-quo, authority), each one paired
  with its ideal reframe and a specific proof point to attach. It is a `brand-brain` COMPONENT: callable
  standalone, or invoked by `brand-brain` during bootstrap/refresh to produce the objection-handling
  layer. It does NOT invent objections, proof, or competitor claims — it surfaces what buyers actually
  say and matches each to the brand's real differentiators and proof. Use whenever the user says
  "objections," "objection handling," "rebuttals," "why we lose deals," "handle this objection," "what
  do buyers push back on," "build an objection library," or hands over call notes / lost-deal reasons /
  churn feedback and asks how to answer them. Writes the objection library and a short top-objections
  pointer in the brand brain; it does not write the sales script, the pricing, or the rest of brand.md.
---

# Objection Library Builder

Feed it the raw friction — call transcripts, ticket threads, churn replies, the CRM "lost reason" field — and get back a structured library: what buyers actually object to, sorted by category, each with the reframe that flips it and the one proof point that lands it. The reframes are written in the brand's real voice and backed only by the brand's real proof, because brand context comes from the shared `brand-brain` skill, not from guessing.

This skill surfaces and answers objections. It does not write the sales deck, redesign the offer, or set pricing. If an objection is *true* — the price really is wrong for that segment, the feature really is missing — it says so and tags it `valid`, rather than inventing a rebuttal that a prospect will see through.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand and loads its context (voice, ICP, real differentiators, real proof, offer/pricing, banned words). This skill does not implement brand scanning, interviewing, or storage; that lives in `brand-brain`, once.
- *(optional, when installed)* `proof-vault` to attach a stronger proof asset to a reframe; `competitive-intelligence-dossier` for sharper competitor-objection rebuttals. Synthesize inline from the brand's real proof/differentiators when absent.

---

## How a run works

```
Step 0  Detect the mode    ──► CALLED-BY-brand-brain | STANDALONE
Step 1  Load the brand     ──► use passed context (called) OR call `brand-brain` (standalone)
Step 2  Mine the objections ──► categorize → reframe → attach proof
Step 3  Self-review, then  ──► RETURN the section (called) OR PERSIST + confirm (standalone)
```

### Step 0 — Detect the mode

- **Called by `brand-brain`** — the request arrives carrying the active **slug**, the current **brand.md** content, and **scanned raw inputs** (call notes, tickets, churn data the bootstrap found). Use that context, do the work, and **RETURN** your two outputs (the objection library + the top-objections pointer block) for `brand-brain` to fold in and persist. **Do not call `brand-brain` back** — no recursion, no writing files yourself in this mode.
- **Standalone** — a user runs you directly with no brand context passed. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. Do the work. Then **persist it yourself** (Step 3 standalone).

### Step 1 — Load the brand

Whichever mode: before mining anything, you must have the brand's **voice + banned words** (so reframes sound on-brand), its **real differentiators + proof** (so reframes are backed by truth), its **ICP + awareness tendency** (objections differ by stage), and its **offer/pricing essentials** (price objections need the real numbers). In standalone mode `brand-brain` returns this digest + the `brand.md` path; read the file if you need the full proof list. **Do no objection work until brand context is loaded.**

**Fallback if `brand-brain` is not installed (standalone only):** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or give you the four things above inline, then proceed. Always prefer the call.

### Step 2 — Mine, categorize, reframe (the craft — below).

### Step 3 — Return or persist

- **Called mode:** RETURN both outputs as text for `brand-brain` to fold in. Do not touch any file.
- **Standalone mode — persist exactly two things, nothing else:**
  1. **Write the companion file** at `<data-root>/brands/<slug>/objections.md` (the full library — see template). This is the system of record for objections.
  2. **Update the `## Notes / do-not` section of `brand.md`** with a short *top-objections pointer* (the 3–5 highest-frequency objections + a one-line "see `objections.md`" link). Read `brand.md`, replace **only** that pointer block (don't disturb the rest of Notes / do-not), bump `updated`, and append yourself to `sources` (e.g. `objection-library-builder YYYY-MM-DD`).
  - Then confirm in one line where you saved both: *"Objection library → `<root>/brands/<slug>/objections.md`; top-objections pointer added to `brand.md` Notes."*

You **never** write or overwrite any other `brand.md` section or file. Where reframes should feed the sales script, CTA, or positioning, you SUGGEST that content — you do not write those skills' outputs.

---

## Mining — turn raw friction into clean objections

Pull from every source the brand has; each has a different bias, so triangulate:

| Source | What it's best for | Watch out for |
|---|---|---|
| **Sales-call notes / transcripts** | Live objections + the *exact words* buyers use | Rep's paraphrase ≠ buyer's real concern; capture the verbatim |
| **Lost-deal / CRM "lost reason"** | The objection that actually killed the deal | "Price" is often a cover for trust/fit — dig past the label |
| **Support tickets** | Post-purchase friction → trust + fit objections | Skewed to existing customers, not prospects |
| **Churn surveys / cancel flows** | Why value didn't land → fit + timing + price | Exit bias; people under-report their own misuse |
| **Review sites / sales-team Slack** | Competitor and status-quo objections | Anonymized — can't verify specifics, tag `[verify]` |

Discipline while mining:
- **Capture the verbatim.** Keep one real quote per objection where you have it — it's the trigger pattern the rep/copy will recognize in the wild. Anonymize names.
- **Separate the stated objection from the real one.** "Too expensive" with high engagement = a *value/ROI* objection, not a price one. Note both.
- **Count frequency.** Rank by how often it appears and how late it kills the deal. A rare objection that ends every late-stage deal outranks a common early one.
- **Don't manufacture objections.** Only log what buyers actually raised. If a source is empty, say the library is thin and list what to collect next — don't pad it with hypotheticals.

---

## Categorize — the seven-bucket taxonomy

Tag every objection with exactly one **primary** category (add a secondary if it genuinely straddles). This is what makes the library queryable by other skills.

| Tag | The buyer is really saying | Reframe strategy |
|---|---|---|
| `price` | "Costs too much / can't justify the spend" | Shift from cost to ROI/payback; right-size to the plan; cost-of-inaction |
| `trust` | "I don't believe it works / that you'll be around / that it's safe" | Proof, specificity, risk reversal, social proof from a peer |
| `fit` | "Not built for my use case / size / stack" | Reframe to the real ICP edge, or honestly tag `valid` if it isn't a fit |
| `timing` | "Not now / next quarter / after X" | Cost of delay, low-commitment first step, anchor to a trigger event |
| `competitor` | "We're looking at / already use [X]" | Differentiator they can't get elsewhere — real ones only, no FUD |
| `status-quo` | "We do this manually / in-house / it's fine" | Surface the hidden cost of the current way; make the gap concrete |
| `authority` | "I need sign-off / it's not my call" | Arm the champion: build their internal business case, not yours |

If an objection is genuinely true and unanswerable with real proof, tag it `valid` and route it upward (Notes / do-not, or a flag to product/pricing) — do not write a spin rebuttal.

---

## Reframe + proof — the answer to each objection

For every objection, produce three things. A reframe without a proof point is an opinion; a proof point without a reframe is a stat nobody asked for.

1. **The ideal reframe** — the on-voice move that turns the objection. Lead by *agreeing with the underlying concern* before pivoting ("Yes, switching tools is real work — here's why teams do it anyway…"). One or two sentences. Never argue with the buyer; reframe what the fact *means*.
2. **The proof point to attach** — the specific, real asset that makes the reframe land: a stat, a named customer/logo, a guarantee, a teardown, a comparison. **Use only the brand's real proof** (from the digest / `brand.md` / `proof-vault`). If the perfect proof doesn't exist, say what proof would close this objection and tag the gap `[need proof]` — never fabricate a number, a customer, or a result.
3. **Tone tag** — `discovery` (educate, low pressure) vs `late-stage` (close, decisive), since the same objection is answered differently early vs at signature.

Calibrate the reframe to the **ICP's awareness stage**: an unaware buyer's "timing" objection needs education; a product-aware buyer's needs a cost-of-delay nudge and a low-commitment next step. Match the move to where their head is.

---

## Output templates

### Companion file — `objections.md`

```markdown
---
brand: <slug>
updated: <YYYY-MM-DD>
source-inputs: [sales-calls, lost-deals, tickets, churn]   # what was mined
confidence: <high | medium | low>
---

# <Brand> — Objection Library

## Top objections (by frequency × deal impact)
1. [tag] short name — freq: high · kills: late-stage
... (3–5)

## Library
### [price] "It's more than we budgeted"
- **Verbatim:** "…" (lost deal, 2026-05)        ← real quote, anonymized
- **Real concern:** value/ROI, not raw cost
- **Reframe (late-stage):** <on-voice reframe>
- **Proof to attach:** <real stat / customer / guarantee>  | or [need proof: …]
- **Frequency:** high · **Stage it appears:** evaluation

### [competitor] "We're already on <X>"
...

## Valid / unanswerable (route upward)
- [fit] "<objection>" — genuinely not our ICP; do not spin. → flag to positioning/product.

## Proof gaps to close
- <objection> needs <proof that doesn't exist yet>.
```

### Pointer block for `brand.md` `## Notes / do-not` (the only thing you touch in brand.md)

```markdown
- **Top objections (see `objections.md`):** price/ROI ("more than we budgeted"), competitor ("already on X"), status-quo ("we do this manually"). Lead reframes with agreement; attach real proof only.
```

Keep the pointer to ~2–4 lines. The depth lives in `objections.md`; `brand.md` just signals what exists and where.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No objection work before brand context is loaded. Voice + banned-words override everything here.
- **Real objections only.** Log what buyers actually said; never invent friction to look thorough.
- **Real proof only.** Every reframe attaches the brand's real proof, or names the gap `[need proof]`. Never fabricate stats, customers, or competitor claims.
- **Agree, then reframe.** Validate the concern before turning it — never argue the buyer is wrong.
- **Some objections are true.** Tag `valid` and route upward; don't spin an honest deal-killer.
- **Categorize everything.** One primary tag per objection — that's what makes the library callable.
- **Stay in your lane.** Own `objections.md` + the Notes pointer. Suggest content for the sales script / CTA / positioning; never write their files.

## What Not to Do

- Don't do objection work before `brand-brain` returns (or the fallback loads the brand).
- Don't call `brand-brain` when you were called *by* it (no recursion).
- Don't write or overwrite any `brand.md` section other than the top-objections pointer in Notes / do-not.
- Don't invent objections, proof, customer names, results, or competitor weaknesses (FUD).
- Don't write spin for an objection that's actually valid — tag and escalate it.
- Don't store the library inside the skill folder — it lives in the data root.
- Don't pad a thin library with hypotheticals; say it's thin and list what to collect.

## Quality checklist (self-review before returning/persisting)

- Mode detected correctly; brand context loaded (called: from caller / standalone: via `brand-brain`)?
- Every objection has a verbatim (where available), a primary tag, a reframe, and a real proof point or `[need proof]`?
- Reframes are on-voice, honor banned words, and lead with agreement before the pivot?
- Top objections ranked by frequency × deal impact; valid/unanswerable ones flagged, not spun?
- No invented objections, stats, customers, or competitor claims; unverifiable items `[verify]`?
- Standalone: wrote `objections.md` AND replaced only the Notes pointer block, bumped `updated`, appended to `sources`, confirmed both paths? Called: returned both outputs, wrote no files?
