# Audit: packages/ui/src/foundations (slice 4)

**Files inspected**

- `packages/ui/src/foundations/utils/icon-sizes.ts`
- `packages/ui/src/foundations/utils/validation-styles.ts`
- `packages/ui/src/foundations/utils/disabled-styles.ts`
- `packages/ui/src/foundations/utils/focus-ring-styles.ts`
- `packages/ui/src/foundations/utils/animation-helpers.ts`
- `packages/ui/src/foundations/utils/type-guards.ts`
- `packages/ui/src/foundations/utils/breadcrumb-helpers.ts`
- `packages/ui/src/foundations/utils/array-helpers.ts`
- `packages/ui/src/foundations/utils/pagination-helpers.ts`
- `packages/ui/src/foundations/utils/truncate-helpers.ts`
- `packages/ui/src/foundations/utils/datetime-helpers.ts`
- `packages/ui/src/foundations/utils/context-helpers.ts`
- `packages/ui/src/foundations/utils/color-helpers.ts`
- `packages/ui/src/foundations/utils/time-helpers.ts`
- `packages/ui/src/foundations/utils/file-validation-helpers.ts`
- `packages/ui/src/foundations/utils/input-helpers.ts`
- `packages/ui/src/foundations/layout/stack.tsx`
- `packages/ui/src/foundations/layout/grid.tsx`
- `packages/ui/src/foundations/layout/container.tsx`
- `packages/ui/src/foundations/layout/section.tsx`

**Findings**: 1 finding

---

## Summary

`icon-sizes.ts` is an exemplary use of `satisfies`. The remaining utilities and layout components are clean: named params, explicit return types, no `any`, and no unnecessary casts. `stack.tsx` uses the same polymorphic `as` prop pattern as `Box`/`Text` with the same known structural cast.

---

## Findings

### Finding 1: Polymorphic `as` prop cast in Stack/StackItem — consistent with Box/Text pattern

- **File**: `packages/ui/src/foundations/layout/stack.tsx`
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — same structural limitation as `Box`, `Text`, and `Flash`; acknowledged pattern
- **Description**: `Stack` and `StackItem` use the `as` polymorphic prop pattern requiring `as React.ElementType` and `ref as React.Ref<HTMLElement>` casts. This is the fourth component that duplicates this pattern. There is currently no shared polymorphic component helper that encapsulates these casts in one place.
- **Suggestion**: Consider extracting a shared `createPolymorphicComponent` helper (or a `PolymorphicBox` base) that centralises the two cast sites. This would reduce duplication across `Box`, `Text`, `Flash`, `Stack`, and `StackItem`.
- **Evidence**: Same cast pattern as `box.tsx` — `const Component = as as React.ElementType`
