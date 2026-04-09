# Audit: apps-api-src-application-28

**Files inspected**: 8
**Findings**: 4

## Summary

These files implement proactive notification logic (VIP triggers, morning brief parsing, priority scoring, waiting-on-you grouping, suggestion mapping) and push-notification error classification helpers. The code is generally clean and well-typed. Three findings involve local type aliases that duplicate types already exported by `@slopweaver/contracts`, and one finding involves a `Record<string, string>` that could use a stricter keyed type.

## Findings

### Finding 1: `ProactiveTriggerType` duplicates `TriggerType` from `@slopweaver/contracts`

- **File**: `apps/api/src/application/proactive/utils/suggestion-mapping.utils.ts:10`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The file declares a local `ProactiveTriggerType` union that is identical to the `TriggerType` type already exported from `@slopweaver/contracts` (`z.enum(["time_based", "event_based", "threshold_based", "pattern_based"])`). If the contract ever adds or renames a trigger type, the local alias will silently drift.
- **Suggestion**: Remove `ProactiveTriggerType` and import `TriggerType` from `@slopweaver/contracts`. Rename usages to `TriggerType` for consistency with the rest of the codebase.
- **Evidence**:

```typescript
// suggestion-mapping.utils.ts:10-11
export type ProactiveTriggerType = "time_based" | "event_based" | "threshold_based" | "pattern_based";
export type ProactiveUrgencyLevel = "critical" | "high" | "medium" | "low";

// packages/contracts/src/contracts/proactive/types.ts
export type TriggerType = z.infer<typeof triggerTypeSchema>; // same four values
export type UrgencyLevel = z.infer<typeof urgencyLevelSchema>; // same four values
```

### Finding 2: `ProactiveUrgencyLevel` duplicates `UrgencyLevel` from `@slopweaver/contracts`

- **File**: `apps/api/src/application/proactive/utils/suggestion-mapping.utils.ts:11`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Same issue as Finding 1 but for urgency levels. The local `ProactiveUrgencyLevel` union (`"critical" | "high" | "medium" | "low"`) is byte-for-byte equivalent to `UrgencyLevel` from contracts. `mapConfidenceToUrgency` returns this type, but it never returns `"critical"` — the `>= 90` branch returns `"high"`, leaving `"critical"` unreachable. This drift would be caught if the return type used the contract's Zod-validated enum.
- **Suggestion**: Import `UrgencyLevel` from `@slopweaver/contracts` and delete `ProactiveUrgencyLevel`. Update the `mapConfidenceToUrgency` signature to return `UrgencyLevel`.
- **Evidence**:

```typescript
// suggestion-mapping.utils.ts:11
export type ProactiveUrgencyLevel = "critical" | "high" | "medium" | "low";

// mapConfidenceToUrgency never returns "critical" despite it being in the union
export function mapConfidenceToUrgency({ confidence }: { confidence: number }): ProactiveUrgencyLevel {
  if (confidence >= 90) return "high"; // "critical" is dead code
  if (confidence >= 70) return "medium";
  return "low";
}
```

### Finding 3: `Record<string, string>` in `getPlatformLabel` should use a const-keyed mapped type

- **File**: `apps/api/src/application/proactive/services/vip-trigger.service.ts:311`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platformLabelMap` is typed as `Record<string, string>`, which means any arbitrary string key is accepted as a valid lookup. The method then throws on a missing key. Typing the object as `Partial<Record<PlatformId, string>>` (using the generated `PlatformId` union from `@slopweaver/contracts`) would make unregistered platform strings a compile-time error and expose the exhaustiveness gap (only 3 of the supported platforms are listed).
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and change the type annotation to `Partial<Record<PlatformId, string>>`. The runtime throw can stay as a safety net, but TypeScript will flag callers that pass unsupported platform strings.
- **Evidence**:

```typescript
// vip-trigger.service.ts:311-316
private getPlatformLabel({ platform }: { platform: string }): string {
  const platformLabelMap: Record<string, string> = {
    "google-gmail": "Gmail",
    linear: "Linear",
    slack: "Slack",
  };
  const label = platformLabelMap[platform];
  if (!label) {
    throw new Error(`Unhandled platform: ${platform}`);
  }
  return label;
}
```

### Finding 4: `{} as Record<string, number>` type cast in `aggregatePlatformCounts`

- **File**: `apps/api/src/application/proactive/utils/cross-platform-thread.utils.ts:41`
- **Category**: type-cast
- **Impact**: low
- **Description**: The `reduce` accumulator is seeded with `{} as Record<string, number>`. The cast is unnecessary because the explicit generic on `reduce` is enough to type the accumulator. Using `as` to cast an empty object literal is a minor smell — it bypasses the type system instead of letting inference work.
- **Suggestion**: Replace `{} as Record<string, number>` with a typed initial value by supplying the generic to `reduce`: `.reduce<Record<string, number>>((counts, m) => { ... }, {})`. This removes the cast while keeping the same runtime behaviour.
- **Evidence**:

```typescript
// cross-platform-thread.utils.ts:36-43
export function aggregatePlatformCounts({
  messages,
}: {
  messages: Array<{ platform: string }>;
}): Record<string, number> {
  return messages.reduce(
    (counts, m) => {
      counts[m.platform] = (counts[m.platform] ?? 0) + 1;
      return counts;
    },
    {} as Record<string, number>, // <-- unnecessary cast
  );
}
```
