# Audit: packages/ui/src/atoms (slice 3)

**Files inspected**

- `packages/ui/src/atoms/PriorityBadge.tsx`
- `packages/ui/src/atoms/badge.tsx`
- `packages/ui/src/atoms/button.tsx`

**Findings**: 2 findings

---

## Summary

`PriorityBadge` has inline size-keyed maps that duplicate a `"sm" | "md" | "lg"` union pattern used across atoms. `badge.tsx` and `button.tsx` use CVA correctly and expose `VariantProps` — no significant issues found there.

---

## Findings

### Finding 1: Inline size-keyed maps duplicate the size union in PriorityBadge

- **File**: `packages/ui/src/atoms/PriorityBadge.tsx` (lines 121–130)
- **Category**: Duplicate type definitions across components
- **Impact**: Low — the duplication is in an implementation detail (map literals), not an exported type
- **Description**: `sizeClasses` and `iconSizes` are inline `Record<"sm" | "md" | "lg", string>` objects. This `"sm" | "md" | "lg"` union is independently declared in at least three atoms (`PriorityBadge`, `Badge`, `Button`). A shared `type SizeVariant = "sm" | "md" | "lg"` in foundations would be a single source of truth.
- **Suggestion**: Extract `type SizeVariant = "sm" | "md" | "lg"` to `packages/ui/src/foundations/utils/type-guards.ts` or a new `types.ts` and import it in components that reuse the same three-way size scale.
- **Evidence**:
  ```typescript
  const sizeClasses: Record<"sm" | "md" | "lg", string> = { ... };
  const iconSizes: Record<"sm" | "md" | "lg", string> = { ... };
  ```

---

### Finding 2: No findings in badge.tsx / button.tsx

- **File**: `packages/ui/src/atoms/badge.tsx`, `packages/ui/src/atoms/button.tsx`
- **Category**: N/A
- **Impact**: N/A
- **Description**: Both files use CVA with `VariantProps` correctly, export explicit types, and have no `any`, unconstrained `Record<string, ...>`, or redundant casts. These serve as good reference implementations.
- **Suggestion**: No action needed.
- **Evidence**: `type BadgeProps = React.ComponentProps<"span"> & VariantProps<typeof badgeVariants>`
