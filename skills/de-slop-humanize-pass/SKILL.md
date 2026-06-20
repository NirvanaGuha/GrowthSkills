---
name: de-slop-humanize-pass
description: >
  Takes AI-drafted or generically "written" text and edits it into something a human would actually
  publish — stripping the tells (the "In today's fast-paced world," the rule-of-three padding, the
  "it's not just X, it's Y," the hedge-everything voice), then rebuilding it with concrete specifics,
  varied sentence rhythm, and a real first-person point of view. It runs a named editorial pass (the
  DESLOP loop) that diagnoses the slop, cuts the filler, injects the brand's real proof and concrete
  detail, varies the cadence, and verifies the result still says what it meant. It does NOT manage brand
  context itself — it calls the `brand-brain` skill to load the active brand's voice, banned words, ICP,
  and proof so the humanized version sounds like the brand and not like a different AI; it calls
  `proof-vault` for real specifics to swap in and `editorial-style-guide` for the brand's mechanical
  rules. Use whenever the user says "de-slop this," "humanize this," "make this sound less like AI,"
  "this reads like ChatGPT wrote it," "remove the AI tells," "make this sound human," "tighten this,"
  or hands over a draft that's grammatically fine but soulless. It edits prose — it does not write from
  scratch, restructure the argument, or change what the piece claims.
---

# De-Slop & Humanize Pass

Hand it a draft that's technically correct and completely lifeless — get back the version a person would actually put their name on. AI text fails in a specific, recognizable way: it's vague where it should be concrete, evenly paced where it should breathe, hedged where it should commit, and decorated with a small set of stock phrases that scream "a model wrote this." This skill names those failures, cuts them, and rebuilds the passage with real specifics and a real voice.

It is an **editing** skill, not a writing one. It preserves the draft's structure, claims, and intent — it does not invent a new argument, add sections, or fabricate facts to sound "specific." When the only way to fix vagueness is a number the draft never had, it asks for it or marks it `[verify]` rather than making one up. The voice it humanizes *toward* is the brand's real voice, loaded from `brand-brain` — not a generic "more casual" default.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice adjectives, banned words/phrases, ICP + awareness, positioning, and real proof. This skill humanizes *toward* that voice; it never re-derives it. It does not implement brand scanning/interviewing/storage — that lives in `brand-brain`, once.
- **`proof-vault`** (when installed) — the source of real, attributable specifics (stats, customer quotes, named results) to swap in where the draft is vague. Pull from here instead of inventing detail. Mark anything not in the vault `[verify]`.
- **`editorial-style-guide`** (when installed) — the brand's mechanical rules (casing, number format, product naming, the canonical banned-words list) so the cleaned copy is consistent, not just de-slopped.
- *(synthesize inline only when a sibling is unavailable; never fabricate proof to do so.)*

---

## How a run works

```
Step 0  Load the brand   ──► call `brand-brain` (it bootstraps on first use)
Step 1  Set the scope    ──► Surgical (default) | Full rewrite-in-voice (on request)
Step 2  Run the DESLOP loop on the passage
Step 3  Self-review (meaning preserved? voice matched? nothing fabricated?) then present
```

### Step 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the digest — voice adjectives, banned words, ICP + awareness, positioning line, and real proof — plus the `brand.md` path. **Do not edit a word until it returns.** Obey voice + banned-words as hard overrides, and pull only real proof when injecting specifics (everything else is `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, ask the user for 3 voice adjectives, the banned-word list, and the ICP, then proceed. Always prefer the call.

### Step 1 — Set the scope
- **Surgical (default).** Edit in place. Preserve structure, paragraphing, and every claim; change only what's slop. This is the everyday job and the safe default — the user trusts the draft's bones.
- **Full rewrite-in-voice (on request).** Triggered by "rewrite this in our voice," "do a heavier pass," or a draft that's slop top to bottom. Same argument and facts, freer sentence-level reconstruction.

When unsure, default to Surgical and offer the heavier pass at the end.

---

## The DESLOP loop (the framework)

Run these six moves over the passage, in order. Each targets a distinct failure mode — don't skip ahead; vagueness can't be paced into life, and rhythm can't fix a hedge.

| | Move | What it kills | What it does instead |
|---|---|---|---|
| **D** | **Detect the tells** | The stock-phrase fingerprint | Flag every AI tell against the inventory below; mark, don't yet rewrite |
| **E** | **Excise the filler** | Empty connective tissue, hedges, throat-clearing | Cut "In today's…," "It's worth noting," "Whether you're X or Y," rule-of-three padding, the "not just X, it's Y" reflex |
| **S** | **Specify** | Abstraction and vagueness | Replace generic claims with concrete nouns, real numbers, and named examples — from `proof-vault`, the draft itself, or `[verify]` |
| **L** | **Loosen the rhythm** | Uniform, metronomic sentence length | Vary cadence: short punch after a long line, sentence fragments where they land, one idea per sentence where it's dense |
| **O** | **Own a POV** | The omniscient hedge-voice | Add a real first-person / brand point of view: a stance, a "we've seen," a willingness to say what's actually true |
| **P** | **Preserve & prove** | Meaning drift and fabrication | Verify the edit still says what the original meant and added no fact that isn't real |

