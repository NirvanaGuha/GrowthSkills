---
name: contractor-agency-brief-builder
description: >
  Turns a job description, campaign goal, and brand guidelines into a ready-to-send contractor
  or agency brief — scoped, deliverable-listed, timeline-bounded, budget-framed, and equipped
  with success criteria so the hired party can't claim they didn't know what "done" looks like.
  Works for any engagement type: freelance copywriter, SEO agency, design studio, paid-ads shop,
  video producer, dev contractor, or full-service retainer. Handles single-project and
  ongoing-retainer structures. The brief is built on SCOPE-PROOF-GATE, our working checklist: Scope
  (what work, what outcome), Proof (how you'll know it succeeded), Gate (what the vendor must
  confirm before starting). Calls brand-brain to load voice, banned words, and positioning so
  the vendor brief itself is on-brand and can be attached alongside a brand guide. Composes with
  campaign-brief-builder when the brief is for a paid or multi-channel campaign, and with
  icp-persona-builder when the vendor needs a persona document included. Saves output to
  ./vendors/<slug>/brief-<vendor-type>.md. Use when the user says "write a brief for a
  contractor," "agency brief," "creative brief for a freelancer," "vendor brief," "write an SOW,"
  "brief for a copywriter / designer / SEO agency / ads agency," or "what should I send to a
  new contractor."
---

# Contractor & Agency Brief Builder

A brief that's vague at the top becomes a scope dispute at the invoice. This skill produces the brief that prevents that: one document a contractor or agency can act on without a follow-up call, that you can enforce without an awkward "that's not what I asked for."

Built on **SCOPE-PROOF-GATE** (our working model, not an external framework) — the three failures that kill vendor relationships before the first delivery.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads voice, banned words, positioning, and ICP so the brief is brand-compliant and can double as a vendor onboarding artifact.
- **`campaign-brief-builder`** (compose when needed) — when the engagement is paid/multi-channel, pull the structured campaign brief and embed or attach it rather than re-deriving objectives and KPIs here.
- **`icp-persona-builder`** (compose when needed) — when the vendor needs audience context (copywriter, agency), pull the persona document instead of writing a thin "our audience is…" paragraph.
- **`offer-pricing-brain`** (compose when needed) — when the brief covers a launch or promotional campaign and the vendor needs offer mechanics.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (voice, banned words, positioning)
Step 1  Gather inputs           ──► job type, deliverables, goal, timeline, budget, success metric
Step 2  Apply SCOPE-PROOF-GATE  ──► build each section; gate-check completeness before saving
Step 3  Compose from siblings   ──► pull campaign-brief-builder / icp-persona-builder if needed
Step 4  Self-review + save      ──► checklist, then write to ./vendors/<slug>/brief-<type>.md
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, positioning, ICP, and real proof. The brief itself must be written in the brand's voice — vendors who receive it also receive a signal about how the brand communicates. Obey banned words in every deliverable description and example you write. Mark any unconfirmed proof `[verify]`.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If neither exists, ask: brand name, ICP in one sentence, 3 voice adjectives, and any hard banned words. Prefer the call.

### Step 1 — Gather inputs

Ask only for what's missing. Common inputs:

| Input | Why it matters |
|---|---|
| Engagement type | Freelancer, agency, retainer, project-based |
| Vendor function | Copywriter, SEO, paid ads, design, video, dev, full-service |
| Business goal | The outcome the work must serve (not the tasks) |
| Deliverables | Named, versioned, format-specified |
| Timeline | Hard deadlines, milestone checkpoints, review rounds |
| Budget | Fixed fee, hourly cap, or retainer ceiling |
| Success metric | How you will judge the work done — not vibes |
| Brand materials available | What you'll hand over (brand.md, style guide, personas) |

Batch into one question block. Don't ask about voice or positioning — brand-brain handled that.

---

## The SCOPE-PROOF-GATE framework

### SCOPE — What the vendor is actually doing

The most common brief failure: deliverables described by effort ("write blog posts") instead of output ("4 × 1,200-word blog posts, SEO-optimized, with a target keyword, internal links, and AIOSEO fields completed, published to WordPress in draft status by [date]").

Each deliverable entry must answer:
- **What** — named, quantified, format-specified
- **For whom** — audience context (link to persona doc if available)
- **Where** — destination channel, platform, CMS, ad account
- **Version rounds** — how many revision rounds are included
- **Hand-off format** — Google Doc, Figma file, MP4, GitHub PR, etc.

Scope also includes explicit **out-of-scope** items. If the copywriter is not responsible for publishing, say so. If the SEO agency does not own paid spend, say so. Disputes almost always start at the boundary.

### PROOF — How you'll know it worked

Every brief needs a success-criterion section the vendor cannot argue with later. Write this in two layers:

**Delivery quality criteria** (can be checked at submission):
- Word count, keyword density, format compliance, brand-voice adherence
- Creative spec compliance (dimensions, file type, safe-zone padding)
- UTM structure, tracking setup, QA checklist passed

**Performance criteria** (retrospective, sets expectations for ongoing work):
- Primary KPI + target (e.g., organic sessions +20% within 90 days `[verify if this is agreed]`)
- Secondary signal (e.g., engagement rate, CTR floor)
- Attribution window and measurement source

