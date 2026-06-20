---
name: positioning-messaging-architect
description: >
  Produces a positioning statement and a value-proposition message house for a brand —
  the competitive frame, market category, unique attributes, and the core message with
  three value pillars and proof under each. It runs April Dunford positioning (competitive
  alternatives → unique attributes → value → who-it's-for → market category), folds that
  into a Geoffrey Moore positioning statement, and structures the value prop as a message
  house every downstream copy skill can pull from. It is a brand-brain COMPONENT — callable
  standalone, or invoked by `brand-brain` during bootstrap/refresh — and it owns exactly two
  brand.md sections: "Positioning / core frame" and "Value proposition & differentiators."
  It does NOT manage brand context itself; it reads the active brand through `brand-brain`.
  Use whenever the user says "positioning," "messaging," "value prop," "message house,"
  "how should we position this," "what's our elevator pitch," "what category are we in,"
  or hands over a product and asks where it sits in the market.
---

# Positioning & Messaging Architect

Give it a product and its market, get the frame the whole brand argues from: a sharp positioning statement, the market category you want to be judged in, and a message house — one core message, three value pillars, proof under each — that every headline, CTA, and deck downstream inherits. Positioning is a deliberate business decision, not a tagline contest; this skill makes that decision explicit and defensible.

It owns the *frame*, not the *facts*. It reads the brand's real differentiators and proof through `brand-brain` and never invents a number, a customer name, or an attribute the product doesn't have. If the product has no defensible difference, it says so — a message house can't manufacture one.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's context (what it is, ICP + awareness, existing positioning, differentiators, proof, voice, banned words). This skill never implements brand scanning, interviewing, or storage; that lives in `brand-brain`, once.
- *(optional, when installed)* `competitive-intelligence-dossier` for the competitive-alternatives set; `proof-vault` for proof under each pillar; `icp-persona-builder` for who-it's-for. Synthesize inline from the brand context when absent.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED-BY-brand-brain  |  STANDALONE
Step 1  Load the brand    ──► use passed context (called)  OR  invoke brand-brain (standalone)
Step 2  Build the frame   ──► Dunford → Moore statement → message house (the craft below)
Step 3  Pressure-test     ──► self-review against the checklist
Step 4  Return OR persist ──► return both section blocks (called)  OR  write them to brand.md (standalone)
```

### Step 0 — Detect the mode (do this first)

- **CALLED BY brand-brain.** The request carries an active brand slug + the current `brand.md` content + scanned raw inputs. **Use the passed context. Do your work. RETURN the two section blocks for brand-brain to fold in. Do NOT call `brand-brain` back** (no recursion) and do NOT write any file — brand-brain owns the write.
- **STANDALONE.** A user ran you directly, no brand context passed. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. Do your work. Then **persist** your two owned sections back into `brand.md` (Step 4) and confirm where you saved.

If you can't tell, look for a passed slug/brand.md in the request → called mode; otherwise standalone.

### Step 1 — Load the brand

You need, from `brand-brain`'s digest + `brand.md`: **what it is**, the **ICP(s) + awareness tendency**, any **existing positioning line**, the **real differentiators**, the **proof assets**, **voice + banned words**. Obey voice and banned-words as hard overrides. Use only real proof; mark anything you can't confirm `[verify]`. Never invent a differentiator to make the house symmetrical.

**Fallback if `brand-brain` is not installed (standalone only):** read `<data-root>/brands/.active` (try `./.brandbrain/` then `~/.brandbrain/`) and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer a 5-question mini-setup — what it is · ICP + the pain they hire it for · the 2–3 competitive alternatives they actually evaluate · the real differentiator(s) · one real proof point — then proceed. Always prefer the call.

---

## The craft — frame first, then the house

Positioning is upstream of messaging. Get the frame wrong and every message below it is well-written and pointed at the wrong target. Build it in this order; do not skip to the pillars.

### Part 1 — Dunford positioning (the five components)

Work the five components in order. Each answers the one before it. Fill the table from the brand's *real* situation; where a row is thin, that's a positioning gap to flag, not a blank to invent past.

| # | Component | The question it answers | How to nail it |
|---|---|---|---|
| 1 | **Competitive alternatives** | What would a customer use if you didn't exist? | The honest set — not just named rivals. Include "a spreadsheet," "a junior hire," "do nothing." This anchors everything; if you skip it the value claims float. |
| 2 | **Unique attributes** | What do you have that the alternatives don't? | Features/capabilities/model that are *demonstrably* true and *hard to copy*. Pull from brand-brain's differentiators. No attribute → no defensible position. |
| 3 | **Value (so what?)** | What can the customer do *because of* those attributes, that they care about? | Translate each attribute → a concrete value. Tie to a metric the ICP owns (retention, CAC, time-to-X). Drop values the ICP doesn't price. |
| 4 | **Who-it's-for** | Who cares a LOT about that value? | The segment for whom the value is acute, not "everyone." Narrow on purpose — best-fit customers, the characteristics that predict they'll love it. |
| 5 | **Market category** | What frame of reference makes the value obvious? | The context that makes you the obvious choice. Choose the category you can *win in* and that makes your unique attributes the buying criteria — don't default to the crowded one. |

**Sequencing rule:** alternatives → attributes → value → who → category. The category is a *consequence* of the first four, chosen last and deliberately. State it explicitly even when it feels obvious — an unstated category lets the customer file you under a competitor's.

### Part 2 — The positioning statement (Geoffrey Moore template)

Compress Part 1 into one disciplined statement. This is internal scaffolding (the source of truth), not customer-facing copy — write it for clarity, not flair.

```
For [target customer / who-it's-for]
who [statement of the need or pain],
[product name] is a [market category]
that [key benefit / the reason to buy].
Unlike [primary competitive alternative],
[product] [the primary differentiation].
```

One statement, not three. If you can't fill a slot from real facts, that slot is the work to do — name the gap (`[verify]` or "positioning gap: …"); don't paper over it. Keep it under ~60 words.

### Part 3 — The message house

The house turns the frame into reusable messaging. One roof, three pillars, proof in the foundation. Every downstream skill (headlines, CTAs, decks, ads) draws from here, so it must be tight and true.

- **Roof — core message.** One sentence the whole brand defends. It restates the positioning as a customer-facing promise: the single most important thing the best-fit customer should believe. Not a tagline (that's a visual-identity asset) — the *argument*.
- **Three pillars — value themes.** The 3 (rarely 4) reasons the core message is true. Each pillar = a benefit theme rolling up the unique attributes from Part 1. Make them *distinct axes of value*, not three rewordings of one. Three is the discipline; if you have five, you have a priority problem — force-rank.
- **Foundation — proof per pillar.** Under each pillar, the real evidence that earns it: a stat, a mechanism, a customer outcome, a named integration, an award. Use only brand-brain's real proof; mark unconfirmed `[verify]`; write "proof gap" where a pillar has none yet (a pillar with no proof is a claim, and you must flag it).

```
## Message house — [Brand]
Core message (roof): [the one promise]
| Pillar | Value theme | Proof (real only; [verify] / proof gap) |
|---|---|---|
| 1 | … | … |
| 2 | … | … |
| 3 | … | … |
Category: [from Dunford #5]   ·   Audience: [who-it's-for]
```

### Elevator pitch (optional, on request)

When asked for an "elevator pitch" / one-liner, derive it from the house — don't write a fresh one. Standard form: **"We help [who] [achieve value] by [the how / category], unlike [alternative] which [shortcoming]."** Keep it speakable in one breath; it must not assert anything the house can't back.

---

## Step 4 — Return or persist

**Called mode:** return TWO clearly-labeled blocks for brand-brain to fold in — ready to drop into `brand.md`:

- **`Positioning / core frame`** — the Dunford five (compact), the Moore positioning statement, the chosen market category, and the core message.
- **`Value proposition & differentiators`** — the headline value prop (one line), the 3–7 real differentiators (the unique attributes), and the message house (pillars + proof).

Do not return or touch any other section. Where your work implies content for a section you don't own (e.g. a sharper ICP from "who-it's-for," or proof you'd want in Proof assets), **suggest** it as a note — do not write it.

**Standalone mode:** persist into `brand.md` yourself, surgically:
1. Read the current `brand.md` at the path brand-brain resolved (`<data-root>/brands/<slug>/brand.md`).
2. Replace **only** the `## Positioning / core frame` and `## Value proposition & differentiators` blocks with your new content. Leave every other section byte-for-byte intact.
3. Bump `updated:` to today; append yourself to `sources:` (e.g. `positioning-messaging-architect YYYY-MM-DD`). Never touch `slug`/`created`/`status` or any section you don't own.
4. Confirm in one line where you saved: *"Positioning + value prop written to `<data-root>/brands/<slug>/brand.md` — every library skill reads it now."*

This skill has no companion file; all output lives in those two `brand.md` sections.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No positioning work before brand context is loaded (passed in called mode, fetched in standalone). Its voice + banned-words override everything here.
- **Own two sections, suggest the rest.** Write only "Positioning / core frame" and "Value proposition & differentiators." For anything else, suggest — never write.
- **Frame before messages.** Run Dunford in order; the category is chosen last as a consequence. Never start at the pillars.
- **Positioning is a choice, not a description.** Pick the category and the who deliberately — narrow to win. "For everyone" is a non-position.
- **Anchor on the real alternative.** Value claims only mean something against what the customer would otherwise do. Always name the alternatives.
- **Pillars are distinct axes.** Three different reasons to believe, not three phrasings of one.
- **No claim without proof.** Every pillar carries real evidence or a flagged proof gap. Unconfirmed → `[verify]`. Never invent proof, differentiators, or customer names.

## What Not to Do

- Don't produce positioning before `brand-brain` context is available.
- Don't write or overwrite any `brand.md` section other than the two you own (in called mode, write nothing at all).
- Don't call `brand-brain` back when you were called by it (no recursion).
- Don't invent a differentiator, a category leadership claim, a metric, or a customer to complete the symmetry of the house.
- Don't default to the crowded market category just because it's familiar — choose the frame you can win.
- Don't ship five "top-priority" pillars; force-rank to three.
- Don't deliver a tagline as the positioning statement, or skip the alternatives because "everyone knows the competitors."
- Don't store brand data inside the skill folder.

## Quality Checklist (self-review before returning/persisting)

- Mode detected correctly; brand context loaded (passed or via `brand-brain`) before any work?
- All five Dunford components filled from real facts, in order, with the market category chosen last and stated explicitly?
- Moore positioning statement complete (every slot real, or the gap flagged), one statement, under ~60 words?
- Core message is an argument, not a tagline; three distinct value pillars; each pillar carries real proof or a flagged proof/`[verify]` gap?
- Value prop + 3–7 real differentiators present; nothing invented; voice + banned-words honored?
- Output scoped to exactly the two owned sections; cross-section content offered as suggestions, not written?
- Standalone: only the two sections replaced, `updated` bumped, `sources` appended, other sections untouched, save location confirmed?
