# Audit: packages/ui/src/foundations (slice 1)

**Files inspected**

- `packages/ui/src/foundations/hooks/use-responsive-value.tsx`
- `packages/ui/src/foundations/hooks/match-media.tsx`
- `packages/ui/src/foundations/hooks/use-media.tsx`

**Findings**: 2 findings

---

## Summary

`use-responsive-value.tsx` contains multiple structurally sound but verbose casts that arise from a complex conditional type. `match-media.tsx` and `use-media.tsx` define identical `MediaQueryFeatures` interfaces independently.

---

## Findings

### Finding 1: Duplicated `MediaQueryFeatures` interface across two hook files

- **File**: `packages/ui/src/foundations/hooks/match-media.tsx` (lines 11–13) and `packages/ui/src/foundations/hooks/use-media.tsx` (lines 8–10)
- **Category**: Duplicate type definitions across components
- **Impact**: Medium — any change to the interface must be made in both files; they can silently diverge
- **Description**: Both files independently declare:
  ```typescript
  interface MediaQueryFeatures {
    readonly [key: string]: boolean | undefined;
  }
  ```
  This is a textually identical definition in two files that are clearly related (both implement media query logic).
- **Suggestion**: Extract `MediaQueryFeatures` to a single shared location (e.g., `packages/ui/src/foundations/hooks/media-query-types.ts`) and import it in both files.
- **Evidence**: Identical `interface MediaQueryFeatures { readonly [key: string]: boolean | undefined; }` in both files

---

### Finding 2: Multiple casts in `flattenResponsiveValue` / `useResponsiveValue`

- **File**: `packages/ui/src/foundations/hooks/use-responsive-value.tsx` (lines 95–141)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — all casts are guarded by null checks or `instanceof` checks; they are structurally sound
- **Description**: The `flattenResponsiveValue` helper uses six type casts: `responsiveValue.narrow as TNarrow`, `responsiveValue.wide as TWide`, `responsiveValue.regular as TRegular`, `value as ResponsiveValue<unknown, unknown, unknown>`, `return value as Exclude<...>`, and `as FlattenResponsiveValue<T> | F`. These arise because TypeScript cannot narrow through the `ResponsiveValue` discriminated union without explicit narrowing.
- **Suggestion**: Refactor the three property accesses (`narrow`, `wide`, `regular`) to use explicit narrowing checks (`"narrow" in responsiveValue` etc.) before accessing, which would eliminate those three casts. The final return-type casts are harder to avoid with the current generic design.
- **Evidence**:
  ```typescript
  responsiveValue.narrow as TNarrow;
  responsiveValue.wide as TWide;
  responsiveValue.regular as TRegular;
  ```
