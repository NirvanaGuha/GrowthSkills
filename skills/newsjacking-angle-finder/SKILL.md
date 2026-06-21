---
name: newsjacking-angle-finder
description: >
  Trending news headline + company positioning → ranked pitch angles connecting the story to your
  expertise before the cycle moves on. Takes a breaking or trending news item and the active brand's
  positioning context, then produces a prioritized set of newsjacking angles — each with a pitch
  hook, a spokesperson frame, and a "speed window" so you know whether to move in hours or days.
  Angles are ranked by fit score (relevance × brand authority × cycle timing), flagged for risk
  (reputational, factual, legal), and matched to an output format (media pitch, LinkedIn exec post,
  op-ed hook, podcast booking angle, or owned-media blog intro). Calls brand-brain for positioning
  and proof so every angle is on-voice and provably credible — never opportunistic noise. Does NOT
  write the finished assets; hands off to media-podcast-pitch-crafter, linkedin-post-writer,
  blog-post-drafting-engine, or press-release-social-blog-amplification-pack. Use whenever the user
  says "newsjack," "jump on this story," "react to the news," "trending story angle," "PR hook,"
  "media angle for this headline," "how do we get coverage from this," or drops a news URL or
  headline and asks what to do with it.
---

# Newsjacking Angle Finder

Breaking news waits for no one. This skill turns a trending headline into a ranked menu of credible angles — each tied to your brand's real expertise, scoped to the time window still open, and flagged for every risk worth knowing before you pitch.

It does the strategy work: relevance scoring, spokesperson framing, output-format routing, and risk flagging. Once the best angle is chosen, the right downstream skill does the writing.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, ICP, positioning, real proof, and banned words. No angle is written before it returns.
- *(downstream — call when the user picks an angle)* `media-podcast-pitch-crafter` for press/podcast pitches; `linkedin-post-writer` for exec reaction posts; `blog-post-drafting-engine` for a timely owned-media piece; `press-release-social-blog-amplification-pack` if the brand itself has a release to amplify; `content-repurposer-atomizer` to fan a single angle across channels.

---

## How a run works

```
Step 0  Load the brand              ──► call brand-brain (voice, positioning, proof, ICP)
Step 1  Classify the story          ──► speed window, story type, risk tier
Step 2  Generate raw angle pool     ──► 6–8 angles via the Newsjacking Matrix
Step 3  Score + rank                ──► Relevance × Authority × Cycle-Timing
Step 4  Risk-flag every angle       ──► reputational, factual, legal
Step 5  Recommend output format     ──► route each angle to the right skill
Step 6  Present the ranked table    ──► user picks; you hand off
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns: voice adjectives, banned words, offer mechanics, real proof, positioning line, ICP + awareness tendency. Do not produce a single angle before it returns.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If none exists, ask the user for: positioning line, ICP description, one real proof point, and 3 voice adjectives + any banned words. Then proceed.

---

### Step 1 — Classify the story

Before angling, establish:

| Dimension | Options |
|---|---|
| **Speed window** | Hot (move in <4 h), Warm (4–24 h), Cool (1–3 days), Fading (>3 days — flag) |
| **Story type** | Regulatory/policy · Market shift · Research/data release · Competitor news · Cultural moment · Crisis/scandal · Technology announcement |
| **Brand's authority surface** | Where does the brand have genuine expertise that intersects this story? |
| **ICP relevance** | Does this story touch the pain, goal, or world of the brand's ICP? |

If the window is Fading: say so clearly and either close or redirect to an evergreen angle.

---

### Step 2 — The Newsjacking Matrix (core framework)

David Meerman Scott's Newsjacking principle: inject your brand's viewpoint into a breaking story while journalists are actively seeking expert sources. The matrix maps story type to brand authority surface to produce six structural angle types:

| Angle type | Core move | Best when |
|---|---|---|
| **Expert commentary** | Brand has specific, credible POV on why this happened or what it means | You have a named domain expert + real data |
| **Contrarian/counter-narrative** | Respectfully disagree with the dominant take, using proof | The consensus narrative has a blind spot you can fill |
| **Data point injection** | You own a stat, report, or original research that reframes the story | The news story is data-driven; your numbers add or correct |
| **ICP impact frame** | Translate macro news into what it means for your specific buyer | ICP is affected but story coverage misses the practitioner angle |
| **Prediction / forward-look** | Use the news event as a launchpad for a credible forecast | You can back the forecast with existing trend data or proof |
| **Case / proof tie-in** | A real customer story or outcome is direct evidence relevant to this story | You have named, verifiable proof (not `[verify]`) |

Generate 6–8 candidate angles, one per angle type where viable, by combining the story's facts with the brand's positioning and proof. Label each with its type.

---

### Step 3 — Score and rank

Score every angle on three dimensions (1–5 each):

| Dimension | What it measures |
|---|---|
| **Relevance** | How directly does the story connect to the brand's core product/category? |
| **Authority** | How credibly can the brand's spokesperson speak to this angle given real proof on file? |
| **Cycle timing** | Is the window still open? Hot = 5, Warm = 4, Cool = 3, Fading = 1 |

**Fit Score = (Relevance + Authority) × Cycle Timing** — multiply by timing, not add, because a brilliant angle at zero timing is worthless.

Sort descending. Surface the top 3 in the presentation; show the rest in a collapsed "further options" section.

---

### Step 4 — Risk flag every angle

Every angle gets a risk tag before it's shown to the user:

- **Reputational risk** — could the angle read as opportunistic, tone-deaf, or crisis-adjacent? Flag if the story involves tragedy, scandal, or a brand the ICP has strong feelings about.
- **Factual risk** — does the angle rely on claims that need verification? Mark each unconfirmed claim `[verify]`. Do not invent proof or infer unconfirmed brand data.
- **Legal risk** — does the angle imply endorsement by the news party, make a comparative claim about a named competitor, or touch regulated areas (financial, medical, legal)? Flag.

A flagged angle is not automatically killed — it's shown to the user with the flag and a mitigation note. The user decides.

---

### Step 5 — Output format routing

Match each top angle to the right downstream skill:

| Format | Skill to call |
|---|---|
| Media pitch / journalist outreach | `media-podcast-pitch-crafter` |
| Exec reaction post (LinkedIn) | `linkedin-post-writer` |
| Owned-media blog post | `blog-post-drafting-engine` |
| Podcast / speaking booking pitch | `media-podcast-pitch-crafter` |
| Cross-channel amplification | `content-repurposer-atomizer` |
| Post tied to a brand release | `press-release-social-blog-amplification-pack` |

Tell the user which skill to invoke next for the angle they pick. If the user is in a hurry, offer to call the downstream skill immediately.

---

## Output format

```
## Newsjacking Angles — [headline / story title]
Brand: [slug, via brand-brain]
Story type: [type]       Speed window: [Hot/Warm/Cool — ⚠ Fading if applicable]
Brand authority surface: [1 line]

