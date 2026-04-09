# Audit: apps-app-src-utils-2

**Files inspected**: 8
**Findings**: 8

## Summary

This slice covers Gmail HTML parsing, inbox data/filter/query utilities, the knowledge-graph adapter, memory-file mapping, message comparison, and proactive-suggestions utilities. The dominant issues are a locally-defined `ParsedGmailEmail` interface that partially duplicates the contract email shape, `Record<string, unknown>` metadata parameters with downstream `as string` bracket casts, a no-op platform branch in `search-result.transformer.ts` (both branches of a conditional return the identical unnarrowed value), `Record<string, string>` for a CATEGORY_COLORS map that should be keyed by a category union, and an undocumented react-force-graph runtime mutation of `GraphLink.source` that forces a `typeof` guard in `mergeGraphData`.

---

## Findings

### Finding 1: Local `ParsedGmailEmail` interface partially duplicates contract email shape

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/googleGmailHtmlParser.ts:1-15`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `ParsedGmailEmail` defines `subject`, `from`, `to`, `date`, `htmlBody`, `textBody`, `attachments` locally. The `@slopweaver/contracts` email expanded-data schema defines a unified email message shape with the same fields (plus thread-level metadata). The local type is not derived from the contract type, so if the contract adds or renames fields (e.g. `bodyHtml` instead of `htmlBody`), the local type will silently diverge. The `attachments` field uses a locally-defined `{ filename: string; mimeType: string; size: number }` object rather than the contract attachment type.
- **Suggestion**: Import the contracts email message type (or its relevant sub-shape) and derive `ParsedGmailEmail` from it using `Pick` or an intersection. At minimum, type `attachments` with the contract attachment type so the compiler catches any field drift.
- **Evidence**: `export interface ParsedGmailEmail { subject: string; from: string; to: string[]; date: string; htmlBody: string; textBody: string; attachments: Array<{ filename: string; mimeType: string; size: number }>; }`

### Finding 2: `metadata: Record<string, unknown>` parameter with downstream `as string` casts

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/inbox-data.utils.ts:38,52`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `buildInboxMetadataDisplay` accepts `metadata: Record<string, unknown>` and then accesses fields via `metadata["snippet"] as string`, `metadata["threadCount"] as number`, etc. Each access requires a cast because the parameter type deliberately erases all field types. If the backend API response has a typed metadata shape (e.g. from the email expanded-data schema), the parameter type should reflect it so these casts become unnecessary.
- **Suggestion**: Import the typed metadata interface from `@slopweaver/contracts` (e.g. the email message metadata or a shared content metadata type) and use it as the parameter type. If metadata varies by content type, define a discriminated union and use a type guard at the call site.
- **Evidence**: `metadata["snippet"] as string` and `metadata["threadCount"] as number` throughout `buildInboxMetadataDisplay`.

### Finding 3: `Record<string, string>` used for CATEGORY_COLORS in knowledge-graph adapter

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/knowledge-graph-adapter.utils.ts:12-22`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `CATEGORY_COLORS: Record<string, string>` maps category names to hex colors. The categories (`"Person"`, `"Organization"`, `"Tool"`, etc.) are well-known constants. Using `Record<string, string>` means: (a) the compiler does not warn if a new category is added to the backend but is missing from the map; (b) any arbitrary string can be used as a key without a compile error. If a `KnowledgeCategory` type or enum is exported from `@slopweaver/contracts`, the map should use it as the key type.
- **Suggestion**: If `@slopweaver/contracts` exports a `KnowledgeCategory` union, change to `Record<KnowledgeCategory, string>` (or `Partial<Record<KnowledgeCategory, string>>` with a fallback). This will cause a compile error when a new category is added without a color.
- **Evidence**: `const CATEGORY_COLORS: Record<string, string> = { Person: "#...", Organization: "#...", ... }`

### Finding 4: Undocumented runtime mutation of `GraphLink.source` by react-force-graph forces a `typeof` guard

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/knowledge-graph-adapter.utils.ts:85-90`
- **Category**: type-cast
- **Impact**: low
- **Description**: `mergeGraphData` contains `typeof l.source === "object"` runtime checks to detect whether react-force-graph has mutated a string `source` field into an object reference. This mutation is a well-known react-force-graph behavior but is not documented here, and `GraphLink` is typed with `source: string`. The result is that the static type and runtime shape diverge silently — if the guard is ever removed, downstream string operations on the mutated `source` would break without a type error.
- **Suggestion**: Add a JSDoc comment on `GraphLink` (or in `mergeGraphData`) explaining the react-force-graph mutation behavior. Consider using a union type `source: string | GraphNode` to make the dual state explicit, and use a type predicate (`isResolvedLink`) to narrow before access.
- **Evidence**: `typeof l.source === "object"` checks in the merge loop without any comment explaining why a string field could be an object.

