# Type-Safety Audit: apps/api/src/interface (Slice 14)

**Files audited**: `oauth/unified-oauth.controller.ts`, `personas/persona.controller.ts`, `prediction/prediction-response.utils.ts`, `prediction/prediction.controller.ts`, `priority-scoring/priority-scoring.controller.ts`, `proactive/proactive-response.utils.ts`, `proactive/proactive.controller.ts`, `push-notifications/push-notifications.controller.ts`

---

## oauth/unified-oauth.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found. The manual `response.status(n).json({...})` pattern is required when using `@Res()` (see API patterns doc).

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Manual response pattern is intentional and documented.

---

## personas/persona.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean file. Gold-standard CRUD pattern.

---

## prediction/prediction-response.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line  | Cast                                                                    | Concern                                                                                                                                                                                                                                                                                                                                                                                                                 |
| ----- | ----------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 42–46 | Multiple `as ContractType` casts in `toContractPrediction`              | The function explicitly models the gap between DB storage types (`string`, `unknown`) and contract types (union literals, `PredictionCompoundAction[]`). These casts are the intended boundary between DB and contract. They are structurally sound but cannot be validated at compile time. Zod schemas matching the contract types and applied to the DB values would replace the casts with validated parse results. |
| 41    | `dbPrediction.contentSignals as PredictionContentSignals \| null`       | `contentSignals` is stored as `unknown` (JSON column). Zod validation would convert this to a proper `Result` instead of a silent cast.                                                                                                                                                                                                                                                                                 |
| 43    | `dbPrediction.intentCategory as PredictionIntentCategory \| null`       | Same: stored as `string \| null`; cast to union literal.                                                                                                                                                                                                                                                                                                                                                                |
| 44    | `dbPrediction.predictedAction as ContractPrediction["predictedAction"]` | Same: stored as `string`, cast to union.                                                                                                                                                                                                                                                                                                                                                                                |
| 45    | `dbPrediction.predictedActions as PredictionCompoundAction[] \| null`   | Stored as `unknown` (JSON), cast to typed array.                                                                                                                                                                                                                                                                                                                                                                        |
| 46    | `dbPrediction.predictedTiming as ContractPrediction["predictedTiming"]` | Stored as `string \| null`, cast to union.                                                                                                                                                                                                                                                                                                                                                                              |

### 3. `any` / Unsafe `unknown` Usage

| Location                             | Usage     | Concern                                                                                                   |
| ------------------------------------ | --------- | --------------------------------------------------------------------------------------------------------- |
| `DbPredictionShape.predictedActions` | `unknown` | This is the correct type for a JSON column before validation. The subsequent cast is where the risk lies. |
| `DbPredictionShape.contentSignals`   | `unknown` | Same.                                                                                                     |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location            | Note                                                                                                                                                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DbPredictionShape` | This type is defined locally as an `Omit<ContractPrediction, ...> & { ... }`. If the DB table's Drizzle-inferred type already matches this shape, prefer importing `typeof predictionsTable.$inferSelect` directly. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## prediction/prediction.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found. All DB→contract mapping is delegated to `toContractPrediction` in the utils file.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

**Assessment**: Clean controller. Error-cause serialisation via `serializeErrorCause` is well-typed.

---

## priority-scoring/priority-scoring.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location                                                                                             | Note                                                                                                                                                                                                                                                                                                               |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| VIP contact response shape built inline in `listVipContacts`, `createVipContact`, `updateVipContact` | The same `{ contactIdentifier, createdAt, displayName, id, notes, platform, updatedAt, userId }` object literal is constructed identically in three separate `.match()` callbacks. A `mapVipContactResponse` helper (similar to what `proactive-response.utils.ts` already has) would eliminate this triplication. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## proactive/proactive-response.utils.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                        | Concern                                                                                                                                                                                                                                                                                                                                                 |
| ---- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 63   | `{ ...(settings as object), enabledChannels } as ContractProactiveSettings` | `settings` is `Record<string, unknown>` and the function spreads it then casts to the contract type. If `settings` were typed as the actual service return type (the DB settings entity), the spread would be type-safe and the cast unnecessary. The function signature should accept the typed settings entity rather than `Record<string, unknown>`. |

### 3. `any` / Unsafe `unknown` Usage

| Location | Usage                                                          | Concern                                                                                                                   |
| -------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Line 62  | `(settings["enabledChannels"] ?? []) as NotificationChannel[]` | Accessing unknown record by string key. Acceptable given the `Record<string, unknown>` input, but see cast concern above. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line                           | Usage                               | Concern                                                                                                                                                  |
| ------------------------------ | ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `toContractSettings` parameter | `settings: Record<string, unknown>` | The function accepts any record and casts it to `ContractProactiveSettings`. A typed settings entity input would make the mapping statically verifiable. |

### 5. Duplicate Type Definitions

| Location | Note                                                           |
| -------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Line 13  | `type NotificationChannel = "desktop" \| "mobile" \| "in_app"` | This union likely exists in `@slopweaver/contracts`. Import it rather than re-defining locally. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## proactive/proactive.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line     | Cast                                                                   | Concern                                                                                                                                                                                                                                                                                  |
| -------- | ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 474, 531 | `serialized as Record<string, unknown>` passed to `toContractSettings` | `serializeDatesToIso` returns `unknown`. Casting to `Record<string, unknown>` before passing to `toContractSettings` is required by the current `toContractSettings` signature. If `toContractSettings` accepted the typed settings entity, this intermediate cast would be unnecessary. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## push-notifications/push-notifications.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

None found.

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Location                                                                         | Note                                                                                                                                                                                                                                                     |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `registerPushToken` and `listPushTokens` response bodies                         | Both endpoints build identical inline object literals `{ appVersion, createdAt, deviceId, deviceOs, id, isActive, lastUsedAt, permissionState, provider, token, userId }` from the service result. A `mapTokenToResponse` helper would deduplicate this. |
| `getNotificationPreferences` and `updateNotificationPreferences` response bodies | Both endpoints build the exact same inline preferences object literal. A `mapPreferencesToResponse` helper would deduplicate.                                                                                                                            |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                                                                 | File                               | Lines    |
| -------- | ----------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------- |
| Medium   | 5 `as ContractType` casts in `toContractPrediction` — Zod parse would replace with validated results                    | `prediction-response.utils.ts`     | 41–46    |
| Medium   | `toContractSettings` accepts `Record<string, unknown>` — use typed settings entity                                      | `proactive-response.utils.ts`      | 61–63    |
| Medium   | `serialized as Record<string, unknown>` cast propagated from `serializeDatesToIso` — fix `toContractSettings` signature | `proactive.controller.ts`          | 474, 531 |
| Medium   | VIP contact response object built identically 3× — extract `mapVipContactResponse` helper                               | `priority-scoring.controller.ts`   | multiple |
| Low      | Push token and preferences response objects duplicated across 2 endpoints each                                          | `push-notifications.controller.ts` | multiple |
| Low      | `NotificationChannel` re-defined locally — import from `@slopweaver/contracts`                                          | `proactive-response.utils.ts`      | 13       |
| Low      | `DbPredictionShape` could be derived from Drizzle `$inferSelect`                                                        | `prediction-response.utils.ts`     | 20–30    |
