# Audit: apps-api-src-shared-18

**Files inspected**: 8
**Findings**: 3

## Summary

Most utilities in this slice are clean. The notable findings are: `OAuthPlatformType` defined inline in `oauth.utils.ts` with a comment warning it must stay in sync with the domain layer (a known duplication risk), a `LlmUserContent = string` trivial type alias in `system-content.utils.ts` that adds no type safety, and an `as TurnstileResponse` cast after `response.json()` in `turnstile.utils.ts` that bypasses runtime validation.

## Findings

### Finding 1: `OAuthPlatformType` local redefinition in `oauth.utils.ts` with manual sync comment

- **File**: `apps/api/src/shared/utils/oauth.utils.ts:14`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `OAuthPlatformType` is defined locally as a string literal union with an inline comment: `// MUST stay in sync with domain layer`. This is a recognised duplication risk: if a platform is added or removed, two definitions must be updated. The canonical source for platform IDs is `@slopweaver/contracts` (`PlatformId`), which is code-generated from `config.ts` files.
- **Suggestion**: Replace the local `OAuthPlatformType` with `PlatformId` from `@slopweaver/contracts`, or with a narrower subset derived from it (e.g. `Extract<PlatformId, "google-gmail" | "microsoft-outlook">`). This eliminates the sync hazard and ties the type to the single source of truth.
- **Evidence**:
  ```ts
  // MUST stay in sync with domain layer
  type OAuthPlatformType = "google-gmail" | "microsoft-outlook" | "slack" | ...;
  ```

### Finding 2: `LlmUserContent = string` trivial alias in `system-content.utils.ts`

- **File**: `apps/api/src/shared/utils/system-content.utils.ts:15`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `LlmUserContent` is defined as `type LlmUserContent = string`. This alias adds no structural constraint over `string` — any string satisfies it and it can be silently widened back to `string` at any call site. It provides nominal documentation value only.
- **Suggestion**: Either remove the alias (use `string` directly) if no additional constraint is intended, or widen to a branded type (`type LlmUserContent = string & { _brand: "LlmUserContent" }`) to prevent accidental mixing with arbitrary strings. If the SDK has a concrete type for user content parts, use that instead.
- **Evidence**:
  ```ts
  export type LlmUserContent = string;
  ```

### Finding 3: `as TurnstileResponse` cast after `response.json()` in `turnstile.utils.ts`

- **File**: `apps/api/src/shared/utils/turnstile.utils.ts:53`
- **Category**: type-cast
- **Impact**: medium
- **Description**: After `await response.json()`, the result is cast directly to `TurnstileResponse` with `as TurnstileResponse`. `response.json()` returns `unknown` (or `any` in older fetch typings); the `as` cast suppresses the type error without validating that the actual JSON matches the expected shape. A malformed or unexpected Cloudflare response would not be caught at the boundary.
- **Suggestion**: Parse the response through a Zod schema (`TurnstileResponseSchema.parse(await response.json())`) to validate the payload at runtime. The project already uses Zod for contract validation — this is consistent with established patterns.
- **Evidence**:
  ```ts
  const data = (await response.json()) as TurnstileResponse;
  ```

## No additional findings

`metadata.utils.ts` (intentional `as Record<string, unknown>` cast, documented), `microsoft-graph-error.utils.ts`, `notion-api-error.utils.ts` (same error extractor pattern as slice 17), `pptx-content-extractor.utils.ts`, `safe-api-call.utils.ts`, and `serialize-dates-to-iso.ts` are otherwise clean.
