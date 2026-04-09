# Audit: packages/ui/src/molecules (slice 1)

**Files inspected**

- `packages/ui/src/molecules/dialog.tsx`
- `packages/ui/src/molecules/dropdown-menu.tsx`

**Findings**: 1 finding

---

## Summary

`dialog.tsx` has one unsafe cast where `document.activeElement` (typed as `Element | null`) is directly cast to `HTMLElement` without an `instanceof` guard. `dropdown-menu.tsx` is clean.

---

## Findings

### Finding 1: `document.activeElement as HTMLElement` without `instanceof` guard

- **File**: `packages/ui/src/molecules/dialog.tsx` (line 100)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Medium — `document.activeElement` can be an `SVGElement` or other non-`HTMLElement` DOM node; calling `.focus()` on those is safe via the `HTMLElement` interface, but the cast suppresses any downstream type mismatch if the element is used for more than just `.focus()`
- **Description**: Inside `handleOpenChange`, the code stores `document.activeElement as HTMLElement` in a ref and later calls `.focus()` on it. `document.activeElement` is typed as `Element | null`. The existing null check on usage (`if (elementToFocus && typeof elementToFocus.focus === "function")`) already guards against null and non-focusable elements at runtime, but the cast at the assignment site is unnecessarily broad.
- **Suggestion**: Replace the cast with a proper narrowing:
  ```typescript
  const active = document.activeElement;
  previousActiveElementRef.current = active instanceof HTMLElement ? active : null;
  ```
  This eliminates the cast and retains the `HTMLElement` type on the ref with a proper guard.
- **Evidence**: `previousActiveElementRef.current = document.activeElement as HTMLElement;`
