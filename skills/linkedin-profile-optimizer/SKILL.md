---
name: linkedin-profile-optimizer
description: >
  Rewrites a LinkedIn profile's headline, About section, and Featured section so the page converts
  visitors into inbound leads, partnerships, or opportunities — aligned to the brand's ICP and voice.
  Two modes: Quick (default) takes a raw profile URL or pasted text and delivers a rewritten headline,
  About copy, and Featured-section recommendation in one pass; Audit mode starts from a scored gap
  analysis before rewriting. Calls brand-brain to load the active brand's ICP, voice, and proof so
  every line speaks to the right buyer in the right tone; calls icp-persona-builder if the ICP is
  missing or thin; calls proof-vault for credentialed social proof; calls cta-variant-generator for
  the About section's closing CTA line. Delegates voice review to post-quality-reviewer-voice-auditor
  before presenting final copy. Output saved to ./linkedin/[slug]-profile.md when requested.
  Use whenever the user says "optimize my LinkedIn," "rewrite my headline," "fix my About section,"
  "LinkedIn profile audit," "make my profile attract [persona]," "LinkedIn bio," or hands over a
  profile URL or raw About/headline text.
---

# LinkedIn Profile Optimizer

A LinkedIn profile is a landing page visited at average 3–5 seconds before a connection decision. Most are written from the inside out — accomplishment lists that mean nothing to a buyer who landed there cold. This skill rewrites from the outside in: ICP-first, proof-anchored, voice-consistent copy that makes the right visitor lean in instead of bounce.

The framework is **the ICP Gravity Stack** — four zones of the profile, each doing a single conversion job at a specific awareness level. Nail all four in sequence and the profile self-selects for quality inbound.

---

## Skills this calls

- **`brand-brain`** (required, always first) — active brand's ICP, voice adjectives, banned words, positioning, and proof digest. Do not write a single line before this returns.
- **`icp-persona-builder`** (conditional) — called when the brand digest has a thin or missing ICP, or when the user says "I'm not sure who I'm targeting." Returns a full ICP persona to anchor every rewrite decision.
- **`proof-vault`** (conditional) — called to surface credential-grade proof points (client names, numbers, award names) that can appear in the About section or Featured copy. Mark anything not returned as `[verify]`.
- **`cta-variant-generator`** (conditional) — called to generate the closing CTA line of the About section. Pass the ICP, offer, and placement ("LinkedIn About section bottom") so it respects the commitment ceiling.
- **`post-quality-reviewer-voice-auditor`** (always, as final pass) — reviews the completed draft against brand voice before presenting. If the skill is absent, run an inline voice check: read back the brand's voice adjectives and banned words, scan each line, flag any mismatch.

Downstream handoff (not a dependency): **`linkedin-post-writer`** creates individual posts. Once the profile is optimized, it's a natural next step.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (mandatory)
Step 1  Assess the input    ──► profile URL / pasted text / "I'll describe"
Step 2  Pick mode           ──► Quick (default) | Audit-first
Step 3  Apply ICP Gravity Stack framework across all four zones
Step 4  Voice review pass   ──► post-quality-reviewer-voice-auditor (or inline)
Step 5  Present + save offer
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool). It returns: ICP (role, company size, awareness tendency), voice adjectives, banned words, positioning line, and real proof. If ICP is thin, invoke `icp-persona-builder` before proceeding. Do not write any profile copy until both return.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if absent, ask the user to install `brand-brain` or answer four questions: (1) What do you/your product do in one sentence? (2) Who is the ICP — role, seniority, industry, company size? (3) What outcome does the ICP care about most? (4) Three voice adjectives + any words to avoid?

### Step 1 — Assess the input

Accept any of:
- A LinkedIn profile URL (read it or ask the user to paste the relevant sections)
- Pasted raw headline + About text (± current Featured section descriptions)
- "I'm starting from scratch" → ask for current role/offer and proceed to rewrite from the framework

If only a URL is given and you cannot fetch it, ask the user to paste the headline and About text. Do not block on the full profile if those two fields are present.

