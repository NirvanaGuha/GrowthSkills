---
name: viral-moment-rapid-response-kit
description: >
  Trending topic URL or keyword spike → a brand-aligned, multi-channel rapid-response kit —
  hot-take blog post, X thread, LinkedIn angle, and social ad copy — delivered while the moment
  is still live. A flagship ORCHESTRATOR: it chains the library's specialist skills end-to-end
  (newsjacking-angle-finder → blog-post-drafting-engine + linkedin-post-writer + x-thread-writer
  + social-ad-copy-writer → post-quality-reviewer-voice-auditor) behind a single speed-and-risk
  gate, so one buyer can go from "this is trending" to publish-ready assets in one run instead of
  spinning up a whole content team. It does NOT write any asset itself — every stage is the
  specialist skill's job; this skill picks the angle, runs the gate, fans the work out in parallel,
  loops weak drafts back through review, and compiles one bundled deliverable. Loads the active
  brand via brand-brain first so every channel is on-voice, on-ICP, and provably credible — never
  opportunistic noise. Use whenever the user says "newsjack this," "we need to respond to this
  trend NOW," "react to this story across channels," "rapid response," "jump on this moment,"
  "this is blowing up — what do we post," or drops a trending URL / keyword spike and wants
  finished multi-channel copy fast.
---

# Viral-Moment Rapid-Response Kit

A trending moment has a half-life measured in hours. This skill is the growth team you don't have time to assemble: point it at a live story or keyword spike and it produces an on-brand, multi-channel response — blog, X thread, LinkedIn angle, and ad copy — fast enough to catch the wave, and reviewed enough that you're not embarrassed tomorrow.

It is an **orchestrator**. It does not write a single line of finished copy itself. It loads the brand once, picks the strongest credible angle, runs a speed-and-risk gate so you don't chase a dead or dangerous story, fans the chosen angle out to the channel specialists in parallel, loops any weak draft back through the reviewer, and compiles everything into one dated kit you can ship. Every craft decision lives in the specialist skills, where it belongs.

---

## Skills this calls

The pipeline, in dependency order. Each is invoked via the **Skill tool** — never re-implemented here.

1. **`brand-brain`** (Layer-0, required, always first) — resolves and loads the active brand: voice adjectives, banned words, ICP + awareness, positioning, offer mechanics + destination URLs, real proof. Nothing downstream runs until it returns.
2. **`newsjacking-angle-finder`** (Stage 1, the strategy brain) — turns the trending input into a ranked, risk-flagged menu of angles, each with a hook, spokesperson frame, and speed window. This is also the run's **gate**.
3. **`blog-post-drafting-engine`** (Stage 2, parallel) — writes the hot-take owned-media blog post from the chosen angle.
4. **`linkedin-post-writer`** (Stage 2, parallel) — writes the LinkedIn exec-reaction post.
5. **`x-thread-writer`** (Stage 2, parallel) — writes the X/Twitter thread.
6. **`social-ad-copy-writer`** (Stage 2, parallel) — writes paid social ad copy to amplify the moment.
7. **`post-quality-reviewer-voice-auditor`** (Stage 3, the gate-back) — scores every draft on hook strength, CTA, platform-fit, and brand-voice consistency; weak drafts loop back to their writer.
8. *(optional, conditional)* `advertising-claims-ftc-disclosure-reviewer` — invoked only if any asset makes a comparative, superlative, or regulated claim; `content-repurposer-atomizer` if the user wants more channels than the core four.

This skill owns no craft. It owns sequencing, the gate, the parallel fan-out, the review loop, and the compiled deliverable.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain (voice, positioning, proof, ICP, banned words)
Step 1  Find + gate the angle ──► call newsjacking-angle-finder → ranked angles + SPEED/RISK gate
        ⮑  GATE: Fading window or unmitigated reputational/legal risk → STOP, surface, get human OK
Step 2  Human picks the angle ──► one chosen angle + hook + spokesperson frame becomes the shared brief
Step 3  Fan out (PARALLEL)    ──► blog-post-drafting-engine ∥ linkedin-post-writer ∥
                                  x-thread-writer ∥ social-ad-copy-writer  (all get the SAME brief)
Step 4  Review (the gate-back)──► post-quality-reviewer-voice-auditor scores each draft
        ⮑  any draft = Revise → loop it back to its writer with the reviewer's notes (max 2 passes)
Step 5  Claims check (cond.)  ──► advertising-claims-ftc-disclosure-reviewer if any claim is risky
Step 6  Compile + hand off    ──► one dated kit folder under ./ ; human approves before publish
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Use the returned digest — voice adjectives, banned words, ICP + awareness tendency, positioning line, offer mechanics + destination URLs, real proof — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first. **Produce nothing until it returns.** Its voice and banned-words are hard overrides for every downstream stage; only its real proof may be cited as fact.

