# Type-Safety Audit: apps-app-src-hooks-2

**Files audited:**

- `apps/app/src/hooks/proposals.utils.ts`
- `apps/app/src/hooks/queue-data-transformers.ts`
- `apps/app/src/hooks/search-data.utils.ts`
- `apps/app/src/hooks/sync-stream.utils.ts`
- `apps/app/src/hooks/useAgendaKeyboardNavigation.ts`
- `apps/app/src/hooks/useAIChatData.ts`
- `apps/app/src/hooks/useAiCostTransparency.ts`
- `apps/app/src/hooks/useAIPageData.ts`

---

## Findings

### 1. `platform: string` and `source: string` in `ApiProposal` — `proposals.utils.ts` line 27

**Category:** Custom types weaker than contract types

**Location:** `apps/app/src/hooks/proposals.utils.ts:27`

**Description:**
`ApiProposal.source.platform` is typed as `string`. Valid values are `PlatformId` values from `@slopweaver/contracts`. Similarly, `source: string` in `ApiEntityIdentity` (seen in hooks-1) follows the same pattern. If `ApiProposal` corresponds to a contract response type, the local interface should either be replaced with or reference the contract type.

**Suggested fix:** Import the proposal response type from `@slopweaver/contracts` or tighten `source.platform` to `PlatformId`:

```typescript
import type { PlatformId } from "@slopweaver/contracts";

export interface ApiProposal {
  // ...
  source: {
    platform: PlatformId | "chat"; // "chat" maps to "manual" at transform time
    from: string;
    subject: string;
  };
}
```

---

### 2. `action.status as WorkItemStatus` cast — `queue-data-transformers.ts` line 293

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/queue-data-transformers.ts:293`

**Description:**
`const status = action.status as WorkItemStatus` casts a `string` field from the input interface to the contract's `WorkItemStatus` union. The `QueueWorkItemTransformInput.status` field is typed as `string` rather than `WorkItemStatus`, requiring a cast at usage.

**Suggested fix:** Tighten `QueueWorkItemTransformInput.status` to `WorkItemStatus`:

```typescript
import type { WorkItemStatus } from "@slopweaver/contracts";

export interface QueueWorkItemTransformInput {
  // ...
  status: WorkItemStatus;
  // ...
}
```

This removes the cast and ensures invalid status values are caught at the call site.

---

### 3. `type: string` and `platform: string` in `QueueWorkItemTransformInput` — `queue-data-transformers.ts` lines 22–23

**Category:** Custom types weaker than contract types

**Location:** `apps/app/src/hooks/queue-data-transformers.ts:22`

**Description:**
`type: string` and `platform: string` in `QueueWorkItemTransformInput` accept any string. `type` is validated at runtime via `isReviewableQueueType()`, and `platform` is used with `isEmailPlatform()` / `isCalendarPlatform()`. Typing these as `WorkItemType | string` (with a narrowing guard) or directly as `WorkItemType` and `PlatformId` would make the contract more explicit at the interface boundary.

**Note:** The existing `isReviewableQueueType` guard is well-implemented and compensates for the loose input type. Tightening the interface would reduce the guard's necessity.

---

### 4. `feedbackOriginalContent` extraction via `as Record<string, unknown>` — `queue-data-transformers.ts` line 310–313

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/queue-data-transformers.ts:310`

**Description:**

```typescript
feedbackOriginalContent: (action.preview as Record<string, unknown>)["feedbackOriginalContent"] as
  | string
  | null
  | undefined,
```

This double cast — first to `Record<string, unknown>`, then to `string | null | undefined` — is required because `WorkItemPreview` does not include `feedbackOriginalContent`. If this field exists in the API response, it should be added to `WorkItemPreview`. If it is a legacy/optional field, it should be typed as `feedbackOriginalContent?: string | null` directly in the interface.

**Suggested fix:**

```typescript
export interface WorkItemPreview {
  // existing fields...
  feedbackOriginalContent?: string | null;
}
```

Then the cast becomes `action.preview.feedbackOriginalContent ?? null`.

---

### 5. `Record<string, number>` for `buildPlatformCounts` and `buildStatusCounts` return types — `queue-data-transformers.ts` lines 349, 362

**Category:** `Record<string, ...>` where stricter types would be safer

**Location:** `apps/app/src/hooks/queue-data-transformers.ts:349,362`

**Description:**
`buildPlatformCounts` returns `Record<string, number>` where keys are `PlatformId` values, and `buildStatusCounts` returns `Record<string, number>` where keys are `WorkItemStatus` values.

**Suggested fix:**

