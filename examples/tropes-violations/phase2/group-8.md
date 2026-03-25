# Phase 2: Cross-Cutting Patterns -- Group 8 (Integrations, Layout, Platform Views, Queue, Settings)

Input slices: integrations-1, integrations-2, layout, platform-content, platform-views-1 through platform-views-3, queue-1 through queue-3, settings-1, settings-2

---

## Pattern 1: Filler reassurance and fake-empathy in progress/error states (8 instances across 3 slices)

Connection wizards and error states pad messages with reassurance phrases that add no information and often sound condescending.

| File                          | Text                                                                |
| ----------------------------- | ------------------------------------------------------------------- |
| `SyncStep.tsx`                | "Usually a short wait"                                              |
| `SyncStep.tsx`                | "Preparing your inbox"                                              |
| `SyncStep.tsx`                | "We'll start syncing automatically" / "We will start automatically" |
| `LearnStep.tsx`               | "You can start working right away."                                 |
| `LearnStep.tsx`               | "Your inbox is ready" (redundant with "Setup complete" heading)     |
| `ErrorStep.tsx`               | "No worries. You can try again whenever you're ready."              |
| `ErrorStep.tsx`               | "This is usually temporary."                                        |
| `connection-wizard-steps.tsx` | "Preparing your connection..."                                      |

**Why this is a pattern**: Every progress step and error state includes at least one reassurance filler sentence. "Usually a short wait", "no worries", "this is usually temporary", and "you can start working right away" all promise comfort without providing information. The first-person "We'll start syncing" anthropomorphizes the system. "Preparing your inbox" and "Preparing your connection" hide what is actually happening behind vague warm language.

**Systemic fix**: Strip all reassurance filler. Progress labels should describe the actual operation ("Importing messages from Gmail", "Waiting for sync capacity"). Error states should state what happened and what the user can do, without emotional padding. Replace first-person "We" with system-focused language ("Sync will start when capacity is available").

---

## Pattern 2: "Connect your tools" as universal onboarding copy (5 instances across 3 slices)

The phrase "Connect your tools" appears as a heading, CTA, or instruction across multiple surfaces with no differentiation by context.

| File                            | Text                                                                               |
| ------------------------------- | ---------------------------------------------------------------------------------- |
| `directory-view.tsx`            | "Connect your tools" (onboarding heading)                                          |
| `directory-view.tsx`            | "Connect your tools" (settings empty-state label)                                  |
| `integration-connect-modal.tsx` | "Connect your [platform] account to start syncing"                                 |
| `core-integration-card.tsx`     | Identical descriptions for Gmail and Outlook ("Email threads + suggested replies") |
| `directory-view.tsx`            | "Choose what to sync" (generic subtitle)                                           |

**Why this is a pattern**: "Connect your tools" is used identically in onboarding and settings contexts. The modal adds "to start syncing" as a filler purpose clause. Platform descriptions are copy-pasted between Gmail and Outlook. "Choose what to sync" is undifferentiated. The entire integration surface reads like a template that was never customized.

**Systemic fix**: Write context-aware copy for each surface. Onboarding should explain what connecting enables. Settings should show current state. Each platform description should name what that specific platform provides. Remove "to start syncing" filler clauses.

---

## Pattern 3: Passive "No X available/provided" empty states (8 instances across 5 slices)

Platform views and content renderers use the same passive sentence structure for every empty state, regardless of whether the state is expected or indicates a problem.

| File                             | Text                                                                                            |
| -------------------------------- | ----------------------------------------------------------------------------------------------- |
| `google-calendar-event-view.tsx` | "No event description." / "No location provided." / "No meeting link." / "No attendees listed." |
| `notion-page-view.tsx`           | "No page content available."                                                                    |
| `linkedin-post-view.tsx`         | "Post preview not available."                                                                   |
| `monday-item-view.tsx`           | "No updates loaded."                                                                            |
| `platform-content-renderer.tsx`  | "Platform content not available for preview"                                                    |

