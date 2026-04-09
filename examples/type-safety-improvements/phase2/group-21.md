# Cross-Cutting Patterns (Group 21)

**Slices analyzed**: packages-ui-src-molecules-2, packages-ui-src-molecules-3, packages-ui-src-molecules-4, packages-ui-src-molecules-5, packages-ui-src-molecules-6, packages-ui-src-organisms-1, packages-ui-src-organisms-2, packages-ui-src-organisms-3, packages-ui-src-organisms-4

## Pattern 1: `T | null` instead of optional `T?` for React props

- **Seen in**: organisms-1 (AiPresenceLogo), organisms-2 (CalendarStrip), organisms-3 (InboxFilters), organisms-4 (UnifiedLogoMark)
- **Category**: missing-strict-typing
- **Combined impact**: Medium -- four separate component interfaces use `T | null` instead of the standard optional `T?` pattern, forcing callers to explicitly pass `null` for every unused prop rather than simply omitting them. This is inconsistent with the rest of the design system and creates unnecessary friction at every call site.
- **What's happening**: Props like `state: AiPresenceState | null`, `events: CalendarEvent[] | null`, `className: string | null` appear across at least four organism-level components. The component implementations then use `?? defaultValue` to handle null. The standard React convention (and the convention used by all molecules in this package) is `state?: AiPresenceState` with default parameter destructuring.
- **Suggestion**: Convert all `T | null` props to optional `T?` props with default values in destructuring. A single codemod or find-and-replace across `packages/ui/src/organisms/` can handle this. Search for `| null` in interface/type definitions within the organisms directory to catch all instances.
- **Estimated scope**: 4 files confirmed, possibly more in organisms not yet audited. Approximately 20+ individual prop declarations.

## Pattern 2: Redundant `as PlatformId` casts after `isPlatformId` type guard

- **Seen in**: organisms-3 (MessageCard.tsx), organisms-4 (SearchResultItem.tsx)
- **Category**: type-cast
- **Combined impact**: Medium -- indicates that `isPlatformId` in `@slopweaver/contracts` may not be declared as a proper type predicate (`value is PlatformId`). If the type predicate is missing, every consumer of this guard must add a redundant cast. If the predicate already exists, these casts are dead code.
- **What's happening**: Both `MessageCard.tsx` and `SearchResultItem.tsx` call `isPlatformId(platform)` and then immediately cast the narrowed variable with `platform as PlatformId`. After a proper type predicate guard, TypeScript should narrow automatically -- the cast should be unnecessary.
- **Suggestion**: First, verify that `isPlatformId` in `@slopweaver/contracts` is declared as `function isPlatformId(value: unknown): value is PlatformId`. If it is not, add the type predicate. Then remove all `as PlatformId` casts that follow an `isPlatformId` guard across the entire codebase (not just these two files).
- **Estimated scope**: 2 files confirmed in the UI package. Likely more across `apps/app/` and `apps/api/` given that `isPlatformId` is a shared utility.

## Pattern 3: `Record<string, ...>` for finite platform-keyed maps

- **Seen in**: molecules-3 (SuggestionCard.tsx), organisms-4 (unified-logo-mark.tsx)
- **Category**: record-weakening
- **Combined impact**: Medium -- both files define maps keyed by platform IDs but type the keys as `string`, allowing typos to pass silently and losing autocomplete for known platform values.
- **What's happening**: `SuggestionCard.tsx` hardcodes `{ slack?: number; "google-gmail"?: number; linear?: number }` as an inline type. `unified-logo-mark.tsx` uses `Record<string, string>` for `CONTEXT_BADGE_COLORS` which contains platform IDs plus UI context keys. Both should use `PlatformId` from `@slopweaver/contracts` as the key type.
- **Suggestion**: For pure platform maps, use `Partial<Record<PlatformId, T>>`. For maps that mix platform IDs with UI context keys, define a union type like `PlatformId | "draft" | "task" | "search"` and use `Partial<Record<ThatUnion, T>>`. Import `PlatformId` from `@slopweaver/contracts` in both cases.
- **Estimated scope**: 2 files confirmed. Search for `Record<string,` in UI components that reference platform-specific values to find others.

## Pattern 4: `platform: string` where `PlatformId` union would be safer

- **Seen in**: organisms-3 (MessageActionBar.tsx, MessageCard.tsx)
- **Category**: missing-strict-typing
- **Combined impact**: Low -- both files already guard with `isPlatformId()` at runtime, so this is a DX/autocomplete issue rather than a safety issue. However, it pairs with Pattern 2: if the prop type were `PlatformId` (or `PlatformId | (string & {})`), the downstream cast after `isPlatformId` would also become unnecessary.
- **What's happening**: `MessageActionBarProps.platform` and `MessageCardData.platform` are both typed as `string`. Callers get no autocomplete for valid platform IDs, and the components must guard with `isPlatformId` before using the value.
- **Suggestion**: Type `platform` as `PlatformId | (string & {})` to preserve autocomplete while accepting arbitrary strings. This is the TypeScript pattern for "string with suggestions."
- **Estimated scope**: 2 files confirmed. The pattern likely exists in other inbox-related components.

## Pattern 5: Unsafe type casts driven by index signatures in chart types

- **Seen in**: molecules-2 (chart.tsx -- Findings 1, 2, 3)
- **Category**: type-cast
- **Combined impact**: Medium -- while concentrated in one file, this is a systemic pattern: a `[key: string]: unknown` index signature forces 3+ downstream `as string` / `as Record<string, unknown>` casts. The root cause (index signature) and symptoms (casts) span multiple findings in the same file, indicating a single targeted fix would eliminate multiple issues.
- **What's happening**: `TooltipPayloadItem` and `LegendPayloadItem` both have `[key: string]: unknown` index signatures. This widens all named properties to `unknown`, forcing `as string` casts in `getPayloadConfigFromPayload` and elsewhere.
- **Suggestion**: Remove the `[key: string]: unknown` index signatures. If arbitrary extra data is needed, add a dedicated `extra?: Record<string, unknown>` field. This single change eliminates 3+ casts.
- **Estimated scope**: 1 file, but 5+ cast instances within it.

## Pattern 6: Missing explicit return types on exported functions

- **Seen in**: organisms-2 (tool-header.tsx)
- **Category**: missing-strict-typing
- **Combined impact**: Low -- only one instance found in this batch, but the project rule mandates explicit return types on all exported functions. This suggests the rule may not be enforced by ESLint (`@typescript-eslint/explicit-function-return-type` or `explicit-module-boundary-types`).
- **What's happening**: `getStatusBadge` is exported without an explicit return type annotation. TypeScript infers correctly, but the project coding standard requires explicit types.
- **Suggestion**: Enable `@typescript-eslint/explicit-module-boundary-types` in ESLint config (or verify it is enabled) so these are caught automatically. Then fix any violations it surfaces.
- **Estimated scope**: 1 instance in this batch; likely more across the full UI package since no lint rule enforces it.
