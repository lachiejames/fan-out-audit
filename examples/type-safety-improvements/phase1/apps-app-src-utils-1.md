# Audit: apps-app-src-utils-1

**Files inspected**: 8
**Findings**: 8

## Summary

This slice covers AI SDK helpers, transparency/analytics transforms, behavioral fingerprint utilities, billing dashboard, and bulk-actions utils. The dominant issues are `Record<string, unknown>` used as a structural type for inbox data (with `[key: string]: unknown` index signatures), `as` casts inside AI SDK message processing, and a locally-defined `BehavioralFingerprint` interface that mirrors an API type without re-using the contracts definition.

---

## Findings

### Finding 1: `InboxQueryData` uses `[key: string]: unknown` index signature

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/bulk-actions.utils.ts:3-7`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `InboxQueryData.messages` is typed as `Array<{ id: string; unread: boolean; [key: string]: unknown }>` and `counts` is `{ messages: number; unread: number; [key: string]: unknown }`. These index signatures make the type extremely permissive — any field access returns `unknown`. This forces callers to cast or re-narrow fields that are already known.
- **Suggestion**: Import the actual inbox message and counts types from `@/hooks/useInboxData` (or `@slopweaver/contracts`) and use them directly. The `applyOptimisticBulkUpdate` function only needs the fields it accesses (`id`, `unread`, `messages`, `unread`, `total`) so a focused `Pick<InboxMessage, "id" | "unread">[]` would suffice.
- **Evidence**: `messages: Array<{ id: string; unread: boolean; [key: string]: unknown }>;`

### Finding 2: `sanitizeMessagesForChatRequest` casts `message as { parts?: unknown }`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/ai-sdk.ts:65-66`
- **Category**: type-cast
- **Impact**: low
- **Description**: `(message as { parts?: unknown }).parts` casts the AI SDK `UIMessage` type to access `.parts`. This suggests the `UIMessage` type from the `ai` package does not expose `parts` directly, or the import is from an older SDK version. The cast allows accessing an unverified field shape.
- **Suggestion**: Check the `ai` package's type definitions for `UIMessage`. In AI SDK v4+, `UIMessage` has a `parts` field. Import the correct version. If the SDK version genuinely lacks the `parts` field in the type, add a comment explaining the SDK version gap.
- **Evidence**: `const sanitizedParts = sanitizeMessagePartsForChatRequest({ parts: (message as { parts?: unknown }).parts }) as UIMessage["parts"];`

### Finding 3: Double cast `sanitizeMessagePartsForChatRequest` result to `UIMessage["parts"]`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/ai-sdk.ts:66`
- **Category**: type-cast
- **Impact**: low
- **Description**: The result of `sanitizeMessagePartsForChatRequest` (typed `MessagePart[]`) is cast to `UIMessage["parts"]`. This cast is needed because `MessagePart[]` and `UIMessage["parts"]` may not be identical in the SDK version in use, but it hides the mismatch rather than resolving it.
- **Suggestion**: Align the `MessagePart` type with `UIMessage["parts"]` by importing the SDK's native part types. If `MessagePart = UiPart` (as set in `assistant.ts`), the cast should not be needed — verify that `UiPart` is assignable to `UIMessage["parts"]`.
- **Evidence**: `as UIMessage["parts"]`

### Finding 4: `isToolPart` uses `(part as { type?: unknown }).type` inside a type predicate

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/ai-sdk.ts:95-96`
- **Category**: type-cast
- **Impact**: low
- **Description**: Inside `isToolPart`, `part` is `MessagePart` (a known type) but the function accesses `type` and `toolCallId` via `as { type?: unknown }` and `as { toolCallId?: unknown }` casts. If `MessagePart` / `UiPart` already has a `type` discriminant, the cast is unnecessary.
- **Suggestion**: Check whether `MessagePart` (`UiPart`) has a `type` discriminant. If so, use it directly. If `MessagePart` is a broad union that includes types without `type`, use a discriminant-aware type guard instead of the freehand cast.
- **Evidence**: `const type = (part as { type?: unknown }).type; const toolCallId = (part as { toolCallId?: unknown }).toolCallId;`

### Finding 5: Local `BehavioralFingerprint` interface duplicates the API type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/behavioral-fingerprint.utils.ts:62-81`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: A local `BehavioralFingerprint` interface is defined with 20+ fields. The file imports `BehavioralFingerprint as ApiBehavioralFingerprint` from `@slopweaver/contracts` and maps from it. The UI type differs primarily in scaled values (`confidence 0-1` vs `0-100`) and `responseTimeByHour` key type (`number` vs `string`). These differences are real — but the local type is not formally derived from the API type (no `Omit`/`Pick` relationship), which means if the API type gains new fields they won't appear in the local type.
- **Suggestion**: Document explicitly that the local type is a UI projection of the API type. Consider using `Omit<ApiBehavioralFingerprint, "confidence" | "responseTimeByHour" | ...> & { confidence: number; responseTimeByHour: Record<number, number>; ... }` to derive the type formally from the API shape.
- **Evidence**: `export interface BehavioralFingerprint { ... }` (lines 62–81), different from `ApiBehavioralFingerprint` imported from contracts.

### Finding 6: `toneByChannel: Record<string, string>` in `BehavioralFingerprint`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/behavioral-fingerprint.utils.ts:73`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `toneByChannel: Record<string, string>` stores tone values as raw strings, but the `ChannelToneRow.tone` type is `"formal" | "casual" | "balanced"`. The `buildChannelToneRows` function casts `tone as "formal" | "casual" | "balanced"`. Tightening the record type to `Record<string, "formal" | "casual" | "balanced">` would eliminate this downstream cast.
- **Suggestion**: Change `toneByChannel` to `Record<string, "formal" | "casual" | "balanced">` in the local interface and in the `mapApiResponseToFingerprint` function (which would need to validate/narrow the API value).
- **Evidence**: `toneByChannel: Record<string, string>;` and `tone: tone as "formal" | "casual" | "balanced"` in `buildChannelToneRows`.

### Finding 7: `Record<string, string>` for `AI_TRANSPARENCY_PERIOD` labels lookup

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/ai-transparency.utils.ts:15,42`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `LABELS: Record<string, string>` is used for `getTaskTypeLabel` and `getModelTierLabel`. Keys are well-known strings. If an entry is added to the backend task-type or model-tier enum, TypeScript won't flag a missing label because the Record accepts any string key. Exhaustiveness checking at compile time is lost.
- **Suggestion**: If the task-type and model-tier sets are exported from `@slopweaver/contracts`, use `Partial<Record<KnownTaskType, string>>` and keep the `?? fallback` for truly unknown keys. This will cause a compile error when a new known type is missing from the map.
- **Evidence**: `const LABELS: Record<string, string> = { ... }` in both functions.

### Finding 8: `sortTaskTypesByCost` and `calculateModelMix` accept `Record<string, number>`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/ai-transparency.utils.ts:120-144`
- **Category**: record-weakening
- **Impact**: low
- **Description**: Both `sortTaskTypesByCost` and `calculateModelMix` accept `Record<string, number>` parameters named `byTaskType` and `byModel`. If a stricter key type (`KnownTaskType` or `AIModelTier`) is available from contracts, using it would prevent accidentally passing unrelated string-keyed records.
- **Suggestion**: If the contracts package exports `AITaskType` or `AIModelTier` unions, use `Partial<Record<AITaskType, number>>` to provide a bounded domain. Keep `| string` only if truly necessary for unknown future keys.
- **Evidence**: `byTaskType: Record<string, number>` and `byModel: Record<string, number>`.
