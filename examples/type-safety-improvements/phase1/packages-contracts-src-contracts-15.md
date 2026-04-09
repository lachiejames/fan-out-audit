# Audit: packages-contracts-src-contracts-15

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers predictions types, proactive suggestions, push notifications, review signals, saved filters, suggested actions, and their types. The suggested actions file is the most complex, implementing a well-structured discriminated union. The main concerns are: `platformCounts: z.record(z.string(), z.number())` on proactive suggestions (intentionally weak but undocumented), the `z.string()` platform field on `suggestionSourceItemSchema`, empty object schemas (`archiveStepDetailsSchema` and `ignoreStepDetailsSchema`) that should use `z.object({}).strict()`, and a `z.string().url()` vs `z.url()` inconsistency in step detail schemas.

## Findings

### Finding 1: `proactiveSuggestionSchema.platformCounts` uses `Record<string, number>` — intentional but undocumented

- **File**: `packages/contracts/src/contracts/proactive/get-proactive-suggestions.ts:74`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platformCounts: z.record(z.string(), z.number().int().min(0)).nullable()` maps platform IDs to message counts for cross-platform suggestions. The comment says "dynamic for 100+ integrations" — this is a valid reason for `Record<string, ...>` rather than a typed union. However the key type should ideally be `PlatformId` from the registry.
- **Suggestion**: Add `// Keyed by PlatformId — using string for 100+ integration scalability (ADR-002)` comment above the field, and consider importing `PlatformId` to annotate the TypeScript type even if the Zod schema remains `z.record(z.string(), ...)`.
- **Evidence**: `platformCounts: z.record(z.string(), z.number().int().min(0)).nullable(),` at line 74.

### Finding 2: `archiveStepDetailsSchema` and `ignoreStepDetailsSchema` are empty objects without `.strict()`

- **File**: `packages/contracts/src/contracts/suggested-actions/schemas.ts:136` and `156`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Both `archiveStepDetailsSchema = z.object({})` and `ignoreStepDetailsSchema = z.object({})` are empty Zod objects without `.strict()`. Zod's default behaviour for `z.object({})` is to strip unknown fields silently. Adding `.strict()` would cause validation to fail if any fields are accidentally passed to these action types, catching bugs where a caller populates the wrong step details schema.
- **Suggestion**: Change both to `z.object({}).strict()`. This is especially important since they serve as sentinel "no additional data" schemas in a discriminated union.
- **Evidence**:
  ```ts
  export const archiveStepDetailsSchema = z.object({}); // line 136
  export const ignoreStepDetailsSchema = z.object({}); // line 156
  ```

### Finding 3: `createTaskStepDetailsSchema.sourceUrl` uses `z.string().url()` — should use `z.url()`

- **File**: `packages/contracts/src/contracts/suggested-actions/schemas.ts:113`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `createTaskStepDetailsSchema` uses `sourceUrl: z.string().url().optional()` while other URL fields in the same file (e.g., `scheduleStepDetailsSchema.schedulingUrl`, `reviewStepDetailsSchema.targetUrl`, `saveResourceStepDetailsSchema.resourceUrl`) also use `z.string().url()`. Zod v4's canonical URL validator is `z.url()` (not `z.string().url()`). These should be consistent with the rest of the codebase.
- **Suggestion**: Replace `z.string().url()` with `z.url()` across all step detail schemas in this file. Check if the broader codebase has adopted `z.url()` as the canonical form.
- **Evidence**: `sourceUrl: z.string().url().optional(),` at line 113.

### Finding 4: `suggestionSourceItemSchema.platform` uses `z.string()` — `PlatformId` exists

- **File**: `packages/contracts/src/contracts/proactive/get-proactive-suggestions.ts:52`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platform: z.string()` on `suggestionSourceItemSchema` with the comment "uses z.string() for 100+ integration scalability (ADR-002)". This is an intentional choice, but unlike other places that reference ADR-002, this one correctly justifies it. No code change required, but the `PlatformId` union from the registry could serve as runtime validation while being a string supertype.
- **Suggestion**: The comment already explains the design. No change needed. This finding is informational.
- **Evidence**: `/** Platform - uses z.string() for 100+ integration scalability (ADR-002) */` at line 50.

### Finding 5: `filterConfigSchema.platforms` uses `z.array(z.string())` — could use `PlatformId`

- **File**: `packages/contracts/src/contracts/saved-filters/schemas.ts:16`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platforms: z.array(z.string()).optional()` in `filterConfigSchema` stores platform filter values. Since filter values are selected by the user from the list of connected integrations, they should always be valid `PlatformId` values. Using `z.string()` allows invalid platform IDs to be saved as filters.
- **Suggestion**: Change `z.array(z.string())` to `z.array(z.string().min(1))` at minimum, or import `integrationPlatformSchema` and use `z.array(integrationPlatformSchema)`.
- **Evidence**: `platforms: z.array(z.string()).optional(),` at line 16.

### Finding 6: `push-notifications` has `z.iso.datetime()` calls instead of the repo-wide `isoDateTimeStringSchema`

- **File**: `packages/contracts/src/contracts/push-notifications/schemas.ts:131-170`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `pushTokenResponseSchema` and related schemas use `z.iso.datetime()` directly rather than importing and using the repo-wide `isoDateTimeStringSchema` alias from `@/schemas.ts`. `isoDateTimeStringSchema` is defined as `z.iso.datetime()` so they are functionally identical, but using the alias is the documented convention and ensures future changes to the timestamp format are centralized.
- **Suggestion**: Replace `z.iso.datetime()` with `isoDateTimeStringSchema` (imported from `@/schemas.ts`) throughout `push-notifications/schemas.ts`.
- **Evidence**: `createdAt: z.iso.datetime(),` at line 131 — direct call instead of the aliased schema.
