# Tropes Audit: apps/app/src/components/analytics (slice 2)

Files audited:

- `apps/app/src/components/analytics/performance-metrics.tsx`
- `apps/app/src/components/analytics/readiness-panel.tsx`
- `apps/app/src/components/analytics/sample-data-banner.tsx`
- `apps/app/src/components/analytics/summary-cards.tsx`

---

## performance-metrics.tsx

No findings.

---

## readiness-panel.tsx

### Finding 1

**File**: `apps/app/src/components/analytics/readiness-panel.tsx`
**Line**: 23 (STATE_CONFIG `learning.subtitle`)
**Text**: `"Now that you have some data, here are your first insights."`
**Trope**: Breathless milestone framing. "First insights" positions routine data accumulation as a meaningful AI discovery moment.
**Suggested fix**: `"Here are your first charts based on the data collected so far."`

### Finding 2

**File**: `apps/app/src/components/analytics/readiness-panel.tsx`
**Line**: 28 (STATE_CONFIG `learning.benchmark`)
**Text**: `"Your first metric charts will appear once enough data accumulates."`
**Trope**: Vague promise. "Enough data" is undefined and creates anticipation without concrete information.
**Suggested fix**: `"Metric charts appear after 3-5 days of activity."`

### Finding 3

**File**: `apps/app/src/components/analytics/readiness-panel.tsx`
**Line**: 34 (STATE_CONFIG `syncing.benchmark`)
**Text**: `"All metrics activate as your data grows. Most are ready within 2 weeks."`
**Trope**: Organic growth metaphor ("as your data grows"). Frames passive data accumulation as a living process.
**Suggested fix**: `"All metrics will be ready within 2 weeks of regular use."`

### Finding 4

**File**: `apps/app/src/components/analytics/readiness-panel.tsx`
**Line**: 41 (STATE_CONFIG `trending.subtitle`)
**Text**: `"Your trends are forming! Most metrics are now active."`
**Trope**: Exclamation mark hype. Routine progress treated as an exciting event. "Forming" anthropomorphizes data patterns.
**Suggested fix**: `"Most metrics are now active. Your trend charts are building."`

### Finding 5

**File**: `apps/app/src/components/analytics/readiness-panel.tsx`
**Lines**: 112-113 (LearningContext)
**Text**: `"After a few more days, you'll unlock trend charts for the remaining ${notReadyCount}."` and `"After a few more days of use, you'll unlock your first metric charts."`
**Trope**: Gamification language ("unlock"). Frames standard feature availability as a reward to be earned.
**Suggested fix**: `"Trend charts for the remaining ${notReadyCount} will appear after a few more days."` and `"Your first metric charts will appear after a few more days of use."`

---

## sample-data-banner.tsx

### Finding 1

**File**: `apps/app/src/components/analytics/sample-data-banner.tsx`
**Line**: 54-55
**Text**: `"This is what your dashboard will look like as your AI learns. Connect your tools and start using SlopWeaver to see your real metrics here."`
**Trope**: "Your AI learns" is vague AI mystification. Learning is treated as a passive background process rather than something concrete (processing messages, extracting patterns).
**Suggested fix**: `"This is what your dashboard will look like once SlopWeaver has processed your messages. Connect your tools to see your real metrics here."`

---

## summary-cards.tsx

No findings.
