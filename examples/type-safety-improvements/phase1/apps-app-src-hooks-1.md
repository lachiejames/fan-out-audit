# Type-Safety Audit: apps-app-src-hooks-1

**Files audited:**

- `apps/app/src/hooks/ai-chat.types.ts`
- `apps/app/src/hooks/billing/useBillingCommerce.ts`
- `apps/app/src/hooks/billing/useNativeIapProducts.ts`
- `apps/app/src/hooks/billing/usePurchaseUpdateListener.ts`
- `apps/app/src/hooks/chat-persistence.utils.ts`
- `apps/app/src/hooks/entities-data.utils.ts`
- `apps/app/src/hooks/expand-message.utils.ts`
- `apps/app/src/hooks/inbox-item-actions.utils.ts`

---

## Findings

### 1. `status?: string` too loose on `updateConversation` params — `ai-chat.types.ts` line 40

**Category:** Custom types duplicating or weaker than contract types

**Location:** `apps/app/src/hooks/ai-chat.types.ts:40`

**Description:**
The `updateConversation` method signature includes `updates: { title?: string; status?: string }`. The `status` field accepts any string, but conversations have a defined status union in the contracts (e.g., `"active" | "archived"`). Using `string` allows invalid status values to be passed without a compile-time error.

**Suggested fix:**

```typescript
import type { ConversationStatus } from "@slopweaver/contracts";

updateConversation: (id: string, updates: { title?: string; status?: ConversationStatus }) => Promise<Conversation>;
```

If `ConversationStatus` is not yet exported from contracts, it should be added there as the source of truth.

---

### 2. `as StoreActionPackInfo[]` cast on dynamic import result — `billing/useBillingCommerce.ts` line 46

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/billing/useBillingCommerce.ts:46`

**Description:**
`(await fetchMobileActionPackProducts()) as StoreActionPackInfo[]` casts the return value of a dynamic function. `StoreActionPackInfo` appears to be a locally-defined type that may duplicate or approximate contract types. If the function's return type is already declared, the cast is unnecessary. If the function returns `unknown`, a Zod schema validation would be safer than a blind cast.

**Suggested actions:**

1. Verify whether `StoreActionPackInfo` duplicates a type already in `@slopweaver/contracts`. If so, replace the local type with the contract type.
2. Add an explicit return type annotation to `fetchMobileActionPackProducts` so the cast is no longer needed at the call site.

---

### 3. `return response.body as T` generic escape hatch — `expand-message.utils.ts` line 64

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/expand-message.utils.ts:64`

**Description:**
`unwrapExpandResponse<T>` returns `response.body as T` when the status is 200. This is an unchecked generic cast — the caller provides `T` and the function blindly asserts the body matches it. If the ts-rest response already types the 200 body, the generic + cast pattern is unnecessary.

**Suggested fix:** If the function is called with a specific contract response type, replace the generic with a concrete return type derived from the contract:

```typescript
// Instead of generic cast, let the ts-rest response type flow through:
export function unwrapExpandResponse({
  response,
  fallbackMessage,
}: {
  response: SomeContractResponse; // ts-rest typed response
  fallbackMessage: string;
}): SomeContractResponse["body"] {
  if (response.status === 200) {
    return response.body;
  }
  // ...
}
```

If the function must remain generic (used for multiple different response types), document the caller's responsibility to ensure `T` matches the actual response body schema.

---

### 4. Local interface definitions for API shapes — `entities-data.utils.ts`

**Category:** Custom types duplicating or weaker than contract types

**Location:** `apps/app/src/hooks/entities-data.utils.ts`

**Description:**
`ApiEntityIdentity`, `ApiEntity`, and `ApiPendingLink` are locally defined interfaces that describe the shape of API responses. These types likely correspond to Zod-generated response types in `@slopweaver/contracts` (e.g., from entity/contacts endpoints). Maintaining local type definitions creates a risk of drift from the actual API response shape.

**Suggested fix:** Import the response types directly from `@slopweaver/contracts`:

```typescript
import type { EntityResponse, PendingLinkResponse } from "@slopweaver/contracts";
```

If those types are not yet exported from contracts, they should be. In the interim, add a comment linking these local types to the relevant contract schema so they can be found and replaced easily.

---

### 5. `platform: string` in `ApiEntityIdentity` — `entities-data.utils.ts` line 21

**Category:** Custom types weaker than contract types

**Location:** `apps/app/src/hooks/entities-data.utils.ts:21`

**Description:**
`ApiEntityIdentity.platform` is typed as `string`. The valid values are `PlatformId` values from `@slopweaver/contracts`. This is a consequence of finding #4 above — the local interface does not use the contract's `PlatformId` union.

**Suggested fix:** When finding #4 is resolved (importing from contracts), this automatically becomes `PlatformId`.

---

## Clean Files

- `billing/useNativeIapProducts.ts` — No issues found.
- `billing/usePurchaseUpdateListener.ts` — No issues found.
- `chat-persistence.utils.ts` — `return parts.map(...) as UIMessage["parts"]` is a necessary cast due to the mismatch between `UiPart[]` (contract type) and `UIMessage["parts"]` (AI SDK type). The cast is acceptable and well-documented. No issues.
- `inbox-item-actions.utils.ts` — Clean; properly uses `ActionTargetIdentifiers` from contracts and discriminated union narrowing for platform-specific identifiers. No issues.
