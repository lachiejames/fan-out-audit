# Type-Safety Audit: apps-app-src-hooks-11

Files audited:

- `apps/app/src/hooks/useWorkItemActions.ts`
- `apps/app/src/hooks/useZoomPersistence.ts`

---

## Finding 1

**File:** `apps/app/src/hooks/useWorkItemActions.ts`

No significant type-safety issues found for the mutation logic itself. The hook correctly uses typed `AcceptWorkItemParams`, `RejectWorkItemParams`, and `RegenerateWorkItemParams` interfaces, checks response status codes before accessing bodies, and uses `ApiResponseError` for error propagation.

One minor observation: the `WorkItemMutationContext` type uses `data: unknown` in the snapshot tuple `[queryKey: readonly unknown[], data: unknown][]`. This is a structural limitation of how `queryClient.getQueriesData` is typed — the `unknown` for `data` is unavoidable at the TanStack Query API level without a generic parameter. The three mutations do provide explicit generics where possible (`getQueriesData<{ counts: MessageCounts; messages: InboxMessage[]; total: number }>`), which is the correct approach.

---

## Finding 2

**File:** `apps/app/src/hooks/useZoomPersistence.ts`

**Category:** `any` or unsafe `unknown` in production code

**Impact:** Low

**Description:**
`cachedTauriUpdate: unknown = null` is used to cache the Tauri SDK `Update` object returned by `check()`. In `downloadAndInstallAppUpdate`, the variable is cast to an inline object type with `downloadAndInstall`. The `unknown` type forces a cast that must be manually maintained in sync with the Tauri SDK's actual type.

This is present in `apps/app/src/lib/app-updates/orchestrator.ts` (not in `useZoomPersistence.ts` itself). `useZoomPersistence.ts` is clean — it properly uses `ZoomStore` interface abstraction and has no unsafe casts. The file uses `isValidStoredZoom` as a type predicate (`value is number`) correctly.

No findings for `useZoomPersistence.ts`.

---

## Summary for Slice 11

`useWorkItemActions.ts` is well-typed relative to other hooks in this audit. The snapshot tuple's `data: unknown` is a TanStack Query API constraint, not a code quality issue. `useZoomPersistence.ts` is clean with no type-safety concerns.
