# Type-Safety Audit: apps/api/src/interface (Slice 12)

**Files audited**: `knowledge-sources/knowledge-sources-import-progress.controller.ts`, `knowledge-sources/knowledge-sources-import-stream.controller.ts`, `knowledge-sources/knowledge-sources-upload-session.controller.ts`, `knowledge-sources/knowledge-sources-upload.controller.ts`

---

## knowledge-sources/knowledge-sources-import-progress.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                              | Concern                                                                                                                                                                                            |
| ---- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 63   | `e.payload as ImportProgressPayload`                                              | SSE event payloads come from the pub/sub system as `unknown`. The cast is required. However, Zod validation at the pub/sub emission point would propagate the typed value, avoiding the cast here. |
| 67   | `e.eventType as "progress" \| "completed" \| "failed" \| "cancelled" \| "paused"` | Same reasoning — if the pub/sub layer emits a discriminated union type, this cast is unnecessary.                                                                                                  |

### 3. `any` / Unsafe `unknown` Usage

| Location | Usage                                        | Concern                                                                                                                                                                                 |
| -------- | -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Line 35  | Missing explicit return type on `listEvents` | The method returns `Promise<unknown>` (ts-rest handler pattern). The missing explicit return type is consistent with the rest of the codebase's ts-rest pattern and not a safety issue. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Note                                                                                                                                                                                                                                                                                   |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The `"progress" \| "completed" \| "failed" \| "cancelled" \| "paused"` literal union at line 67 is likely also defined in the contracts package as part of `ImportProgressEvent`. If so, import the type from `@slopweaver/contracts` instead of duplicating the literal union inline. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## knowledge-sources/knowledge-sources-import-stream.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line                       | Cast                                   | Concern                                                                                                                                                                     |
| -------------------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ~374 (and other locations) | `row.payload as ImportProgressPayload` | Repeated from the SSE pattern. Same root cause as in `import-progress.controller.ts`: if pub/sub events were typed at the emission point, these casts would be unnecessary. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

| Note                                                                                                                                                                                                                                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `as ImportProgressPayload` appears in both this file and `knowledge-sources-import-progress.controller.ts`. This is the same cast at two different consumption points of the same pub/sub event stream. A shared typed helper that reads and validates the payload would eliminate both. |

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## knowledge-sources/knowledge-sources-upload-session.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                         | Concern                                                                                                                                                                                                                                                                               |
| ---- | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 136  | `file: { ... } as Express.Multer.File`       | When Multer processes a multipart upload, `req.file` is typed as `Express.Multer.File \| undefined`. The cast here asserts it is non-undefined after a prior null check — acceptable, but `@UploadedFile()` from NestJS already provides the typed value directly without casting.    |
| 197  | `provider as Parameters<...>[0]["provider"]` | Complex utility-type cast to extract the `provider` parameter type from a generic function signature. This pattern is fragile — if the function signature changes, the cast silently accepts the wrong shape. Extract the provider type as a named exported type and use it directly. |

### 3. `any` / Unsafe `unknown` Usage

None found.

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

None found.

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## knowledge-sources/knowledge-sources-upload.controller.ts

### 1. Custom Types Duplicating SDK Types

None found.

### 2. Type Casts That Could Be Avoided

| Line | Cast                                                                             | Concern                                                                                                                                                                                                                                                                                                                                  |
| ---- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 106  | `(req.body as Record<string, unknown>)?.["providerHint"] as string \| undefined` | Double cast: first widens `req.body` (which is `any` in Express) to `Record<string, unknown>`, then narrows the accessed value to `string \| undefined`. The outer cast is unavoidable given `req.body: any`, but the inner `as string \| undefined` could be replaced by a type guard: `typeof value === "string" ? value : undefined`. |
| 135  | Same double-cast pattern for `sourceId`                                          | Same recommendation: use a type-guard helper (`asStringOrUndefined`) rather than a cast.                                                                                                                                                                                                                                                 |

### 3. `any` / Unsafe `unknown` Usage

| Location          | Usage                                | Concern                                                                                                                                                          |
| ----------------- | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `req.body` access | `req.body` is typed `any` by Express | Standard Express limitation. The `as Record<string, unknown>` widening is the safe approach; the secondary `as string \| undefined` cast is where the risk lies. |

### 4. `Record<string, ...>` Where Stricter Types Would Be Safer

| Line     | Usage                                 | Concern                                                                                                                            |
| -------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 106, 135 | `req.body as Record<string, unknown>` | Acceptable intermediate step, but the double-cast pattern should use a type-guard instead of a secondary cast for the final value. |

### 5. Duplicate Type Definitions

None found.

### 6. Missing Strict tsconfig/eslint Flags

None found.

---

## Summary

| Severity | Finding                                                                                                                            | File                                                                                               | Lines    |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | -------- |
| Medium   | `as ImportProgressPayload` repeated in two files — a typed pub/sub emission point or shared validation helper would eliminate both | `knowledge-sources-import-progress.controller.ts`, `knowledge-sources-import-stream.controller.ts` | 63, ~374 |
| Medium   | `provider as Parameters<...>[0]["provider"]` fragile utility-type cast — export `ProviderType` as a named type                     | `knowledge-sources-upload-session.controller.ts`                                                   | 197      |
| Medium   | Double-cast `req.body` pattern — use `asStringOrUndefined` type-guard helper                                                       | `knowledge-sources-upload.controller.ts`                                                           | 106, 135 |
| Low      | `e.eventType as "progress" \| ..."` literal union — import from contracts                                                          | `knowledge-sources-import-progress.controller.ts`                                                  | 67       |
| Low      | `file: { ... } as Express.Multer.File` — prefer `@UploadedFile()` which provides the typed value directly                          | `knowledge-sources-upload-session.controller.ts`                                                   | 136      |
