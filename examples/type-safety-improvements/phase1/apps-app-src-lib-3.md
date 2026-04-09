# Type-Safety Audit: apps-app-src-lib-3

**Files inspected**: 8
**Findings**: 4

## Summary

The diagnostics subsystem has a repeated window-marker cast pattern across four files that could be consolidated into a single augmented interface. `command-palette-context.tsx` has two avoidable `as` casts from imprecise return-type annotations. `connection-store.ts`, `csrf-store.ts`, and `onboarding-context.tsx` are clean.

---

## Findings

### 1

**File**: `apps/app/src/lib/diagnostics/captureConsole.ts`, `captureErrors.ts`, `captureNetwork.ts`, `index.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — functionally correct; but the cast is repeated four times across four files
**Description**: Each diagnostics file uses the pattern `(window as typeof window & Record<string, boolean>)[marker]` to read/write a string-keyed boolean property on `window`. This cast is necessary because `window` does not declare these properties, but the identical workaround is duplicated in every file.
**Suggestion**: Declare a single module augmentation (e.g. in `apps/app/src/lib/diagnostics/types.ts` or a new `diagnostics-window.d.ts`):

```typescript
interface Window {
  __SLOPWEAVER_CONSOLE_CAPTURE_ACTIVE__: boolean;
  __SLOPWEAVER_ERROR_CAPTURE_ACTIVE__: boolean;
  __SLOPWEAVER_NETWORK_CAPTURE_ACTIVE__: boolean;
  __SLOPWEAVER_DIAGNOSTICS_INITIALIZED__: boolean;
}
```

Then all four files can drop the `as` cast entirely.
**Evidence**:

- `captureConsole.ts` lines 34, 37: `(window as typeof window & Record<string, boolean>)[marker]`
- `captureErrors.ts`: same pattern
- `captureNetwork.ts` line 61: same pattern
- `index.ts`: same pattern

---

### 2

**File**: `apps/app/src/lib/diagnostics/export.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low
**Description**: `window as Window & { requestIdleCallback?: (cb: () => void) => void }` in `scheduleDiagnosticsInit`. The `requestIdleCallback` API is now broadly supported; alternatively, a dedicated `.d.ts` augmentation could declare it.
**Suggestion**: Add `requestIdleCallback?: (callback: IdleRequestCallback, options?: IdleRequestOptions) => number;` to the `Window` interface augmentation described in finding 1. This removes the inline cast.
**Evidence**: `(window as Window & { requestIdleCallback?: (cb: () => void) => void }).requestIdleCallback`

---

### 3

**File**: `apps/app/src/lib/diagnostics/types.ts`
**Category**: `Record<string, ...>` / loose string types where stricter unions exist
**Impact**: Low — no runtime impact; weakens autocomplete and exhaustiveness checking
**Description**: (a) `DiagnosticsFeatureState.billing.provider: string | null` could be narrowed to `"ios" | "android" | "macos" | "direct" | null` since the billing provider is a finite set. (b) `DiagnosticsBundle.connectivity.connectionStatus: string` could use the `ConnectionStatus` union type from `connection-store.ts` (`"connected" | "partial" | "disconnected" | "unknown"`).
**Suggestion**: Import the `ConnectionStatus` type from `@/lib/connection-store` and the platform union from `@/lib/billing/native-iap` (or a shared types file), and use them in the diagnostics type declarations.
**Evidence**:

- `DiagnosticsFeatureState.billing.provider: string | null`
- `DiagnosticsBundle.connectivity.connectionStatus: string`

---

### 4

**File**: `apps/app/src/lib/command-palette-context.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the casts resolve a union-narrowing problem that could be addressed with overloads or a discriminated union
**Description**: Line 176 casts `result as Promise<void>` and line 224 casts `result as CommandActionResult | void`. Both casts exist because `CommandAction.handler` returns `Promise<CommandActionResult | void>` but the caller checks `result instanceof Promise` first. The narrowing logic is correct but TS cannot prove it without the cast.
**Suggestion**: Redesign the `handler` return type as `CommandActionResult | void` (synchronous) or narrow by adding an explicit type guard `isPromise<T>` instead of `instanceof Promise`.
**Evidence**:

- Line 176: `result as Promise<void>`
- Line 224: `result as CommandActionResult | void`
