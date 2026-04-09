# Type-Safety Audit: apps-api-src-integrations-4

**Batch 65** — `apps/api/src/integrations/core/registry/`, `relevance/`, and `services/` (attachment, auto-work-item, base-sync-lifecycle)

## Summary

10 files reviewed. The most notable findings are in the tiered relevance engine and its plan types, where cursor state is typed as `Record<string, unknown>` throughout. The registry validator uses a cast to bypass strict index access. The services reviewed are generally clean.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/registry/plugin-registry.validator.ts`
- **Line**: 64
- **Category**: Type casts
- **Impact**: Low
- **Description**: `(PLATFORMS as Record<string, PublicPlatformConfig>)[plugin.id]` — casts the `PLATFORMS` registry (likely a typed const object) to a generic record to allow string-key access without a type error. An `Object.hasOwn` check would be safer.
- **Suggestion**: Use `if (Object.hasOwn(PLATFORMS, plugin.id)) { const config = PLATFORMS[plugin.id as keyof typeof PLATFORMS]; }` instead of casting the whole object.
- **Evidence**: `const platformConfig = (PLATFORMS as Record<string, PublicPlatformConfig>)[plugin.id];`

---

### 2

- **File**: `apps/api/src/integrations/core/relevance/engine.tiered.ts`
- **Lines**: 103, 118
- **Category**: `Record<string, ...>`
- **Impact**: Medium
- **Description**: `cursorState?: Record<string, unknown>` for tiered selection state. The cursor state's shape is implicitly versioned (checked via `cursorState["v"] !== 1`) but the type doesn't express this.
- **Suggestion**: Define a versioned discriminated union: `type TieredCursorState = { v: 1; tierIndex: number; offset: number } | { v: undefined }` and use it instead of `Record<string, unknown>`.
- **Evidence**: `const nextCursorState: Record<string, unknown> = { v: 1, tierIndex, offset };` and `cursorState?.["v"] !== 1` reset.

---

### 3

- **File**: `apps/api/src/integrations/core/relevance/plan.types.ts`
- **Line**: 25
- **Category**: Unsafe `unknown`
- **Impact**: Medium
- **Description**: `TierPlan.fetch: (ctx: TierPlanContext, cursor?: unknown) => ...` — the cursor parameter is `unknown`, forcing every `fetch` implementation to cast or narrow its own cursor before use.
- **Suggestion**: Make `TierPlan` generic: `TierPlan<TCursor = unknown>` so platforms can express their cursor type and get compile-time checking.
- **Evidence**: `fetch: (ctx: TierPlanContext, cursor?: unknown) => Promise<TierFetchResult>;`

---

### 4

- **File**: `apps/api/src/integrations/core/relevance/plan.types.ts`
- **Line**: 52
- **Category**: `Record<string, ...>`
- **Impact**: Low
- **Description**: `cursorState: Record<string, unknown>` in `SelectionResult` — consistent with the engine but still too wide.
- **Suggestion**: Use the versioned cursor state type suggested in finding 2, or at minimum `Record<string, JsonPrimitive>`.
- **Evidence**: `cursorState: Record<string, unknown>;`

---
