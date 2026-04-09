# Audit: apps-api-src-interface-16

**Findings count**: 5
**Summary**: Five type-safety issues in the sync stream and todo-extraction controllers. The most impactful is `mapSyncProgressEventRowToPayload` returning an anonymous `Record<string, unknown>` instead of a named interface. Two unsafe `as SyncPhase` casts access a bag-typed payload field via bracket notation. Two `req.body as { … }` casts in SSE endpoints skip Zod validation of raw request bodies.

---

## Finding 1

**File**: `apps/api/src/interface/http/sync/sync-stream-event-payload.utils.ts`
**Line**: 4
**Category**: Weak `Record<string, …>` type
**Impact**: Medium — callers receive an opaque `Record<string, unknown>` and cannot see which keys are guaranteed present without reading the implementation.

**Description**: `mapSyncProgressEventRowToPayload` is declared to return `Record<string, unknown>`. The function body actually populates a fixed set of keys (`phase`, `messagesIndexed`, `messagesSynced`, `errors`, etc.) that match the DB row schema. Using a named interface would make the contract explicit and allow callers to access fields without casts.

**Suggestion**: Define `SyncProgressEventPayload` as a named interface reflecting the exact keys populated, and change the return type accordingly.

**Evidence**:

```typescript
export const mapSyncProgressEventRowToPayload = (row: SyncProgressEventRow): Record<string, unknown> => {
```

---

## Finding 2

**File**: `apps/api/src/interface/http/sync/sync-stream.controller.ts`
**Line**: 98
**Category**: Type cast (`as`)
**Impact**: Low — the cast is redundant: `normalizedBody` is already narrowed to `object` by the preceding `typeof` check, so the `as Record<string, unknown>` is a no-op assertion rather than a narrowing.

**Description**: After `if (typeof normalizedBody !== "object" || normalizedBody === null) return;`, the code immediately writes `(normalizedBody as Record<string, unknown>)`. The outer narrowing already guarantees `object`; the cast adds no safety and masks the fact that arbitrary property access on `object` is not guaranteed.

**Suggestion**: Remove the cast; use a type guard function (`isRecord`) that returns `value is Record<string, unknown>` to narrow properly.

**Evidence**:

```typescript
(normalizedBody as Record<string, unknown>)["platform"];
```

---

## Finding 3

**File**: `apps/api/src/interface/http/sync/sync-stream.controller.ts`
**Line**: 401
**Category**: Type cast (`as`)
**Impact**: Medium — `event.payload["phase"]` reads from the `Record<string, unknown>` returned by `mapSyncProgressEventRowToPayload` and blindly casts to `SyncPhase` with no runtime check. A stale or unexpected payload value will silently propagate as a wrong enum value.

**Description**: The `SyncPhase` union is cast from an unvalidated bracket-access on a `Record<string, unknown>`. If the payload key is missing or holds an unrecognised string, the cast succeeds at compile time but the runtime value will not satisfy the union.

**Suggestion**: Use a type guard `isSyncPhase(value: unknown): value is SyncPhase` and handle the `false` branch explicitly.

**Evidence**:

```typescript
const phase = event.payload["phase"] as SyncPhase;
```

---

## Finding 4

**File**: `apps/api/src/interface/http/sync/sync-stream.controller.ts`
**Line**: 463
**Category**: Type cast (`as`)
**Impact**: Medium — same pattern as Finding 3 at a different call site. Both casts originate from the same root cause: `mapSyncProgressEventRowToPayload` returning `Record<string, unknown>`.

**Description**: Second occurrence of `as SyncPhase` on a bracket-access from an opaque payload map.

**Suggestion**: Fix the root cause (Finding 1) so the payload is strongly typed, eliminating both casts.

**Evidence**:

```typescript
const phase = event.payload["phase"] as SyncPhase;
```

---

## Finding 5

**File**: `apps/api/src/interface/http/todos/todo-extraction.controller.ts`
**Lines**: 396, 460
**Category**: Type cast (`as`)
**Impact**: High — two raw-body endpoints cast `req.body` to inline structural types without Zod validation. A missing or wrong-typed field will silently produce `undefined` at runtime.

**Description**: Lines 396 and 460 both pattern-match `req.body` directly via a TypeScript cast:

- `req.body as { contentId: string }` (line 396, SSE endpoint for extract-by-content)
- `req.body as { files: ...; textContent?: string }` (line 460, SSE upload endpoint)

These endpoints use `@Res()` for SSE streaming so ts-rest does not perform automatic body validation.

**Suggestion**: Parse both bodies through Zod schemas before using them, and return a typed 400 response on parse failure.

**Evidence**:

```typescript
const { contentId } = req.body as { contentId: string };
// ...
const { files, textContent } = req.body as { files: Express.Multer.File[]; textContent?: string };
```
