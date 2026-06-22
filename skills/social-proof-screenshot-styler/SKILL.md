---
name: social-proof-screenshot-styler
description: >
  Transforms raw proof — a plain screenshot, raw review text, a tweet/DM grab, an approved
  testimonial — into a social-ready branded card. Produces a render-ready annotation spec
  (border style, brand color fills, watermark placement, blur zones, overlay copy) PLUS an
  optional 30–60s spoken-script for a talking-head or voiceover video that amplifies the same
  proof point. Calls brand-brain for colors, font stack, banned words, and voice; calls
  proof-vault for the canonical proof library so you never surface an unverified or
  superseded number; calls screenshot-annotator-bug-reporter for the technical annotation
  layer. Uses a Proof-Strength Rubric (this skill's own five-rung scale, from authority/named
  expert down to anonymous) to grade each asset and style it to its honest strength, grounded
  in Cialdini's actual finding that social proof bites hardest under uncertainty and similarity
  ("people like me"). Outputs a card spec the user can execute in
  Canva, Figma, or any image editor, plus (on request) a spoken script. Use when the user
  says "style this screenshot," "make this review shareable," "brand this testimonial,"
  "turn this DM into a social card," "social proof graphic," "proof card," "screenshot card,"
  "testimonial image," "quote graphic," "review screenshot," or hands over raw proof and asks
  how to make it look good and on-brand.
---

# Social Proof & Screenshot Styler

Raw proof rots on a clipboard. This skill turns it into a branded, trust-signaling asset — a render-ready card spec with every styling decision made and a video spoken-script when you need to go further. It does not redesign your brand; it reads the brand and applies it. It does not invent proof; it checks the proof vault and flags anything unverified.

