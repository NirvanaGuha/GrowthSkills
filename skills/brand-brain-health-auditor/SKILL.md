---
name: brand-brain-health-auditor
description: >
  Audits the active brand's full brain bundle — brand.md plus all companion files
  (proof.md, personas.md, competitors.md, objections.md, style-guide.md) — and
  produces a structured health report before any downstream content skill runs off
  stale, contradictory, or incomplete data. Runs a four-axis diagnostic: Freshness
  (how old are the core facts?), Completeness (which required fields are absent or
  skeletal?), Consistency (are there internal contradictions between sections or
  companion files?), and Truth Integrity (which figures are unverified, which proof
  points need citation?). Outputs a prioritized fix list — Critical / High / Low —
  that the user or the `brand-brain` refresh command can act on immediately. Pairs
  naturally with brand-brain's own `brand-brain audit` command (this skill extends it
  with a structured scoring rubric and a save-to-file report) and with downstream
  auditors like `brand-consistency-auditor` (creative assets) and
  `positioning-reviewer` (positioning statement critique). Use whenever the user says
  "audit the brand brain," "check the brand brain," "is the brand brain stale,"
  "health check," "what's wrong with my brand data," "before we relaunch," "refresh
  the brain," or any downstream skill surfaces a `[verify]` warning and needs its
  root cause traced.
---

# Brand Brain Health Auditor

The rest of the library is only as good as the data it reads. A stale positioning line, an unverified proof stat, or a missing ICP section silently corrupts every CTA, every email sequence, and every piece of ad copy that follows. This skill catches those problems at the source — before they ship.

It does not rebuild the brand. It audits what `brand-brain` already wrote and tells you exactly what to fix, in priority order.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand and serves its `brand.md` + companion file paths. This skill reads what `brand-brain` wrote; it does not reimplement brand resolution.
- *(optional, when installed)* **`positioning-reviewer`** — called to score the positioning statement if a deep critique is requested; this skill flags staleness/gaps, `positioning-reviewer` scores the statement's quality.
- *(optional, when installed)* **`brand-consistency-auditor`** — covers creative-asset drift; this skill covers data-file drift. Both flag different surfaces of the same underlying problem.
- *(optional, when installed)* **`proof-vault`** — consulted to cross-check whether a proof point in `brand.md` is also registered (and verified) in the proof vault.

---

## How a run works

```
Step 0  Load the brand   ──► call brand-brain skill (fast Serve path)
Step 1  Inventory files  ──► which companion files are present vs missing?
Step 2  Score on 4 axes  ──► Freshness · Completeness · Consistency · Truth Integrity
Step 3  Build fix list   ──► Critical / High / Low with owner and repair action
Step 4  Save & present   ──► write ./brand-audit/[slug]-health-[date].md; show summary
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and receive the digest + `brand.md` path. Do not begin auditing until it returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, tell the user to run `brand-brain` first — there is nothing to audit yet.

Then read `brand.md` in full. Read every companion file that is present alongside it (`proof.md`, `personas.md`, `competitors.md`, `objections.md`, `style-guide.md`). Note which expected companions are absent.

---

## The FCCI Framework (Freshness · Completeness · Consistency · Truth Integrity)

This is the scoring rubric. Apply all four axes to every brand you audit. Each axis produces a verdict and a set of findings.

### Axis 1 — Freshness

Brand data decays. Pricing changes quarterly. Proof stats go stale. Competitors move. Judge every time-sensitive field against its last confirmed update.

**Staleness signals to flag:**
- `updated` timestamp in `brand.md` is > 90 days old → **Critical** if core fields (positioning, pricing, proof) have not been re-verified since.
- `updated` is 30–90 days → **High** if the brand has active campaigns touching those fields.
- Any proof stat or pricing figure with no `updated` date at all → **High** (assume unknown vintage).
- Competitor entries in `competitors.md` with no date → **Low** (likely stale but lower blast radius).

Freshness verdict: `Fresh` (< 30d, core fields verified) | `Aging` (30–90d) | `Stale` (> 90d or no date).

### Axis 2 — Completeness

The `brand.md` schema has required and optional fields. Missing required fields break downstream skills. Skeletal fields (present but ≤ 10 words) are almost as bad.

**Required fields in brand.md** (flag Critical if absent, High if skeletal):
- `positioning_line` — one crisp differentiated sentence
- `icp` — job title / company type / key pain / awareness stage tendency
- `voice_adjectives` — ≥ 3 adjectives
- `banned_words` — explicit list (even if short)
- `offer_mechanics` — free-trial / freemium / card-required / guarantee + primary destination URL
- `proof` — ≥ 2 verified proof points with source

**Optional but high-value** (flag Low if absent when brand is mature):
- `objections.md` companion
- `competitors.md` companion with at least 3 named rivals
- `style-guide.md` companion
- `personas.md` companion

Completeness verdict: `Complete` (all required + ≥ 3 optional) | `Partial` (all required, some optional missing) | `Incomplete` (≥ 1 required field absent or skeletal).

### Axis 3 — Consistency

Files written at different times drift. The ICP in `brand.md` may contradict the personas in `personas.md`. The positioning line may conflict with the competitive moat described in `competitors.md`. The voice adjectives may conflict with the style guide's tone.

**Consistency checks to run:**
- `brand.md` ICP ↔ `personas.md` primary persona: same job title range? Same awareness stage? Same core pain?
- `brand.md` positioning_line ↔ `competitors.md` differentiation column: does the stated moat appear in the competitive analysis?
- `brand.md` proof points ↔ `proof.md` (if present): are the headline stats aligned? Or did one get updated and the other didn't?
- `brand.md` voice_adjectives ↔ `style-guide.md` tone section: do they describe the same register?
- `brand.md` banned_words ↔ `style-guide.md` language rules: any conflicts or one-way mentions?
- `brand.md` offer_mechanics (price/tier names) ↔ any pricing copy in `style-guide.md` or `objections.md`: same tier names, same price points?

For each contradiction: quote both conflicting passages, name the files, and state which one the user should treat as canonical (or flag it as unknown).

Consistency verdict: `Aligned` (no contradictions) | `Minor Drift` (1–2 low-stakes discrepancies) | `Contradicted` (≥ 1 core-field conflict).

### Axis 4 — Truth Integrity

`[verify]` markers in the brain mean a downstream skill will silently inherit an unconfirmed claim. Find them all and rank by blast radius.

**Truth integrity checks:**
- Scan every field in `brand.md` and all companions for `[verify]` tags → list each, note which downstream skills are most likely to use it.
- Flag any numeric claim (conversion rate, user count, revenue figure, NPS score) with no cited source, even if not tagged `[verify]`.
- Flag superlatives ("the only," "the fastest," "#1") with no evidence.
- Flag named differentiators ("enterprise-grade security," "real-time sync") that have no supporting proof point anywhere in the bundle.

Blast-radius scoring: a `[verify]` in `positioning_line` or `proof` is **Critical** (used by nearly every downstream skill). A `[verify]` in `competitors.md` is **Low** (fewer skills read it). A `[verify]` in a persona's quote is **Low**.

Truth Integrity verdict: `Clean` (0 unverified claims in core fields) | `Flagged` (unverified claims in non-core fields only) | `Compromised` (unverified claims in positioning, proof, or offer_mechanics).

---

## The Fix List

After scoring all four axes, compile a single prioritized fix list:

```
## Brand Brain Health Report — [slug] — [date]