**Why this is a pattern**: Every empty state follows the template "No [noun] [available/provided/loaded]." This passive construction says what is missing without explaining why or what to do about it. "Not available for preview" and "No updates loaded" leak implementation details ("preview", "loaded") into user-facing copy. The calendar empty states are less problematic (they describe genuinely optional fields), but the consistency of the pattern across all views suggests a template rather than considered copy.

**Systemic fix**: For genuinely optional fields (calendar location, description), the current copy is acceptable. For states that indicate a data gap or load failure, replace with actionable copy ("Content couldn't be loaded. Open in [platform] to see the original."). Remove implementation terms like "preview", "loaded", "provided".

---

## Pattern 4: AI anthropomorphism and "learning" language in settings (7 instances across 2 slices)

Settings pages describe system behavior using language that frames the AI as a conscious entity accumulating knowledge.

| File                                 | Text                                                                                          |
| ------------------------------------ | --------------------------------------------------------------------------------------------- |
| `settings-ai-snapshot-card.tsx`      | "{N} items learned"                                                                           |
| `settings-ai-context-modal.tsx`      | "What the AI Knows"                                                                           |
| `settings-voice-section.tsx`         | "Still learning your style. Keep sending messages and SlopWeaver will pick up your patterns." |
| `settings-ai-snapshot-card.tsx`      | "It fills in as you use your connected tools."                                                |
| `settings-current-focus-section.tsx` | "SlopWeaver will surface your focus areas as you work."                                       |
| `settings-people-section.tsx`        | "high-context relationships"                                                                  |
| `settings-people-section.tsx`        | "important people will show up here"                                                          |

**Why this is a pattern**: Every settings section uses a different flavor of the same anthropomorphism: "learned", "knows", "learning your style", "pick up your patterns", "surface your focus areas", "show up here". The settings page should be the most concrete surface in the product (showing what the system has computed), but instead it uses the vaguest language. "High-context relationships" is jargon with no user definition.

**Systemic fix**: Replace all anthropomorphic verbs with process descriptions. "Items learned" becomes "items imported". "What the AI Knows" becomes "AI Profile Context". "Still learning your style" becomes "Not enough messages processed yet." "Surface your focus areas" becomes "Projects appear here once messages are synced." Replace "high-context relationships" with "frequent contacts".

---

## Pattern 5: Developer-facing error messages leaking into user UI (6 instances across 4 slices)

Error toasts and fallback states use implementation terminology that users cannot interpret or act on.

| File                    | Text                                              |
| ----------------------- | ------------------------------------------------- |
| `jira-issue-view.tsx`   | "Copy not available in this context"              |
| `linear-issue-view.tsx` | "Copy not available in this context"              |
| `slack-thread-view.tsx` | "Missing thread context for reply"                |
| `slack-thread-view.tsx` | "Failed to send reply" / "Failed to add reaction" |
| `teams-thread-view.tsx` | "Failed to send reply"                            |
| `monday-item-view.tsx`  | "No updates loaded."                              |

**Why this is a pattern**: "In this context", "thread context", and "loaded" are all developer vocabulary. "Copy not available in this context" appears identically in two different platform views. Generic "Failed to [verb]" toasts appear across Slack and Teams with no actionability. These errors look like log messages that were promoted to user-facing text without rewriting.

**Systemic fix**: Replace "in this context" with a specific reason or "Copy failed". Replace "Missing thread context" with "Can't reply, open in Slack to reply directly." Add minimal guidance to all "Failed to X" toasts ("Couldn't send, check your connection" or provide a retry action).

---

## Pattern 6: "Suggested replies" and "Generate suggestion" as AI hedge language (4 instances across 3 slices)

AI-generated drafts are consistently labeled with tentative language that undersells the product and sounds like every other AI tool.

| File                        | Text                                                               |
| --------------------------- | ------------------------------------------------------------------ |
| `inline-reply-composer.tsx` | "Generate suggestion" / "Generating..."                            |
| `core-integration-card.tsx` | "Email threads + suggested replies" (Gmail and Outlook, identical) |
| `queue-empty-state.tsx`     | "AI-suggested actions will appear here with confidence scores"     |
| `queue-filter-bar.tsx`      | "Proposed actions ready for your review"                           |

