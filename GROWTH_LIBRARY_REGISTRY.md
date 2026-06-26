# Growth Skill Library — Registry & Call Graph

The map of the built library. **Tier-1 + Tier-2 are complete.** Total = **269 skill folders** (the brand-brain Layer-0 system + 115 Tier-1 + 149 Tier-2; 2 Tier-2 catalog entries were exact dupes of brand-brain components and are covered by them). Every skill calls `brand-brain` for context and composes built siblings.

Last updated: 2026-06-22  ·  **269 skills · Tier-1 + Tier-2 complete · only Tier-3 (≈142) remains**

**Library home:** `~/GrowthSkills/` — canonical source in `skills/<slug>/`, each symlinked into `~/.claude/skills/`. **GitHub:** https://github.com/NirvanaGuha/GrowthSkills (public, single repo, no collaborators).

---

## Architecture rules

1. **One brand brain, many readers.** Every skill calls `brand-brain` first to load the active brand; none re-derives it. Mandatory fallback line in each skill (thin `~/.brandbrain/brands/.active` + `brand.md` read, else ask).
2. **Data outside skill folders.** Brand data → `~/.brandbrain/brands/<slug>/`. Skill artifacts → project-relative `./<folder>/`. Survives `amskills` rm-rf.
3. **Skills call skills.** Compose over re-implement; reviewers/auditors review siblings rather than rebuild them; orchestrators chain specialists.
4. **Graceful degradation + no recursion.**

## Data roots & build method

- **Brand data:** `~/.brandbrain/brands/<slug>/{brand.md, personas.md, competitors.md, proof.md, objections.md, style-guide.md}` · active pointer `.active`. Built brands: `pushengage`.
- **Build method:** reusable wave workflows — parallel build (sonnet; orchestrators on opus) → sonnet verify against the library contract → fix MUST issues → symlink → commit/push. Tier-1 in 7 waves; Tier-2 in 7 waves.

---

## Layer 0 — brand-brain system

`brand-brain` (resolve/serve/bootstrap/refresh/audit, multi-brand) + 8 components: `brand-voice-codifier`, `icp-persona-builder`, `positioning-messaging-architect`, `competitive-intelligence-dossier`, `offer-pricing-brain`, `proof-vault`, `objection-library-builder`, `editorial-style-guide`.

---

## Tier-1 catalog — 115 skills (the MVP spine)

