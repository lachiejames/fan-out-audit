# Audit: packages/ui/src/molecules (slice 3)

**Files inspected**

- `packages/ui/src/molecules/form-control-context.tsx`
- `packages/ui/src/molecules/form-control.tsx`
- `packages/ui/src/molecules/CitationChip/CitationChip.tsx`
- `packages/ui/src/molecules/SuggestionCard/SuggestionCard.tsx`

**Findings**: 5 findings

---

## Summary

`form-control-context.tsx` contains the most unsafe casts in the entire molecules layer: `return externalProps as never` and `useContext(...) ?? ({} as FormControlContextValue)`. `form-control.tsx` casts child props without a type guard. `CitationChip.tsx` has redundant casts that should be unnecessary given the source type. `SuggestionCard.tsx` hardcodes platform IDs as object literal keys.

---

## Findings

### Finding 1: `return externalProps as never` — extreme unsafe cast

- **File**: `packages/ui/src/molecules/form-control-context.tsx` (line 53)
- **Category**: Type cast (`as SomeType`)
- **Impact**: High — `as never` completely silences type checking on the return value; any incompatible type will pass through without a compile error
- **Description**: The `mergeFormControlProps` function returns `externalProps as never` to satisfy a complex overloaded return type. This is the most unsafe cast available; it tells TypeScript the value is of type `never` (which is assignable to everything), completely bypassing type checking on the return.
- **Suggestion**: Refactor the overloads into a single union return type that accurately describes what the function returns. If the function's behaviour is genuinely ambiguous, use `as ReturnType<typeof mergeFormControlProps>` rather than `as never`.
- **Evidence**: `return externalProps as never;`

---

### Finding 2: `useContext(FormControlContext) ?? ({} as FormControlContextValue)` — empty object cast

- **File**: `packages/ui/src/molecules/form-control-context.tsx` (line 30)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Medium — an empty `{}` cast as `FormControlContextValue` means any code relying on the context outside a provider silently receives an object with all fields `undefined`, rather than a clear null/error signal
- **Description**: When `FormControlContext` is not provided, the hook falls back to `{} as FormControlContextValue`. This masks the absence of a provider: callers receive a fake context object with `undefined` for all fields rather than `null` (which would force explicit null handling).
- **Suggestion**: Return `null` when outside the provider and require callers to handle the null case, or use a `createStrictContext` helper that throws a descriptive error.
- **Evidence**: `return useContext(FormControlContext) ?? ({} as FormControlContextValue);`

---

### Finding 3: `child.props as FormControlValidationProps` cast in form-control.tsx

- **File**: `packages/ui/src/molecules/form-control.tsx` (line 87)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — the cast happens after a check for element type but without a typed type guard
- **Description**: `(child.props as FormControlValidationProps).variant` casts child props to a specific type to access the `variant` field. This is avoidable with a type guard function that checks whether the element is a `FormControlValidation` component.
- **Suggestion**: Create a type guard `function isFormControlValidationElement(el: React.ReactElement): el is React.ReactElement<FormControlValidationProps>` and use it to narrow before the prop access.
- **Evidence**: `(child.props as FormControlValidationProps).variant`

---

### Finding 4: Redundant `as React.CSSProperties` casts in CitationChip.tsx

- **File**: `packages/ui/src/molecules/CitationChip/CitationChip.tsx` (lines 105, 119)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — the casts are redundant because `PlatformStyleConfig.chipStyle` is already typed as `React.CSSProperties`
- **Description**: `styles.chipStyle as React.CSSProperties` and `styles.headerStyle as React.CSSProperties` cast values that are already typed as `React.CSSProperties` in the `PlatformStyleConfig` type. The casts add noise without providing any type benefit.
- **Suggestion**: Remove the casts. If the TypeScript compiler requires them, it indicates a type mismatch upstream in `PlatformStyleConfig` that should be fixed at the source.
- **Evidence**: `style={styles.chipStyle as React.CSSProperties}`

---

### Finding 5: Hardcoded platform IDs as object literal keys in SuggestionCard.tsx

- **File**: `packages/ui/src/molecules/SuggestionCard/SuggestionCard.tsx` (lines 25–28)
- **Category**: `Record<string, ...>` where stricter union keys would be safer
- **Impact**: Low — the keys are a hardcoded subset of platforms; if new platforms are added, this type won't be updated automatically
- **Description**: `relatedItems` is typed as `{ slack?: number | null; "google-gmail"?: number | null; linear?: number | null }`. This hardcodes three platform IDs as literal object keys. Using `Partial<Record<PlatformId, number | null>>` would make the type flexible and consistent with the platform system.
- **Suggestion**: Replace the inline type with `Partial<Record<PlatformId, number | null>>` imported from `@slopweaver/contracts`.
- **Evidence**: `relatedItems?: { slack?: number | null; "google-gmail"?: number | null; linear?: number | null }`
