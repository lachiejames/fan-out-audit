# Type-Safety Audit: apps-api-src-integrations-2

**Batch 63** — `apps/api/src/integrations/core/adapters/` (continued) and `base/`, `config/`, `errors/`, `interceptors/`, `interfaces/`

## Summary

9 files reviewed. The highest-impact finding is an unsafe widening cast in `base-client.provider.ts` where a `PreparedIntegration` is cast to a generic `TContext`, bypassing the generic constraint. A redundant non-null cast in `work-item-executor.adapter.ts` and two BudgetConfig local aliases are lower priority.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/adapters/work-item-executor.adapter.ts`
- **Line**: 88
- **Category**: Type casts
- **Impact**: Low
- **Description**: `targetIdentifiers as NonNullable<typeof targetIdentifiers>` — the outer `if (targetIdentifiers != null)` check already narrows the type; the cast is redundant.
- **Suggestion**: Remove the cast; TypeScript's control-flow narrowing handles this.
- **Evidence**: `const narrowed = targetIdentifiers as NonNullable<typeof targetIdentifiers>;`

---

### 2

- **File**: `apps/api/src/integrations/core/adapters/work-item-executor.adapter.ts`
- **Line**: 179
- **Category**: Type casts
- **Impact**: Low
- **Description**: Same redundant `as NonNullable<...>` cast inside the `undo` path.
- **Suggestion**: Remove; same rationale as finding 1.
- **Evidence**: Same pattern repeated in `undo` method.

---

### 3

- **File**: `apps/api/src/integrations/core/base/base-client.provider.ts`
- **Line**: 245
- **Category**: Type casts
- **Impact**: High
- **Description**: `return ok(result.value as TContext)` — widening cast from `PreparedIntegration` to the generic `TContext`. If a subclass sets `TContext` to a type incompatible with `PreparedIntegration`, this cast silently lies to TypeScript.
- **Suggestion**: Either constrain `TContext extends PreparedIntegration` or use a type guard that verifies compatibility at runtime before the cast.
- **Evidence**: `return ok(result.value as TContext);`

---

### 4

- **File**: `apps/api/src/integrations/core/base/base-client.provider.ts`
- **Line**: 539
- **Category**: Unsafe `unknown`
- **Impact**: Medium
- **Description**: Private helper uses `AuthStrategyGoogleOAuth<TContext, unknown>` replacing the `TAuth` generic with `unknown`. This bypasses the original generic constraint and loses type information about the auth strategy's token type.
- **Suggestion**: Thread `TAuth` through the private helper or use the concrete token type from the Google OAuth SDK.
- **Evidence**: `private helper(..., strategy: AuthStrategyGoogleOAuth<TContext, unknown>)`

---

### 5

- **File**: `apps/api/src/integrations/core/base/base-integration-plugin.ts`
- **Line**: 29
- **Category**: SDK type duplication
- **Impact**: Low
- **Description**: `export type BudgetConfig = PlatformBudgetConfig` — local alias for an SDK-exported type, adding indirection without benefit.
- **Suggestion**: Remove the alias; callers should import `PlatformBudgetConfig` directly from `@slopweaver/contracts`.
- **Evidence**: `export type BudgetConfig = PlatformBudgetConfig;`

---

### 6

- **File**: `apps/api/src/integrations/core/interceptors/sync-metrics.interceptor.ts`
- **Line**: 41
- **Category**: Type casts
- **Impact**: Low–Medium
- **Description**: `const syncResult = result as SyncResultResponse | undefined` — casts the `unknown` tap result without any runtime validation of the shape.
- **Suggestion**: Use a Zod schema or a `isSyncResultResponse` type guard instead of a blind cast.
- **Evidence**: `const syncResult = result as SyncResultResponse | undefined;`

---

### 7

- **File**: `apps/api/src/integrations/core/interfaces/integration-plugin.interface.ts`
- **Line**: 29
- **Category**: SDK type duplication
- **Impact**: Low
- **Description**: `export type BudgetConfig = PlatformBudgetConfig` — same alias present in both `base-integration-plugin.ts` and this interfaces file, creating two identical re-exports.
- **Suggestion**: Remove both aliases and import `PlatformBudgetConfig` directly.
- **Evidence**: `export type BudgetConfig = PlatformBudgetConfig;`

---