**Why this is a pattern**: "Suggested", "suggestion", "proposed" are all hedge words that frame AI output as inherently tentative. Every AI startup uses "suggested reply" or "suggested action." The language positions SlopWeaver as apologetic about its own outputs rather than confident. The content playbook says to show real data and confidence scores, not hedge with soft labels.

**Systemic fix**: Replace "Generate suggestion" with "Draft reply". Replace "suggested replies" with what the integration actually provides. Replace "AI-suggested actions" with "Drafted actions" or just "Actions". Let confidence scores (which are already shown) communicate the uncertainty, not the label.

---

## Pattern 7: In-product marketing copy in queue and knowledge surfaces (5 instances across 3 slices)

Product-marketing language appears inside the app where users expect functional guidance.

| File                                | Text                                                                                              |
| ----------------------------------- | ------------------------------------------------------------------------------------------------- |
| `queue-empty-state.tsx`             | "All caught up!" / "As SlopWeaver learns your preferences, AI-suggested actions will appear here" |
| `queue-slide-over-footer.tsx`       | Five-sentence AI disclaimer on every action card                                                  |
| `queue-slide-over.tsx`              | "AI will learn from this" toast                                                                   |
| `knowledge-sources-empty-state.tsx` | "Bring your AI context" / "give SlopWeaver a head start on learning your patterns"                |
| `quick-import-section.tsx`          | "What SlopWeaver Learned" / "Top insights" / "AI context" jargon                                  |

**Why this is a pattern**: Queue empty states use "All caught up!" (a stock cliche from every email app) followed by marketing-register promises about future AI behavior. The queue footer includes a multi-sentence legal disclaimer on every action card that undermines confidence. Knowledge source surfaces use jargon ("AI context", "Top insights") that sounds meaningful but explains nothing concrete.

**Systemic fix**: Queue empty state: replace "All caught up!" with "No pending actions." Remove the marketing sentence about future AI behavior. Queue footer: move the disclaimer to a first-use modal or settings page, not every action card. Knowledge sources: replace "AI context" with "what SlopWeaver knows about you", "Top insights" with "extracted items", and "learned" with "imported".

---

## Pattern 8: Vague progress labels hiding actual system operations (4 instances across 2 slices)

Sync and connection progress steps use warm but uninformative labels instead of describing what is happening.

| File                          | Text                                                                    |
| ----------------------------- | ----------------------------------------------------------------------- |
| `SyncStep.tsx`                | "Preparing your inbox"                                                  |
| `SyncStep.tsx`                | "Learning signals" (section label for extracted patterns)               |
| `connection-wizard-steps.tsx` | "Preparing your connection..."                                          |
| `connection-wizard-steps.tsx` | "Connecting to [platform]..." (during OAuth redirect, not a connection) |

**Why this is a pattern**: "Preparing your X" appears twice as a progress label for operations that have concrete descriptions (importing data, confirming OAuth). "Learning signals" is jargon that sounds meaningful but is undefined. "Connecting to" is inaccurate during an OAuth redirect. All four labels obscure the actual operation behind soft language.

**Systemic fix**: Use specific verbs: "Importing messages from Gmail", "Confirming connection", "Redirecting to [platform] for authorization". Replace "Learning signals" with "Extracted patterns" or remove the label.

---

## Summary Table

| #   | Pattern                                     | Instance Count | Slices Affected | Severity |
| --- | ------------------------------------------- | -------------- | --------------- | -------- |
| 1   | Filler reassurance in progress/error states | 8              | 3               | High     |
| 2   | "Connect your tools" universal copy         | 5              | 3               | Medium   |
| 3   | Passive "No X available" empty states       | 8              | 5               | Medium   |
| 4   | AI anthropomorphism in settings             | 7              | 2               | High     |
| 5   | Developer-facing error messages in UI       | 6              | 4               | High     |
| 6   | "Suggested" hedge language for AI outputs   | 4              | 3               | Medium   |
| 7   | In-product marketing copy                   | 5              | 3               | High     |
| 8   | Vague progress labels hiding operations     | 4              | 2               | Medium   |
