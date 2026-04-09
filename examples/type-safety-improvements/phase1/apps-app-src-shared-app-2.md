# Audit: apps-app-src-shared-app-2

**Files inspected**: 8
**Findings**: 5

## Summary

This slice covers shared utility files including billing, clipboard, integration helpers, platform runtime detection, and void-async. The main issues are `as` casts on OAuth error bodies, a `Record<string, string>` used for AppDistribution env casting, unsafe `as TauriRuntime` casting to construct a synthetic runtime, and filter-array casts required by TypeScript but avoidable with better typing.

---

## Findings

### Finding 1: `as { message?: string[] }` cast on OAuth error response

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/utils/integration-helpers.ts:164`
- **Category**: type-cast
- **Impact**: medium
- **Description**: When the OAuth init call returns a non-200 status, `response.body` is cast to `{ message?: string[] }`. The ts-rest client should have the error body typed from the contract's error responses (`standardResponses`). Using `as` bypasses that narrowing.
- **Suggestion**: Check the inferred type of `response.body` in the non-200 branch from the ts-rest client. If the contract uses `standardResponses`, the body should already be typed as `{ message: string[]; ... }`. Use the narrowed type directly rather than casting.
- **Evidence**: `const errorBody = response.body as { message?: string[] };`

### Finding 2: `as [string, string][]` cast on `Object.entries` filter result

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/utils/integration-helpers.ts:200,245`
- **Category**: type-cast
- **Impact**: low
- **Description**: `Object.entries(queryParams).filter(([, value]) => value !== undefined) as [string, string][]` requires a cast because TypeScript cannot prove after `.filter()` that the remaining values are non-undefined. This cast is technically correct at runtime but forces a trust assertion.
- **Suggestion**: Use a typed filter-map: `Object.entries(queryParams).flatMap(([k, v]) => v !== undefined ? [[k, String(v)] as [string, string]] : [])` — no cast needed. Alternatively, use a type predicate `(entry): entry is [string, string] => entry[1] !== undefined`.
- **Evidence**: `Object.entries(queryParams).filter(([, value]) => value !== undefined) as [string, string][]`

### Finding 3: `as AppDistribution | undefined` cast for build-time env var

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/utils/platform-runtime.ts:347-348`
- **Category**: type-cast
- **Impact**: low
- **Description**: `import.meta.env["VITE_APP_DISTRIBUTION"] as AppDistribution | undefined` casts without validation. An invalid build-time string would silently produce an invalid `AppDistribution` value that passes all downstream checks.
- **Suggestion**: Add a validation step: check that the value is one of `["direct", "app_store", "play_store"]` before using it, and fall back to `"direct"` if it is not. This is a build-time constant, so a one-time validation on startup is cheap.
- **Evidence**: `const APP_DISTRIBUTION: AppDistribution = (import.meta.env["VITE_APP_DISTRIBUTION"] as AppDistribution | undefined) ?? "direct";`

### Finding 4: Synthetic `TauriRuntime` construction uses `as` to force type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/utils/platform-runtime.ts:273-275`
- **Category**: type-cast
- **Impact**: low
- **Description**: `invokeBridge` is cast to `TauriRuntime["core"]["invoke"]` to satisfy the `getTauriRuntime` return type while constructing a synthetic runtime from `__TAURI_INTERNALS__`. This is a known shim and works correctly — but if the `Window["__TAURI__"]` type ever adds more required members to `core.invoke`, the cast will hide the incompatibility.
- **Suggestion**: This is a legitimate shim for a real interop boundary. Document explicitly with a comment why the cast is needed (`// __TAURI_INTERNALS__ lacks full type but invoke signature is compatible`) and consider a `satisfies` check to at least verify the signature shape at compile time.
- **Evidence**: `const invokeBridge = (...) as TauriRuntime["core"]["invoke"];`

### Finding 5: `as TauriInvoke` cast when checking `__TAURI_INTERNALS__`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/utils/platform-browser.ts:74-76`
- **Category**: type-cast
- **Impact**: low
- **Description**: `(internals as { invoke?: unknown }).invoke` is extracted and then cast to `TauriInvoke` with `return maybeInvoke as TauriInvoke`. The intermediate extraction into `maybeInvoke: unknown` followed by the cast means there's no structural validation of the function signature.
- **Suggestion**: The runtime `typeof maybeInvoke === "function"` check is already present, which is the best we can do without calling the function. This is acceptable; add a JSDoc comment noting the runtime invariant that this is the Tauri IPC bridge.
- **Evidence**: `return maybeInvoke as TauriInvoke;`
