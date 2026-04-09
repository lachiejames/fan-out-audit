# Audit: packages-contracts-src-contracts-17

**Files inspected**: 8
**Findings**: 5

## Summary

This slice covers training types, triage constants/schemas, user settings (refresh-all), user types, voice constants and voice schemas. The triage file is large and well-organized with strong constants patterns. The refresh-all endpoint's `refreshStepSchema` uses `z.string()` for timestamps rather than `isoDateTimeStringSchema`. The voice schemas are clean. The main concerns are: triage `platforms: z.array(z.string())` fields, the `TriageSessionStatus` type duplication, and `refreshStepSchema` timestamp types.

## Findings

### Finding 1: `TriageSessionStatus` and `TriageItemStatus` types are exported from both `schemas.ts` and `constants.ts`

- **File**: `packages/contracts/src/contracts/triage/schemas.ts:11-12` and `packages/contracts/src/contracts/triage/constants.ts:13-16`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `triage/schemas.ts` exports `type TriageSessionStatus = z.infer<typeof triageSessionStatusSchema>` and `type TriageItemStatus = z.infer<typeof triageItemStatusSchema>`. These same types are re-derived in `triage/constants.ts` via `(typeof triageSessionStatusSchema.options)[number]`. Having two derivation points means TypeScript sees them as structurally compatible but semantically distinct types coming from different files.
- **Suggestion**: Remove the type exports from `triage/schemas.ts` and import them from `triage/constants.ts` (or vice versa). Establish one canonical source for each type. The constants file is the better home since it already imports from schemas and owns the full type ecosystem for triage.
- **Evidence**:
  ```ts
  // schemas.ts line 11
  export type TriageSessionStatus = z.infer<typeof triageSessionStatusSchema>;
  // constants.ts line 13
  export type TriageSessionStatus = (typeof triageSessionStatusSchema.options)[number];
  ```

### Finding 2: `baseSessionSchema.platforms` uses `z.array(z.string())` — should be `PlatformId[]`

- **File**: `packages/contracts/src/contracts/triage/schemas.ts:34`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `platforms: z.array(z.string())` on `baseSessionSchema` stores the list of platforms in a triage session. Since triage sessions only cover connected integrations, the values should always be valid `PlatformId` strings. Using `z.string()` allows arbitrary values.
- **Suggestion**: Change to `platforms: z.array(integrationPlatformSchema)` (importing `integrationPlatformSchema` from `contracts/integrations/schemas.ts`) or `z.array(z.string().min(1))` as a minimum improvement.
- **Evidence**: `platforms: z.array(z.string()),` at line 34.

### Finding 3: `refreshStepSchema` timestamps use `z.string()` instead of `isoDateTimeStringSchema`

- **File**: `packages/contracts/src/contracts/user/settings/refresh-all.ts:22`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `refreshStepSchema` uses `completedAt: z.string().nullable()`, `startedAt: z.string().nullable()`, and the outer `refreshAllResponseSchema` uses `startedAt: z.string()`, `completedAt: z.string().nullable()`. None of these use the repo-wide `isoDateTimeStringSchema` convention.
- **Suggestion**: Replace all `z.string()` timestamp fields in `refresh-all.ts` with `isoDateTimeStringSchema` from `@/schemas.ts`.
- **Evidence**: `completedAt: z.string().nullable(),` and `startedAt: z.string().nullable(),` at lines 23-24 in `refreshStepSchema`.

### Finding 4: `startSessionBodySchema.platforms` uses `z.array(z.string())` without min-length

- **File**: `packages/contracts/src/contracts/triage/schemas.ts:132`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `platforms: z.array(z.string()).min(1).optional()` — while the `.min(1)` is good, individual platform strings within the array have no validation. An empty string `""` would pass. The `integrationPlatformSchema` already enforces `.min(1)` on individual values.
- **Suggestion**: Change `z.array(z.string())` to `z.array(integrationPlatformSchema)` to validate individual platform values as well as array length.
- **Evidence**: `platforms: z.array(z.string()).min(1).optional(),` at line 132.

### Finding 5: `voiceProfileCatalogSchema` uses `z.record(voiceProfileIdSchema, ...)` — correct but should be `satisfies`

- **File**: `packages/contracts/src/contracts/voice/constants.ts:28`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `export const voiceProfileCatalogSchema = z.record(voiceProfileIdSchema, voiceProfileSchema)` is used to validate the `VOICE_PROFILE_CATALOG` constant. However, `VOICE_PROFILE_CATALOG` is declared as `as const` (line 12) with known keys. The `z.record` schema accepts any `VoiceProfileId` key, but the actual catalog is a fixed map. Using `satisfies` on the schema would provide tighter compile-time validation.
- **Suggestion**: This is a minor improvement opportunity — the `voiceProfileCatalogSchema` is used at the API layer to validate responses, not just to type the constant. The current approach is acceptable. No change required.
- **Evidence**: `export const voiceProfileCatalogSchema = z.record(voiceProfileIdSchema, voiceProfileSchema);` at line 28.
