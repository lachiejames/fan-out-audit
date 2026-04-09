# Cross-Cutting Patterns (Group 14)

**Slices analyzed**: apps-app-src-components-3, apps-app-src-components-4, apps-app-src-components-5, apps-app-src-components-6, apps-app-src-components-7, apps-app-src-components-8, apps-app-src-components-9, apps-app-src-components-25, apps-app-src-components-26, apps-app-src-components-27, apps-app-src-components-28, apps-app-src-env.ts

## Pattern 1: `Record<string, ...>` maps with closed key sets that should use contract union types

- **Seen in**: slices 3, 4, 5, 6, 7, 8, 9, 25
- **Category**: record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety gap across the frontend; every slice except 26, 27, 28 (voice internals), and env.ts has at least one instance
- **What's happening**: Constant lookup maps (e.g. `STATUS_TO_ACTION`, `SHORT_NAMES`, `METRIC_LABELS`, `MODEL_COLORS`, `tierUpgrades`, `statusColors`, `RESPONSE_LABELS`, `TRANSACTION_ICON_MAP`, task priority weights) are all typed as `Record<string, ...>` even though their keys come from a known, closed set. This means (a) lookups with invalid keys compile without error, (b) adding a new value to the domain union does not produce a compile error for missing entries, and (c) exhaustiveness checking is impossible.
- **Suggestion**: For each map, import or define the appropriate union type (`TriageStatus`, `PlatformId`, `SubscriptionTier`, `MetricId`, `AttendeeResponse`, `TaskPriority`, `TransactionType`, `InvoiceStatus`, etc.) and use `Record<UnionType, ValueType>`. Where not all keys are always present, use `Partial<Record<UnionType, ValueType>>`. This is a mechanical find-and-replace per instance.
- **Estimated scope**: ~15-20 constant declarations across ~12-15 files in `apps/app/src/components/`

## Pattern 2: Props and fields typed as `string` instead of contract union types (`PlatformId`, `SubscriptionTier`, `ReadinessState`, etc.)

- **Seen in**: slices 3, 4, 5, 6, 8, 9
- **Category**: sdk-type-duplication
- **Combined impact**: high -- weakens the entire prop-passing chain from API response to leaf component
- **What's happening**: Component props, interface fields, and Zustand store fields use `string` where a contract-exported union type exists. Examples: `platform: string` instead of `PlatformId`, `readinessState: string` instead of `ReadinessState`, tier props as `string` instead of `SubscriptionTier`, billing provider comparisons against string literals. This means type errors from contract changes are not surfaced at compile time; they become silent runtime bugs.
- **Suggestion**: Import the canonical type from `@slopweaver/contracts` at each site. For Zustand stores (slice 27), update the store type definition so downstream consumers do not need casts. For string literal comparisons (slice 5), replace with typed equality checks using the imported union.
- **Estimated scope**: ~10-15 prop/field definitions across ~10 files spanning billing, calendar, contacts, analytics, triage, and voice modules

## Pattern 3: Repeated `as { details?: unknown }` / `as { message?: unknown }` casts on ts-rest error response bodies

- **Seen in**: slices 25, 26, 27
- **Category**: type-cast
- **Combined impact**: high -- the same unsafe cast pattern appears in 5+ files within the voice module alone
- **What's happening**: When a ts-rest call returns a non-200 status, the response body is typed as `unknown`. Each call site independently casts to `{ details?: unknown }` or `{ message?: string }` to extract error information. The cast is duplicated in `message-tts.api.ts`, `message-tts.utils.ts`, `use-message-tts.ts` (twice), and `voice-conversation.api.ts`. This is fragile (if the error shape changes, every site must be updated) and unsafe (no runtime validation).
- **Suggestion**: Create a single `extractApiErrorBody` type guard utility (e.g. in `message-tts.utils.ts` or a shared API utils file) that validates the shape at runtime and returns a typed result. Replace all cast sites with calls to this function. Slice 26 already proposes a concrete implementation.
- **Estimated scope**: 5-7 cast sites across 4-5 files in the voice module; likely more instances elsewhere in the app

