# Tropes Audit: apps/app/src/components/analytics (batch 1)

**Files audited**:

- `ai-chart-sections.tsx`
- `ai-headline-stats.tsx`
- `ai-metrics.tsx`
- `analytics-header.tsx`
- `metric-state.tsx`
- `milestone-celebration.tsx`
- `onboarding-connect-cards.tsx`
- `performance-chart-sections.tsx`
- `analytics-copy.utils.ts` (drives all cold-start and unlock copy; audited as part of this batch)

---

## Findings

### 1. Stakes inflation + false suspense — `milestone-celebration.tsx`, lines 28–63

The milestone messages treat routine data accumulation as dramatic progress events.

| Line | Text                                                                                    | Problem                                                                                                                                           |
| ---- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 28   | `"Your first 10 AI interactions logged! Your AI is starting to learn your patterns."`   | Exclamation mark on a routine threshold. "Starting to learn" implies the AI was not learning before — false suspense.                             |
| 33   | `"3 metrics are now active. Your dashboard is coming to life."`                         | "Coming to life" is ornate filler. Three metrics loading is not a resurrection event.                                                             |
| 44   | `"First writing pattern learned! The AI is adapting to your communication style."`      | Exclamation + progress narration the user did not ask for.                                                                                        |
| 50   | `"100 messages processed. Your AI is getting smarter with every interaction."`          | "Getting smarter with every interaction" is a marketing claim, not a fact the UI can assert. It mirrors the exact wording used in AI product ads. |
| 56   | `"Your trends are forming! You can now see how your AI is improving over time."`        | "Trends are forming!" is false suspense. "How your AI is improving" repeats the unverified "getting smarter" claim.                               |
| 63   | `"All metrics are now powered by real data. Your analytics dashboard is fully active."` | "Powered by real data" / "fully active" are marketing register, not status copy.                                                                  |

**Pattern**: Every milestone message is a mini press release. The text announces what happened, then explains its significance in excited tones the user did not ask for.

---

### 2. Pedagogical voice — `analytics-copy.utils.ts`, `METRIC_GOOD_DESCRIPTIONS`, lines 34–45

These strings appear as guidance text (rendered via `getMetricGoodDescription`). Each one tells the user what a good value means, unsolicited.

| Key                  | Text                                                                     | Problem                                                                |
| -------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| `approvalRate`       | `"Higher is better. Shows how well the AI matches your style."`          | Tells the user how to read a percentage. Condescending.                |
| `chatFeedback`       | `"More positive feedback means the AI chat is genuinely helpful."`       | Circular and patronizing. The user knows what positive feedback means. |
| `editFrequency`      | `"Lower means the AI is getting closer to your voice."`                  | Marketing claim embedded in guidance copy.                             |
| `estimatedTimeSaved` | `"More time saved as the AI learns your patterns."`                      | Future-tense prediction delivered as fact.                             |
| `patternsLearned`    | `"More patterns means better predictions and drafts."`                   | Unqualified causal claim.                                              |
| `responseTime`       | `"Faster responses mean the AI is helping you stay on top of messages."` | Attributes causation to the AI without evidence.                       |
| `trainingCategories` | `"Improvement across more categories means broader learning."`           | "Broader learning" is vague AI-speak.                                  |
| `triageOverrides`    | `"Fewer overrides means the AI is triaging more like you would."`        | Reasonable but still didactic.                                         |
| `volume`             | `"Higher volume means more data for the AI to learn from."`              | Acceptable, but consistent with the pedagogical pattern throughout.    |

**Pattern**: Every description follows the formula "X is better/worse because the AI [does something good]." This is a pedagogical voice applied to every metric simultaneously, turning a data dashboard into a tutorial.

---

### 3. Magic adverb + stakes inflation — `analytics-copy.utils.ts`, `getMetricUnlockText`, line 132

```
After ${remaining} more ${unit}, you'll unlock your ${label} chart.
```

"Unlock" frames a chart becoming visible as a reward or achievement gate. This is gamification language borrowed from mobile apps. The chart is not locked; it simply requires a data minimum. Consider: "After N more X, this chart will have enough data to display."

---

### 4. Vague AI-speak noun — `analytics-copy.utils.ts`, `METRIC_DESCRIPTIONS`, line 29

```
"Writing and workflow patterns the AI has identified"
```

"Patterns the AI has identified" is accurate enough, but "patterns" is used throughout the copy as an ornate container noun that never gets defined. Users see "Patterns Learned," "writing patterns," "communication patterns" across multiple surfaces with no indication of what a pattern actually is (a rule? a tendency? a template?).

---

### 5. Redundant progress narration — `onboarding-connect-cards.tsx`, line 48

```
"Connect your tools to start learning"
```

This is a section heading on a connect-platform card grid. "Start learning" anthropomorphizes the AI as a student waiting to begin. The heading is functional — it could simply read "Connect your tools" or "Connect platforms to enable analytics."

---

### 6. Ornate verb — `analytics-copy.utils.ts`, `METRIC_GOOD_DESCRIPTIONS`, line 41

```
"Faster responses mean the AI is helping you stay on top of messages."
```

"Stay on top of" is a filler idiom. The metric is average response time. The guidance could state what the number means without narrating a productivity story.

---

### 7. False attribution of causation — multiple locations

The copy routinely assigns AI causation to outcomes that could have other explanations:

- "Faster responses mean the AI is helping you stay on top of messages" (`analytics-copy.utils.ts`)
- "Your AI is getting smarter with every interaction" (`milestone-celebration.tsx`)
- "The AI is adapting to your communication style" (`milestone-celebration.tsx`)
- "The AI is triaging more like you would" (`analytics-copy.utils.ts`)

None of these are verifiable claims the UI can make. They inflate the perceived intelligence of the system and will read as puffery to skeptical users.

---

## Summary

| File                             | Tropes present                                                             |
| -------------------------------- | -------------------------------------------------------------------------- |
| `milestone-celebration.tsx`      | Stakes inflation, false suspense, unverified causal claims                 |
| `analytics-copy.utils.ts`        | Pedagogical voice, magic adverb ("unlock"), ornate verb, false attribution |
| `onboarding-connect-cards.tsx`   | Anthropomorphism ("start learning")                                        |
| `ai-chart-sections.tsx`          | None                                                                       |
| `ai-headline-stats.tsx`          | None                                                                       |
| `ai-metrics.tsx`                 | None                                                                       |
| `analytics-header.tsx`           | None                                                                       |
| `metric-state.tsx`               | None (copy sourced from utils)                                             |
| `performance-chart-sections.tsx` | None                                                                       |

The chart and stat components are clean. All violations are in the copy layer (`analytics-copy.utils.ts` and `milestone-celebration.tsx`), which makes them straightforward to fix in one place.
