# Phase 2: Cross-Cutting Patterns -- Group 6 (App UI Components)

Input slices: components-2 through components-8, ai-chat-widget, analytics-1, analytics-2, auth-1, auth-2

---

## Pattern 1: "Learning your patterns" as universal filler (9 instances across 7 slices)

The phrase "learn(ing) your patterns" (or close variants like "learn your style", "learn your preferences", "pick up your patterns") appears as a catch-all description of what SlopWeaver does, used identically across wildly different contexts without ever defining what a "pattern" is.

| File                             | Text                                                              |
| -------------------------------- | ----------------------------------------------------------------- |
| `ai-chat-widget-empty-state.tsx` | "I'm still learning your patterns."                               |
| `inbox-empty-state.tsx` (x2)     | "learning your communication patterns" / "learning your patterns" |
| `settings-profile.tsx`           | "needs more activity to learn your patterns"                      |
| `auth-branding-sidebar.tsx`      | "Your AI starts learning from messages you sync"                  |
| `auth-branding-sidebar.tsx`      | "Watch the AI learn"                                              |
| `analytics-copy.utils.ts`        | "More time saved as the AI learns your patterns"                  |
| `readiness-panel.tsx`            | "as your data grows"                                              |
| `sample-data-banner.tsx`         | "as your AI learns"                                               |

**Why this is a pattern, not isolated incidents**: The word "patterns" is used as an ornate container noun that never gets defined. Users see "patterns learned", "writing patterns", "communication patterns" across multiple surfaces with no indication of what a pattern actually is. The phrase has become a product-wide verbal tic that substitutes for concrete descriptions of behavior.

**Systemic fix**: Define a glossary of what SlopWeaver actually extracts (voiceprint traits, response preferences, triage rules, contact priorities) and use those concrete nouns instead of "patterns" everywhere. Each surface should name the specific thing relevant to its context.

---

## Pattern 2: Anthropomorphizing the AI as a learning student (8 instances across 6 slices)

Across analytics, auth, settings, and empty states, copy consistently frames SlopWeaver as a sentient student progressing through a learning journey, rather than a system processing data.

| File                        | Text                                                |
| --------------------------- | --------------------------------------------------- |
| `milestone-celebration.tsx` | "Your AI is starting to learn your patterns"        |
| `milestone-celebration.tsx` | "The AI is adapting to your communication style"    |
| `milestone-celebration.tsx` | "Your AI is getting smarter with every interaction" |
| `auth-branding-sidebar.tsx` | "Watch your AI improve over time"                   |
| `auth-branding-sidebar.tsx` | "Connect your tools. Watch the AI learn."           |
| `analytics-copy.utils.ts`   | "the AI is getting closer to your voice"            |
| `settings-profile.tsx`      | "needs more activity to learn your patterns"        |
| `readiness-panel.tsx`       | "Your trends are forming!"                          |

**Why this matters**: These phrases make unverifiable causal claims about AI improvement. "Getting smarter", "adapting", "improving" are marketing assertions the UI cannot prove. They also set expectations that the system has conscious intent, which erodes trust when the system makes mistakes.

**Systemic fix**: Replace agency language with observable outcomes. Instead of "the AI is learning", state what changed: "Acceptance rate: 72% (up from 65%)". Let metrics prove improvement rather than prose claiming it.

---

## Pattern 3: Pedagogical "X means Y" guidance copy in analytics (9 instances in 1 slice, pattern echoed in 2 others)

The analytics `METRIC_GOOD_DESCRIPTIONS` object contains nine strings that all follow the formula "Higher/Lower/More X means the AI is [doing something good]." This didactic voice also appears in milestone celebrations and readiness panels.

| File                      | Text                                                                   |
| ------------------------- | ---------------------------------------------------------------------- |
| `analytics-copy.utils.ts` | "Higher is better. Shows how well the AI matches your style."          |
| `analytics-copy.utils.ts` | "More positive feedback means the AI chat is genuinely helpful."       |
| `analytics-copy.utils.ts` | "Lower means the AI is getting closer to your voice."                  |
| `analytics-copy.utils.ts` | "More time saved as the AI learns your patterns."                      |
| `analytics-copy.utils.ts` | "More patterns means better predictions and drafts."                   |
| `analytics-copy.utils.ts` | "Faster responses mean the AI is helping you stay on top of messages." |
| `analytics-copy.utils.ts` | "Improvement across more categories means broader learning."           |
| `analytics-copy.utils.ts` | "Fewer overrides means the AI is triaging more like you would."        |
| `readiness-panel.tsx`     | "Now that you have some data, here are your first insights."           |

**Why this is a pattern**: Every guidance string treats the user as someone who cannot interpret a dashboard metric without a sentence explaining whether "higher" is good. The voice is patronizing and formulaic. It also embeds unverifiable causal claims (the AI "helping you stay on top of messages") into what should be neutral data descriptions.

**Systemic fix**: Remove the "X means Y" descriptions entirely. If contextual help is needed, use a tooltip with the metric's definition (what it measures), not an interpretation (what it proves about the AI).

---

## Pattern 4: Gamification language -- "unlock" for standard feature availability (4 instances across 2 slices)

