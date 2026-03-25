# Tropes Audit — apps/app/src/components/billing (slice 4)

Files audited:

- transaction-item.tsx
- trial-banner-wrapper.tsx
- trial-banner.tsx
- usage-chart-content.tsx
- usage-chart-empty.tsx
- usage-chart-skeleton.tsx
- usage-chart.tsx

## Findings

### usage-chart-empty.tsx — line 14

**File**: `apps/app/src/components/billing/usage-chart-empty.tsx`
**Line**: 14
**Trope**: Filler copy / empty-state placeholder text that reads like a prompt output rather than a product voice.

```tsx
<p className="mt-1 text-muted-foreground text-sm">Start using actions to see usage trends here.</p>
```

**Problem**: "Start using actions to see usage trends here" is bland instructional filler. It restates the obvious (the chart is empty because there's no data) without adding value or personality. This phrasing is a hallmark of AI-generated placeholder text.

**Suggested fix**: Either remove the sub-line entirely and let the heading stand alone, or rewrite to be specific and direct, e.g. "Actions you take will appear here."

---

### usage-chart.tsx — line 49

**File**: `apps/app/src/components/billing/usage-chart.tsx`
**Line**: 49
**Trope**: Redundant subtitle that restates the heading in different words.

```tsx
<h3 className="font-semibold text-foreground">Usage Trends</h3>
<p className="text-muted-foreground text-sm">Usage over time</p>
```

**Problem**: "Usage Trends" and "Usage over time" say the same thing. The subtitle adds no information. This double-heading pattern is a common AI writing trope where a subtitle is added reflexively even when it contributes nothing.

**Suggested fix**: Drop the subtitle entirely. "Usage Trends" is sufficient on its own.

---

No other findings in this slice.
