# Audit: apps-api-src-application-38

**Files inspected**: 8
**Findings**: 7

## Summary

The batch is generally well-typed with consistent use of `neverthrow`, named-object params, and contract-derived types. The main issues are: (1) a type cast on `actionRaw` in `context-rules.utils.ts` that could be replaced with a safe Zod parse; (2) ten `as string` casts in `export.service.ts` that are caused by a mismatch between Drizzle enum columns and the `z.string()`-typed export contract schemas; (3) a loose `Record<string, "urgent" | "high" | "medium" | "low">` map that should be keyed by `TodoPriority`; (4) `Record<string, boolean>` used internally where `FeatureFlags` from the contract is already the correct type; (5) a `TriageSessionForStats` local interface whose `status: string` field could be tightened; and (6) an `Object.assign` mutation pattern in `fetchConversationsWithMessages` that loses the structural type of `conversations` items.

---

## Findings

### Finding 1: Unsafe type cast on parsed triage action

- **File**: `apps/api/src/application/triage/utils/context-rules.utils.ts:64`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `actionRaw.toLowerCase() as TriageAction` blindly casts the regex-captured string to `TriageAction` without validation. If the regex ever matches an unexpected value (e.g. from a future regex change or a character-case edge case), the cast silently produces an invalid `TriageAction` that propagates downstream.
- **Suggestion**: Replace the cast with a Zod parse using the already-imported `triageActionSchema` (from `@slopweaver/contracts`). Use `triageActionSchema.safeParse(actionRaw.toLowerCase())` and treat a failure as an invalid-syntax rule, which the existing `onInvalidRule` callback already handles.
- **Evidence**:

```typescript
rules.push({
  action: actionRaw.toLowerCase() as TriageAction, // unsafe cast
  conditions,
  raw: line,
});
```

---

### Finding 2: `Record<string, boolean>` intermediate type when `FeatureFlags` already matches

- **File**: `apps/api/src/application/user/services/feature-flags.service.ts:67`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `flagsObject` is typed as `Record<string, boolean>`, but the service return type is `Result<FeatureFlags, UserError>` and `FeatureFlags` is already `ServerInferResponses<typeof contract.user.getFeatures, 200>["body"]["flags"]`, which resolves to `Record<string, boolean>`. The intermediate annotation is correct but redundant — a future change to the contract's `featureIdSchema` to a union literal type would not be caught here because the intermediate annotation escapes to the wider type.
- **Suggestion**: Remove the explicit `Record<string, boolean>` annotation and let TypeScript infer the type directly from the construction, so it can flow to the `return ok(flagsObject)` and be checked against `FeatureFlags` at the `ok(...)` call site.
- **Evidence**:

```typescript
const flagsObject: Record<string, boolean> = {};
for (const flag of flags) {
  flagsObject[flag.featureId] = flag.enabled;
}
return ok(flagsObject);
```

---

### Finding 3: `TODO_PRIORITY_MAP` keyed by `string` instead of the canonical `TodoPriority` type

- **File**: `apps/api/src/application/triage/utils/triage-mapping.utils.ts:74`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `TODO_PRIORITY_MAP` is declared as `Record<string, "urgent" | "high" | "medium" | "low">`. The key type should be the contract's `TodoPriority` (or at least `"high" | "medium" | "low"` matching the `ItemAction.todoData.priority` field), which would make exhaustiveness checkable at compile time. The current `string` key means any typo in a lookup key silently returns `undefined`, and the map is missing the `"urgent"` case (the value type allows it, but the map body only defines `high`, `medium`, `low`).
- **Suggestion**: Key the map by `NonNullable<ItemAction["todoData"]>["priority"]` (i.e. `"high" | "medium" | "low"`) and import / use `TodoPriority` from `@slopweaver/contracts` for the value type.
- **Evidence**:

```typescript
export const TODO_PRIORITY_MAP: Record<string, "urgent" | "high" | "medium" | "low"> = {
  high: "high",
  low: "low",
  medium: "medium",
};
```