### Step 2 — Pick mode

- **Quick mode (default):** analyze then rewrite in one pass. The everyday job.
- **Audit mode:** triggered by "audit," "score," "what's wrong," or "review before rewriting." Run the four-zone scorecard first, present the gap analysis, then offer the rewrite. Useful when the user wants to understand why, not just receive the fix.

---

## The ICP Gravity Stack (the core framework)

LinkedIn profiles fail because writers stack credentials for an imaginary recruiter instead of writing for the actual buyer who lands there cold. The ICP Gravity Stack maps four profile zones to four conversion jobs, each at a specific Schwartz awareness level.

### Zone 1 — The Headline (Unaware → Problem-Aware)

**Job:** stop the scroll and signal relevance within 3 seconds.

**Reality check:** LinkedIn's default is "Job Title at Company." That's a credential, not a hook. The ICP's first question is *"Does this person solve my kind of problem?"* — the headline must answer it before the visitor reads another word.

**Formula:** `[ICP outcome] | [How/who you serve] | [optional: differentiator or proof stub]`

Rules:
- Lead with the outcome the ICP cares about — not your job title.
- Include the ICP role or industry if the offer is niche ("for SaaS founders" / "eCommerce operators").
- Headline limit: 220 characters. Aim for 120–160 so it renders fully on mobile.
- No `||` decoration, no emoji unless brand voice allows it, no buzzwords the brand bans.
- At least one version must be testable against a simpler, title-led variant (present both; label the recommended primary).

### Zone 2 — The About Section (Problem-Aware → Solution-Aware)

**Job:** make the ICP feel understood, credible-ize the person/brand, and hand off to action.

