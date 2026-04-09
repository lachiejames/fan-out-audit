# Type-Safety Audit: apps-api-src-integrations-6

**Batch 67** — `apps/api/src/integrations/core/services/` (oauth-base.service, oauth-base.wizard, query-expansion, reply-classifier, reranking, semantic-search-assignment, semantic-search-fallback, semantic-search.queries)

## Summary

8 files reviewed. The main findings are in `oauth-base.wizard.ts`, where `WizardDeps` uses pervasive `unknown` types for tokenData/userInfo to avoid threading generics, and `oauth-base.service.ts` has several `as TTokenResponse`/`as TUserInfo` casts at the generic boundary. Most other files are well-typed. `query-expansion.service.ts` and `reply-classifier.service.ts` have a minor duplicate `usage` extraction pattern. Reranking, semantic-search-assignment, and semantic-search-fallback are clean.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/services/oauth-base.wizard.ts`
- **Lines**: 28–46
- **Category**: Unsafe `unknown`
- **Impact**: Medium
- **Description**: `WizardDeps` interface uses `unknown` for `tokenData`, `userInfo`, `updates`, and the return types of `findById`/`findByPlatformUserId` (all `Result<unknown | null, IntegrationError>`). This is intentional to avoid threading `TTokenResponse`/`TUserInfo` generics, but every access to these values requires an unsafe cast.
- **Suggestion**: The comment "Duck-typed to avoid threading generics" explains the trade-off. Consider using a small typed adapter interface for the token/user fields (`{ accessToken: string; expiresIn: number; refreshToken: string | null }`) that captures only what the wizard needs, rather than `unknown`.
- **Evidence**: `exchangeCodeForTokens: (...) => Promise<Result<unknown, IntegrationError>>; extractAccessToken: (tokenData: unknown) => string;`

---

### 2

- **File**: `apps/api/src/integrations/core/services/oauth-base.wizard.ts`
- **Lines**: 180–184, 248
- **Category**: Type casts
- **Impact**: Medium
- **Description**: `const existingIntegration = existingResult.value as { id: string; platform: PlatformId; platformUserId?: string | null }` — casts `unknown` to a concrete shape after retrieving from the duck-typed `findById`. No runtime validation.
- **Suggestion**: Add a narrow Zod schema or type guard at this point to validate the integration shape before use.
- **Evidence**: `const existingIntegration = existingResult.value as { id: string; platform: PlatformId; ... };`

---

### 3

- **File**: `apps/api/src/integrations/core/services/oauth-base.service.ts`
- **Lines**: 233, 239–240, 248–249
- **Category**: Type casts
- **Impact**: Low–Medium
- **Description**: The `handleCallback` method creates `CallbackDeps` by wrapping abstract methods with arrow functions that cast `tokenData as TTokenResponse` and `userInfo as TUserInfo`. These casts are load-bearing to bridge between the erased generic in `CallbackDeps` (which uses `unknown`) and the concrete generic in `OAuthBaseService`. They are safe by construction but fragile if `CallbackDeps` is ever changed.
- **Suggestion**: Low priority given the intentional architecture, but document the invariant with a comment explaining why the casts are safe at these specific sites.
- **Evidence**: `tokenData: tokenData as TTokenResponse`, `userInfo: userInfo as TUserInfo` in callback lambda wrappers.

---

### 4

- **File**: `apps/api/src/integrations/core/services/oauth-base.service.ts`
- **Line**: 726
- **Category**: Type casts
- **Impact**: Low
- **Description**: `updates as { connectionName?: string; syncSettings?: Record<string, unknown> }` — the `updates` parameter from `WizardDeps.updateIntegration` is typed `unknown`, requiring a cast to use it.
- **Suggestion**: Part of the broader `WizardDeps` `unknown`-typing issue (finding 1). Fixing the root would eliminate this.
- **Evidence**: `const normalizedUpdates = updates && typeof updates === "object" ? (updates as { connectionName?: string; syncSettings?: Record<string, unknown> }) : {};`

---

### 5

- **File**: `apps/api/src/integrations/core/services/query-expansion.service.ts`
- **Line**: 347
- **Category**: Unsafe `unknown`
- **Impact**: Low
- **Description**: `const usage = typeof result === "object" && result !== null && "usage" in result ? (result as { usage?: unknown }).usage : undefined` — the Vercel AI SDK `generateText` result is typed but `usage` is accessed via an existence check + cast rather than using the typed property directly.
- **Suggestion**: Use `result.usage` directly since `generateText` returns a typed object with a `usage` field. The extra check is unnecessary.
- **Evidence**: `const usage = typeof result === "object" && ... ? (result as { usage?: unknown }).usage : undefined;`

---

### 6

- **File**: `apps/api/src/integrations/core/services/reply-classifier.service.ts`
- **Line**: 317
- **Category**: Unsafe `unknown`
- **Impact**: Low
- **Description**: Same `usage` extraction pattern as finding 5 — accessing `generateText` result's `usage` via a defensive `"usage" in result` check and cast instead of using the typed SDK property.
- **Suggestion**: Same fix as finding 5.
- **Evidence**: `const usage = typeof result === "object" && result !== null && "usage" in result ? (result as { usage?: unknown }).usage : undefined;`

---
