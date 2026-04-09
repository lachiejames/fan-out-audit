# Type-Safety Audit: apps-api-src-integrations-10

**Batch 71** — `apps/api/src/integrations/core/utils/` (sync-settings, thread-aggregation, token-refresh, transcript-chunking), `integrations/errors/`, `integrations/platforms/asana/` (types, errors, provider)

**Files inspected**: 8
**Findings**: 7

## Summary

Key issues: Two `unknown`-cast patterns in `sync-settings.utils.ts` where Zod validation would be cleaner. `AsanaPlatformMetadata` uses `extends Record<string, unknown>` index signature weakening. Asana client provider has two unavoidable JSON-boundary casts. Other files are clean.

## Findings

### Finding 1: Repeated unknown casts in sync-settings.utils.ts

- **File**: `src/integrations/core/utils/sync-settings.utils.ts:11`
- **Category**: type-cast
- **Impact**: low
- **Description**: `syncSettings` is typed `unknown`; two separate inline casts (`as { messageLimit?: unknown }` and `as { syncProfile?: unknown }`) are used to access fields. These casts bypass structural validation.
- **Suggestion**: Define a Zod schema for sync settings covering both fields; parse once at the call site to eliminate the per-field casts.
- **Evidence**:

```typescript
const limit = (syncSettings as { messageLimit?: unknown }).messageLimit;
// ...
const profile = (syncSettings as { syncProfile?: unknown }).syncProfile;
```

---

### Finding 2: AsanaPlatformMetadata extends Record<string, unknown>

- **File**: `src/integrations/platforms/asana/asana.types.ts:197`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `AsanaPlatformMetadata extends Record<string, unknown>` adds an index signature that weakens all the typed fields below. Any unknown key access compiles without error.
- **Suggestion**: Remove the `Record<string, unknown>` base. If storing to a `Json` DB column, cast only at the database boundary (`metadata as unknown as Json`).
- **Evidence**:

```typescript
export interface AsanaPlatformMetadata extends Record<string, unknown> {
  platform: "asana";
  workspaceId: string;
  workspaceName: string;
  ...
}
```

---

### Finding 3: customFieldsMap typed Record<string, unknown>

- **File**: `src/integrations/platforms/asana/asana.types.ts:220`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `customFieldsMap: Record<string, unknown>` uses an overly wide value type. The actual values have a known structure (Asana custom field objects).
- **Suggestion**: Define an `AsanaCustomField` interface matching the Asana SDK type and use `Record<string, AsanaCustomField>`.
- **Evidence**:

```typescript
customFieldsMap: Record<string, unknown>;
```

---

### Finding 4: 204 response cast {} as T in asana-client.provider.ts

- **File**: `src/integrations/platforms/asana/providers/asana-client.provider.ts:168`
- **Category**: type-cast
- **Impact**: low
- **Description**: `return okResult({} as T)` casts an empty object to the generic type `T` for 204 No Content responses. Structurally unsound.
- **Suggestion**: Add a unit type for 204 responses (`type EmptyResponse = Record<string, never>`) and document why the cast is acceptable at this HTTP boundary.
- **Evidence**:

```typescript
if (response.status === 204) {
  return okResult({} as T);
}
```

---

### Finding 5: JSON parse cast (await response.json()) as T

- **File**: `src/integrations/platforms/asana/providers/asana-client.provider.ts:172`
- **Category**: type-cast
- **Impact**: low
- **Description**: `(await response.json()) as T` is an unavoidable cast at the JSON deserialization boundary since `fetch.json()` returns `any` in older TS libs. Acceptable at the HTTP client layer.
- **Suggestion**: Consider adding an optional Zod schema parameter to `request<T>()` so callers can validate the parsed shape rather than relying on the cast alone.
- **Evidence**:

```typescript
const data = (await response.json()) as T;
return okResult(data);
```

---

### Finding 6: thread-aggregation, token-refresh, transcript-chunking utilities

- **File**: `src/integrations/core/utils/thread-aggregation.utils.ts`, `token-refresh.utils.ts`, `transcript-chunking.utils.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: All three files are clean — well-typed pure functions using explicit param/return types. No unsafe casts, no `any`, no `Record` weakening.
- **Suggestion**: No changes needed.

---

### Finding 7: integration.errors.ts discriminated union

- **File**: `src/integrations/errors/integration.errors.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Clean discriminated union error factory using the `createPlatformErrors` pattern. No issues.
- **Suggestion**: No changes needed.
