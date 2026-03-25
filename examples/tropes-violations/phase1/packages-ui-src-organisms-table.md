# Slice 200: packages/ui/src/organisms/table (use-table)

Files audited:

- `packages/ui/src/organisms/table/use-table.tsx`
- `packages/ui/src/organisms/table/row.ts`
- `packages/ui/src/organisms/table/column.ts`
- `packages/ui/src/organisms/table/sorting.ts`
- `packages/ui/src/organisms/table/utils.ts`

---

## use-table.tsx

Infrastructure hook for table state management. No user-facing strings — only internal error messages thrown for developer misuse (e.g. `"Expected either an 'id' or 'field' to be defined for a Column"`). These are programming errors, not UI copy. No violations.

---

## row.ts

Type definition only (`UniqueRow` interface). No strings. No violations.

---

## column.ts

Column type definitions and `createColumnHelper` factory. No user-facing strings. No violations.

---

## sorting.ts

Sort utility functions and type definitions. No user-facing strings. No violations.

---

## utils.ts

TypeScript utility types (`ObjectPaths`, `ObjectPathValue`). No strings at all. No violations.

---

## Findings

No AI writing tropes found in this slice.