### Summary Scorecard
| Axis               | Verdict       |
|--------------------|---------------|
| Freshness          | Fresh / Aging / Stale |
| Completeness       | Complete / Partial / Incomplete |
| Consistency        | Aligned / Minor Drift / Contradicted |
| Truth Integrity    | Clean / Flagged / Compromised |

Overall health: GREEN / AMBER / RED
(GREEN = all Fresh/Complete/Aligned/Clean; RED = any Critical finding; AMBER = otherwise)

### Critical Findings  (fix before any content ships)
[Finding ID] — [field/file] — [what's wrong] — [exact repair action]

### High Findings  (fix this sprint)
[Finding ID] — [field/file] — [what's wrong] — [exact repair action]

### Low Findings  (backlog)
[Finding ID] — [field/file] — [what's wrong] — [exact repair action]

### Recommended next command
brand-brain refresh [slug]  ← re-scan + fill gaps + bump updated
```

**Exact repair actions, not advice.** Don't write "consider updating the proof stat." Write: "Replace `[verify]` on `proof[0].stat` — confirm the current figure directly from [source] and update `brand.md` line 34."

---

## Output and persistence

Save the full report to `./brand-audit/[slug]-health-[YYYY-MM-DD].md` (project-relative; create the folder if absent). Present the Summary Scorecard and Critical findings inline. Offer to walk through High findings if there are more than three.

After presenting: if the overall health is RED or there are Critical findings, recommend running `brand-brain refresh [slug]` immediately. Do not run the refresh automatically — the user decides.

---

## Principles (Non-Negotiable)

- **Audit the brain, don't rebuild it.** This skill reads and critiques. `brand-brain` bootstraps and refreshes. Never write a new `brand.md` or modify the existing one.
- **Quote the problem exactly.** Every finding cites the specific field, file, and line context. No vague "the positioning could be stronger."
- **Rank by blast radius.** A `[verify]` in `positioning_line` corrupts dozens of downstream outputs. A stale competitor entry affects one skill. Severity must reflect actual propagation risk.
- **Both directions of contradiction.** When two files conflict, name both, quote both, and state the canonical source (or flag it as unknown) — don't just flag the "newer" one.
- **Never overwrite brand.md.** This skill has read-only intent on all brand files. The user's repair command is `brand-brain refresh`.
- **Truth only.** If a field looks wrong but you can't confirm it, flag it as `[verify]` in the report — don't assert that it's incorrect.

## What Not to Do

- Don't produce any marketing copy, positioning, or CTA output. That is downstream work.
- Don't call `brand-brain` in Bootstrap mode — if no brain exists, tell the user to run `brand-brain` first.
- Don't re-run `brand-brain refresh` automatically after the audit. Surface the fix list, let the user trigger the refresh.
- Don't flag every old fact as Critical — stale competitor pricing is not the same blast radius as a stale proof stat used in every ad headline.
- Don't skip the Consistency axis just because the files look superficially coherent. The worst contradictions are subtle (same metric, different number across two companion files).
- Don't write the report without saving it. The saved file is the paper trail.

## Quality Checklist (self-review before presenting)

- `brand-brain` called first and the active brand loaded before any analysis?
- All four FCCI axes scored with a named verdict?
- Every finding has: the field/file name, a quoted passage, a severity level, and an exact repair action?
- Critical findings are genuinely Critical (blast radius: used by multiple downstream skills in core output)?
- Consistency check ran across all present companion files (not just brand.md in isolation)?
- All `[verify]` tags found and ranked by blast radius?
- Report saved to `./brand-audit/[slug]-health-[date].md`?
- Recommended next command included?
- Did not write to or modify any brand file?