**Structure (five beats, ~250–320 words — LinkedIn's indexed sweet spot):**

1. **Opening hook (1–2 lines):** Name the problem or frustration the ICP lives with daily. Not "I help X do Y" — the reader's pain, in the reader's language. First-person is fine; opener should not start with "I" (LinkedIn de-prioritizes it).
2. **Credibility statement (2–3 lines):** Who you are and why that matters to this specific buyer. One real proof point (number, name, outcome) if available from `proof-vault` — else `[verify]`.
3. **Core offer / mechanism (2–3 lines):** What you do and how it works, at the *solution-aware* level. Specific enough to self-select; general enough not to limit. Use the brand's actual positioning line here.
4. **Social proof anchor (2–3 lines):** A second proof point or a specific outcome — client category, volume, result. Not a quote (quotes in About look forced); a fact.
5. **CTA line (1 line):** Low-commitment, matches the ICP's awareness ceiling. Delegate to `cta-variant-generator` with placement = "LinkedIn About section, solution-aware ICP." Use the primary returned; present the alternate as a test option.

Format rules: Short paragraphs (2–3 sentences max). No bullet lists in About (LinkedIn renders them poorly and they read as a resume, not a person). No first word "I" in the opener. Plain prose, not markdown.

### Zone 3 — The Featured Section

**Job:** hand off the most-aware visitor to a conversion asset without them leaving for a search.

Three-slot framework (prioritize in this order):

| Slot | Best asset type | Why |
|---|---|---|
| 1 | Lead magnet, case study, or key landing page | Highest-intent action for a warm visitor |
| 2 | Best-performing piece of content (post or article) | Social proof via engagement numbers |
| 3 | Media mention, award, or external credential | Third-party validation |

For each slot, provide:
- **Asset recommendation:** what to put there (or what to create if missing)
- **Title copy:** 55-char max (the visible card title on desktop)
- **Description copy:** 120-char max (visible below title)
- Flag if the user has no lead magnet or case study — note that Slot 1 is the highest-leverage gap to fill.

### Zone 4 — The Experience + Skills Skimmer (Product-Aware)

**Job:** confirm the visitor's due-diligence read before they DM or book.

This zone is not rewritten in full — that would require the full work history. Instead, provide:
- **Top-of-experience guidance:** how to write the first 2–3 lines of the current role entry so it reinforces the headline promise (offer + outcome, not duties).
- **Skills section:** recommend 5 skills to pin (the three that appear above the fold drive endorsement requests). Pick for ICP search relevance, not vanity.

---

## Deliverable format

```
## LinkedIn Profile — [Brand / Person name]

### Recommended Headline
[Primary — 120–160 chars]
[Alternate (test option)]

### About Section
[Full rewritten About, five-beat structure, ~250–320 words, plain prose]

### Featured Section Recommendations
Slot 1: [Asset type] | Title: "[55-char copy]" | Desc: "[120-char copy]"
Slot 2: [Asset type] | Title: "[55-char copy]" | Desc: "[120-char copy]"
Slot 3: [Asset type] | Title: "[55-char copy]" | Desc: "[120-char copy]"
[Gap flag if no lead magnet exists]

### Experience Entry Guidance (current role)
[2–3 lines on how to open the current role description]

### Skills to Pin (top 3 + 2)
1. [Skill] — why it fits ICP search
2. …

### Voice Audit
[Result of post-quality-reviewer-voice-auditor pass, or inline check]

### What to test first
[The single change with the highest expected return — usually the headline]
```

Save to `./linkedin/[brand-slug]-profile.md` when the user asks, or after a full Audit run.

---

## Audit mode scorecard

Run this before rewriting in Audit mode. Score each zone 1–5. Present the table, call out the two lowest scores as priorities, then offer to proceed to the rewrite.

| Zone | What you're scoring | Score (1–5) | Top gap |
|---|---|---|---|
| Headline | ICP-outcome clarity, no generic title, char range | | |
| About opener | Does NOT start "I", names ICP pain, not self-intro | | |
| About structure | Five beats present, proof anchored, CTA present | | |
| Featured | Lead magnet or case study in Slot 1, titles under 55 chars | | |
| Voice match | Brand adjectives present, banned words absent | | |

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before brand-brain returns the active brand digest. Its voice and banned words override everything here.
- **ICP-out, not credential-in.** Every line earns its place by answering a buyer question, not by listing an achievement.
- **Five beats, plain prose.** About section is always structured around the five beats; never a bulleted resume.
- **Real proof or `[verify]`.** If proof-vault doesn't surface it, mark the claim `[verify]`; never invent a number or client name.
- **Commitment ceiling respected.** The CTA matches the ICP's awareness level — no transactional ask on a cold profile visit.
- **Char limits are hard constraints.** Headline ≤220 (aim 120–160 for mobile), Featured title ≤55, Featured desc ≤120.
- **Present a test option.** Always give a headline alternate on a different angle — the user needs something to test against.

## What Not to Do

- Don't rewrite the full work history — that's out of scope and dilutes focus; guide the current-role opener only.
- Don't invent proof points (client names, revenue figures, award names) — only use what proof-vault returns.
- Don't open the About section with "I" — LinkedIn's algorithm and human readers both penalize it.
- Don't put bullet lists in the About section — prose converts better and doesn't read as a resume.
- Don't ignore the Featured section — it's the only click-out on the profile and most profiles leave it empty or stale.
- Don't skip the voice audit pass — brand-brain loads the rules; post-quality-reviewer-voice-auditor (or the inline check) enforces them.
- Don't recommend generic skills ("Leadership," "Strategy") — only skills the ICP would actually search for.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and returned ICP + voice before any copy was written?
- `icp-persona-builder` called if ICP was thin or missing?
- `proof-vault` consulted; all unverified claims marked `[verify]`?
- `cta-variant-generator` called for the About CTA line with correct placement?
- `post-quality-reviewer-voice-auditor` run (or inline voice check done) on final draft?
- Headline: primary + alternate, each on a different angle, ≤220 chars, ICP outcome leads?
- About: five beats present, opener does not start "I," plain prose (no bullets), ~250–320 words?
- Featured: three slots addressed, Slot-1 gap flagged if no lead magnet/case study exists?
- Experience guidance and top-5 skills provided?
- "What to test first" recommendation included?
- Output saved to `./linkedin/[slug]-profile.md` if a full run or user requested save?
