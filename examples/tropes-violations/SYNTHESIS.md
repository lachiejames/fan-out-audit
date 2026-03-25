# Fan-Out Audit Synthesis: AI Writing Tropes

**Task**: Check all user-facing text against ~/Downloads/tropes.md
**Date**: 2026-03-25
**Slices audited**: 201
**Total files inspected**: 809
**Phase 1 slices with findings**: 90 (45%)
**Phase 2 cross-cutting patterns**: 108

## Top 10 Highest-Impact Opportunities

### 1. "Please reconnect your account" -- formulaic filler across ALL 17 platforms

- **Impact**: High (user-facing error copy, ~50+ instances)
- **Evidence**: Phase 2 groups 4, 5 -- every platform error file uses "Please reconnect your account" or "Please connect your [Platform] account" mechanically
- **What to do**: Global search-replace: drop "Please" from all reconnect messages. Use direct imperatives: "Reconnect your account" or "Reconnect your account to continue."
- **Estimated scope**: ~50 instances across `apps/api/src/integrations/platforms/*/errors/*.errors.ts`

### 2. "Learning your patterns" -- universal filler phrase (9+ instances)

- **Impact**: High (appears in onboarding, empty states, settings, analytics, auth sidebar)
- **Evidence**: Phase 2 group 6 -- "learn(ing) your patterns/style/preferences" used identically across 7+ components without ever defining what a "pattern" is
- **What to do**: Replace each instance with what the AI actually does in that context. In inbox: "calibrating reply tone from your sent messages." In analytics: "tracking edit rate on drafts." Generic "learning" language is the #1 SaaS AI cliche.
- **Estimated scope**: 9+ instances across `apps/app/src/`

### 3. Fractal summaries on every docs page

- **Impact**: High (every user doc page, ~11+ pages)
- **Evidence**: Phase 2 group 11 -- nearly every docs page ends with a closing callout that re-explains the section. Follows predictable formula: "[Feature] improves as the AI learns more / the more platforms you connect"
- **What to do**: Delete closing callout boxes that restate the section content. If something needs a CTA, make it an action ("Try it: connect Gmail and check your analytics after 48 hours") not a restatement.
- **Estimated scope**: 11+ doc pages in `apps/docs/src/app/`

### 4. Stock SaaS billing copy ("Choose Your Plan", "Sorry to see you go", "POPULAR")

- **Impact**: High (conversion-critical surfaces)
- **Evidence**: Phase 2 group 7 -- billing modals use phrases found verbatim on hundreds of SaaS products. "Upgrade anytime. Cancel anytime." "POPULAR" badge in ALL CAPS. "Maybe later" dismissal.
- **What to do**: Rewrite billing copy with SlopWeaver's voice. Replace "Choose Your Plan" with something tied to the product's differentiator. Remove ALL CAPS badges. Replace "Sorry to see you go" with direct copy about what happens next.
- **Estimated scope**: 7+ instances across `apps/app/src/components/billing/`

### 5. Generic integration marketing copy ("without context-switching", "stay on top of")

- **Impact**: High (marketing integration pages + docs)
- **Evidence**: Phase 2 groups 12, 13 -- integration descriptions reuse worn-out productivity phrases. "Better Context" appears as a placeholder use-case title in 6+ integrations. "Without context-switching" in jira + monday.
- **What to do**: Replace "Better Context" headings with integration-specific titles derived from the descriptions. Kill "without context-switching" and "stay on top of" -- describe what the integration actually does instead.
- **Estimated scope**: 20+ integration pages across marketing + docs

### 6. Filler reassurance in connection wizard ("No worries", "Usually a short wait")

- **Impact**: Medium (onboarding flow -- first impression)
- **Evidence**: Phase 2 group 8 -- SyncStep, LearnStep, ErrorStep use padding phrases: "Usually a short wait", "No worries. You can try again whenever you're ready.", "We'll start syncing automatically"
- **What to do**: Cut the filler. "Syncing..." needs no qualifier. Error states should give actionable instructions, not reassurance. "Try again" is sufficient without "No worries" or "whenever you're ready."
- **Estimated scope**: 8+ instances in connection wizard components

### 7. "Please try again later" -- stock error fallback across platforms

