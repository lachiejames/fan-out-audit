# Type-Safety Audit: apps-app-src-lib-1

Files audited:

- `apps/app/src/lib/ai-chat-controller.tsx`
- `apps/app/src/lib/ai-chat-controller.utils.ts`
- `apps/app/src/lib/ai-chat-runtime.ts`
- `apps/app/src/lib/ai-chat-store.ts`
- `apps/app/src/lib/ai-chat-types.ts`
- `apps/app/src/lib/analytics-events.ts`
- `apps/app/src/lib/api-client.ts`
- `apps/app/src/lib/app-updates/orchestrator.ts`

---

## Finding 1

**File:** `apps/app/src/lib/ai-chat-controller.tsx` (lines 286–309)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
Inside `deferToolApprovalToWorkItem`, `toolInput` is typed as `Record<string, unknown>` (from `tool.args ?? tool.input ?? {}`), and then two specific fields are extracted with `typeof toolInput["sourceContentId"] === "string"` guards. This pattern is sound at runtime, but the need for it indicates that `ToolCall.args` and `ToolCall.input` are typed too loosely.

Additionally, the error description in the `toast.error` call uses a nested cast chain: `(result.body as { message?: unknown }).message` to extract an error message. This is a less safe path than using `extractErrorMessage`.

**Suggestion:**

- Replace the nested cast chain for the error toast with `extractErrorMessage({ body: result.body, fallback: undefined })` from `@slopweaver/contracts`.
- Consider whether `ToolCall.args` could carry per-tool typed payloads via a discriminated union keyed by `ToolCall.name`, which would eliminate all bracket-access narrowing.

**Evidence:**

```typescript
typeof (result.body as { message?: unknown }).message === "string"
  ? String((result.body as { message?: unknown }).message)
  : undefined;
```

---

## Finding 2

**File:** `apps/app/src/lib/ai-chat-controller.utils.ts` (line 49)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`getToolResultRecord` casts `toolCall.result as Record<string, unknown>` after checking that the result is a non-null object. Since `toolCall.result` is typed as `unknown` (see Finding 3 below), the cast is the only way to access its fields. The function's purpose — normalizing an opaque result into a record — is well-defined, but the root cause is the loose type on `ToolCall.result`.

**Suggestion:**
See Finding 3: tighten `ToolCall.result` to `Record<string, unknown> | string | null | undefined`. With a tighter source type, `getToolResultRecord` can narrow without a cast.

**Evidence:**

```typescript
return toolCall.result as Record<string, unknown>;
```

---

## Finding 3

**File:** `apps/app/src/lib/ai-chat-types.ts` (line 24)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
`ToolCall.result` is typed as `unknown | undefined`. This is effectively `unknown` (since `unknown | T = unknown` for any `T`). Every consumer of `result` must cast or narrow it. Given that tool results in this system are always JSON-serializable objects, a tighter base type is appropriate.

**Suggestion:**
Change `result?: unknown` to `result?: Record<string, unknown> | string | null`. This covers all known result shapes (object results, error strings, null/absent) while eliminating the need for casts in `getToolResultRecord` and any other consumer.

**Evidence:**

```typescript
/** Result from tool execution */
result?: unknown | undefined;
```

---

## Finding 4

**File:** `apps/app/src/lib/ai-chat-types.ts` (lines 14–27)

**Category:** Custom types duplicating SDK/contract types

**Impact:** Medium

**Description:**
`ToolCall` defines its own `status` union (`"running" | "completed" | "error" | "pending" | "approval-requested" | "approval-responded" | "output-denied"`). The AI SDK (`ai` package) has its own tool call state type. The local `ToolCall` type was likely necessary to extend or adapt the SDK type for UI purposes, but if it diverges from what the SDK actually emits, UI state machines will break silently.

**Suggestion:**
Check whether the `ai` SDK exports a ToolCall or ToolInvocation type that could be extended via intersection rather than fully redeclared. Document explicitly which fields are added by SlopWeaver vs. which come from the SDK.

**Evidence:**

```typescript
export interface ToolCall {
  id: string;
  name: string;
  status: "running" | "completed" | "error" | "pending" | "approval-requested" | "approval-responded" | "output-denied";
  // ...
}
```

---

## Finding 5

**File:** `apps/app/src/lib/analytics-events.ts` (line 5)

**Category:** `Record<string, ...>` where stricter types would be safer

**Impact:** Low

**Description:**
`AnalyticsPayload = Record<string, unknown>` is a loose catch-all for analytics event properties. While analytics payloads are inherently flexible, using a more constrained base type would catch accidental nesting of non-serializable values (functions, class instances).

**Suggestion:**
Tighten to `Record<string, string | number | boolean | null | undefined | string[] | number[]>` or use a JSON-serializable recursive type. This prevents accidentally passing non-serializable values to analytics providers.

**Evidence:**

```typescript
type AnalyticsPayload = Record<string, unknown>;
```

---

## Finding 6

**File:** `apps/app/src/lib/api-client.ts` (line 602)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
`(await response.json()) as { csrfToken: string }` is used to extract the CSRF token from the bootstrap response. A Zod parse would validate the shape at runtime rather than silently accepting any JSON.

**Suggestion:**
Add a minimal Zod schema `z.object({ csrfToken: z.string() })` and use `.parse()` or `.safeParse()`. The CSRF token endpoint is a bootstrap call; runtime validation here provides early detection of server-side contract breaks.

**Evidence:**

```typescript
const data = (await response.json()) as { csrfToken: string };
```

---

## Finding 7

**File:** `apps/app/src/lib/api-client.ts` (line 951)

**Category:** Type cast (`as SomeType`) that could be avoided

**Impact:** Low

**Description:**
The 403 retry path casts the error body as `{ message?: string | string[] } | null` to extract a user-facing message for the forbidden toast. This is the same error-body cast pattern found throughout the hooks.

**Suggestion:**
Use `extractErrorMessage` from `@slopweaver/contracts` for consistency with the rest of the codebase.

**Evidence:**

```typescript
const body = retryResult.body as { message?: string | string[] } | null;
const msg = Array.isArray(body?.message) ? body.message[0] : body?.message;
```

---

## Finding 8

**File:** `apps/app/src/lib/app-updates/orchestrator.ts` (line 174)

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Medium

**Description:**
`cachedTauriUpdate: unknown = null` stores the `Update` object returned by the Tauri `check()` function. In `downloadAndInstallAppUpdate`, it is cast to an inline object type with a `downloadAndInstall` method. The cast must be kept manually in sync with the Tauri SDK's actual `Update` type.

**Suggestion:**
Import the `Update` type from `@tauri-apps/plugin-updater` and type `cachedTauriUpdate` as `Update | null`. Then `cachedTauriUpdate.downloadAndInstall(...)` is type-safe without a cast. This is a dynamic import context, but the type can still be imported without tree-shaking the plugin itself (use `import type`).

**Evidence:**

```typescript
let cachedTauriUpdate: unknown = null;

// ...later:
const update = cachedTauriUpdate as {
  downloadAndInstall: (onProgress: (...) => void) => Promise<void>;
} | null;
```

---

## Finding 9

**File:** `apps/app/src/lib/ai-chat-store.ts`

No type-safety issues found. The store uses explicit type parameters throughout, imports contract types (`VoiceProfileId`, `ConversationSourceAnchor`), and all Zustand state is fully typed via the `AIChatStoreState` interface. The `VoicePersistedState` schema uses `z.object()` with `.catch()` fallbacks for runtime resilience.
