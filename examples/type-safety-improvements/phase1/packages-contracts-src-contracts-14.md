# Audit: packages-contracts-src-contracts-14

**Files inspected**: 8
**Findings**: 4

## Summary

This slice covers memory types, metadata helpers, navigation types, notification endpoints, OAuth schemas, and personas types. Most files are clean thin type-aggregation modules. The metadata index is notable for its `unknownRecordSchema` fallback. The OAuth authorize schema uses `.loose()` intentionally. The main concerns are minor: the `oauthAuthorizeQuerySchema` uses `.loose()` where extra query params from OAuth providers could be silently accepted, and the `exportedAt` timestamp in `exportMemoryFilesResponseSchema` uses `z.string()` instead of `isoDateTimeStringSchema`.

## Findings

### Finding 1: `exportMemoryFilesResponseSchema.exportedAt` uses `z.string()` instead of `isoDateTimeStringSchema`

- **File**: `packages/contracts/src/contracts/memory/export-all.ts:16`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `exportedAt: z.string()` on `exportMemoryFilesResponseSchema` is a timestamp field, but unlike other timestamp fields throughout the contracts package, it does not use the repo-wide `isoDateTimeStringSchema` convention. This means the contract would accept any string (e.g., `"yesterday"`) for `exportedAt` without format validation.
- **Suggestion**: Replace `exportedAt: z.string()` with `exportedAt: isoDateTimeStringSchema`. Import `isoDateTimeStringSchema` from `@/schemas.ts`.
- **Evidence**: `exportedAt: z.string(),` at line 16 in `export-all.ts`.

### Finding 2: `oauthAuthorizeQuerySchema` uses `.loose()` — unknown OAuth provider params silently pass

- **File**: `packages/contracts/src/contracts/oauth/platform/schemas.ts:23`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `oauthAuthorizeQuerySchema` uses `.loose()` to accept extra query parameters from OAuth providers (e.g., `state` from Slack, `hd` from Google Workspace). This is intentional per the comment "Platform-specific providers may ignore/extend with extra query params." However, the `.loose()` call does not have a comment explaining why it is needed, which may lead future maintainers to incorrectly tighten it.
- **Suggestion**: Add an inline comment above `.loose()`: `// Providers may append platform-specific query params (e.g., Slack state, Google hd domain hint)`.
- **Evidence**: `}).loose();` at line 33 in `schemas.ts`.

### Finding 3: `safeParseUnknownRecord` parameter could be more specific

- **File**: `packages/contracts/src/contracts/metadata/index.ts:20`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `safeParseUnknownRecord({ metadata }: { metadata: unknown })` accepts `unknown` and handles null/undefined with an early return. The function is correct, but the null/undefined check (`if (metadata === null || metadata === undefined)`) could be simplified to `if (metadata == null)` (using loose equality as permitted by the TypeScript patterns rule). This is a code quality note, not a type-safety issue.
- **Suggestion**: Replace `metadata === null || metadata === undefined` with `metadata == null` per the `typescript-patterns.md` rule that "Loose equality for null checks (`== null`, `!= null`) is acceptable."
- **Evidence**: `if (metadata === null || metadata === undefined) {` at line 21.

### Finding 4: Navigation `types.ts` depends on a Zod schema but exports only inferred types — no type guard

- **File**: `packages/contracts/src/contracts/navigation/types.ts`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `navigation/types.ts` exports `SidebarCountsResponse = z.infer<typeof sidebarCountsResponseSchema>`. If `sidebarCountsResponseSchema` is a complex or nested schema, this file could be a source of TS7056 in the future. More importantly, there is no exported type guard (`isSidebarCountsResponse`) for use in SSE or WebSocket event handlers that receive `unknown` data.
- **Suggestion**: Add a type guard function `isSidebarCountsResponse(value: unknown): value is SidebarCountsResponse` using `sidebarCountsResponseSchema.safeParse(value).success` for use in event parsing code.
- **Evidence**: `export type SidebarCountsResponse = z.infer<typeof sidebarCountsResponseSchema>;` is the only export in the file.
