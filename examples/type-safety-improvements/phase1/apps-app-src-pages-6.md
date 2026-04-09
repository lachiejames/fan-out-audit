# Audit: apps-app-src-pages-6

**Files inspected**: 3
**Findings**: 4

## Summary

This slice covers the three inbox page hooks. The main issues are `as SomeType` casts when parsing URL search params (bypassing actual validation), a locally duplicated `SelectedMessageOrigin` union type defined twice (once in `useInboxDeepLink.ts` and once in `useInboxPage.ts`), and an unsafe `as Content` cast for query-cache retrieval.

---

## Findings

### Finding 1: Duplicate `SelectedMessageOrigin` type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/useInboxDeepLink.ts:17` and `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/useInboxPage.ts:38`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `type SelectedMessageOrigin = "list" | "deeplink" | null` is defined identically in both files. If the allowed origin values change (e.g. a new `"shortcut"` value is added), both definitions must be updated in sync.
- **Suggestion**: Export `SelectedMessageOrigin` from `useInboxDeepLink.ts` (or a shared types file) and import it in `useInboxPage.ts`.
- **Evidence**: Both files contain `type SelectedMessageOrigin = "list" | "deeplink" | null;`

### Finding 2: `as InboxFilters["folder"]` cast without validation

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/useInboxFilters.ts:31`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `searchParams.get("folder")` returns `string | null`. This is cast immediately to `InboxFilters["folder"] | null` via `as`. The code does subsequently validate against `["inbox", "archived", "all"]`, but the cast happens before the check — so TypeScript trusts the type before it has been proven.
- **Suggestion**: Remove the upfront cast and perform the membership check first: `const validFolders: InboxFilters["folder"][] = ["inbox", "archived", "all"]; const folder = validFolders.includes(rawFolder) ? rawFolder : null;`. This makes the validation visible to the type system.
- **Evidence**: `const folder = searchParams.get("folder") as InboxFilters["folder"] | null;` (same pattern applied to `status`, `priority`, `sortBy`, `sortDirection`, `view` on lines 33–37).

### Finding 3: Multiple `as InboxFilters[...]` casts for URL params

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/useInboxFilters.ts:33-37`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Five additional URL params (`status`, `priority`, `sortBy`, `sortDirection`, `view`) are cast from `string | null` to their InboxFilters union types before validation. Each subsequently has a manual inclusion check, but the casts precede the guards.
- **Suggestion**: Create a small helper `castIfValid<T extends string>(value: string | null, allowed: readonly T[]): T | null` and use it for all five params. This eliminates the `as` cast entirely.
- **Evidence**: Lines 33–37 of `useInboxFilters.ts`.

### Finding 4: `as Content` cast for query cache retrieval

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/useInboxPage.ts:227`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `queryClient.getQueryData(...)` returns `unknown`. The result is cast directly to `Content` before being passed to `buildInboxMessageDetail`. This skips runtime validation — if the cache holds a stale or incompatible shape (e.g. after a contract update), the cast will silently pass invalid data through.
- **Suggestion**: Check the cached value with a type guard or a Zod parse before passing it to `buildInboxMessageDetail`. At minimum narrow with a simple `instanceof`-style guard or check for required fields before casting.
- **Evidence**: `return buildInboxMessageDetail({ apiMessage: cached as Content });`