---

### Finding 4: Ten `as string` casts in `export.service.ts` caused by enum-to-string widening mismatch

- **File**: `apps/api/src/application/user/services/export.service.ts:122–183`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Ten Drizzle-inferred enum columns (`contentLevel`, `type`, `role`, `status`, `category`, `priority`, `effort`) are cast with `as string` to satisfy `DataExportResponse`, whose contract schemas use `z.string()` for these fields. The casts are technically correct (enum values satisfy `string`), but they suppress TypeScript's ability to catch mismatches if either side changes. The comment at line 118 ("Cast to DataExportResponse to handle enum-to-string type widening") acknowledges this but treats the cast as unavoidable.
- **Suggestion**: The contract schemas (e.g. `exportedTodoSchema`) already accept `z.string()` for these columns, so the `as string` casts could be eliminated by using `satisfies` or by widening the Drizzle query's select to `text(...)` columns that TypeScript already infers as `string`. Alternatively, a typed mapping helper per entity (similar to the existing `mapSettingsToExport`) would centralise the widening in one place and make it explicit without scatter.
- **Evidence**:

```typescript
contents: contents.map((c) => ({
  ...c,
  contentLevel: c.contentLevel as string,   // Drizzle enum -> z.string()
  type: c.type as string,
  ...
})),
todos: todos.map((t) => ({
  ...t,
  category: t.category as string,
  effort: t.effort as string,
  priority: t.priority as string,
  status: t.status as string,
  ...
})),
```

---

### Finding 5: `Object.assign` in `fetchConversationsWithMessages` loses structural type

- **File**: `apps/api/src/application/user/services/export.service.ts:334`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `Object.assign(conv, { messages })` returns `typeof conv & { messages: ... }`. While TypeScript does infer the intersection, using object spread (`{ ...conv, messages }`) is idiomatic TypeScript, makes the resulting type transparent to readers and tooling, and avoids the runtime mutation of `conv`.
- **Suggestion**: Replace `Object.assign(conv, { messages })` with `{ ...conv, messages }`.
- **Evidence**:

```typescript
return Object.assign(conv, { messages });
```

---

### Finding 6: `TriageSessionForStats.status` typed as `string` instead of a narrower enum type

- **File**: `apps/api/src/application/triage/utils/triage-stats.utils.ts:32`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The local `TriageSessionForStats` interface declares `status: string`. The source table (`triageSessionsTable`) uses `triageSessionStatusEnum` which is a Drizzle pgEnum. Keeping `status: string` here means callers can pass any string, losing the discriminated-union safety that an enum-derived type would provide.
- **Suggestion**: Import `TriageSession` from `@/shared/db/tables/triage-sessions.table` and type `status` as `TriageSession["status"]`, or import the enum type directly.
- **Evidence**:

```typescript
interface TriageSessionForStats {
  // ...
  status: string; // should be TriageSession["status"]
}
```

---

### Finding 7: `platformMetadata` cast via `Object.assign` in `fetchIntegrations`

- **File**: `apps/api/src/application/user/services/export.service.ts:357–359`
- **Category**: type-cast
- **Impact**: low
- **Description**: `(i.platformMetadata ?? {}) as Record<string, unknown>` is a type assertion. The Drizzle column is already `jsonb`, which Drizzle infers as `unknown` (or a custom type). The cast papers over the mismatch rather than properly narrowing the type. Additionally, the `Object.assign` pattern mutates `i` in place, which is surprising for a `map` callback.
- **Suggestion**: Replace with object spread `{ ...i, platformMetadata: (i.platformMetadata ?? {}) as Record<string, unknown> }` to avoid mutation. If the column is declared with a typed JSON mode in Drizzle, define it as `Record<string, unknown>` in the table schema to remove the cast entirely.
- **Evidence**:

```typescript
return integrations.map((i) =>
  Object.assign(i, { platformMetadata: (i.platformMetadata ?? {}) as Record<string, unknown> }),
);
```
