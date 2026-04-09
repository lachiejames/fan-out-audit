# Audit: packages/ui/src/molecules (slice 4)

**Files inspected**

- `packages/ui/src/molecules/select.tsx`
- `packages/ui/src/molecules/inline-message.tsx`
- `packages/ui/src/molecules/flash.tsx` (molecule-level re-exports if any)
- `packages/ui/src/molecules/enhanced-card.tsx`
- `packages/ui/src/molecules/EntityBadge/EntityBadge.tsx`

**Findings**: 1 finding

---

## Summary

`enhanced-card.tsx` has a single-member CVA variant that provides no type narrowing value. The other molecules are clean.

---

## Findings

### Finding 1: Single-member `variant` union in `enhancedCardVariants`

- **File**: `packages/ui/src/molecules/enhanced-card.tsx` (line 13)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — not a runtime issue, but a design signal that the variant system was planned but not yet implemented
- **Description**: `enhancedCardVariants` defines `variant: { default: "..." }` with only one option. A single-member CVA variant is semantically meaningless — `variant` can only ever be `"default"`, making it equivalent to a constant. It adds noise to the props interface without providing any type-checking benefit.
- **Suggestion**: Either remove the `variant` prop entirely (simplifying the API) or extend it with the additional variants that were presumably planned when it was introduced (`"elevated"`, `"outlined"`, etc.).
- **Evidence**: `variants: { variant: { default: "rounded-xl border border-border bg-card ..." } }`
