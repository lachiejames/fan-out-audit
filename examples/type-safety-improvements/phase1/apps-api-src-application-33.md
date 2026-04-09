# Audit: apps-api-src-application-33

**Files inspected**: 8
**Findings**: 6

## Summary

The most significant issues are local type aliases (`FixtureWorkItemStatus`, `FixtureTodoStatus`) that duplicate already-exported Drizzle enum types, an unsafe `as keyof PatternsByType` cast that sidesteps a proper type guard, and a `Record<string, unknown>` update payload that loses column-level type safety. Two additional `as`-casts (`as VoiceProfileId`, `as NewContent["platformIdentifiers"]`) are avoidable with narrowing. The remaining files are clean.

## Findings

### Finding 1: `FixtureWorkItemStatus` and `FixtureTodoStatus` duplicate existing enum types

- **File**: `apps/api/src/application/test-fixtures/services/test-fixtures-entities.service.ts:26-44`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: Two local union types are defined that exactly mirror the Drizzle enum value arrays already exported from `apps/api/src/shared/db/tables/enums.ts` as `WorkItemStatus` and the inferred todo status type. Because they are maintained separately, any future addition to `workItemStatusEnum` or `todoStatusEnum` in the DB schema will silently not be reflected in the fixture helpers, and callers cannot interchange the fixture types with the schema types.
- **Suggestion**: Remove both local aliases and import the canonical types:
  ```typescript
  import type { WorkItemStatus } from "@/shared/db/tables/enums";
  import type { Todo } from "@/shared/db/tables/todo.table";
  type FixtureTodoStatus = Todo["status"]; // or (typeof todoStatusEnum.enumValues)[number]
  ```
  `WorkItemStatus` is already exported from `enums.ts`; `todoStatusEnum` exports its values the same way.
- **Evidence**:

```typescript
type FixtureWorkItemStatus =
  | "proposed"
  | "pending"
  | "scheduled"
  | "executing"
  | "executed"
  | "failed"
  | "expired"
  | "discarded"
  | "undone";
type FixtureTodoStatus =
  | "proposed"
  | "accepted"
  | "pending"
  | "in_progress"
  | "completed"
  | "snoozed"
  | "rejected"
  | "cancelled";
```

### Finding 2: Unsafe `as keyof PatternsByType` cast instead of a type guard

- **File**: `apps/api/src/application/system/context-status/utils/context-status-calculations.utils.ts:144`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `stat.eventType` is typed as `string` (from the DB query result). It is cast with `as keyof PatternsByType` before the `in` check, meaning TypeScript sees the narrowing as already done and cannot detect if the `in` guard is ever removed or reordered. The cast also suppresses any future compiler error if `EventTypeStat.eventType` is widened further. The `in` guard already does the correct runtime check; the cast is both redundant and misleading.
- **Suggestion**: Remove the cast and narrow via the `in` check, assigning to a typed variable only after confirmation:
  ```typescript
  const key = stat.eventType;
  if (key in patterns) {
    patterns[key as keyof PatternsByType] = stat.count;
  }
  ```
  Or define a type guard:
  ```typescript
  function isPatternKey(k: string): k is keyof PatternsByType {
    return k in createEmptyPatternsByType();
  }
  if (isPatternKey(stat.eventType)) {
    patterns[stat.eventType] = stat.count;
  }
  ```
- **Evidence**:

```typescript
const eventType = stat.eventType as keyof PatternsByType;
if (eventType in patterns) {
  patterns[eventType] = stat.count;
}
```

### Finding 3: `buildCategoryStats` returns `Record<string, number>` where a stricter type is possible

- **File**: `apps/api/src/application/system/context-status/utils/context-status-calculations.utils.ts:169`
- **Category**: record-weakening
- **Impact**: low
- **Description**: The function accumulates counts keyed by `eventType`, which in practice is always a member of `keyof PatternsByType`. Returning `Record<string, number>` loses that constraint at the call site. The service passes this directly into the contract response `stats.byCategory`, so inaccurate keys would not be caught.
- **Suggestion**: Tighten the return type to `Partial<Record<keyof PatternsByType, number>>`. This aligns with the companion `PatternsByType` interface and ensures no arbitrary string key is silently accepted:
  ```typescript
  export function buildCategoryStats({
    stats,
  }: {
    stats: EventTypeStat[];
  }): Partial<Record<keyof PatternsByType, number>> {
    const byCategory: Partial<Record<keyof PatternsByType, number>> = {};
    for (const stat of stats) {
      if (isPatternKey(stat.eventType)) {
        byCategory[stat.eventType] = stat.count;
      }
    }
    return byCategory;
  }
  ```