Charts, metrics, and features that simply require a data minimum are framed as locked rewards to be earned.

| File                               | Text                                                            |
| ---------------------------------- | --------------------------------------------------------------- |
| `analytics-copy.utils.ts`          | "you'll unlock your ${label} chart"                             |
| `readiness-panel.tsx`              | "you'll unlock trend charts for the remaining ${notReadyCount}" |
| `readiness-panel.tsx`              | "you'll unlock your first metric charts"                        |
| `sync-limit-paywall.tsx` (Group 7) | "unlock reply features"                                         |

**Why this is a pattern**: "Unlock" frames passive data accumulation as achievement gating. The chart is not locked; it lacks sufficient data. This gamification language borrowed from mobile apps is explicitly banned in the content playbook.

**Systemic fix**: Replace "unlock" with "appear" or "become available" throughout. State the concrete threshold when known ("After 5 more days of use, this chart will have enough data").

---

## Pattern 5: False suspense via ellipsis and dramatic progress labels (5 instances across 4 slices)

Loading states and progress indicators use language that dramatizes latency rather than reporting it neutrally.

| File                              | Text                       |
| --------------------------------- | -------------------------- |
| `ai-typing-indicator.tsx`         | "Thinking..."              |
| `ai-chat-widget.messages.tsx`     | "Still working on this..." |
| `ai-chat-widget-file-preview.tsx` | "Loading..."               |
| `ai-tool-group.tsx`               | "Searching..."             |
| `readiness-panel.tsx`             | "Your trends are forming!" |

**Why this is a pattern**: Each instance attributes a human cognitive process to the system ("thinking", "working on this") or uses an exclamation to inflate routine data processing into an event. These are ubiquitous AI-product loading tropes that users have learned to ignore.

**Systemic fix**: Use neutral, specific status labels. Replace "Thinking" with nothing (dots suffice) or with a concrete verb ("Checking your tools"). Remove all dramatic ellipses and exclamation marks from status copy.

---

## Pattern 6: Stakes inflation in milestone/progress celebrations (6 instances in 2 slices)

Milestone messages treat routine data accumulation as dramatic achievements worthy of excitement.

| File                        | Text                                                                                  |
| --------------------------- | ------------------------------------------------------------------------------------- |
| `milestone-celebration.tsx` | "Your first 10 AI interactions logged!"                                               |
| `milestone-celebration.tsx` | "3 metrics are now active. Your dashboard is coming to life."                         |
| `milestone-celebration.tsx` | "100 messages processed. Your AI is getting smarter with every interaction."          |
| `milestone-celebration.tsx` | "Your trends are forming!"                                                            |
| `milestone-celebration.tsx` | "All metrics are now powered by real data. Your analytics dashboard is fully active." |
| `readiness-panel.tsx`       | "Your trends are forming! Most metrics are now active."                               |

**Why this is a pattern**: Every milestone is written as a mini press release with exclamation marks and marketing register. "Coming to life", "powered by real data", "fully active" are sales phrases, not status copy. The tone is inconsistent with the rest of the product's functional voice.

**Systemic fix**: Rewrite milestones as neutral status updates without exclamation marks. "10 interactions logged. First metric chart available." State facts, not celebrations.

---

## Pattern 7: Internal jargon leaking into user-facing copy (5 instances across 3 slices)

Technical or model-specific terminology appears in UI copy where users have no context for it.

| File                         | Text                                                                 |
| ---------------------------- | -------------------------------------------------------------------- |
| `model-selector.tsx`         | "Supports extended reasoning" / "No extended reasoning"              |
| `voice-settings-menu.tsx`    | "v3 voice profile preview"                                           |
| `voice-settings-menu.tsx`    | "Voice provider is unavailable" / "Voice catalog" / "curated voices" |
| `voice-settings-content.tsx` | "Generating assistant voice" / "Playing assistant voice"             |
| `ai-chat-widget.tsx`         | "Reasoning enabled" / "Reasoning disabled"                           |

**Why this is a pattern**: Model capabilities ("extended reasoning"), internal versioning ("v3"), infrastructure terms ("voice provider", "voice catalog"), and capability flags ("reasoning enabled") are surfaced directly to users who have no context for them.

**Systemic fix**: Map every technical capability to a user-facing benefit description. "Extended reasoning" becomes "better at complex problems" or is omitted. Version numbers never appear in UI. Infrastructure terms ("provider", "catalog") are replaced with the feature name ("voice", "voices").

---

## Summary Table

| #   | Pattern                                  | Instance Count | Slices Affected | Severity |
| --- | ---------------------------------------- | -------------- | --------------- | -------- |
| 1   | "Learning your patterns" filler          | 9              | 7               | High     |
| 2   | AI anthropomorphized as learning student | 8              | 6               | High     |
| 3   | Pedagogical "X means Y" guidance         | 9+             | 3               | Medium   |
| 4   | Gamification "unlock" language           | 4              | 2               | Medium   |
| 5   | False suspense in loading states         | 5              | 4               | Medium   |
| 6   | Stakes inflation in milestones           | 6              | 2               | Medium   |
| 7   | Internal jargon in user-facing copy      | 5              | 3               | Medium   |
