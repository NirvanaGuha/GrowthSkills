---
name: esp-map-platform-builder
description: >
  Translates a campaign brief or lifecycle strategy into a platform-specific automation spec —
  complete flow canvas, node-by-node logic, conditional branch conditions, timing rules, and
  paste-ready JSON or import recipe — for Klaviyo, ActiveCampaign, Braze, HubSpot, or Marketo.
  Absorbs platform syntax differences so the operator works at the strategy layer, not the
  click-by-click UI layer. Calls brand-brain for voice/ICP/offer context, then applies the
  Signal-Trigger-Branch-Message (STBM) framework to map every entry condition, wait step,
  branch condition, message slot, and exit rule before writing a single line of copy or JSON.
  Outputs a human-readable canvas spec (markdown table), optional platform-native JSON/recipe
  import block, and a QA checklist scoped to the target ESP/MAP. Calls subject-line-preview-text-
  optimizer, push-notification-copy-generator, cta-variant-generator, and proof-vault for the
  message slots, rather than redoing their work inline. Use when the user says "build this flow
  in Klaviyo," "set up this automation in HubSpot," "design this sequence in ActiveCampaign,"
  "write the Braze canvas," "map out the Marketo program," "build a triggered workflow for
  [platform]," "turn this lifecycle stage into an automation," or hands over a brief and names
  a platform.
---

# ESP / MAP Platform Builder

Turn a campaign brief into a platform-native automation spec. Give it the strategy and the target tool; get back a ready-to-build canvas with every node, branch, timing rule, and message slot defined — plus optional import-ready JSON or recipe copy. Every output is scoped to the platform's actual data model so a junior operator can build it without translating the logic.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, ICP, offer/pricing, proof, banned words before any copy or spec is produced.
- **`subject-line-preview-text-optimizer`** — called for every email node's subject + preview text slot.
- **`push-notification-copy-generator`** — called for push/SMS nodes when the channel is in scope.
- **`cta-variant-generator`** — called for email body CTA buttons and landing-page handoffs.
- **`proof-vault`** — called when a message node calls for social proof, testimonials, or case-study snippets.
- *(optional)* `lifecycle-journey-mapper` to pre-define the stage map before this skill builds the flow; `lead-nurture-drip-builder` when the output is purely nurture-sequence copy rather than a canvas spec.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (voice, ICP, offer, proof, banned words)
Step 1  Scope the build      ──► platform + flow type + entry signal + goal metric
Step 2  STBM mapping         ──► Signal → Trigger → Branches → Message slots → Exit
Step 3  Canvas spec          ──► markdown node table with timing, conditions, channel
Step 4  Message slots        ──► call sibling skills for copy at each node
Step 5  Platform export      ──► JSON / recipe block scoped to target ESP/MAP (if requested)
Step 6  QA checklist         ──► platform-specific pass before handing to the operator
Step 7  Persist artifact     ──► offer to save to ./sequences/<platform>/<slug>-canvas.md
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned digest: ICP + awareness tendency drives branch conditions and message tone; offer mechanics drive CTAs and incentive logic; voice adjectives and banned words override every message slot. Do not write copy or define a flow before `brand-brain` returns.

### Step 1 — Scope the build

Collect (or confirm) four inputs before proceeding:

| Input | Why it matters |
|---|---|
| **Platform** | Klaviyo / ActiveCampaign / Braze / HubSpot / Marketo — each has its own node types, data model, and JSON schema |
| **Flow type** | Trigger-based automation vs. scheduled broadcast vs. program/campaign vs. canvas journey |
| **Entry signal** | The specific event, property change, list join, or behavioral trigger that enrolls a contact |
| **Goal metric** | What does success look like? (activation event, purchase, upgrade, re-engagement) |

If any are missing, ask once — batched, not one at a time.

---

## The STBM Framework (Signal → Trigger → Branch → Message)

This is the core method. Apply it to every flow before writing nodes.

**Signal** — the real-world event that matters (a user completes onboarding step 1, a cart is abandoned, a lead hits a score threshold). Distinguish signal from platform trigger; they are not the same.

**Trigger** — the platform-level enrollment condition (event occurred, property equals, list added, webhook received). Each platform expresses this differently; map it correctly.

**Branch** — the conditional logic that personalizes behavior. Good branches are based on data the platform actually has at the time of evaluation: segment membership, property value, prior message engagement, time of day. Bad branches assume data that isn't resolved yet.

**Message** — the channel-specific touchpoint (email, push, SMS, in-app, ad audience sync). Every message slot gets: channel, subject/title, body intent (one sentence), CTA destination, timing offset, and a call to the relevant sibling skill for copy.

