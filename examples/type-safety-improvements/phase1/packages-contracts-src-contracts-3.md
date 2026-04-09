# Audit: packages-contracts-src-contracts-3

**Files inspected**: 8
**Findings**: 9

## Summary

This slice covers chat, content, and context schemas — the most complex area of the contracts. Key issues: multiple `z.unknown()` fields in chat tool parts, widespread `z.record(z.string(), z.unknown())` for metadata without discriminated alternatives, `.loose()` on nearly every chat-stream schema, and a weak `citationPlatformSchema` (bare `z.string()`). The content schemas have strong platform-specific sub-schemas but the base `metadata` field is still fully opaque.

## Findings

### Finding 1: `uiPartToolSchema` uses `z.unknown()` for `input`, `output`, and `providerMetadata`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/chat/schemas.ts:138-140`
- **Category**: any-usage
- **Impact**: high
- **Description**: The tool call part schema has three `z.unknown()` fields: `input`, `output`, and `providerMetadata`. These flow into frontend rendering and backend tool execution. The lack of typing means the frontend cannot safely render tool results without additional runtime checks.
- **Suggestion**: At minimum, define a base tool input/output interface with `{ [key: string]: unknown }` and bind it as `z.ZodType<BaseToolInput>`. For known tools (from `AgentToolMetadata`), consider discriminated sub-schemas keyed by `toolName`. `providerMetadata` is genuinely opaque (AI SDK internals) and `z.unknown()` is acceptable there.
- **Evidence**: `input: z.unknown().optional(), output: z.unknown().optional(), providerMetadata: z.unknown().optional(),` at lines 138-140.

### Finding 2: `citationSchema.metadata` is `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/chat/schemas.ts:57`
- **Category**: any-usage
- **Impact**: medium
- **Description**: Citation metadata is an opaque record. The frontend uses this to render citation previews but has no typed contract for what keys are present.
- **Suggestion**: Define a `CitationMetadata` interface with optional known fields (`pageNumber?: number`, `sectionTitle?: string`, etc.) and use it as `z.ZodType<CitationMetadata>`. Unknown extras can be captured with `[key: string]: unknown`.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()).default({}),` at line 57.

### Finding 3: `citationPlatformSchema` is bare `z.string()` rather than referencing `PlatformId`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/chat/schemas.ts:39`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The comment says "String for 100+ integration scalability (aligned with entityPlatformSchema pattern)". While intentional, the `platform` field on a citation goes through rendering logic in the frontend that pattern-matches on platform IDs. A typo here produces a citation with no icon/label.
- **Suggestion**: Use `z.string()` but add a runtime validation hint via `.refine(s => s.length > 0)` and document the known values. Alternatively, keep `z.string()` but add a type brand: `type CitationPlatform = PlatformId | (string & {})` to allow extensions while still surfacing known values in autocomplete.
- **Evidence**: `export const citationPlatformSchema = z.string();` at line 39.

### Finding 4: `.loose()` on every `uiPart*` schema and `chatStreamRequestSchema`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/chat/schemas.ts:78-62` and `chat-stream.schema.ts:53`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Every UI part sub-schema and the stream request itself uses `.loose()` (passthrough). While this is intentional for forward-compat with new AI SDK message types, it means extra unknown fields are silently accepted and re-serialized, which can mask bugs where a field name was changed or a part type was incorrectly constructed.
- **Suggestion**: The forward-compat rationale is sound. Document explicitly which schemas use `.loose()` and why. Consider adding a stricter "validate only known fields" variant used in tests, to catch accidental unknown keys during integration testing.
- **Evidence**: `.loose()` at the end of `uiPartTextSchema`, `uiPartReasoningSchema`, `uiPartFileSchema`, etc.

### Finding 5: `baseContentSchema.metadata` is `z.record(z.string(), z.unknown())` — opaque base field

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/content/schemas.ts:147`
- **Category**: any-usage
- **Impact**: high
- **Description**: The base `metadata` field on `Content` is typed as `Record<string, unknown>`. The file defines 12+ named platform-specific metadata schemas below (e.g., `slackContentMetadataSchema`, `gmailContentMetadataSchema`) but the base schema still uses the opaque type. The typed sub-schemas require manual type guards (`isSlackContent`).
- **Suggestion**: This is a known architectural pattern (the file even exports type guards). The immediate improvement is to ensure all platform metadata schemas that use `.loose()` (like `githubContentMetadataSchema`) document what the loose fields represent, and add a comment linking the base `metadata` field to the type-guard pattern.
- **Evidence**: `metadata: z.record(z.string(), z.unknown()),` at line 147.

### Finding 6: `asanaContentMetadataSchema.customFields` is `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/content/schemas.ts:399`
- **Category**: any-usage
- **Impact**: low
- **Description**: `customFields` inside the nested `asana` object is fully opaque. Asana custom fields have a known shape (type, value, display_value). An opaque record makes it impossible to render custom fields in the UI without ad-hoc null checks.
- **Suggestion**: Define an `AsanaCustomField` interface with at minimum `{ name: string; type: string; display_value: string | null; [key: string]: unknown }` and use it as the record value type.
- **Evidence**: `customFields: z.record(z.string(), z.unknown()).optional(),` at line 399.

### Finding 7: `mondayContentMetadataSchema.columnValues` is `z.record(z.string(), z.unknown())`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/content/schemas.ts:369`
- **Category**: any-usage
- **Impact**: low
- **Description**: Monday.com column values are opaque. Monday has a rich column type system; without typed column values the UI cannot render them.
- **Suggestion**: Similar to Finding 6 — define a minimal `MondayColumnValue` interface or leave as unknown but add JSDoc documenting Monday's column value shapes.
- **Evidence**: `columnValues: z.record(z.string(), z.unknown()).optional(),` at line 369.

### Finding 8: `ChatMessage` and `VoiceTranscript` type helpers in `content/types.ts` use intersection with string literal, not discriminated union

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/content/types.ts:27-30`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `ChatMessage = Content & { type: "integration_message" }` is an intersection — TypeScript treats this as a refinement. However, `Content` has `type: ContentType` already, so this creates a subtle structural subtype instead of a proper discriminated union member. The name `ChatMessage` for a type discriminated as `integration_message` is also semantically confusing (a chat message is not an integration message).
- **Suggestion**: Either derive these from a proper discriminated union on `Content`, or rename them to accurately reflect their discriminant value (e.g., `IntegrationMessageContent`, `VoiceTranscriptContent`). The intersection approach works but is fragile if `Content.type` changes.
- **Evidence**: `export type ChatMessage = Content & { type: "integration_message" };` at line 27.

### Finding 9: `currentContextSchema` and `uiMessageLikeSchema` use `.loose()` in `chat-stream.schema.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/chat/chat-stream.schema.ts:25-61`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `currentContextSchema`, `uiMessageLikeSchema`, and `chatStreamRequestSchema` all use `.loose()`. The comment "intentionally permissive for forward-compat" applies to `currentContextSchema`, but `chatStreamRequestSchema` being loose means malformed extra fields pass through to backend processing.
- **Suggestion**: Consider using `.loose()` only on `currentContextSchema` (where forward-compat is explicitly desired) and using strict object schemas for `chatStreamRequestSchema` and `uiMessageLikeSchema`. This catches client bugs where extra fields are accidentally sent.
- **Evidence**: `.loose()` on `chatStreamRequestSchema` at line 61.