### D — the AI-tell inventory (cut or rewrite on sight)
- **Opener clichés:** "In today's fast-paced/digital world," "In the ever-evolving landscape of," "Picture this," "Let's dive in."
- **Stock connectives & hedges:** "It's worth noting that," "It's important to remember," "That being said," "At the end of the day," "Needless to say."
- **The slop syntax patterns:** *"It's not just X — it's Y."* · *"Whether you're a [A] or a [B], …"* · forced **rule-of-three** lists ("efficient, scalable, and reliable") · "From X to Y, …" sweeping ranges · em-dash-everywhere pacing.
- **Empty intensifiers & abstractions:** "robust," "seamless," "powerful," "game-changing," "in order to," "leverage," "utilize," "a myriad of," "navigate the complexities of," "unlock the power of," "in the realm of."
- **The wrap-up tic:** "In conclusion," "Ultimately, the key takeaway is," a final paragraph that restates everything already said.
- **Brand-specific banned words** from `brand-brain` / `editorial-style-guide` — these are *hard* cuts on top of the generic list.

### S — vague → concrete (the highest-leverage move)
Slop's deepest tell isn't a phrase, it's *vagueness*. Every fix swaps an abstraction for a specific:

| Slop (abstract) | Humanized (concrete) |
|---|---|
| "significantly improved results" | "cut cart abandonment from 71% to 58%" `[verify if not in proof-vault]` |
| "many businesses" / "leading companies" | "a 40-person Shopify store" or the named, real customer |
| "leverage powerful tools to streamline" | "use [actual tool] to [actual task]" |
| "in today's competitive landscape" | the actual condition: "when three rivals all undercut you on price" |
| "a wide range of benefits" | the two or three benefits that actually matter, named |

If a specific would require a fact the draft doesn't have and `proof-vault` can't supply, **leave it `[verify]` or ask** — never invent a number or a customer to win the sentence.

---

## What humanized actually means (the bar, not just the absence of tells)

Removing tells gets you to *clean*. These get you to *human*:
- **Rhythm you can hear.** Read it aloud. If every sentence is the same length, break the pattern. A short sentence after a long one is the cheapest way to sound alive.
- **A stance.** Humans commit. Replace "there are several factors to consider" with the take: which factor matters and why.
- **Concrete over comprehensive.** One vivid real example beats three abstract ones. Cut the list to its load-bearing items.
- **Earned transitions.** Ideas connect because they actually follow, not because "Moreover" / "Furthermore" / "Additionally" was inserted.
- **Voice math:** humanized ≠ unprofessional. Match the brand's actual register from `brand-brain` — for a confident-but-precise B2B voice, "human" means direct and specific, not slangy.

---

## Output

**Surgical (default):**
```
## De-slopped — [what it is]  (brand: <slug>, via brand-brain · scope: Surgical)
[the edited passage]

### What changed (and why)
- [tell/vagueness cut] → [the fix] — [the DESLOP move it served]
- … (group the meaningful changes; don't list every comma)
[⚠ specifics needed: any [verify] facts the user must confirm or supply]
```

**Full rewrite-in-voice:** the rewritten passage + a short before→after on the 3–5 highest-leverage moves + the same `[verify]` list.

If the piece is long or the user is iterating, offer to save to `./edits/<slug>-deslop-[topic].md` (a project path — never inside the skill folder, never `brand.md`).

---

## Principles (Non-Negotiable)
- **Brand-brain first.** No edits before it returns. Its voice + banned-words override the generic inventory here.
- **Edit, don't author.** Preserve structure, claims, and intent. If the *argument* is the problem, say so — don't quietly rewrite it.
- **Specificity is the cure.** The deepest de-slop move is vague→concrete, not phrase-swapping. A cleaner cliché is still a cliché.
- **Never fabricate to sound specific.** Real proof from `proof-vault`/the draft, or `[verify]`. A made-up stat is worse slop than the vagueness it replaced.
- **Humanize toward the brand, not toward "casual."** The target voice is the brand's real register, loaded from `brand-brain`.
- **Read it aloud.** Rhythm is verified by ear, not by rule.

## What Not to Do
- Don't edit before `brand-brain` returns the active brand.
- Don't reimplement brand resolution/scanning/storage — call `brand-brain`. Don't re-derive the banned-word list — pull it from `brand-brain`/`editorial-style-guide`.
- Don't invent numbers, customers, quotes, or differentiators to fill a "specific" — `[verify]` instead.
- Don't change what the piece claims or add new sections — that's writing, not de-slopping; flag it and stop.
- Don't trade one tic for another (every em-dash → every semicolon; every "leverage" → every "utilize").
- Don't strip a brand's *intentional* signature phrasing because it pattern-matches a tell — confirm against `brand-brain` voice first.
- Don't sand it down to bland. Removing slop without adding stance and specifics just makes shorter slop.

## Quality Checklist (self-review before presenting)
- `brand-brain` called and the active brand loaded (or bootstrapped) before any edit?
- Every tell from the inventory + every brand banned word cut or rewritten?
- At least the load-bearing vague claims made concrete — with real proof or marked `[verify]`?
- Sentence rhythm varied (read it aloud — not all one length)? A real POV present, not the hedge-voice?
- Meaning preserved, structure intact, **nothing fabricated**; claims unchanged?
- Result matches the brand's actual register (humanized ≠ off-voice)? `[verify]` list surfaced for the user?