Two things govern every output: a **Proof-Strength Rubric** (defined below — this skill's own scale, not borrowed) that grades how much trust a piece of proof can honestly carry, and a small set of **conversion heuristics** that decide what actually makes a card persuade rather than just look tidy. The rubric decides *how strong* the proof is; the heuristics decide *how to render it so a buyer believes it*.

A note on attribution: this skill does **not** claim a Cialdini "social proof ladder." Cialdini never published a five-tier social-proof hierarchy. What he established is the *mechanism* — social proof is strongest under **uncertainty** (the buyer is unsure what to do) and **similarity** ("people like me are doing this"). That finding shapes the heuristics here. Authority and Liking are **separate Cialdini principles**, not rungs of social proof; this skill treats a named expert or a recognizable face as strong proof because of *who* they are, not because they sit higher on a social-proof scale. The five-rung rubric below is ours, owned and labeled as such.

---

## Skills this calls

- **`brand-brain`** (required) — loads brand colors, font stack, voice, banned words, and watermark/logo assets. This skill does not implement brand resolution itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand colors (hex), font name, logo path, and any banned words before proceeding.
- **`proof-vault`** (required when available) — retrieves the canonical proof library and flags any numbers the user supplied that conflict with or are superseded by the vault. Never style a card with an unverified or stale stat.
  Fallback if proof-vault is absent: use only proof the user explicitly supplies and mark any numeric claims `[verify]`.
- **`screenshot-annotator-bug-reporter`** (optional, when installed) — handles the pixel-level annotation layer (blur zones, redaction boxes, callout arrows). Synthesize annotation instructions inline when absent.
- **`canva-figma-workflow-accelerator`** (optional, for downstream execution) — if the user wants a Canva Bulk Create CSV or a Figma component spec for batch production, route there after this skill produces the card spec.
- **`brand-consistency-auditor`** (optional, for review pass) — invoke after producing a batch of card specs to flag any off-brand color or tone violations before the assets go live.

---

## How a run works

```
Step 0  Load brand + proof  ──► call brand-brain, then proof-vault
Step 1  Grade the proof      ──► score it on the Proof-Strength Rubric (R1–R5)
Step 2  Apply the heuristics  ──► number-vs-quote hierarchy, real-frame vs retype, raw-vs-polished
Step 3  Choose the card mode  ──► Quick card (default) | Batch spec | + Video script
Step 4  Build the card spec   ──► styling decisions, annotation layer, copy
Step 5  Optional video script ──► 30–60s spoken framing
Step 6  Self-review + present
```

### Step 0 — Load brand and proof (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). Use the returned digest for brand colors, font stack, voice adjectives, banned words, and any available logo/watermark path.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand colors (hex), font name, logo path, and any banned words before proceeding.

Then invoke `proof-vault` if installed. Check every numeric claim in the user's proof against the vault. If a number conflicts or is superseded, flag it and use the vault version. If the vault is absent, accept user-supplied proof and mark any unverified numbers `[verify]`.

Do not produce any card spec until both calls return (or fallbacks are applied).

---

## The Proof-Strength Rubric (this skill's grading scale)

This is **our** rubric — a five-rung scale for how much trust a piece of proof can honestly carry. It is not a Cialdini taxonomy and is not presented as one. Rung order is a proxy for credibility-per-pixel: how much belief the asset buys before the viewer's skepticism kicks in. Grade every incoming item at its **honest** rung and style it to that strength. Never inflate upward.

Two of these rungs lean on Cialdini principles that are *distinct from* social proof and are flagged as such:

| Rung | What it is | Trust source | Styling treatment |
|---|---|---|---|
| **R1 Authority / known face** | Named expert, recognized brand logo, press masthead, verified-badge endorsement | **Cialdini's Authority + Liking principles** — borrowed credibility from *who said it*, not from crowd behavior | Full-bleed logo or headshot, high-contrast brand border, masthead-style attribution; let the name/logo be the loudest element |
| **R2 Third-party certification** | G2/Capterra badge, award seal, compliance cert, platform verification mark | Vetted by a neutral institution the buyer already trusts | Badge prominent and uncropped, muted background so the seal reads, one line of trust copy beneath |
| **R3 Volume / the crowd** | Review count, "X customers," aggregate star rating | **True social proof under uncertainty** — "lots of people like me chose this" | Lead with the number set huge (see heuristic 1), source attribution small beneath; secondary brand fill |
| **R4 Named peer** | First name + job title + company + outcome, an identifiable customer DM | **Social proof via similarity** — strongest when the named peer visibly resembles the target buyer | Real screenshot frame preferred (heuristic 2), avatar/initials, company logo if cleared, outcome highlighted |
| **R5 Anonymous** | Screenshot with no name, paraphrased feedback, an un-clearable DM | Weakest — no name, no institution, no crowd | Honest minimal styling; add `[name on file]` or `[shared with permission]`; never fabricate identity |

**Match the rung to the audience, not just the asset.** A R4 named-peer card only fires on the similarity principle if the named peer *looks like* the viewer — a VP-of-Eng testimonial converts engineers, not procurement. When the brand-brain ICP and the proof's author don't match, say so and prefer a different item.

**Inflate nothing.** A R5 anonymous DM is not an R1 endorsement. Do not strip attribution, imply institutional backing, or imply a named identity the source doesn't contain.

---

## Conversion heuristics (what makes a card *convert*, not just look clean)

A proof card and a generic quote tile use the same fonts and the same canvas. The difference is judgment. Apply these on every card; they override "make it pretty."

**1. Number beats quote — so set the number, not the sentence, as the hero.**
If the proof contains a hard outcome ("cut churn 34%", "4.9★ from 5,000 reviews", "ROI in 11 days"), the *number* is the persuasion engine and must dominate the visual hierarchy: oversized (60–120px on a 1080 canvas), brand-accent color, top third of the card. The supporting sentence shrinks to caption weight beneath it. The common failure is centering a 40-word quote at uniform size with the number buried mid-sentence — the eye finds nothing and the card reads as decoration. Rule of thumb: if a viewer can't extract the single most persuasive fact in under one second at thumbnail size, the hierarchy is wrong.
*Exception:* when the proof's power is the **emotional language** ("I almost cancelled, then your team saved the account in an hour"), the phrase is the hero — pull-quote it large, drop the metric to a footnote.

**2. A real screenshot frame out-converts a retyped quote — keep the receipts.**
A retyped quote in your brand font is indistinguishable from copy you wrote yourself; it carries zero verification weight. The actual screenshot — the real G2 review chrome, the iMessage bubble, the tweet with its handle and timestamp — is the *evidence*. Keep the native frame and brand *around* it (border, header bar, watermark, callout arrow) rather than rebuilding the content inside it. Retype only when you legally must anonymize, the source resolution is unusable, or platform ToS forbids reposting the chrome — and when you do, add a `[verbatim — original on file]` note so the claim is auditable. Default to the receipt; reach for the retype reluctantly.

**3. Raw beats polished when authenticity is the scarce resource.**
Over-design reads as advertising, and advertising is discounted. A slightly ugly real artifact — the unstyled DM, the screenshot with the status bar still in it, the review with a minor typo left intact — often out-converts the glossy card because it *looks unbought*. Lean raw for: organic social, founder-voice posts, "early customer" or beta proof, and any audience primed to distrust marketing. Lean polished for: paid placements, sales decks, the website's proof wall, enterprise buyers who expect production value. When in doubt on organic, keep one authenticity tell (real timestamp, real avatar, native frame) rather than scrubbing the asset clean.

**4. One card, one claim.**
A card that argues two things proves neither. If the proof bundles multiple outcomes, pick the single most ICP-relevant one for the hero and let the rest live in the body or split into a second card. Crowded proof cards test worse than focused ones for the same reason cluttered landing pages do.

---

## Worked example (before → after)

Same raw proof, graded and styled two ways. This is the judgment the rubric + heuristics are meant to produce.

**Raw input (pasted by user):**
> "honestly we were about to churn off [Brand] but the new automation cut our support backlog like 30-something % in the first month and now my team isn't drowning. life saver" — DM from Priya, Head of CX at a 40-person SaaS

**Generic-tile version (what a templating skill produces — the 3/5 output):**
A 1080×1080 navy square. The full 38-word quote centered in 28px white text. "Priya, Head of CX" in 16px beneath. Brand logo bottom-right. Looks clean. Converts poorly: no visual entry point, the number ("30-something %") is trapped mid-sentence and reads as vague, and a retyped quote in brand font carries no proof that Priya is real.

**Rubric + heuristic version (what this skill should produce):**

```
Rubric grade: R4 Named peer — first name + title + company-size; fires on similarity
              for the SaaS-CX-leader ICP. Strong, because the named peer = the buyer.
Heuristics applied: #1 (number is hero, but emotional phrase is co-lead — see exception),
                    #2 (keep the real DM frame), #3 (lean raw — this is organic + founder voice)

Canvas: 1080×1350 (vertical, feed-optimized)
Hero zone (top 40%): pull the metric clean — "30%+ support backlog gone — in month one"
   set 88px brand-accent. The vague "30-something" is sharpened to "30%+" (honest floor,
   not inflation) and flagged for the user to confirm against proof-vault before publish.
Evidence zone (mid 45%): the ACTUAL DM screenshot, lightly cropped to Priya's bubble,
   native iMessage chrome and timestamp INTACT (heuristic 2 + 3). Thin brand border around
   the screenshot, not redrawn inside it. Blur the contact-photo + phone-number PII only.
Attribution: "Priya · Head of CX · 40-person SaaS" — 20px, under the screenshot.
   Confirm name-use permission; if not cleared, render "Head of CX, 40-person SaaS team
   [name on file]" and crop the avatar.
Authenticity tell kept: the "life saver" sign-off stays — it's the emotional co-hook and
   it reads unbought. Do NOT scrub the lowercase/typos.
Watermark: logo bottom-right, 96px, muted so it never competes with the screenshot.
Overlay trust copy: none needed — the real frame IS the trust signal.
Alt text: "Customer DM from Priya, Head of CX: new automation cut their support backlog
   over 30% in the first month."
Why this converts: a real receipt from someone the viewer recognizes as themselves, with
   the one number set big enough to read at thumbnail. Belief before skepticism.
```

The two cards cost the same to make. The second one is graded, sharpened, and keeps its receipts — that is the whole job.

---

## Card spec format (Quick mode, default)

One proof item → one render-ready card spec. Inline output unless the user asks to save.

```
## Card spec — [brief proof description]

Brand: [slug, from brand-brain]
Proof rung: R[n] — [rung name] · fires on [authority / certification / volume / similarity]
Heuristics applied: [which of #1–#4, in one line — forces the judgment call to be explicit]
Canvas: [recommended dimensions, e.g. 1080×1080 / 1080×1350 / 1200×628]

### Background
[color hex + fill style: solid / gradient / branded texture suggestion]

### Border / frame
[px weight · color hex · radius · shadow spec — or "none"]

### Proof content zone
- Quote / stat text: "[exact text]" — [font: weight size, brand font stack from brand-brain]
- Max characters to display: [n] — truncation rule if source is longer
- Attribution line: "[Name, Title, Company]" — [font: weight size]
- Source badge / logo: [placement · max px · clear space]

### Watermark / brand lock-up
- Logo: [placement — e.g. bottom-right 24px margin · max 120px wide]
- Color: [hex — should not compete with proof content]

### Annotation layer (blur / redaction / callout)
[If screenshot: blur zones (PII, unrelated UI regions) · callout arrows if directing attention]
[If text-only proof: "no annotation needed"]

### Overlay copy (optional trust-builder)
[e.g. "Verified G2 review" · "4.9 ★ on Capterra" · "5,000+ teams" — use only vault-confirmed numbers]

### Accessibility
- Alt text: "[draft alt text for the final image]"
- Contrast check: confirm overlay text meets WCAG AA against the background hex above

### Execution path
[Canva: step-by-step from blank · Figma: component suggestion · or "export spec only"]
```

---

## Batch mode (on request)

Triggered by "multiple," "batch," "bulk," or a list of ≥3 proof items.

1. Grade every item on the Proof-Strength Rubric first; group by rung. Lead the set with the highest-rung item — the strongest proof earns the first slot.
2. Produce a **shared base spec** (background, border, watermark, font stack) common to all cards.
3. Produce a **per-card delta table** — only the fields that differ per item (rung, hero number-or-quote per heuristic 1, attribution, rung-specific badge, canvas size).
4. Flag any item where proof cannot be verified — do not omit it, but annotate it.
5. If `canva-figma-workflow-accelerator` is available, offer to produce a Bulk Create CSV after the specs are confirmed.

Save batch output to `./social-proof/[brand-slug]-card-specs.md` when asked.

---

## Video spoken-script (on request)

Triggered by "video," "script," "voiceover," "talking head," or "reel." A proof video is not a read-aloud of the card — it is a 30–60s story that earns the proof before showing it. The same rubric + heuristics drive it: the on-screen receipt is the evidence; the script is the framing that makes a stranger care.

**Open on tension, not the testimonial.** The single biggest failure of proof videos is opening with "Here's a great review we got." Nobody cares about your review. They care about the problem the review proves you solve. Lead with the viewer's *current* pain, then reveal the proof as the resolution.

```
[0–3s]   Pattern interrupt — state the viewer's pain as a fact, not a question.
         "Your support team is drowning and you're three weeks from churning a tool."
         (Pull this pain from the brand-brain ICP, not from the testimonial.)
[3–10s]  The turn — "Here's what happened when [peer like you] hit the same wall."
         Name the peer's similarity to the viewer (role, company size) — this is the
         Cialdini similarity lever doing the work, said out loud.
[10–25s] The proof, shown not just said — read the ONE hero number from heuristic 1,
         and put the REAL screenshot on screen as you say it (heuristic 2). The spoken
         line and the on-screen receipt must reinforce, not duplicate, each other.
[25–45s] Mechanism — one sentence on WHY it worked, so the result feels repeatable
         rather than lucky. Without this, the proof reads as a fluke.
[45–58s] Transfer — "If you're the [ICP role] who [pain], this is the part that matters
         for you." Make the viewer the next protagonist.
[58–60s] CTA — one action, brand-voice compliant, no banned words.
```

**Rung changes the framing, not just the visuals:**
- **R1 authority** → lead with the name/credential ("The [Title] at [known brand] said this on the record"); authority is the hook itself.
- **R3 volume** → lead with scale and the uncertainty it resolves ("4,000 teams already made this call, so you don't have to guess").
- **R4 named peer** → lead hardest on similarity; the more the on-screen peer mirrors the viewer, the shorter the hook needs to be.
- **R5 anonymous** → do not build a video around it; anonymous proof can't carry a talking-head. Use it as a B-roll receipt inside a script anchored on stronger proof, or decline.

Deliver as labeled sections with timestamps, plus a parallel **on-screen / B-roll column** noting when the real screenshot appears (so the spoken word and the receipt stay in sync). Write to be heard, not read: short sentences, one idea per breath. Use only vault-confirmed numbers; mark others `[verify]`. If the proof is R5 anonymous, say so and recommend a stronger item before scripting.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No card spec before brand-brain returns. Its colors, fonts, and banned words are hard overrides.
- **Proof vault gates the numbers.** Any numeric claim not in the vault or not user-supplied gets `[verify]` — never invent a stat to make a card look more compelling.
- **Rung honesty.** Grade proof at its actual rung. Never style an R5 anonymous piece as R1 by removing attribution, implying institutional backing, or inventing a name, company, or photo.
- **Attribution is not optional.** Every card carries a source attribution line. Anonymous proof carries an explicit permission note.
- **Receipts over re-types.** Default to the real screenshot frame; only retype when legally or technically forced, and mark it `[verbatim — original on file]`.
- **Accessibility is output, not afterthought.** Every spec includes a draft alt-text string and a contrast check note.
- **Spec is the deliverable.** This skill produces render-ready instructions, not the rendered image. The spec must be complete enough that a non-designer can execute it.

## Traps (failure modes, not restatements of the rules above)

These are the specific ways a competent operator still ships a weak card. They are distinct from the Principles — read them as "even after you've followed the rules, watch for these."

- **The pretty-tile trap.** A card that's clean, on-brand, and centered but has no number-vs-quote hierarchy (heuristic 1) is decoration, not proof. Tidy is not the same as persuasive.
- **The over-scrubbed receipt.** Rebuilding a real DM/review in brand font to "clean it up" destroys the very thing that made it credible. Polish around the receipt, not over it.
- **The mismatched peer.** Strong R4 proof aimed at the wrong audience converts no one — a developer testimonial on a card for CFOs wastes the similarity lever. Match author to ICP or pick another item.
- **The buried lede video.** Opening a proof video with "we got a great review" instead of the viewer's pain. Tension first, testimonial second.
- **The kitchen-sink card.** Cramming two or three outcomes onto one card so it proves none of them (heuristic 4). Split it.
- **The borrowed-framework tell.** Don't reintroduce a "Cialdini ladder" or any invented hierarchy in the output — the rubric is ours and labeled as ours; Authority and Liking are separate principles, not social-proof rungs.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; colors, font, banned words applied?
- `proof-vault` checked; all numeric claims vault-confirmed or marked `[verify]`?
- Proof graded at its honest rung (R1–R5) — no inflation — and matched to the ICP audience?
- Heuristics applied and stated on the spec: is the number (or the emotional phrase) the visual hero, not buried? Real screenshot frame kept over a retype? Raw-vs-polished called for the placement? One claim per card?
- Card spec complete: background, border, content zone, attribution, watermark, annotation layer, alt text, contrast note, execution path?
- Attribution line present on every card; anonymous proof carries a `[shared with permission]` note?
- Batch mode: highest-rung item leads; shared base spec + per-card delta table produced?
- Video script (if requested): opens on viewer pain (not the testimonial); similarity/authority lever named; hero number read while the real receipt is on screen; mechanism beat present; CTA on-voice; no unverified stats?
- No banned words; no invented proof; no fabricated identity; no "Cialdini ladder" or borrowed-hierarchy language anywhere in the output?