### Top angles

| # | Angle type | Hook (one line) | Spokesperson frame | Fit score | Risks |
|---|---|---|---|---|---|
| 1 | … | … | … | ## | [flags] |
| 2 | … | … | … | ## | [flags] |
| 3 | … | … | … | ## | [flags] |

### Further options (ranked #4+)
[collapsed or brief]

### Recommended next step
Pick angle #_ → call [skill] with this hook: "[hook]"
```

Save to `./pr/newsjacking-[story-slug]-[YYYY-MM-DD].md` if the user wants to preserve the run for team review.

---

## Principles

- **Brand-brain first.** No angle before the brand is loaded. Voice + banned-words are hard overrides; real proof is the only proof.
- **Speed is the point.** Always show the speed window. If it is Fading, say so — don't let the user spend two hours on a dead story.
- **Authority over opportunism.** Only pitch angles where the brand has genuine, verifiable expertise. Manufactured relevance is worse than silence.
- **Risk flag, don't self-censor.** Show flagged angles with context and let the user decide. Your job is to inform, not to gatekeep.
- **Proof discipline.** Every factual claim in an angle is either sourced from `brand.md` (real proof) or tagged `[verify]`. No invented statistics, no fabricated differentiators.
- **Hand off, don't overreach.** This skill produces angles, frames, and hooks — not finished assets. Route to the right downstream skill.

---

## What not to do

- Do not produce angles before `brand-brain` returns the active brand.
- Do not invent proof, customer stories, or statistics to make an angle stronger.
- Do not pitch angles on tragedies, crises, or scandals involving human harm without an explicit, brand-appropriate reason and a strong mitigation note.
- Do not write the finished pitch, post, or article here — call the right skill.
- Do not ignore a Fading speed window; surface it and let the user decide whether to continue.
- Do not generate near-identical angles under different labels — if you can't find six structurally distinct takes, say the brand's authority surface doesn't intersect this story well enough.

---

## Quality checklist

- `brand-brain` called and active brand loaded before any angle was produced?
- Speed window classified and shown prominently?
- 6–8 angles generated across structurally distinct angle types (not synonyms)?
- Every angle scored on Relevance × Authority × Cycle Timing; top 3 surfaced?
- Every angle risk-flagged (reputational / factual / legal) and mitigation noted?
- Every unconfirmed claim tagged `[verify]`; no invented proof or statistics?
- Output format table complete with hook, spokesperson frame, fit score, and routing?
- Downstream skill identified for the user's next step?