```typescript
import type { PlatformId, WorkItemStatus } from "@slopweaver/contracts";

export function buildPlatformCounts(workItems: QueueItem[]): Partial<Record<PlatformId, number>> { ... }
export function buildStatusCounts(workItems: QueueItem[]): Partial<Record<WorkItemStatus, number>> { ... }
```

Using `Partial<Record<...>>` correctly models that not every platform/status will necessarily appear in the result.

---

### 6. `(body as Record<string, unknown>)` pattern in `parseSseErrorResponse` — `sync-stream.utils.ts` line 244

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/sync-stream.utils.ts:244`

**Description:**

```typescript
const body = errorBody as Record<string, unknown>;
const bodyMessage = body["message"];
```

`errorBody` is already narrowed to a non-null object by the `if (errorBody && typeof errorBody === "object")` guard above. The additional `as Record<string, unknown>` cast is technically a valid way to access properties, but a typed helper would be cleaner.

**Assessment:** This is a low-severity issue. The `errorBody && typeof errorBody === "object"` guard makes the cast safe. The code is acceptable as-is, but a `parseErrorBody` helper shared with other SSE parsers would eliminate the repeated pattern.

---

### 7. `updateConversation` positional-style call wrapping — `useAIChatData.ts` line 716

**Category:** Project rule violation (named params)

**Location:** `apps/app/src/hooks/useAIChatData.ts:716`

**Description:**

```typescript
updateConversation: async (id: string, updates: { title?: string; status?: string }): Promise<Conversation> =>
  updateConversationMutation.mutateAsync({ id, title: updates.title ?? "" }),
```

The `updateConversation` function in the returned object takes two positional parameters (`id` and `updates`). Per the project's mandatory named-params rule, this should use a single named-param object. Additionally, `status?: string` is too loose (see hooks-1 finding #1).

**Suggested fix:**

```typescript
updateConversation: async ({
  id,
  updates,
}: {
  id: string;
  updates: { title?: string; status?: ConversationStatus };
}): Promise<Conversation> => updateConversationMutation.mutateAsync({ id, title: updates.title ?? "" }),
```

---

### 8. `(p.data as { citations?: unknown }).citations` — `useAIChatData.ts` line 133

**Category:** Avoidable type casts (`as SomeType`)

**Location:** `apps/app/src/hooks/useAIChatData.ts:133`

**Description:**
In `extractCitationsFromParts`, `(p.data as { citations?: unknown }).citations` casts the data part's payload to access the `citations` field. The Vercel AI SDK `MessagePart` types don't expose a strongly typed `data` field for custom parts, making this cast necessary. However, a type guard would be cleaner:

**Suggested fix:**

```typescript
function hasCitationsField(v: unknown): v is { citations: unknown } {
  return typeof v === "object" && v !== null && "citations" in v;
}
const citations = hasCitationsField(p.data) ? p.data.citations : undefined;
```

---

### 9. `AiCostTransparencyData.byModel` and `byTaskType` as `Record<string, number>` — `useAiCostTransparency.ts` lines 21–24

**Category:** `Record<string, ...>` where stricter types would be safer

**Location:** `apps/app/src/hooks/useAiCostTransparency.ts:21`

**Description:**
`byTaskType: Record<string, number>` and `byModel: Record<string, number>` use open `string` keys. If the contracts define known AI model IDs (`AIModelId`) and task type strings, these could be more tightly typed. At minimum, using `Partial<Record<AIModelId, number>>` for `byModel` would catch invalid model IDs.

**Suggested fix:**

```typescript
import type { AIModelId } from "@slopweaver/contracts";

interface AiCostTransparencyData {
  byModel: Partial<Record<AIModelId, number>>;
  byTaskType: Record<string, number>; // keep as string if task types aren't enumerated in contracts
  // ...
}
```

---

## Clean Files

- `search-data.utils.ts` — Well-structured; uses `ServerInferRequest` for type derivation from the contract. `Record<string, unknown>` for `metadata` and `Record<string, number>` for date range lookup are acceptable here (metadata is genuinely unstructured, and the date lookup is a local constant). No significant issues.
- `useAgendaKeyboardNavigation.ts` — Clean; no type-safety issues.
- `useAIPageData.ts` — Clean; correctly imports `CoreProfileResponse`, `Knowledge`, `UpdateIdentityBody` from `@slopweaver/contracts`. The `addFact`/`updateFact`/`deleteFact` helpers use positional parameters, which violates the named-params rule, but this is a project-rule violation rather than a type-safety issue. The `useCoreProfileData` hook uses a manual `useQuery` with a documented reason — this is a legitimate exception, well-commented.
