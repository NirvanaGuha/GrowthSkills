# Growth Skill Library — Registry & Call Graph

The map of the built library and how it interconnects. **Tier-1 is complete: 115/115 skills built.** Total library = **120 skill folders** (Tier-1 + the Layer-0 brand-brain system extras). Every skill calls `brand-brain` for context and composes built siblings; graphify can ingest this for a queryable dependency graph.

Last updated: 2026-06-21  ·  **120 skills · Tier-1 complete**

**Library home:** `~/GrowthSkills/` — canonical source in `~/GrowthSkills/skills/<slug>/`, each symlinked into `~/.claude/skills/` for global discovery. **GitHub:** https://github.com/NirvanaGuha/GrowthSkills (public, single repo, no collaborators).

---

## Architecture rules (every skill follows these)

1. **One brand brain, many readers.** No skill re-derives brand context. Every copy/marketing skill calls **`brand-brain`** first to load the active brand.
2. **Data outside skill folders.** Brand data → `~/.brandbrain/brands/<slug>/` (per-project `./.brandbrain/` overrides). Survives `amskills` updates (rm-rf safe). Skill artifacts → project-relative `./<folder>/` (the user's CWD, never the skill folder).
3. **Skills call skills.** Compose existing skills over re-implementing; declare them in each SKILL.md's "## Skills this calls".
4. **Graceful degradation.** If a called skill is absent, fall back inline — never hard-fail.
5. **No recursion.** When `brand-brain` (or an orchestrator) calls a component it passes context; the component must not call back up.

---

## Layer 0 — the brand-brain system (system of record)

- **`brand-brain`** — resolves the active brand, serves the `brand.md` digest, bootstraps (scan→delegate→interview→write), refreshes, audits, multi-brand. Called by every skill; delegates to the 8 components below.
- **8 components** (each owns one part of `brand.md` or a companion file, callable standalone or by brand-brain): `brand-voice-codifier`, `icp-persona-builder`, `positioning-messaging-architect`, `competitive-intelligence-dossier` (→`competitors.md`), `offer-pricing-brain`, `proof-vault` (→`proof.md`), `objection-library-builder` (→`objections.md`), `editorial-style-guide` (→`style-guide.md`).

---

## Tier-1 catalog — 115 skills by layer

- **L1 Foundation / Brand Brain** (6): brand-voice-codifier, icp-persona-builder, positioning-messaging-architect, battlecard-objection-handler, win-loss-interview-synthesizer, objection-library-builder
- **L2 Strategy & Planning** (3): gtm-launch-planner, experiment-pipeline-backlog-prioritizer, okr-suite
- **L3 Audience Research & Insight** (1): voice-of-customer-mining-pipeline
- **L4 SEO & Organic Search** (8): on-page-seo-optimizer, serp-analysis-report, topic-cluster-pillar-architect, content-brief-builder, aeo-geo-llm-visibility-optimizer, content-refresh-briefer, keyword-research-clustering-suite, seo-content-health-decay-audit
- **L5 Content Creation & Editorial** (8): headline-hook-generator, landing-product-page-copy-writer, content-repurposer-atomizer, de-slop-humanize-pass, blog-post-drafting-engine, editorial-calendar-builder, content-qa-reviewer, cta-variant-generator
- **L6 Social Media & Community** (5): linkedin-post-writer, x-thread-writer, short-form-video-script-writer, social-content-calendar-builder, faceless-short-form-video-producer
- **L7 Email, Push & Lifecycle / Retention** (8): subject-line-preview-text-optimizer, welcome-onboarding-email-sequence-builder, push-notification-copy-generator, lifecycle-journey-mapper, abandon-flow-writer, lead-nurture-drip-builder, full-email-push-asset-builder-copy-responsive-html, esp-map-platform-builder
- **L8 Paid Acquisition & Performance** (4): campaign-brief-builder, ad-copy-variant-generator, weekly-paid-performance-summary, bid-budget-pacing-checker
- **L9 CRO & Experimentation** (4): a-b-multivariate-test-designer, experiment-results-analyzer, landing-page-heuristic-live-cro-auditor, funnel-drop-off-analyzer
- **L10 Analytics, Measurement & Reporting** (6): channel-roi-scorecard, ltv-cac-payback-calculator, data-qa-measurement-gotcha-checker, ga4-weekly-traffic-digest, growth-diagnostic-deep-dive, sql-query-generator-for-ga4-bigquery
- **L11 Product-Led Growth & Activation** (3): usage-triggered-message-sequencer, aha-moment-activation-metric-definer, in-app-microcopy-writer-auditor
- **L12 ABM & Sales Enablement** (5): meeting-prep-follow-up-pack, account-dossier-builder, cold-outreach-sequence-architect, account-list-builder-icp-scorer, sales-asset-reviewer
- **L13 Creative & Design Ops** (3): deck-presentation-writer, ai-image-generator-on-brand-assets, creative-feedback-translator
- **L14 Marketing Ops, Governance & Compliance** (3): utm-parameter-bulk-builder, campaign-qa-launch-checklist-generator, automation-workflow-designer-debugger
- **L15 Decision-Making & Mental Models** (4): prioritization-framework-suite, pre-mortem-post-mortem-generator, analytical-reasoning-toolkit, eisenhower-matrix-weekly-priorities-sorter
- **L16 Brainstorming, Ideation & Creativity** (3): creative-ideation-framework-suite, campaign-concept-developer, how-might-we-10x-reframer
- **L17 Personal Productivity & Time Management** (4): meeting-agenda-action-item-builder, doc-note-summarizer, daily-weekly-planning-sprint, async-standup-writer
- **L18 Organization & Knowledge Management (PKM)** (1): sop-builder-reviewer
- **L19 Internal Communication & Collaboration** (2): stakeholder-update-status-writer, quick-tone-softener-diplomacy-pass
- **L20 PR & Corporate Communications** (4): media-podcast-pitch-crafter, newsjacking-angle-finder, press-release-social-blog-amplification-pack, quote-polisher
- **L21 Growth Engineering (technical)** (1): landing-page-builder-html-tailwind
- **L22 Visual Documentation & Screen Capture** (1): product-demo-gif-mockup-builder
- **L24 Budget, Finance & Business-Case** (1): vendor-cost-comparison-renewal-brief
- **L25 Vendor, Agency & Team Management** (2): agency-deliverable-qa-pulse-reviewer, contractor-agency-brief-builder
- **L26 Events, Webinars & Field Marketing** (5): webinar-event-campaign-planner, webinar-email-sequence-writer, event-registration-on-demand-page-copywriter, event-social-promotion-pack, event-recap-blog-post-writer
- **L27 Customer Marketing, Advocacy & Referral** (5): case-study-customer-spotlight-production-suite, churn-save-sequence-writer, upsell-cross-sell-campaign-builder, referral-program-brief-builder, review-response-drafter
- **L28 Partnerships, Influencer & Affiliate** (1): partnership-outreach-personalization-writer
- **L30 AI/Automation Meta-Skills** (3): prompt-engineering-library-suite, multi-source-deep-research-report-builder, custom-ai-agent-builder
- **L31 eCommerce Platform Marketing** (1): shopify-product-collection-page-seo-builder
- **L32 Pricing, Packaging & Monetization** (1): pricing-page-copywriter-reviewer
- **L33 App Store & Mobile Marketing (ASO)** (1): app-store-listing-aso-optimizer-ios-android
- **L35 Flagship Orchestrators** (8): see pipelines below

---

## Flagship orchestrators (L35) — the pipelines

Each chains specialist skills end-to-end into one bundled deliverable; all call `brand-brain` first.

| Orchestrator | Pipeline |
|---|---|
| `brand-brain-bootstrapper` | drives brand-brain bootstrap → 8 components in waves → full `brand.md` + companions |
| `seo-topic-cluster-factory` | keyword-research → pillar-architect → (per spoke) serp-analysis → content-brief → editorial-calendar |
| `keyword-to-published-post-pipeline-runner` | serp-analysis → content-brief → blog-draft → on-page-seo → aeo-geo → content-qa → publish handoff |
| `content-decay-refresh-sweep` | seo-content-health-decay-audit → (per URL) content-refresh-briefer → blog-draft → on-page-seo → content-qa |
| `full-campaign-launch-orchestrator` | campaign-brief → campaign-concept → ad-copy + landing + email + social → utm-builder → campaign-qa |
| `product-launch-gtm-orchestrator` | gtm-launch-planner → positioning → press + landing + email + social + deck → campaign-qa |
| `activation-onboarding-orchestrator` | aha-moment → lifecycle-mapper → welcome-onboarding → usage-triggered-sequencer → in-app-microcopy |
| `growth-reporting-auto-compiler-weekly-monthly` | ga4-digest + channel-roi + paid-summary + growth-diagnostic (via data-qa gotcha-check) → stakeholder-update |

---

## Call graph

```
   every skill ──calls──► brand-brain (Layer 0) ──delegates──► 8 components
   orchestrators (L35) ──chain──► specialist skills (L1–L33), each of which ──calls──► brand-brain
```

## Status / next

- **Tier-1: COMPLETE** (115/115). Built in 7 verified waves (Wave 1 opus; Waves 2–6 sonnet; Wave 7 orchestrators opus); each wave brand-brain-contract-verified, symlinked, committed, pushed.
- **Publishing:** GitHub public (this repo). AM Skills publishing paused per user.
- **Next candidates:** a library-wide quality polish pass; Tier-2 (152 skills) from the prioritized list; package/sell.