### STBM node table format

```
| # | Node type     | Wait / timing   | Branch condition (if any)         | Message intent                  | Sibling skill called |
|---|---------------|-----------------|-----------------------------------|---------------------------------|----------------------|
| 1 | Trigger       | immediate       | entry: [event / property]         | —                               | —                    |
| 2 | Wait          | +2h             | —                                 | —                               | —                    |
| 3 | Branch (Y/N)  | —               | has_purchased = true              | —                               | —                    |
| 4 | Email         | +0              | branch: N path                    | Remind + show top use case      | subject-line-optimizer, cta-variant-generator |
| 5 | Wait          | +3d             | —                                 | —                               | —                    |
| 6 | Branch (Y/N)  | —               | opened email #4                   | —                               | —                    |
| 7 | Email         | +0              | branch: N path (non-opener)       | Social proof + urgency nudge    | proof-vault, subject-line-optimizer |
| 8 | Exit          | —               | goal: purchase_event OR 14d lapse | —                               | —                    |
```

---

## Platform-specific rules

Apply these when generating the canvas spec and any JSON export. Never invent platform capabilities; mark uncertain syntax `[verify in platform UI]`.

### Klaviyo
- **Data model:** Profiles + Events + Lists/Segments. Flows enroll on event or list/segment membership change.
- **Key nodes:** Trigger, Time Delay, Conditional Split (profile property or event-history based), A/B Split (message-level only), Email, SMS, Update Profile Property, Webhook.
- **Branch limits:** Conditional splits are binary (Y/N); chain splits for multi-way logic.
- **JSON:** Klaviyo has no public flow import API [verify]. Deliver the STBM table + copy slots; note manual build steps.
- **Gotcha:** Metric-triggered flows only re-enroll a profile after the `flow_filter` window expires; confirm re-enrollment setting matches the intended frequency.

### ActiveCampaign
- **Data model:** Contacts + Custom Fields + Tags + Deals (CRM). Automations enroll on tag added, form submitted, date-based, or goal reached.
- **Key nodes:** Start (trigger), Wait, If/Else (contact field, tag, engagement), Send Email, Add/Remove Tag, Goal, Webhook, CRM Deal action.
- **Branch limits:** If/Else is binary; use Goal nodes to jump contacts across paths.
- **JSON:** Automations export as `.xml` (not JSON) [verify]; deliver the STBM table.
- **Gotcha:** Goals pull contacts from *earlier* in the automation — mis-placing a Goal exits contacts prematurely; place it at the intended conversion checkpoint only.

### Braze
- **Data model:** Users + Custom Events + Custom Attributes + Segments. Canvas is the primary multi-step journey tool.
- **Key nodes:** Entry (schedule/action/API/audience-change), Experiment Path (A/B), Action Path (waits for specific event), Delay, Message (email/push/SMS/in-app/content card/webhook), Audience Path (segment split), User Update.
- **Branch limits:** Experiment Path for true A/B; Audience Path for property-based splits; Action Path for event-triggered forks.
- **JSON:** Canvas can be created/updated via Braze REST API (`/canvas/create`); export the full Canvas JSON block with `name`, `schedule`, `entry_audience`, `components[]` array.
- **Gotcha:** Action Paths have a configurable evaluation window; if the window is too short the majority falls to the "Everyone Else" path — set the window to match realistic user behavior, not platform default.

### HubSpot (Workflows)
- **Data model:** Contacts/Companies/Deals + Properties + Lists. Workflows enroll on property change, form submission, list membership, or manual trigger.
- **Key nodes:** Enrollment trigger, Delay (date/time or relative), If/Else (contact/company/deal property), Send Email, SMS, Internal notification, Create/Update record, Set Property, Enroll in another workflow, Goal.
- **Branch limits:** If/Else is binary; chain for multi-way; use Goal to advance contacts out of long sequences.
- **JSON:** HubSpot has no public workflow import [verify]. Deliver the STBM table with property names matching the HubSpot schema the user provides.
- **Gotcha:** Enrollment re-entry is off by default; explicitly confirm whether a contact should re-enroll (e.g. repeat-purchase flows) and note the setting.

### Marketo (Engagement Programs / Smart Campaigns)
- **Data model:** Leads + Activities + Smart Lists. Engagement Programs stream content; Smart Campaigns handle triggers + filters + flow steps.
- **Key nodes (Smart Campaign):** Smart List (trigger + filter), Flow (Send Email, Change Data Value, Add to List, Wait, If/Else via Advanced Choices), Schedule (batch or triggered).
- **Key nodes (Engagement Program):** Streams, Cast (content send), Nurture track transitions.
- **JSON:** Marketo assets are managed via REST API (`/asset/v1/smartCampaigns`) [verify]; for most users deliver the Smart List filter spec + Flow step table.
- **Gotcha:** Marketo qualification rules (each campaign has a "once" / "every time" / "once per period" setting) control re-qualification — wrong setting silently blocks contacts from flowing.