Mark performance targets `[verify]` if they are aspirational rather than contractually bound. Separate what is a pass/fail gate from what is a stretch goal.

### GATE — What the vendor must confirm before starting

The gate section prevents misaligned kickoffs. Before any billable work begins, the vendor confirms:

1. They have received and read: brand brief, persona docs, style guide, existing assets
2. They confirm access: CMS logins, ad account, Drive folder, Slack channel
3. They agree on: the revision-round definition (what counts as a "round"), the approval chain (who signs off), and the escalation path (what triggers a scope-change conversation)
4. They flag any ambiguities within [N] business days of receiving the brief

The gate is a two-sentence confirmation email template you embed in the brief, making it frictionless for the vendor to complete it.

---

## Brief structure (output template)

```markdown
# [Vendor type] Brief — [Project/Campaign name]
**Brand:** [slug]  |  **Date issued:** [date]  |  **Brief owner:** [name]

## 1. Engagement overview
[2–3 sentences: what the business is trying to achieve and why this vendor engagement serves it]

## 2. Scope of work
### Deliverables
[Table or list: name | quantity | format | destination | revision rounds | due date]

### Out of scope
[Explicit list]

## 3. Audience context
[1 paragraph or link to persona doc — pulled from icp-persona-builder if available]

## 4. Brand context
[Voice adjectives, 2–3 positioning sentences, banned words, tone guidance — from brand-brain]
[Attach: brand.md / style guide]

## 5. Campaign / project context
[Link to or embed from campaign-brief-builder if applicable]

## 6. Success criteria
### Delivery quality (pass/fail at submission)
[Checklist]

### Performance targets (retrospective)
[KPI | target | measurement source | window]

## 7. Timeline & milestones
[Milestone | deliverable | owner | due date]

## 8. Budget
[Total or per-deliverable fee | rate structure | invoicing schedule | late-delivery clause if applicable]

## 9. Assets provided
[List every file, login, folder, or document you're handing over]

## 10. Gate confirmation (vendor to complete before starting)
[Confirmation email template — fill and return to [owner] within [N] business days]
```

---

## Engagement-type adaptations

| Type | Key adjustments |
|---|---|
| Freelance copywriter | Emphasize voice guide + persona doc; specify SEO fields if WordPress; note who does publishing |
| SEO agency | Include GSC/GA4 access, current baseline metrics `[verify]`, keyword list or brief source |
| Paid-ads agency | Embed or link campaign-brief-builder output; specify ad account access, creative approval flow, bid strategy scope |
| Design / creative studio | Add brand color/font spec, file format requirements, usage rights clause |
| Video producer | Script ownership, b-roll direction, caption format, thumbnail deliverable, publishing access |
| Dev contractor | Acceptance criteria per ticket, code review process, deployment ownership, rollback responsibility |
| Full-service retainer | Add monthly reporting spec, communication cadence, retainer renewal clause |

---

## Principles

- **Specificity prevents disputes.** Every deliverable gets a number, a format, and a due date. "Some social posts" is not a deliverable.
- **The brief is not the contract, but it feeds it.** Write it precisely enough that it could become a contract exhibit.
- **Brand-brain first, always.** The brief itself is a brand artifact — vendors form their first impression of the brand by reading it.
- **Compose, don't rebuild.** Campaign goals come from campaign-brief-builder; persona context comes from icp-persona-builder. Don't write thin proxies when the real output exists.
- **Gate early or negotiate late.** A two-sentence confirmation email up front saves a three-email scope dispute at delivery.
- **Mark unconfirmed numbers.** Performance targets that are aspirational get `[verify]` — the vendor should not be held to a number you invented.

---

## What not to do

- Don't start writing the brief before brand-brain returns — voice and banned-word violations in a brief sent to a vendor are embarrassing and hard to walk back.
- Don't describe deliverables by effort ("write emails") — describe them by output ("3 × onboarding emails, 200–300 words each, plain HTML, drafted in HubSpot, reviewed within 5 business days").
- Don't omit the out-of-scope section — it exists to protect you, not the vendor.
- Don't invent performance targets — use real baselines `[verify]` or mark them as stretch goals, not contractual gates.
- Don't skip the gate section because it feels bureaucratic — it is the single highest-leverage page in the document.
- Don't rebuild ICP or campaign structure inline — call the sibling skills.

---

## Quality checklist (self-review before delivering)

- [ ] `brand-brain` called and voice/banned-words applied throughout the brief?
- [ ] Every deliverable has: name, quantity, format, destination, revision rounds, due date?
- [ ] Out-of-scope section present and specific?
- [ ] Audience context sourced from `icp-persona-builder` or explicitly written — not omitted?
- [ ] Campaign objectives sourced from `campaign-brief-builder` if applicable?
- [ ] Success criteria split into delivery-quality (pass/fail) and performance targets (retrospective)?
- [ ] Performance targets marked `[verify]` if aspirational?
- [ ] Gate confirmation template included with a named owner and deadline?
- [ ] Budget section specifies rate structure and invoicing schedule?
- [ ] Assets-provided list is complete so the vendor cannot claim missing access?
- [ ] Saved to `./vendors/<slug>/brief-<vendor-type>.md`?