## Pattern 4: `unknown` values accessed via intermediate object casts instead of type guards

- **Seen in**: slices 3, 9, 25, 26, 27
- **Category**: type-cast
- **Combined impact**: medium -- each instance is a potential runtime error hiding behind a cast
- **What's happening**: A recurring pattern of `(value as { prop?: unknown }).prop` to access properties on `unknown` values. Seen in: BroadcastChannel message handling (slice 3), command palette metadata access (slice 9), voice error detail extraction (slices 25-27), and voice conversation failure context (slice 27). The cast silently succeeds even if the value is null, a string, or an array -- it just returns `undefined` instead of throwing, masking bugs.
- **Suggestion**: Replace each instance with a type guard function. The pattern is always the same: `function hasX(v: unknown): v is { x: SomeType } { return typeof v === 'object' && v !== null && 'x' in v; }`. A generic `hasProperty` utility could serve most of these cases.
- **Estimated scope**: ~8-10 cast sites across ~6-7 files

## Pattern 5: Duplicate type/constant definitions across sibling components

- **Seen in**: slices 6+7 (tierUpgrades), 9 (SuggestionChip), 26+27 (DetailedStatus)
- **Category**: duplicate-type
- **Combined impact**: medium -- each duplicate is a maintenance hazard that will silently drift
- **What's happening**: Three distinct cases of the same type or constant being defined in two places: (1) `tierUpgrades` record in both `soft-paywall.tsx` and `sync-limit-paywall.tsx` with identical structure, (2) `SuggestionChip` interface in both `contact-detail-panel.tsx` and `contacts-empty-state.tsx`, (3) `DetailedStatus` type in `voice-conversation-composer.tsx` duplicating the export from `voice-conversation.types.ts`.
- **Suggestion**: For each case, extract the shared definition to a single canonical location and import it at all use sites. Specifically: (1) `billing.constants.ts` for tier upgrades, (2) `contacts.types.ts` for SuggestionChip, (3) import from `voice-conversation.types.ts` for DetailedStatus.
- **Estimated scope**: 3 duplicate pairs, 6 files total

## Pattern 6: Window global debug properties cast inline instead of declared in a `.d.ts` file

- **Seen in**: slices 26, 27
- **Category**: type-cast
- **Combined impact**: low -- only affects debug/development code paths, but the pattern adds noise
- **What's happening**: Debug globals like `window.__SLOPWEAVER_VOICE_METRICS__` and `window.__SLOPWEAVER_LAST_VOICE_START__` are assigned using inline window type augmentation casts (`window as Window & { __SLOPWEAVER_X__?: ... }`). Each assignment site re-declares the augmented window type independently.
- **Suggestion**: Add all `__SLOPWEAVER_*` debug properties to the global `Window` interface in a single `apps/app/src/types/globals.d.ts` declaration. This eliminates all inline casts and makes the debug surface discoverable via IDE autocomplete.
- **Estimated scope**: 3-4 assignment sites across 2 files (`message-tts.utils.ts`, `voice-conversation-debug.ts`)

## Pattern 7: Local interfaces duplicating contract/API response shapes

- **Seen in**: slices 4, 9
- **Category**: sdk-type-duplication
- **Combined impact**: medium -- silent divergence between frontend types and API contracts
- **What's happening**: Frontend components define local interfaces that mirror API response shapes instead of importing from `@slopweaver/contracts`. Slice 4 has four `Sample*Stats` interfaces in `sample-data.utils.ts` that are documented as matching API response types. Slice 9 has a local `Contact` interface that likely duplicates a contract response type. These local copies will silently drift if the API contract changes.
- **Suggestion**: Replace local interfaces with types derived from contracts: either direct imports or `z.infer<typeof responseSchema>`. If the local type intentionally differs from the contract (e.g. for sample/mock data), document the divergence explicitly with a comment linking to the canonical contract type.
- **Estimated scope**: ~5-6 local interface definitions across ~3 files
