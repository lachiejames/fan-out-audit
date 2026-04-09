# Audit: apps-api-src-application-41

**Files inspected**: 8
**Findings**: 6

## Summary

The batch is generally well-typed and uses `neverthrow` + discriminated unions correctly. The dominant issue is a **duplicate subscription status literal union** that is re-declared inline in at least 8 places across the codebase when `SubscriptionStatus` already exists in `@slopweaver/contracts`. Two secondary findings involve a type-cast inside `isCalendarActionType`, an open index signature on `AuditEventDetails` that erases type safety for known keys, an overly-wide `Record<string, unknown>` on `WorkItemError.details`, and a local `WorkItemSource.type` field typed as `string | null` where a narrower union would be more useful.

---

## Findings

### Finding 1: `subscriptionStatus` inline literal union duplicates `SubscriptionStatus` from contracts

- **File**: `apps/api/src/application/work-item-generation/services/work-item-generation.service.ts:140`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: Line 140 declares a local variable with the inline type `"active" | "cancelled" | "expired" | "past_due" | "paused" | "trialing" | undefined`. This exact six-member union already exists as `SubscriptionStatus` in `@slopweaver/contracts` (exported from `packages/contracts/src/contracts/billing/schemas.ts:18`). The same inline literal appears in at least 7 other files (`ai-work-item-generation.port.ts:43`, `ai-model.port.ts:32/42`, `chat-agent.factory.ts:54/214`, `prompt-generation.utils.ts:15`, `ai-model.utils.ts:44/101/123/219`, `billing.types.ts:11`), making `SubscriptionStatus` from contracts the canonical definition that is being ignored. Any future addition of a new billing status requires updating every inline occurrence individually, with no compiler error if one is missed.
- **Suggestion**: Replace the inline literal with `import type { SubscriptionStatus } from "@slopweaver/contracts"` and type the variable as `SubscriptionStatus | undefined`. The contracts package already exports this type; it just needs to be imported.
- **Evidence**:

  ```typescript
  // work-item-generation.service.ts:140 — current
  let subscriptionStatus: "active" | "cancelled" | "expired" | "past_due" | "paused" | "trialing" | undefined;

  // contracts/billing/schemas.ts:17-18 — already exists
  export const subscriptionStatusSchema = z.enum(["trialing", "active", "past_due", "cancelled", "expired", "paused"]);
  export type SubscriptionStatus = z.infer<typeof subscriptionStatusSchema>;
  ```

---

### Finding 2: Type cast inside `isCalendarActionType`

- **File**: `apps/api/src/application/work-items/services/work-items-calendar-metadata.utils.ts:26`
- **Category**: type-cast
- **Impact**: low
- **Description**: `isCalendarActionType` passes `type as CalendarActionType` to `Set.has()`. `Set<T>.has()` accepts `T`, so when the set is `Set<CalendarActionType>` the argument must be cast because the parameter is `string`. The cast is technically sound at runtime because `Set.has` does not mutate and the value never escapes, but the cast can be avoided entirely by widening the set type.
- **Suggestion**: Type the set as `Set<string>` (or `ReadonlySet<string>`) and keep the return type predicate. This removes the cast while keeping the narrowing behaviour.
- **Evidence**:

  ```typescript
  // current
  const calendarActionTypeSet = new Set<CalendarActionType>(["create_event", "delete_event", "update_event"]);
  export function isCalendarActionType(type: string): type is CalendarActionType {
    return calendarActionTypeSet.has(type as CalendarActionType); // cast required
  }

  // suggested
  const calendarActionTypeSet: ReadonlySet<string> = new Set<CalendarActionType>([
    "create_event",
    "delete_event",
    "update_event",
  ]);
  export function isCalendarActionType(type: string): type is CalendarActionType {
    return calendarActionTypeSet.has(type); // no cast needed
  }
  ```

---

### Finding 3: `AuditEventDetails` index signature erases known-key types

