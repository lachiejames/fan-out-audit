# Type-Safety Audit: apps-api-src-integrations-8

**Batch 69** — `apps/api/src/integrations/core/services/` (sync-stream-db, thread-aggregation, token-refresh) and `utils/` (batch-db, cursor-state, search-candidate-mapping)

## Summary

6 files reviewed. This batch is largely clean. The main finding is the `token-refresh.service.ts` JSON response cast `(await response.json()) as Record<string, unknown>` — intentional given OAuth responses vary per provider. `cursor-state.utils.ts` returns `Record<string, unknown>` by design (cursor state is opaque). `search-candidate-mapping.utils.ts` has a minor DB string-to-union cast and a `Map<string, unknown>` parameter. `sync-stream-db.service.ts` and `thread-aggregation.service.ts` are clean.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/services/token-refresh.service.ts`
- **Line**: 316
- **Category**: Type casts
- **Impact**: Low
- **Description**: `(await response.json()) as Record<string, unknown>` — necessary since OAuth token responses differ per provider. The per-provider `parseResponse` functions then extract and validate individual fields using `tokenData["access_token"]` etc.
- **Suggestion**: Low priority given that each provider's `parseResponse` function handles field extraction. Could be improved with per-provider Zod schemas if validation strictness is desired, but the current pattern is a known intentional trade-off.
- **Evidence**: `const tokenData = (await response.json()) as Record<string, unknown>;`

---

### 2

- **File**: `apps/api/src/integrations/core/services/token-refresh.service.ts`
- **Line**: 160
- **Category**: Type casts
- **Impact**: Low
- **Description**: `const authedUser = tokenData["authed_user"] as { access_token?: string } | undefined` in `parseSlackTokenResponse` — inline cast to extract the nested Slack token struct. No Zod validation.
- **Suggestion**: Define a small `SlackTokenResponseSchema = z.object({ authed_user: z.object({ access_token: z.string().optional() }).optional(), ... })` and validate before access.
- **Evidence**: `const authedUser = tokenData["authed_user"] as { access_token?: string } | undefined;`

---

### 3

- **File**: `apps/api/src/integrations/core/utils/cursor-state.utils.ts`
- **Lines**: 11, 31
- **Category**: `Record<string, ...>`
- **Impact**: Low
- **Description**: Both `parseCursorState` and `parseTieredSelectionState` return `Record<string, unknown>`. This is intentional since cursor state is platform-specific and opaque to the core layer.
- **Suggestion**: No change required. Documented here for completeness. The callers (e.g., `engine.tiered.ts`) should use their own typed cursor state at the call site.
- **Evidence**: `export function parseCursorState(...): Record<string, unknown>`

---

### 4

- **File**: `apps/api/src/integrations/core/utils/search-candidate-mapping.utils.ts`
- **Line**: 337
- **Category**: Type casts
- **Impact**: Low
- **Description**: `contentLevel: content.contentLevel as "message" | "thread"` — the DB column is typed as `string` but the function return type requires the narrower union. No runtime guard.
- **Suggestion**: Add a `isContentLevel` type predicate or Zod enum parse before the cast.
- **Evidence**: `contentLevel: content.contentLevel as "message" | "thread",`

---

### 5

- **File**: `apps/api/src/integrations/core/utils/search-candidate-mapping.utils.ts`
- **Line**: 283
- **Category**: `Record<string, ...>`
- **Impact**: Low
- **Description**: `semanticResultsMap: Map<string, unknown>` parameter in `extractKeywordOnlyIds` — the map is typed `unknown` so the function can accept both `SemanticSearchRow` and other types. The actual semanticResultsMap at call sites holds `SemanticSearchRow` values.
- **Suggestion**: Change the parameter type to `Map<string, SemanticSearchRow>` (or `Map<string, unknown>` is fine if the function only uses `.has()`). Current usage only calls `.has()` so `Map<string, unknown>` is intentionally permissive for reuse.
- **Evidence**: `semanticResultsMap: Map<string, unknown>;` in `extractKeywordOnlyIds` params.

---