- **Evidence**:

```typescript
export function buildCategoryStats({ stats }: { stats: EventTypeStat[] }): Record<string, number> {
  const byCategory: Record<string, number> = {};
  for (const stat of stats) {
    byCategory[stat.eventType] = stat.count;
  }
  return byCategory;
}
```

### Finding 4: `updateMessage` builds a `Record<string, unknown>` update payload losing column types

- **File**: `apps/api/src/application/test-fixtures/services/test-fixtures-messages.service.ts:241`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `updateMessage` method builds an update payload typed as `Record<string, unknown>` and passes it directly to `db.update(contentsTable).set(payload)`. Drizzle's `.set()` accepts the table's column-keyed update type, so using `Record<string, unknown>` completely bypasses column-level type safety — a typo in the key name or a wrong value type will not be caught at compile time.
- **Suggestion**: Use a typed partial that matches the Drizzle update shape:
  ```typescript
  import type { NewContent } from "@/shared/db/tables/contents.table";
  const payload: Partial<Pick<NewContent, "isArchived" | "isRead" | "snoozedUntil">> = {};
  ```
  Drizzle's `$inferInsert` or a manually written `Partial<{...}>` with the exact column names will restore full type checking.
- **Evidence**:

```typescript
const payload: Record<string, unknown> = {};
if (updates.isArchived !== undefined) {
  payload["isArchived"] = updates.isArchived;
}
// ...
this.database.db.update(contentsTable).set(payload);
```

### Finding 5: `as VoiceProfileId` cast without narrowing

- **File**: `apps/api/src/application/test-fixtures/services/test-fixtures-voice.service.ts:312`
- **Category**: type-cast
- **Impact**: low
- **Description**: `input.voiceProfileId` is declared as `string` in the method signature. It is cast to `VoiceProfileId` with a bare `as` without any runtime validation. `VoiceProfileId` is a `z.enum` type (`"mark_convoai"` only) — if an invalid string is passed (e.g. from a test) the cache will be seeded with a bad key silently.
- **Suggestion**: Narrow the input type at the function signature level (accept `VoiceProfileId` directly) or validate at the boundary using the exported schema:
  ```typescript
  import { voiceProfileIdSchema } from "@slopweaver/contracts";
  const parsed = voiceProfileIdSchema.safeParse(input.voiceProfileId);
  if (!parsed.success) return err(TestFixturesErrors.databaseError("Invalid voiceProfileId"));
  const typedVoiceProfileId = parsed.data;
  ```
  Or simply change the parameter type to `VoiceProfileId` and let callers provide the correct type.
- **Evidence**:

```typescript
voiceProfileId: string;
// ...
const typedVoiceProfileId = input.voiceProfileId as VoiceProfileId;
```

### Finding 6: `as NewContent["platformIdentifiers"]` cast on an already-compatible type

- **File**: `apps/api/src/application/test-fixtures/services/test-fixtures-messages.service.ts:66`
- **Category**: type-cast
- **Impact**: low
- **Description**: `input.platformIdentifiers` is typed as `Record<string, unknown> | null | undefined`. After the `?? null` coalesce, the value is `Record<string, unknown> | null`. This is cast to `NewContent["platformIdentifiers"]` with `as`. If `NewContent["platformIdentifiers"]` is a JSON column type (e.g. `PlatformIdentifiers | null`), the cast hides a real mismatch — `Record<string, unknown>` is structurally incompatible with a narrower JSON schema type and the cast will suppress that error.
- **Suggestion**: Accept `NewContent["platformIdentifiers"]` directly in the input type to push the type responsibility to callers, eliminating the need for a cast entirely:
  ```typescript
  platformIdentifiers?: NewContent["platformIdentifiers"] | undefined;
  ```
  This is a test-fixtures service, so callers are test code and can easily supply the right type.
- **Evidence**:

```typescript
platformIdentifiers?: Record<string, unknown> | null | undefined;
// ...
platformIdentifiers: (input.platformIdentifiers ?? null) as NewContent["platformIdentifiers"],
```
