# Audit: packages-contracts-src-contracts-1

**Files inspected**: 8
**Findings**: 5

## Summary

Most files in this slice are thin type-export wrappers over Zod schemas (activity-stream, activity, analytics, auth, behavioral) and are clean. The substantive schemas live in co-located `schemas.ts` files not audited here. The main findings are in `agents/tool-schemas.ts` (loose `string` for platforms), `ai-costs/constants.ts` (two weak `Record<string, string>` maps), and `behavioral/types.ts` (duplicate `chatRoleSchema` definition).

## Findings

### Finding 1: `MODEL_TIER_LABELS` and `TASK_TYPE_LABELS` use `Record<string, string>` instead of constrained key unions

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/ai-costs/constants.ts:28-46`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `MODEL_TIER_LABELS` is typed as `Record<string, string>` but the only valid keys are `(typeof ALL_MODEL_TIERS)[number]` ("haiku" | "sonnet" | "opus"). Similarly `TASK_TYPE_LABELS` keys should be constrained to `(typeof ALL_TASK_TYPES)[number]`. As written, any string key lookup returns `string | undefined` at runtime but is typed as `string`, hiding potential missing-key bugs.
- **Suggestion**: Change both to use the derived union as the key type:
  ```typescript
  export const MODEL_TIER_LABELS: Record<(typeof ALL_MODEL_TIERS)[number], string> = { ... }
  export const TASK_TYPE_LABELS: Record<(typeof ALL_TASK_TYPES)[number], string> = { ... }
  ```
- **Evidence**: `export const MODEL_TIER_LABELS: Record<string, string>` at line 28; `export const TASK_TYPE_LABELS: Record<string, string>` at line 37.

### Finding 2: `agentToolMetadataSchema.platforms` is `z.array(z.string())` — should reference known platform IDs

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/agents/tool-schemas.ts:48`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: The `platforms` field in `agentToolMetadataSchema` accepts any array of strings. The codebase has a `PlatformId` union and `isPlatformId` type guard. This means a tool metadata record with `platforms: ["gmailz"]` would pass Zod validation.
- **Suggestion**: Use `z.array(z.string().refine(isPlatformId, { message: "Unknown platform" }))` or keep as `z.string()` with a JSDoc noting the expected values. The loose typing here propagates to the frontend's capability gating logic.
- **Evidence**: `platforms: z.array(z.string()),` at line 48.

### Finding 3: `toolActionSchema` — `platform` in `oauth_popup` branch is `z.string()` rather than a platform literal

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/agents/tool-schemas.ts:26`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The `oauth_popup` action variant includes a `platform: z.string()` field. Like Finding 2, this could be narrowed to a known `PlatformId`.
- **Suggestion**: Use `platform: z.string().refine(isPlatformId)` or import and use a Zod enum derived from `PLATFORM_IDS`.
- **Evidence**: `platform: z.string(),` at line 26.

### Finding 4: `AnalyticsResponse` deprecated alias creates duplicate type surface

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/analytics/types.ts:50-52`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `AnalyticsResponse` is a `@deprecated` alias for `AnalyticsOverviewResponse`. Both types are exported and identical. This creates dual import paths and can cause confusion in IDEs.
- **Suggestion**: Add a tracking item to remove `AnalyticsResponse` in a future cleanup. Ensure all usages in `apps/` have been migrated. Once migrated, delete the alias.
- **Evidence**: `export type AnalyticsResponse = AnalyticsOverviewResponse;` at lines 50-52.

### Finding 5: `WorkItemRevision` and `WorkItemRevisionResponse` are duplicate type exports

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/contracts/feedback-loops/types.ts:26-28`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Both `WorkItemRevision` and `WorkItemRevisionResponse` are exported and resolve to the same `z.infer<typeof workItemRevisionResponseSchema>`. This creates two names for the same shape, which adds noise.
- **Suggestion**: Pick one canonical name (likely `WorkItemRevision`) and remove the alias, or document the difference if they are intended to diverge.
- **Evidence**: `export type WorkItemRevision = z.infer<typeof workItemRevisionResponseSchema>;` and `export type WorkItemRevisionResponse = z.infer<typeof workItemRevisionResponseSchema>;` at lines 26-27.