- **Impact**: Medium (user-facing error copy)
- **Evidence**: Phase 2 groups 4, 5 -- rate-limit and generic errors use "Please try again later" as a vague catch-all. Tells user nothing about when "later" is or why it failed.
- **What to do**: Replace with specific timing ("Rate limited. Try again in 60 seconds.") or specific actions ("Too many requests. Wait a minute, then retry."). Drop "Please."
- **Estimated scope**: 15+ instances across platform error files

### 8. Anthropomorphized AI ("your AI starts learning", "watch the AI learn")

- **Impact**: Medium (onboarding, settings, analytics)
- **Evidence**: Phase 2 group 6 -- 8 instances across 6 slices. AI framed as a learning student with human qualities. "Your AI starts learning from messages" treats the system as a separate entity being nurtured.
- **What to do**: Replace with concrete descriptions of what happens. "Messages you sync are used to calibrate draft style" instead of "Your AI starts learning from messages you sync."
- **Estimated scope**: 8+ instances across onboarding/settings/analytics

### 9. "Your data stays yours" and privacy cliches in trust signals

- **Impact**: Medium (marketing CTAs, security/privacy pages)
- **Evidence**: Phase 2 groups 13, 14 -- "Your data stays yours", self-applied virtue labels in privacy/security contexts. Generic trust signals that every SaaS uses.
- **What to do**: Replace with specific claims: "No model training on your data" or "OAuth tokens AES-256 encrypted" -- things that are actually differentiating and verifiable.
- **Estimated scope**: 5+ instances across marketing and CTA components

### 10. Vague AI labels ("Insights", "Context", "AI confidence") leaking system internals

- **Impact**: Medium (UI labels across app)
- **Evidence**: Phase 2 group 16 -- KnowledgeSparkles shows "Insights" when it means "extracted facts about you." ContextBar shows "Context" when it means "the email the AI is looking at." ConfidenceBar uses "AI confidence" as an aria-label.
- **What to do**: Replace abstract labels with concrete descriptions. "Insights" -> "What the AI knows about you." "Context" -> the actual subject line or message title. "AI confidence: X%" -> "Style match: X%."
- **Estimated scope**: 10+ labels across UI components

## Cross-Cutting Patterns (Deduplicated)

### Systemic: "Please" politeness filler in all error messages

50+ instances of "Please" prefixing error instructions. Universal across API error files. Single search-replace fix.

### Systemic: "Get Started Free" / "Ready to get started?" CTA boilerplate

Appears in marketing hero, pricing, integration pages. Invisible due to ubiquity. Replace with product-specific CTAs.

### Systemic: Empty states that market instead of help

Empty states across inbox, queue, tasks, contacts use marketing language ("as your AI learns...") instead of helping the user take the next action.

### Systemic: "Manage your X, Y, and Z" subtitle formula

Settings sections use "Manage your subscription, security, and data" type subtitles that restate the heading. Remove or replace with actionable descriptions.

### Systemic: "In your voice/style" as AI writing cliche

Multiple surfaces use "in your voice" -- the same phrase every AI writing tool uses. Replace with SlopWeaver's actual differentiator (measurable style metrics).

### Clean zones (no action needed)

- **UI atoms/foundations/molecules**: Zero tropes in 60+ component files. Text is prop-driven.
- **Backend error infrastructure**: application/\*/errors/ and utils/error-mapper.ts files are structurally clean (no prose).
- **Storybook stories**: Fixture data uses functional labels.
- **Auth/form pages**: Login, signup, forgot-password are consistently trope-free.

## Slices with zero findings

111 of 201 slices had zero findings. Primary clean zones:

- All `packages/ui/src/atoms/` (icons, primitives)
- All `packages/ui/src/foundations/` (hooks, layout)
- Most `packages/ui/src/molecules/` (structural UI)
- Most `apps/api/src/application/*/errors/` (terse error factories)
- Auth/form pages in `apps/app/src/pages/`

These are genuinely clean -- the agents checked and confirmed no user-facing prose exists in these files.

## Raw data

- Phase 1: `docs/audits/tropes-violations/phase1/` (201 files)
- Phase 2: `docs/audits/tropes-violations/phase2/` (17 files)
- Slice plan: `docs/audits/tropes-violations/SLICE_PLAN.md`
- Task: `docs/audits/tropes-violations/TASK.md`
