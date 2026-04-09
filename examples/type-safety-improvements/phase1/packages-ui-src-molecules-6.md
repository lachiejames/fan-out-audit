# Audit: packages/ui/src/molecules (slice 6)

**Files inspected**

- `packages/ui/src/molecules/input-otp.tsx`
- `packages/ui/src/molecules/calendar.tsx` (molecule-level)
- `packages/ui/src/molecules/carousel.tsx`
- `packages/ui/src/molecules/date-picker.tsx`
- `packages/ui/src/molecules/skeleton.tsx` (molecule-level)
- `packages/ui/src/molecules/toggle-group.tsx`
- `packages/ui/src/molecules/hover-card.tsx`
- `packages/ui/src/molecules/aspect-ratio.tsx`

**Findings**: 0 findings

---

## Summary

All eight molecules in this slice are either thin wrappers (re-exporting atom-level components) or straightforward compositions with explicit prop types and no casts. No type-safety improvements needed.
