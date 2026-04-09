# Audit: apps/app/src/components — Slice 3 (ai-tool and analytics area)

**Files inspected**: 8
**Findings**: 6

## Summary

The ai-tool and analytics files have two distinct problem clusters: unsafe casts on external data boundaries (BroadcastChannel messages and opaque result objects) and a set of four parallel `Record<string, string>` constants whose keys form a well-defined closed set that should be typed as a union.

## Findings

### Finding 1: Unvalidated BroadcastChannel message cast

- **File**: `apps/app/src/components/ai-tool-action-button.tsx:55`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: high
- **Description**: `const data = event.data as { type?: string; platform?: string; success?: boolean } | undefined` assumes the message shape without runtime validation. BroadcastChannel messages come from other browsing contexts and could be any shape.
- **Suggestion**: Define a discriminated union for the message payload and use a type guard function (e.g. `isToolActionEvent(data): data is ToolActionEvent`) to validate before use. This eliminates the cast and prevents runtime errors if an unexpected message is received.
- **Evidence**: `event.data as { type?: string; platform?: string; success?: boolean } | undefined` at line 55.

### Finding 2: Cast on opaque `result` object to access `.action`

- **File**: `apps/app/src/components/ai-tool-action-button.tsx:28`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: medium
- **Description**: `(result as Record<string, unknown>)["action"]` casts an opaque result to access a property. If `result` is typed as `unknown`, a proper type guard should be used; if it has a known type, the property should be accessed directly.
- **Suggestion**: Use a type guard: `function hasAction(v: unknown): v is { action: string }` and call `if (hasAction(result)) { ... result.action ... }`.
- **Evidence**: `(result as Record<string, unknown>)["action"]` at line 28.

### Finding 3: `SHORT_NAMES` with `string` keys should use `PlatformId`

- **File**: `apps/app/src/components/ai-tool-activity.utils.ts:272`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `const SHORT_NAMES: Record<string, string>` maps platform IDs to display names. The keys are a closed set of known platform IDs plus `"workspace"`. Using `string` allows silent typos.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and type as `Partial<Record<PlatformId | 'workspace', string>>`. TypeScript will warn if a new platform is added to contracts but not to this map.
- **Evidence**: `const SHORT_NAMES: Record<string, string> = { gmail: ..., slack: ..., ... }` at line 272.

### Finding 4: `readinessState` typed as `string` instead of `ReadinessState`

- **File**: `apps/app/src/components/analytics/ai-headline-stats.tsx:82`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: The prop `readinessState: string | undefined` accepts any string. `ReadinessState` is a union type exported from `@slopweaver/contracts` covering the known states. Using `string` loses exhaustiveness checking in any switch or conditional.
- **Suggestion**: Import `ReadinessState` from `@slopweaver/contracts` and change the prop type to `ReadinessState | undefined`.
- **Evidence**: `readinessState: string | undefined` at line 82 in the component props.

### Finding 5: Four parallel `Record<string, string>` metric maps with closed key sets

- **File**: `apps/app/src/components/analytics/analytics-copy.utils.ts:8-57`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `METRIC_LABELS`, `METRIC_DESCRIPTIONS`, `METRIC_GOOD_DESCRIPTIONS`, and `METRIC_UNITS` are all `Record<string, string>` maps whose keys are the same closed set of metric ID strings. There is no compile-time guarantee that all four maps cover the same keys, or that a new metric added to one is added to all.
- **Suggestion**: Define `type MetricId = 'responseTime' | 'triageAccuracy' | ...` (matching the actual keys). Type all four constants as `Record<MetricId, string>`. The compiler will then flag any map that is missing an entry for a known metric.
- **Evidence**: Four separate `const X: Record<string, string> = { ... }` declarations starting at line 8.

### Finding 6: Unsafe `unknown` access pattern on analytics data without type guards

- **File**: `apps/app/src/components/analytics/ai-headline-stats.tsx`
- **Category**: `any` or unsafe `unknown` in production code
- **Impact**: low
- **Description**: Several analytics data fields from API responses are accessed without explicit type narrowing, relying on implicit inference from `any`-typed API clients. As `tsr.*` typing improves, these accesses should be validated.
- **Suggestion**: Ensure all data destructuring from `tsr.*` hooks uses `select: (d) => d.body` to get the typed body, and avoid property access via bracket notation on loosely-typed objects.
- **Evidence**: General pattern across the component; most impactful on the `readinessState` prop noted in Finding 4.