- **L1** brand-voice-codifier · icp-persona-builder · positioning-messaging-architect · battlecard-objection-handler · win-loss-interview-synthesizer · objection-library-builder
- **L2** gtm-launch-planner · experiment-pipeline-backlog-prioritizer · okr-suite
- **L3** voice-of-customer-mining-pipeline
- **L4** on-page-seo-optimizer · serp-analysis-report · topic-cluster-pillar-architect · content-brief-builder · aeo-geo-llm-visibility-optimizer · content-refresh-briefer · keyword-research-clustering-suite · seo-content-health-decay-audit
- **L5** headline-hook-generator · landing-product-page-copy-writer · content-repurposer-atomizer · de-slop-humanize-pass · blog-post-drafting-engine · editorial-calendar-builder · content-qa-reviewer · cta-variant-generator
- **L6** linkedin-post-writer · x-thread-writer · short-form-video-script-writer · social-content-calendar-builder · faceless-short-form-video-producer
- **L7** subject-line-preview-text-optimizer · welcome-onboarding-email-sequence-builder · push-notification-copy-generator · lifecycle-journey-mapper · abandon-flow-writer · lead-nurture-drip-builder · full-email-push-asset-builder-copy-responsive-html · esp-map-platform-builder
- **L8** campaign-brief-builder · ad-copy-variant-generator · weekly-paid-performance-summary · bid-budget-pacing-checker
- **L9** a-b-multivariate-test-designer · experiment-results-analyzer · landing-page-heuristic-live-cro-auditor · funnel-drop-off-analyzer
- **L10** channel-roi-scorecard · ltv-cac-payback-calculator · data-qa-measurement-gotcha-checker · ga4-weekly-traffic-digest · growth-diagnostic-deep-dive · sql-query-generator-for-ga4-bigquery
- **L11** usage-triggered-message-sequencer · aha-moment-activation-metric-definer · in-app-microcopy-writer-auditor
- **L12** meeting-prep-follow-up-pack · account-dossier-builder · cold-outreach-sequence-architect · account-list-builder-icp-scorer · sales-asset-reviewer
- **L13** deck-presentation-writer · ai-image-generator-on-brand-assets · creative-feedback-translator
- **L14** utm-parameter-bulk-builder · campaign-qa-launch-checklist-generator · automation-workflow-designer-debugger
- **L15** prioritization-framework-suite · pre-mortem-post-mortem-generator · analytical-reasoning-toolkit · eisenhower-matrix-weekly-priorities-sorter
- **L16** creative-ideation-framework-suite · campaign-concept-developer · how-might-we-10x-reframer
- **L17** meeting-agenda-action-item-builder · doc-note-summarizer · daily-weekly-planning-sprint · async-standup-writer
- **L18** sop-builder-reviewer · **L19** stakeholder-update-status-writer · quick-tone-softener-diplomacy-pass · **L20** media-podcast-pitch-crafter · newsjacking-angle-finder · press-release-social-blog-amplification-pack · quote-polisher
- **L21** landing-page-builder-html-tailwind · **L22** product-demo-gif-mockup-builder · **L24** vendor-cost-comparison-renewal-brief · **L25** agency-deliverable-qa-pulse-reviewer · contractor-agency-brief-builder
- **L26** webinar-event-campaign-planner · webinar-email-sequence-writer · event-registration-on-demand-page-copywriter · event-social-promotion-pack · event-recap-blog-post-writer
- **L27** case-study-customer-spotlight-production-suite · churn-save-sequence-writer · upsell-cross-sell-campaign-builder · referral-program-brief-builder · review-response-drafter
- **L28** partnership-outreach-personalization-writer · **L30** prompt-engineering-library-suite · multi-source-deep-research-report-builder · custom-ai-agent-builder · **L31** shopify-product-collection-page-seo-builder · **L32** pricing-page-copywriter-reviewer · **L33** app-store-listing-aso-optimizer-ios-android
- **L35 orchestrators (8)** brand-brain-bootstrapper · seo-topic-cluster-factory · keyword-to-published-post-pipeline-runner · content-decay-refresh-sweep · full-campaign-launch-orchestrator · product-launch-gtm-orchestrator · activation-onboarding-orchestrator · growth-reporting-auto-compiler-weekly-monthly

---

## Tier-2 catalog — 149 skills (depth & breadth)