---

## Timing and frequency rules

- State every delay as a relative offset from the *previous node's completion*, not from enrollment (ambiguity causes bugs).
- Respect send-time optimization where the platform offers it (Klaviyo Smart Send Time, Braze Intelligent Timing, HubSpot send-time optimization) — note when enabling it.
- Flag quiet-hours constraints if the brand's `brand.md` specifies them or if the flow includes push/SMS.
- For drips > 7 days, add a global unsubscribe / suppression exit branch.

---

## Message slot protocol

For each message node in the canvas:

1. State: channel, timing, one-sentence intent, CTA destination, and any personalization token needed.
2. Call the relevant sibling skill to fill the copy:
   - Email subject + preview text → `subject-line-preview-text-optimizer`
   - Push / SMS → `push-notification-copy-generator`
   - Email CTA button → `cta-variant-generator`
   - Proof block needed → `proof-vault`
3. Inline the returned copy (or [pending — call sibling skill]) if running in a single pass.
4. Mark any unconfirmed proof `[verify]`; never invent testimonials, stats, or customer names.

---

## Platform JSON / recipe export

When the user requests an importable artifact:

- **Braze:** Deliver the full Canvas JSON (`components[]` array with `name`, `messages{}`, `next_paths[]`, delay configuration).
- **Other platforms:** Deliver the STBM node table (human-readable) plus a structured spec block the operator pastes into the platform builder, since most lack public import APIs. Note this explicitly.
- Always wrap JSON in a fenced code block labeled with the platform. Mark any field where the exact property name must match the user's live schema: `[replace: YOUR_EVENT_NAME]`.

---

## Principles

- **Brand-brain first.** No node, branch, or message until `brand-brain` returns. Voice + banned words override all copy.
- **STBM before copy.** Map Signal → Trigger → Branch → Message in the canvas table before calling sibling skills. Skipping to copy produces structurally broken flows.
- **Platform-accurate, never invented.** Only use node types and API fields the platform actually supports. Uncertain syntax gets `[verify in platform UI]`.
- **Compose, don't duplicate.** Subject lines come from `subject-line-preview-text-optimizer`; push copy from `push-notification-copy-generator`; CTAs from `cta-variant-generator`; proof from `proof-vault`. Don't rewrite these inline.
- **Branches need resolved data.** Every If/Else or Conditional Split must reference a property or event the platform can evaluate at that exact moment in the flow. Never branch on data that isn't hydrated yet.
- **State delays explicitly.** "Send 2 hours after previous step completion" not "send 2 hours after enrollment" — ambiguity here causes silent bugs in production.
- **Honest proof only.** Real proof points from `proof-vault` or `brand.md`. Mark anything else `[verify]`.

---

## What not to do

- Don't produce copy or a flow spec before `brand-brain` returns the active brand.
- Don't use node types or API fields the target platform doesn't support — ask or mark `[verify]`.
- Don't invent proof, customer names, or open/click benchmarks as `[verify]`-unmarked facts.
- Don't skip the STBM table and jump straight to message copy — structural logic errors are invisible in prose.
- Don't build the flow without confirming the entry signal; a wrong trigger is unfixable after build.
- Don't assume re-enrollment behavior; call it out as a required setting for the operator to confirm.
- Don't store canvas artifacts inside the skill folder. Persist to `./sequences/<platform>/<slug>-canvas.md`.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any spec or copy?
- All four scope inputs confirmed: platform, flow type, entry signal, goal metric?
- STBM table complete — every node has a type, timing offset, branch condition (where applicable), and message intent?
- Platform-specific node types used correctly; no invented capabilities; uncertain syntax marked `[verify]`?
- Every email node has a subject/preview slot called out to `subject-line-preview-text-optimizer`?
- Every push/SMS node called out to `push-notification-copy-generator`?
- CTAs called out to `cta-variant-generator`; proof blocks called out to `proof-vault`?
- Re-enrollment setting explicitly noted for the operator to confirm?
- Delays stated as offsets from previous node, not from enrollment?
- Global unsubscribe exit branch present for flows > 7 days?
- JSON (if requested) wrapped in fenced code block with `[replace: ...]` markers for schema-specific fields?
- Artifact offered for save to `./sequences/<platform>/<slug>-canvas.md`?
