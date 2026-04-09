# Audit: packages-contracts-src-contracts-4

**Files inspected**: 8
**Findings**: 7

## Summary

This slice covers conversations, core-profile, digest, entities, and feedback-loops. The entity schemas are rich and well-designed with proper type guards. The main issues are: loose `string` for entity/citation platform fields, `confidence` stored as a decimal string rather than a number in `pendingLinkResponseSchema`, and some loose `string` timestamp fields where `isoDateTimeStringSchema` exists.

## Findings

### Finding 1: `entityPlatformSchema` is bare `z.string()` — same pattern as `citationPlatformSchema`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/entities/schemas.ts:16`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `entityPlatformSchema = z.string()` with the comment "string for 100+ integration scalability". This is intentional but means entity platform values are unvalidated. A `PlatformIdentity` with `platform: "gmailz"` passes Zod.
- **Suggestion**: Use `z.string().min(1)` at minimum, or a soft-validation `refine` that checks against known platforms and logs a warning (not an error) for unknown ones. Consider `PlatformId | (string & {})` at the TypeScript level for autocomplete.
- **Evidence**: `export const entityPlatformSchema = z.string();` at line 16.

### Finding 2: `pendingLinkResponseSchema.confidence` is `z.string()` (decimal string) instead of `z.number()`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/entities/schemas.ts:172`
- **Category**: missing-strict-typing
- **Impact**: high
- **Description**: `confidence: z.string()` with comment "Decimal as string (0.00 - 1.00)". This is a numeric value being transmitted as a string, likely because Drizzle returns `numeric` columns as strings. The frontend must parse this with `parseFloat` before use, and Zod offers no validation of the numeric range.
- **Suggestion**: Use `z.string().transform(s => parseFloat(s)).pipe(z.number().min(0).max(1))` to convert at the boundary, or change the DB column to `real` and use `z.number().min(0).max(1)` directly. The current shape means callers working with `PendingLinkResponse.confidence` get a string where they expect a number, creating silent bugs in any arithmetic.
- **Evidence**: `confidence: z.string(), // Decimal as string (0.00 - 1.00)` at line 172.

### Finding 3: Timestamp fields on entities use `z.string()` rather than `isoDateTimeStringSchema`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/entities/schemas.ts:93-103`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `createdAt`, `updatedAt`, `lastInteraction`, and `linkedAt` fields in entity schemas use `z.string()` (plain, no datetime validation) rather than `isoDateTimeStringSchema` used consistently across other contracts.
- **Suggestion**: Replace `z.string()` with `isoDateTimeStringSchema` (imported from `@/schemas.ts`) for all timestamp fields in entity schemas. This provides format validation at API boundaries and aligns with the rest of the codebase convention.
- **Evidence**: `createdAt: z.string(),` at line 93; `updatedAt: z.string(),` at line 94; `lastInteraction: z.string().nullable(),` at line 99; `linkedAt: z.string(),` at line 82.

### Finding 4: `resolvedEntityResponseSchema.interactionScore` has no range constraint

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/entities/schemas.ts:97`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `interactionScore: z.number()` with comment "0-100" but no `.min(0).max(100)` constraint. The range is documented but not enforced.
- **Suggestion**: `interactionScore: z.number().int().min(0).max(100)`.
- **Evidence**: `interactionScore: z.number(), // 0-100` at line 97.

### Finding 5: `conversationSourceItemTypeSchema` uses a complex `.refine()` where a `z.union`/`z.enum` would be cleaner

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/conversations/schemas.ts:42-51`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The schema validates a string as a `PlatformId | "email" | "task" | "work_item" | "search"` using a `.refine()` callback. Zod's discriminated unions / enum composition would be more readable and produce better error messages than a refine.
- **Suggestion**: Consider `z.union([z.enum(["email", "task", "work_item", "search"]), z.string().refine(isPlatformId)])` which preserves the two-part structure but makes the valid values explicit in schema metadata.
- **Evidence**: `.refine((value): value is PlatformId | "email" | "task" | "work_item" | "search" => ...` at lines 44-50.

### Finding 6: `chatRoleSchema` is defined twice — in `conversations/schemas.ts` and `chat/schemas.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/conversations/schemas.ts:40`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `chatRoleSchema = z.enum(["user", "assistant", "system"])` is defined in both `conversations/schemas.ts` (line 40) and `chat/schemas.ts` (line 9). The conversations file even has a comment "Note: ChatRole type is exported from @/contracts/chat/types.ts". The local re-definition creates two separate schema instances that could drift.
- **Suggestion**: Remove the `chatRoleSchema` definition from `conversations/schemas.ts` and import it from `chat/schemas.ts`.
- **Evidence**: `export const chatRoleSchema = z.enum(["user", "assistant", "system"]);` at line 40 of `conversations/schemas.ts`, duplicating line 9 of `chat/schemas.ts`.

### Finding 7: `WritingPattern` and `WritingPatternResponse` are duplicate type exports in `feedback-loops/types.ts`

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/feedback-loops/types.ts:31-32`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `WritingPattern` and `WritingPatternResponse` resolve to the same inferred type from `writingPatternResponseSchema`. Two names for the same type creates maintenance overhead.
- **Suggestion**: Keep only `WritingPattern` (the semantic name) and remove `WritingPatternResponse`, or document their intended divergence.
- **Evidence**: `export type WritingPattern = z.infer<typeof writingPatternResponseSchema>;` and `export type WritingPatternResponse = z.infer<typeof writingPatternResponseSchema>;` at lines 31-32.