> **Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; else ask the user for the positioning line, ICP description, one real proof point, and 3 voice adjectives + any banned words, then proceed. This thin read is the sanctioned fallback — do not reimplement brand scanning, interviewing, or storage here, and never write `brand.md` yourself. Always prefer the call.

### Step 1 — Find the angle and run the gate

**Invoke `newsjacking-angle-finder`**, passing the trending URL/keyword + the brand digest. It returns 6–8 ranked angles, each with a hook, spokesperson frame, **speed window** (Hot / Warm / Cool / Fading), and **risk flags** (reputational / factual / legal). This is the orchestrator's gate — read its output before anything is written:

- **Speed gate.** If the top angle's window is **Fading**, stop and tell the user the moment is likely dead; offer an evergreen reframe instead of burning a parallel fan-out on a cold story. Hot/Warm/Cool → proceed, and carry the window into the deliverable so the human knows the publish deadline.
- **Risk gate.** If the chosen angle carries an **unmitigated reputational or legal flag** (tragedy-adjacent, names a competitor, touches a regulated claim, could read as opportunistic), **STOP and surface it to the human** with the mitigation note before any asset is written. Do not self-censor and do not push through — the human decides.

If no angle clears Relevance × Authority (the brand has no genuine surface on this story), say so and stop. Manufactured relevance is worse than silence.

### Step 2 — Human picks the angle (the shared brief)