### Finding 5: No-op type narrowing in `search-result.transformer.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/search-result.transformer.ts:68`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `const platform = isMessagingPlatform(item.platform) ? item.platform : item.platform` — both branches of the conditional assign `item.platform` unchanged. The intent was presumably to narrow `platform` to `MessagingPlatform` in the true branch and keep it as the wider type in the false branch, but because both branches are identical, the resulting type is still the union of both. The `isMessagingPlatform` check has no effect on the assigned value and is dead code.
- **Suggestion**: Either (a) use the narrowed value only in the true branch — `const platform: MessagingPlatform | undefined = isMessagingPlatform(item.platform) ? item.platform : undefined` — or (b) remove the conditional if `item.platform` is always a `MessagingPlatform`. Audit what downstream consumers expect and fix the narrowing logic accordingly.
- **Evidence**: `const platform = isMessagingPlatform(item.platform) ? item.platform : item.platform;`

### Finding 6: `metadata?: Record<string, unknown>` passthrough in search result transformer

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/search-result.transformer.ts:80`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `transformSearchResultToCard` passes `metadata: item.metadata` where `item.metadata` is `Record<string, unknown> | undefined`. The resulting `SearchResultCard` also types `metadata` as `Record<string, unknown>`, so consumers must cast fields they access. If the search result metadata has a known shape (distinct per content type), a discriminated union keyed on `contentType` would eliminate downstream casts.
- **Suggestion**: Define per-content-type metadata interfaces and use a union type for `SearchResultCard.metadata`, discriminated by `contentType`. At minimum, document the expected shape for each content type via JSDoc so consumers know what to expect.
- **Evidence**: `metadata: item.metadata` where both sides are `Record<string, unknown> | undefined`.

### Finding 7: `relatedItems?: Record<string, number>` in proactive suggestions where keys could be `PlatformId`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/proactive-suggestions.utils.ts:18`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `ProactiveSuggestion.relatedItems` is typed as `Record<string, number>`. In practice these are counts keyed by platform or content type. If the keys are `PlatformId` values, using `Partial<Record<PlatformId, number>>` would bound the domain and prevent accidentally passing unrelated string keys.
- **Suggestion**: Confirm what keys appear in `relatedItems` at runtime. If they are all `PlatformId` values, change the type to `Partial<Record<PlatformId, number>>`. If they are mixed (platform IDs plus other keys), define an explicit union and use `Partial<Record<KnownRelatedItemKey, number>>`.
- **Evidence**: `relatedItems?: Record<string, number>;`

### Finding 8: `ToolCall` import from `@/lib/ai-chat-types` instead of `@/types/assistant` in message-comparison utils

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/utils/message-comparison.utils.ts:9`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `message-comparison.utils.ts` imports `ToolCall` from `@/lib/ai-chat-types` rather than from `@/types/assistant`. If `ai-chat-types.ts` re-exports or redeclares `ToolCall` independently, there are two definitions of the same type in the codebase, which will cause structural incompatibilities when comparing tool calls sourced from different parts of the app.
- **Suggestion**: Verify that `@/lib/ai-chat-types` and `@/types/assistant` define `ToolCall` identically (or that one re-exports the other). If they are the same, consolidate to a single import source (`@/types/assistant`). If they differ, add a comment explaining why two variants are needed.
- **Evidence**: `import type { ToolCall } from "@/lib/ai-chat-types";` (line 9) instead of importing from `@/types/assistant`.
