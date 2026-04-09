# Audit: apps/app/src/components — Slice 11 (inbox components)

**Files inspected**: 8
**Findings**: 6

## Summary

The inbox components contain two high-impact issues: unsafe string-split casts to construct sort tuples (duplicated across two files), and a local `Message` interface using `string` for the `platform` field. There is also a duplicate `iconMap` pattern from the ai-suggestion-chips file appearing again here.

## Findings

### Finding 1: String-split cast to sort tuple in `inbox-filter-bar.tsx`

- **File**: `apps/app/src/components/inbox/inbox-filter-bar.tsx:357`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: high
- **Description**: `const [sortBy, sortDirection] = v.split("-") as ["timestamp" | "priorityScore", "asc" | "desc"]` casts the result of `String.prototype.split()` directly to a typed tuple. If the input string `v` does not follow the expected format (e.g. a Select option value changes), the cast silently produces incorrect typed values with no runtime error.
- **Suggestion**: Replace the cast with explicit runtime validation. Define a helper: `function parseSortValue(v: string): { sortBy: 'timestamp' | 'priorityScore'; sortDirection: 'asc' | 'desc' } | null`. Parse the split parts, validate each against the known union, and return null if invalid. At the call site, either throw or fall back to a default sort. This eliminates the unsafe cast entirely.
- **Evidence**: `v.split("-") as ["timestamp" | "priorityScore", "asc" | "desc"]` at line 357.

### Finding 2: Same string-split cast pattern duplicated in `mobile-filter-sheet.tsx`

- **File**: `apps/app/src/components/inbox/mobile-filter-sheet.tsx:308`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: high
- **Description**: Identical cast pattern to Finding 1: `v.split("-") as [...]`. This is a cross-file duplicate of the same unsafe pattern.
- **Suggestion**: Extract the `parseSortValue` helper (from Finding 1's suggestion) to a shared module (e.g. `apps/app/src/components/inbox/inbox-filter.utils.ts`) and use it in both `inbox-filter-bar.tsx` and `mobile-filter-sheet.tsx`. This fixes the type safety issue and eliminates the duplication in one step.
- **Evidence**: `v.split("-") as ["timestamp" | "priorityScore", "asc" | "desc"]` at line 308.

### Finding 3: Local `Message` interface uses `string` for `platform` field

- **File**: `apps/app/src/components/inbox/message-card.tsx:15-37`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: high
- **Description**: The local `Message` interface has `platform: string`. Messages come from known platforms and this field should be `PlatformId` from `@slopweaver/contracts`. Additionally, this local `Message` interface likely duplicates or diverges from a `MessageResponse` type in contracts.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and change `platform: string` to `platform: PlatformId`. Then check whether contracts exports a `MessageResponse` type that covers the same shape — if so, replace the local interface with an import or a type derived from it.
- **Evidence**: `platform: string` in the local `Message` interface at lines 15-37.

### Finding 4: `iconMap` in inbox uses `Record<string, React.ReactNode>` with closed key set

- **File**: `apps/app/src/components/inbox/compound-suggestion-chip.tsx:19`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: low
- **Description**: `const iconMap: Record<string, React.ReactNode>` is the same closed-key-set pattern seen in `ai-suggestion-chips.tsx` (slice 2). The two maps cover different icon sets but have the same type weakness.
- **Suggestion**: Define a union type for the valid icon name strings used in inbox suggestion chips and type the map as `Partial<Record<InboxSuggestionIconName, React.ReactNode>>`. If many icon maps follow this pattern, consider a shared pattern for icon registry types.
- **Evidence**: `const iconMap: Record<string, React.ReactNode>` at line 19.

### Finding 5: Inbox filter values not sharing a canonical type with the filter bar and sheet

- **File**: `apps/app/src/components/inbox/inbox-filter-bar.tsx` and `apps/app/src/components/inbox/mobile-filter-sheet.tsx`
- **Category**: Duplicate type definitions
- **Impact**: medium
- **Description**: Both the desktop filter bar and mobile filter sheet define or assume the same set of sort/filter values (sort field names, sort directions, filter categories). There is no shared type definition — each component implicitly knows the values through its JSX option values and the casts in Findings 1 and 2.
- **Suggestion**: Define an `InboxFilterConfig` type or a set of const arrays (`SORT_BY_OPTIONS`, `SORT_DIRECTION_OPTIONS`) in a shared `inbox-filter.utils.ts` file. Derive union types from these using `typeof SORT_BY_OPTIONS[number]`. Both components import from this shared source.
- **Evidence**: Parallel sort/filter value handling in both filter components with no shared type source.

### Finding 6: `platform` fields in inbox message list data typed as `string`

- **File**: `apps/app/src/components/inbox/` (multiple inbox components in slice 11)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: Several inbox components (beyond `message-card.tsx`) accept or pass `platform` values as plain `string` in props or filter state. This pattern is pervasive in the inbox and represents a systemic gap where `PlatformId` from `@slopweaver/contracts` should be used.
- **Suggestion**: Audit all inbox component props with a `platform` field and replace `string` with `PlatformId`. This is a single import that provides exhaustiveness checking across all platform-conditional rendering logic in the inbox.
- **Evidence**: `platform: string` in multiple inbox component props and filter state types.