Surface the top 3 angles and let the human choose one (in a true hurry, recommend #1 and proceed on confirmation). The chosen angle — its hook, spokesperson frame, the key proof point, the destination URL, and the speed window — becomes the **single shared brief** passed identically to every Stage-2 writer. One angle, four channels: this is what keeps the kit coherent instead of four skills riffing in four directions.

### Step 3 — Fan out to the channel specialists (parallel)

The four writers have **no dependencies on each other**, so invoke them concurrently, each with the same shared brief plus its channel constraints:

| Channel | Skill | Brief it receives | Returns |
|---|---|---|---|
| Hot-take blog | `blog-post-drafting-engine` | angle, hook, spokesperson frame, proof, target keyword, internal-link/CTA destination | owned-media post draft |
| LinkedIn | `linkedin-post-writer` | angle, hook, exec POV, one proof point, awareness stage | exec-reaction post (plain prose, no markdown) |
| X thread | `x-thread-writer` | angle, hook, the contrarian/insight beat, proof, CTA | numbered thread |
| Social ad | `social-ad-copy-writer` | angle, offer, ICP, objective, destination URL | ad variants (headline + primary text) |

Each writer calls `brand-brain` itself for full context — pass the digest so it doesn't re-bootstrap. Do not paraphrase one writer's output into another's; they share the **brief**, not each other's drafts.

### Step 4 — Review (the gate-back loop)

**Invoke `post-quality-reviewer-voice-auditor`** on every draft (batch them in one call where supported). It scores hook strength, CTA presence, platform-fit, and brand-voice consistency, returning Pass / Revise per asset.

- **Pass** → asset goes to the compiled kit.
- **Revise** → loop that asset **back to its own writer** with the reviewer's specific notes, then re-review. **Cap at 2 revise passes per asset.** If a draft still fails after two loops, include it flagged `NEEDS HUMAN EDIT` with the open reviewer notes rather than shipping it silently or faking a pass.

Authoring and review are separate lanes: the writer revises, the reviewer judges — never let one stage approve its own output.

### Step 5 — Claims check (conditional)

If any drafted asset makes a comparative claim ("better than X"), a superlative ("the #1…"), an unverifiable statistic, or touches a regulated area, **invoke `advertising-claims-ftc-disclosure-reviewer`** on those assets and apply its required fixes before the kit is final. Skip this stage entirely when no asset makes a risky claim.

### Step 6 — Compile the bundled deliverable

Write the kit to a **project-relative path under the user's CWD** (never the skill folder):

```
./rapid-response/[story-slug]-[YYYY-MM-DD]/
  00-README.md          # the run sheet (below)
  01-angle.md           # chosen angle, hook, spokesperson frame, speed window, risk notes
  02-blog.md            # blog-post-drafting-engine output
  03-linkedin.md        # linkedin-post-writer output (plain prose)
  04-x-thread.md        # x-thread-writer output
  05-social-ad.md       # social-ad-copy-writer output
  06-review.md          # reviewer scores + revise history + any NEEDS HUMAN EDIT flags
```

`00-README.md` is the run sheet a human approves at a glance:

```
# Rapid-Response Kit — [story title]
Brand: [slug, via brand-brain]   Speed window: [Hot/Warm/Cool] — publish by [time]
Chosen angle: [type] — "[hook]"   Spokesperson frame: […]
Gate: [risk flags + mitigation, or "clear"]
Assets:  Blog ✓  LinkedIn ✓  X thread ✓  Social ad ✓   (✓ Pass / ⚠ needs human edit)
Claims check: [run / not needed]
Publish order + destinations: […]
```

If the user only wants one compiled doc instead of a folder, concatenate the same sections into `./rapid-response/[story-slug]-[YYYY-MM-DD].md`. **The human approves the kit before anything is published** — this skill drafts and compiles; it does not publish (hand to `publishing-integration-hub` if the user wants that next).

---

## Orchestration logic (gates, branches, parallelism, human approval)

- **Brand gate (Step 0).** No stage runs before `brand-brain` returns. Hard stop.
- **Speed gate (Step 1).** Fading window → stop or offer evergreen reframe. The window sets the publish deadline carried through the whole kit.
- **Risk gate (Step 1).** Unmitigated reputational/legal flag → STOP, surface to human, wait for an explicit go. Never push through silently.
- **Human-in-the-loop #1 (Step 2).** Human picks the angle before any asset is written — this is the cheapest place to course-correct.
- **Parallel fan-out (Step 3).** The four writers are independent; run them concurrently and pass each the identical brief, not each other's drafts.
- **Review loop (Step 4).** Revise → back to the same writer with notes → re-review, max 2 passes; still failing → flag `NEEDS HUMAN EDIT`, never fake a pass.
- **Conditional claims gate (Step 5).** Only if an asset makes a risky claim.
- **Human-in-the-loop #2 (Step 6).** Human approves the compiled kit before publish.
- **Degrade gracefully.** If a specialist skill is unavailable, say which one and what's missing; fill the gap with a clearly-labeled inline placeholder rather than silently reimplementing that stage's craft.

---

## Principles

- **Orchestrate, don't author.** Every asset is a specialist skill's output. This skill sequences, gates, parallelizes, loops, and compiles — it writes no finished copy itself.
- **Brand-brain first, always.** Voice and banned-words are hard overrides on every stage; only real proof from `brand.md` is fact.
- **Speed is the whole point.** Always show the speed window and the publish-by deadline. A perfect kit that lands after the moment died is a loss.
- **One angle, many channels.** All four writers share one chosen brief so the response is coherent across surfaces — not four disconnected takes.
- **Authority over opportunism.** Only respond where the brand has a genuine, verifiable surface on the story. Manufactured relevance is worse than silence.
- **Risk-flag, then let the human decide.** Surface reputational/legal flags with mitigation; don't push through and don't quietly self-censor a viable angle.
- **Truth discipline.** Every factual claim is sourced from `brand.md` or tagged `[verify]`. No invented stats, customers, or differentiators.
- **Review is a separate lane.** The writer revises; the reviewer judges. Never self-approve in the authoring context.

## What not to do

- Don't write any blog, post, thread, or ad copy directly — call the specialist skill.
- Don't run any stage before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage, and never write `brand.md` yourself — the thin fallback read is the only exception.
- Don't push a Fading-window story into a full fan-out; surface it and offer an evergreen reframe.
- Don't ship an asset that's tragedy-adjacent, names a competitor, or makes a regulated claim without the risk gate and (where relevant) the claims reviewer clearing it.
- Don't fan the writers off each other's drafts — share the chosen brief, not derivative copy.
- Don't loop a Revise more than twice; flag `NEEDS HUMAN EDIT` instead of faking a pass.
- Don't write the kit into the skill folder — save under the user's CWD `./rapid-response/…`.
- Don't publish; compile and hand off for human approval.

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any stage ran?
- `newsjacking-angle-finder` run; speed window shown and the publish-by deadline carried into the kit?
- Speed gate honored (Fading → stopped/reframed) and risk gate honored (unmitigated flag → surfaced to human)?
- Human picked the angle; the same single brief was passed to all four writers?
- All four channel skills invoked (blog, LinkedIn, X thread, social ad) — none authored inline here?
- Every draft scored by `post-quality-reviewer-voice-auditor`; Revises looped back to their writer (≤2 passes); leftovers flagged `NEEDS HUMAN EDIT`?
- Claims reviewer run if and only if an asset made a comparative/superlative/regulated claim?
- Every factual claim sourced from `brand.md` or tagged `[verify]`; voice + banned-words honored on every asset?
- Kit compiled to `./rapid-response/[story-slug]-[YYYY-MM-DD]/` (or single doc) with a run-sheet README, and handed off for human approval before publish?