- **L1** product-feature-knowledge-base-curator · brand-brain-health-auditor *(+ competitive/offer/proof/editorial = the components)*
- **L2** growth-model-builder · marketing-plan-generator · weekly-ops-digest · board-exec-summary-writer · positioning-reviewer · channel-strategy-selector · initiative-business-case-writer · marketing-roadmap-builder
- **L3** jtbd-customer-interview-suite · competitor-review-gap-spotter · synthetic-persona-interview · reddit-forum-listening-digest · survey-designer-analyzer · sentiment-shift-detector
- **L4** internal-linking-planner · meta-title-description-bulk-writer · content-gap-finder · technical-seo-audit-fix-prioritizer · gsc-monitoring-suite · schema-markup-generator
- **L5** content-format-writer-suite · publishing-integration-hub
- **L6** linkedin-carousel-builder · social-ad-copy-writer · post-quality-reviewer-voice-auditor · linkedin-profile-optimizer · instagram-caption-reel-script-writer · tiktok-script-hook-generator · youtube-description-channel-seo-optimizer · ugc-creator-brief-writer · bulk-scheduling-csv-builder · social-performance-review-summarizer
- **L7** win-back-re-engagement-campaign-builder · segmentation-rfm-strategy-builder · post-purchase-nurture-sequence-builder · newsletter-issue-builder · email-a-b-test-planner-performance-analyzer · lifecycle-email-push-copy-reviewer · notification-opt-in-prompt-optimizer · lead-scoring-routing-model-designer · transactional-email-copywriter · sms-whatsapp-message-writer · email-compliance-auditor-gdpr-can-spam · weekly-email-push-metrics-digest · send-time-cadence-recommender
- **L8** keyword-list-builder-segmenter · search-term-report-triage · creative-fatigue-monitor · ad-to-landing-page-message-match-auditor · competitor-ad-library-spy · audience-targeting-spec-writer · linkedin-campaign-spec-builder
- **L9** pricing-page-optimizer · segmented-conversion-rate-breakdown · sample-size-calculator · post-test-learning-logger · form-friction-auditor · heatmap-session-recording-synthesizer · validity-threat-checker · cro-reporting-audit-packager
- **L10** ga4-custom-report-exploration-builder · weekly-wins-anomalies-slack-ping · seo-to-ga4-gap-analysis · ga4-audience-custom-segment-builder · tracking-plan-taxonomy-builder-auditor · looker-studio-live-dashboard-builder · analytics-report-reviewer · ga4-anomaly-detector · kpi-tree-builder · attribution-model-configurator
- **L11** onboarding-flow-builder · upgrade-expansion-prompt-writer · user-segment-activation-playbook · customer-onboarding-time-to-value-program-designer · feature-adoption-campaign-planner · nps-csat-feedback-loop-designer
- **L12** crm-data-hygiene-dedup-runner · roi-business-case-calculator · sales-partner-collateral-creator · sdr-daily-prioritization-engagement-digest · hubspot-sequence-workflow-builder · intent-signal-summarizer · deal-communication-pack
- **L13** canva-figma-workflow-accelerator · creative-brief-image-prompt-crafter · alt-text-accessibility-copy-batch-writer · infographic-data-viz-spec-writer · brand-consistency-auditor · design-qa-handoff-pack-builder · stock-asset-mood-board-brief
- **L14** martech-stack-auditor-mapper · consent-privacy-compliance-auditor · utm-campaign-naming-enforcer · gtm-tag-builder-server-side-conversion-setup · advertising-claims-ftc-disclosure-reviewer
- **L15** go-no-go-gate-evaluator · decision-log-brief-writer · tradeoff-memo-writer · **L16** idea-evaluation-stress-test-suite · constraint-based-ideator · **L17** quarterly-goal-decomposer · comms-inbox-processor
- **L18** knowledge-base-internal-faq-manager · onboarding-enablement-knowledge-pack-builder · **L19** escalation-note-change-announcement-drafter · feedback-recognition-framer · loom-async-video-script-writer · **L20** press-release-writer-reviewer · crisis-communications-writer · haro-source-response-writer
- **L21** google-sheets-excel-power-tools-builder · pop-up-sticky-bar-widget-builder · page-speed-core-web-vitals-fixer · **L22** screenshot-annotator-bug-reporter · video-loom-summarizer · annotated-dashboard-report-builder · social-proof-screenshot-styler
- **L23** deadline-drift-detector · project-plan-generator-reviewer · risk-log-builder · **L24** budget-variance-p-l-reconciliation-analyst · cfo-ready-budget-summary-slide-builder · seo-cost-per-keyword-roi-calculator · **L25** marketing-job-description-hiring-scorecard-writer
- **L26** webinar-script-run-of-show-generator · event-roi-calculator · cfp-abstract-speaker-outreach-writer · **L27** referral-email-sequence-writer · review-testimonial-solicitation-sequence · **L28** co-marketing-sponsorship-campaign-planner · partner-onboarding-checklist-generator
- **L29** transcreation-multi-market-copy-adapter · **L30** ai-tool-evaluator · ai-skill-gap-learning-plan-builder · **L31** ecommerce-seasonal-promotional-campaign-planner · **L32** packaging-tiering-designer · competitor-price-benchmarking-analyst · **L33** aso-keyword-research-gap-analyst · **L34** community-led-acquisition-advocacy-amplifier · owned-community-platform-selector-architecture-planner
- **L35 orchestrators (7)** new-competitor-detected-response-kit · persona-to-full-funnel-messaging-kit · abm-account-to-outreach-kit · win-back-sweep-orchestrator · paid-campaign-spin-up-orchestrator · viral-moment-rapid-response-kit · event-end-to-end-orchestrator

---

## Call graph

```
   every skill ──calls──► brand-brain (Layer 0) ──delegates──► 8 components
   15 L35 orchestrators ──chain──► specialist skills (L1–L34), each of which ──calls──► brand-brain
```

## Status / next

- **Tier-1: COMPLETE (115).  Tier-2: COMPLETE (149).** Both built in 7 verified waves each; structural audit 269/269 clean.
- **Quality:** opus depth-audit sweeps on stratified samples — Tier-1 avg 3.79, Tier-2 avg 3.93 — both now uniformly ≥4/5 after polishing the flagged few (mostly framework-accuracy fixes).
- **Publishing:** GitHub public. AM Skills paused per user.
- **Remaining on the prioritized list:** Tier-3 (~142, long tail). Other options: dogfood a brand end-to-end, full per-skill quality grade.
