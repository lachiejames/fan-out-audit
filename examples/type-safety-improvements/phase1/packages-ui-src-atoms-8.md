# Audit: packages/ui/src/atoms (slice 8)

**Files inspected**

- `packages/ui/src/atoms/flash.tsx`
- `packages/ui/src/atoms/breadcrumb.tsx`
- `packages/ui/src/atoms/pagination.tsx`
- `packages/ui/src/atoms/aspect-ratio.tsx`
- `packages/ui/src/atoms/resizable.tsx`
- `packages/ui/src/atoms/sonner.tsx`

**Findings**: 1 finding

---

## Summary

`flash.tsx` uses the same polymorphic `as` prop pattern as `Box` and `Text`, requiring the same structural casts. The remaining five atoms are clean wrappers with no type-safety issues.

---

## Findings

### Finding 1: Polymorphic `as` prop cast in flash.tsx

- **File**: `packages/ui/src/atoms/flash.tsx`
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — same structural limitation as `Box` and `Text`; cast is unavoidable without a complete overload-based polymorphic API
- **Description**: Like `Box` and `Text`, `Flash` accepts an `as` prop to render as different HTML elements. The implementation must cast the runtime element value to `React.ElementType` and the ref to `React.Ref<HTMLElement>` to satisfy JSX. This is the same known limitation documented in `box.tsx`.
- **Suggestion**: Consistent with `Box` and `Text`, add a comment citing the same TypeScript limitation so the three polymorphic atoms are self-documenting as a group.
- **Evidence**: Pattern mirrors `const Component = as as React.ElementType` seen in `box.tsx`
