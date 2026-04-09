# Audit: packages/ui/src/atoms (slice 4)

**Files inspected**

- `packages/ui/src/atoms/scroll-area.tsx`
- `packages/ui/src/atoms/skeleton.tsx`
- `packages/ui/src/atoms/input.tsx`
- `packages/ui/src/atoms/textarea.tsx`

**Findings**: 2 findings

---

## Summary

`scroll-area.tsx` still uses `React.forwardRef` while the rest of the codebase has migrated to React 19 ref-as-prop. `skeleton.tsx` has a duplicate CSS class expression. `input.tsx` and `textarea.tsx` are clean.

---

## Findings

### Finding 1: `React.forwardRef` in scroll-area — inconsistent with React 19 ref-as-prop

- **File**: `packages/ui/src/atoms/scroll-area.tsx` (lines 17, 49)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — `forwardRef` is still valid in React 19, but inconsistency with the rest of the design system creates confusion about which pattern to follow
- **Description**: `ScrollArea` and `ScrollBar` both use `React.forwardRef(...)`. The rest of the design system (`Button`, `Input`, `Table`, etc.) pass `ref` as a plain prop using the React 19 pattern. Two different ref patterns in the same package is a maintenance risk.
- **Suggestion**: Migrate `ScrollArea` and `ScrollBar` to the React 19 ref-as-prop pattern consistent with `Button`, `Input`, and `Table`.
- **Evidence**: `const ScrollArea = React.forwardRef<HTMLDivElement, ScrollAreaProps>(({ ... }, ref) => ...)`

---

### Finding 2: Duplicate `animate-pulse` class expression in skeleton.tsx

- **File**: `packages/ui/src/atoms/skeleton.tsx` (lines 52–53)
- **Category**: Missing explicit prop types / inferred-any leaking (code correctness)
- **Impact**: Low — produces redundant output but does not break anything
- **Description**: The expression `animation === "pulse" && "animate-pulse"` appears twice consecutively in the `cn()` call. This is a copy-paste error; one occurrence is sufficient.
- **Suggestion**: Remove the duplicate occurrence.
- **Evidence**:
  ```typescript
  animation === "pulse" && "animate-pulse",
  animation === "pulse" && "animate-pulse",
  ```
