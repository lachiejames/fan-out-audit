# Type-Safety Audit: apps-app-src-hooks-6

Files audited:

- `apps/app/src/hooks/useKnowledgeGraphData.ts`
- `apps/app/src/hooks/useKnowledgeSourceArchive.ts`
- `apps/app/src/hooks/useKnowledgeSources.ts`
- `apps/app/src/hooks/useMemoryFiles.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useKnowledgeGraphData.ts` (lines 83–84)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`body.edges` and `body.nodes` are cast to `KnowledgeGraphEdge[]` and `KnowledgeGraphNode[]` respectively. This occurs because the ts-rest contract response body is typed more loosely than the actual shape. The casts bypass compile-time field validation.

**Suggestion:**
Ensure the ts-rest contract for this endpoint declares the 200 response body with `edges: KnowledgeGraphEdge[]` and `nodes: KnowledgeGraphNode[]` explicitly. With a well-typed contract, `select: (d) => d.body` in the `useQuery` call propagates the exact type without needing casts.

**Evidence:**

```typescript
body.edges as KnowledgeGraphEdge[];
body.nodes as KnowledgeGraphNode[];
```

---

## Finding 2

**File:** `apps/app/src/hooks/useKnowledgeSourceArchive.ts` (lines 47, 53–54, 163)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
Three separate casts target error response bodies:

- Line 47: `response.body as Record<string, unknown>` for a paywall error body
- Lines 53–54 and 163: `response.body as { message?: string | string[] }` for general error responses

All three represent the same underlying pattern: extracting an error message from a non-200 ts-rest response body.

**Suggestion:**
Use `extractErrorMessage` from `@slopweaver/contracts` consistently. This function already handles the standard error body shape safely and eliminates all three casts.

**Evidence:**

```typescript
response.body as Record<string, unknown>; // line 47
response.body as { message?: string | string[] }; // lines 53-54
response.body as { message?: string | string[] }; // line 163
```

---

## Finding 3

**File:** `apps/app/src/hooks/useKnowledgeSources.ts` (lines 267, 328, 340, 385, 431)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Medium

**Description:**
Five `response.body as { message?: string[] }` casts appear across different mutation functions. This is the most repeated error-body cast pattern in the hooks directory. Each cast is fragile in the same way: if the backend changes to return `message: string` (non-array), the cast won't fail but the downstream `.join()` or array access will return unexpected results.

**Suggestion:**
Replace all five with `extractErrorMessage` from `@slopweaver/contracts`, which already handles both `string` and `string[]` variants of the `message` field in the standard error schema.

**Evidence:**

```typescript
response.body as { message?: string[] }; // 5 occurrences across mutation functions
```

---

## Finding 4

**File:** `apps/app/src/hooks/useKnowledgeSources.ts`

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`(await res.json()) as KnowledgeSourceUploadResponse` is used for a raw fetch call (the multipart file upload path). Since this is a raw `fetch` (not ts-rest), there is no schema validation on the response.

**Suggestion:**
Parse the response with a Zod schema derived from the contract (`KnowledgeSourceUploadResponse` should have a corresponding Zod schema in `@slopweaver/contracts`). Use `.parse()` or `.safeParse()` rather than a type cast.

**Evidence:**

```typescript
(await res.json()) as KnowledgeSourceUploadResponse;
```

---

## Finding 5

**File:** `apps/app/src/hooks/useKnowledgeSources.ts`

**Category:** Custom types duplicating SDK/contract types

**Impact:** Low

**Description:**
`PaywallSentinelError` extends `Error` and adds a non-standard `__isPaywall?: boolean` discriminator property. This is a custom pattern for detecting paywall errors across catch blocks. The codebase already has `isPaywallStatus` and related utilities in `@/lib/paywall-event`; using a custom error class with a boolean discriminator is a duplicate mechanism.

**Suggestion:**
Replace `PaywallSentinelError` with the existing paywall detection utilities (`isPaywallStatus`, `dispatchPaywallEvent`) from `@/lib/paywall-event`. The `__isPaywall` sentinel pattern makes it harder for callers to determine the error type without accessing a non-standard property.

**Evidence:**

```typescript
class PaywallSentinelError extends Error {
  __isPaywall?: boolean;
  ...
}
```

---

## Finding 6

**File:** `apps/app/src/hooks/useMemoryFiles.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** High

**Description:**
Three hooks — `useWriteMemoryFile`, `useExportMemoryFiles`, and `useClearSessionMemory` — expose their `mutateAsync` return types as `ReturnType<typeof useMutation<unknown, Error, ..., unknown>>`. The `unknown` generic parameters propagate into the public API, making the mutation result type opaque to callers.

**Suggestion:**
Derive explicit return types from the contract response schemas. For `useWriteMemoryFile`, the 200 body is known from the ts-rest contract; use `Awaited<ReturnType<typeof tsr.memory.writeFile.mutate>>['body']` (when status 200) as the resolved type.

**Evidence:**

```typescript
ReturnType<typeof useMutation<unknown, Error, { path: string; content: string }, unknown>>;
```

---

## Finding 7

**File:** `apps/app/src/hooks/useMemoryFiles.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Low

**Description:**
The same `internal?: unknown` escape-hatch field present in `useEntitiesData.ts` appears here as well. This is a second occurrence of the same structural issue.

**Suggestion:**
Either type `internal` with its actual shape or remove the field if it is unused.

**Evidence:**

```typescript
internal?: unknown
```