- **File**: `apps/api/src/application/work-items/services/work-item-events.utils.ts:11-17`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `AuditEventDetails` declares named optional fields (`type`, `content`, `error`, `undoAvailable`) alongside an index signature `[key: string]: unknown`. The index signature supersedes the named fields — TypeScript resolves any property access through the index signature, meaning code that reads `details.content` or `details.type` gets back `unknown` rather than `string | undefined`, throwing away the value of the named fields entirely. Callers must then narrow or cast even for the explicitly declared fields. Additionally, `WorkItemEvent["details"]` (from the DB table) is already `Record<string, unknown> | null`, so the interface is adding nominal structure on top of an already-opaque JSON column without providing real compile-time guarantees.
- **Suggestion**: Remove the index signature. If open-ended extra properties are needed, use `& Record<string, unknown>` as an intersection instead, or rely on the underlying `Record<string, unknown>` DB type and only declare `AuditEventDetails` as a strict named-key type without the catch-all.
- **Evidence**:
  ```typescript
  // current — index signature makes named fields effectively unknown
  export interface AuditEventDetails {
    type?: string;
    content?: string;
    error?: string;
    undoAvailable?: boolean;
    [key: string]: unknown; // defeats the named fields above
  }
  ```

---

### Finding 4: `WorkItemError.details` typed as `Record<string, unknown>` instead of known shapes

- **File**: `apps/api/src/application/work-items/errors/work-item.errors.ts:26`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `WorkItemError.details` is typed as `Record<string, unknown>`. Each factory in `WorkItemErrors` always produces a details object with a specific known shape — e.g., `notFound` always produces `{ workItemId: string }`, `invalidPlatform` always produces `{ invalidPlatforms: string[]; supportedPlatforms: string }`, and `executionFailed` always produces `{ reason: string; workItemId: string }`. Because the type is `Record<string, unknown>`, callers that switch on `error.type` and then read from `error.details` must narrow or cast every field. A discriminated union with typed `details` per error variant would make all reading call-sites safe without casts.
- **Suggestion**: Either convert `WorkItemError` into a discriminated union where each `type` variant carries a narrowly-typed `details` field, or at minimum use per-factory generic typing via function overloads. A lighter-weight alternative is to keep `WorkItemError` flat but add per-type interfaces exported alongside the error constants so consumers can assert the shape when they need it.
- **Evidence**:

  ```typescript
  // current
  export interface WorkItemError {
    type: WorkItemErrorType;
    message: string;
    details?: Record<string, unknown>;   // too wide
  }

  // e.g. notFound always produces { workItemId: string }
  notFound: (workItemId: string): WorkItemError => ({
    details: { workItemId },
    ...
  }),
  ```

---

### Finding 5: `WorkItemSource.type` typed as `string | null` instead of a known union

- **File**: `apps/api/src/application/work-item-generation/services/work-item-generation.service.ts:43`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `WorkItemSource.type` is `string | null`. At the single construction site within `generateWorkItem` (line 255), the only value ever assigned is the literal `"message"`. This suggests the field is intended to carry a small discriminated set of content-source types. Using `string | null` means consumers cannot safely switch on the value and the type provides no documentation about valid states.
- **Suggestion**: Introduce a `WorkItemSourceType` literal union (e.g. `"message" | "document" | "attachment"`) and type `WorkItemSource.type` as `WorkItemSourceType | null`. If the set is still evolving, even a union of currently-used literals narrows the surface and makes the intent explicit.
- **Evidence**:

  ```typescript
  export interface WorkItemSource {
    type: string | null;   // only ever "message" at the call site
    ...
  }

  const triggerSource: WorkItemSource = {
    type: "message",   // literal that could be the union member
    ...
  };
  ```

---

### Finding 6: `WorkItemMessage` and `WorkItemSource` are locally-defined shapes that parallel contracts types

- **File**: `apps/api/src/application/work-item-generation/services/work-item-generation.service.ts:31-55`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `WorkItemMessage` and `WorkItemSource` are defined locally in the service file and are not exported from contracts or a shared module. `WorkItemMessage` mirrors the shape of message fields already present in the content/inbox domain model, and `WorkItemSource` mirrors search-result response shapes. If these types are consumed by other files (currently `content.work-items.ts` imports `GeneratedWorkItem` from this service), the shapes will spread as informal contracts rather than canonical types. They are not yet causing active unsafety, but they are a vector for future drift.
- **Suggestion**: If `WorkItemMessage`/`WorkItemSource`/`GeneratedWorkItem` are used across more than one file, move them to a shared types file (e.g. `work-item-generation.types.ts`) or into `@slopweaver/contracts`. This is low-priority until a second consumer appears but worth noting for the next refactor pass.
- **Evidence**:
  ```typescript
  // Defined locally in the service — not in contracts or a shared types file
  export interface WorkItemMessage { ... }
  export interface WorkItemSource { ... }
  export interface GeneratedWorkItem { ... }
  ```
