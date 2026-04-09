# Audit: packages/ui/src/organisms (slice 3)

**Files inspected**

- `packages/ui/src/organisms/inbox/EmailIframeViewer.tsx`
- `packages/ui/src/organisms/inbox/InboxFilters.tsx`
- `packages/ui/src/organisms/inbox/MessageActionBar.tsx`
- `packages/ui/src/organisms/inbox/MessageCard.tsx`
- `packages/ui/src/organisms/inbox/platform-views/GmailThreadView.tsx`
- `packages/ui/src/organisms/inbox/platform-views/LinearIssueView.tsx`
- `packages/ui/src/organisms/inbox/platform-views/SlackThreadView.tsx`

**Findings**: 5 findings

---

## Summary

`InboxFilters.tsx` repeats the `T | null` pattern and also has a duplicate `AdvancedCondition` type definition. `MessageActionBar.tsx` and `MessageCard.tsx` use `platform: string` where `PlatformId | string` would improve autocomplete. `LinearIssueView.tsx` has `PRIORITY_CONFIG` colors using light-mode Tailwind classes that will not work in the dark-first design system. `GmailThreadView.tsx` uses a non-null assertion.

---

## Findings

### Finding 1: `InboxFiltersProps` uses `T | null` for optional props

- **File**: `packages/ui/src/organisms/inbox/InboxFilters.tsx` (lines 105–118)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Medium — same `T | null` pattern as `AiPresenceLogo` and `CalendarStrip`; inconsistent with standard React optional prop convention
- **Description**: Six props (`savedFilters`, `onSaveFilter`, `onUpdateFilter`, `onDeleteSavedFilter`, `onLoadSavedFilter`, `isLoading`, `className`) are typed as `T | null` instead of optional `T?`.
- **Suggestion**: Convert to optional props with default parameter values.
- **Evidence**: `savedFilters: SavedFilter[] | null;`, `className: string | null;`

---

### Finding 2: `AdvancedCondition` interface duplicated in InboxFilters.tsx

- **File**: `packages/ui/src/organisms/inbox/InboxFilters.tsx` (lines 54–61)
- **Category**: Duplicate type definitions across components
- **Impact**: Medium — `AdvancedCondition` is also defined in `foundations/utils/condition-helpers.ts` with an identical structure (including the `pro` field). Two independent definitions can silently diverge.
- **Description**: Both `InboxFilters.tsx` and `condition-helpers.ts` define `AdvancedCondition` independently. The `InboxFilters.tsx` version re-declares the same `field`, `pro`, and `value` fields. Only one definition should exist.
- **Suggestion**: Import `AdvancedCondition` from `condition-helpers.ts` rather than re-declaring it in `InboxFilters.tsx`.
- **Evidence**: Identical `interface AdvancedCondition { field: AdvancedConditionField; pro: AdvancedConditionOperator; value: string; }` in both files

---

### Finding 3: `platform: string` in `MessageActionBarProps` and `MessageCardData`

- **File**: `packages/ui/src/organisms/inbox/MessageActionBar.tsx` (line 12), `packages/ui/src/organisms/inbox/MessageCard.tsx` (line 33)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — both files already call `isPlatformId(platform)` before using the platform value, so the runtime behaviour is correct; the string type just misses autocomplete
- **Description**: Both `MessageActionBarProps.platform` and `MessageCardData.platform` are typed as `string`. Using `PlatformId | (string & {})` would provide autocomplete for known platform IDs while still accepting arbitrary strings.
- **Suggestion**: Type `platform` as `PlatformId | (string & {})` in both interfaces.
- **Evidence**: `platform: string;` in both prop interfaces

---

### Finding 4: `getPlatformAccentColor` uses `platform as PlatformId` after `isPlatformId` guard

- **File**: `packages/ui/src/organisms/inbox/MessageCard.tsx` (line 119)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — the cast is directly after an `isPlatformId` guard; the only reason it's needed is because `isPlatformId` doesn't narrow the local variable `platform` in all TypeScript configurations
- **Description**: `PLATFORMS[platform as PlatformId]` appears inside an `if (!isPlatformId(platform)) return null;` early-return guard. After the guard, `platform` should be narrowed to `PlatformId`. If `isPlatformId` is declared as a proper type predicate (`platform is PlatformId`), the cast becomes unnecessary.
- **Suggestion**: Verify that `isPlatformId` in `@slopweaver/contracts` is declared as `function isPlatformId(value: unknown): value is PlatformId`. If it is, remove the cast.
- **Evidence**: `return PLATFORMS[platform as PlatformId].display.brandColor;`

---

### Finding 5: `emails.at(-1)!.isStarred` non-null assertion in GmailThreadView.tsx

- **File**: `packages/ui/src/organisms/inbox/platform-views/GmailThreadView.tsx` (line 557)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — the assertion is guarded by a `typeof emails.at(-1)?.isStarred === "boolean"` check, so runtime null access cannot occur; the compiler just doesn't narrow through the conditional spread
- **Description**: `emails.at(-1)!.isStarred` uses a non-null assertion inside a conditional spread expression. The spread pattern `{...(typeof emails.at(-1)?.isStarred === "boolean" ? { isStarred: emails.at(-1)!.isStarred } : {})}` is overly complex.
- **Suggestion**: Extract `const lastEmail = emails.at(-1)` before the JSX return to avoid repeated `.at(-1)` calls and eliminate the non-null assertion:
  ```typescript
  const lastEmail = emails.at(-1);
  // ...
  {...(lastEmail?.isStarred !== undefined ? { isStarred: lastEmail.isStarred } : {})}
  ```
- **Evidence**: `emails.at(-1)!.isStarred`
