# Audit: apps/app/src/components — Slice 9 (command palette and contacts)

**Files inspected**: 8
**Findings**: 6

## Summary

This slice has the highest density of duplicate type definitions in the audit: `SuggestionChip` is defined identically in two contacts files. Additionally, the command palette accesses `Record<string, unknown>` metadata fields without type guards, and contact interfaces use `string` for platform fields that should use `PlatformId`.

## Findings

### Finding 1: Duplicate `SuggestionChip` interface across two contacts files

- **File**: `apps/app/src/components/contacts/contact-detail-panel.tsx:33-38` and `apps/app/src/components/contacts/contacts-empty-state.tsx:11-17`
- **Category**: Duplicate type definitions
- **Impact**: high
- **Description**: `interface SuggestionChip { id: string; label: string; icon: React.ReactNode; action: () => void; primary?: boolean }` is defined identically in both files. Any change to the chip shape must be synchronized manually.
- **Suggestion**: Extract `SuggestionChip` to a shared types file (e.g. `apps/app/src/components/contacts/contacts.types.ts`) and import it in both components. If this type is also used elsewhere in the app, consider placing it in `apps/app/src/types/`.
- **Evidence**: Identical `interface SuggestionChip { ... }` blocks at the specified lines in both files.

### Finding 2: `Contact` interface uses `string` for platform fields

- **File**: `apps/app/src/components/contacts/contact-card.tsx:7-27`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: high
- **Description**: The local `Contact` interface has `platforms: string[]` and `identities: { platform: string; ... }[]`. These `platform` fields represent actual platform IDs and should use `PlatformId` from `@slopweaver/contracts` for type safety and consistency.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts`. Change `platforms: string[]` to `platforms: PlatformId[]` and `identities: { platform: string; ... }[]` to `identities: { platform: PlatformId; ... }[]`. Check whether the contracts package already exports a `ContactResponse` type that could replace this local interface entirely.
- **Evidence**: `platforms: string[]` and `identities: { platform: string; ... }[]` in the local `Contact` interface at lines 7-27.

### Finding 3: Command palette metadata accessed via unguarded bracket notation

- **File**: `apps/app/src/components/command-palette/command-palette.tsx:165-167`
- **Category**: `any` or unsafe `unknown` in production code
- **Impact**: medium
- **Description**: `const metadata = result.metadata ?? {}` is typed as `Record<string, unknown>`, and then `metadata["isAttachment"]` and `metadata["contentId"]` are accessed without type guards. These accesses return `unknown` and any downstream use requires implicit or explicit casts.
- **Suggestion**: Define a typed metadata interface for command palette results (e.g. `interface CommandResultMetadata { isAttachment?: boolean; contentId?: string }`). Either type `result.metadata` using this interface, or use type guards: `if (typeof metadata["isAttachment"] === 'boolean') { ... }`.
- **Evidence**: `metadata["isAttachment"]` and `metadata["contentId"]` accessed on `Record<string, unknown>` at lines 165-167.

### Finding 4: `filter-dropdown.tsx` event target cast

- **File**: `apps/app/src/components/common/filter-dropdown.tsx:47`
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: low
- **Description**: `!ref.current.contains(event.target as Node)` casts `event.target` (typed as `EventTarget | null`) to `Node`. The cast is technically safe because `Node` extends `EventTarget` in the DOM, but it obscures the null check. The `contains` method accepts `Node | null`, so the cast is unnecessary.
- **Suggestion**: Remove the cast: `!ref.current.contains(event.target as Node | null)`. Or better, check null first: `event.target !== null && !ref.current.contains(event.target as Node)`. Actually, `Node.contains()` accepts `null` — so `ref.current.contains(event.target as Node | null)` is correct. But since `EventTarget` is a supertype of `Node`, a simple null check without cast is cleaner: `event.target instanceof Node && !ref.current.contains(event.target)`.
- **Evidence**: `event.target as Node` at line 47.

### Finding 5: `Contact` interface potentially duplicating a contracts type

- **File**: `apps/app/src/components/contacts/contact-card.tsx:7-27`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: The local `Contact` interface may duplicate a `ContactResponse` or similar type that is or should be in `@slopweaver/contracts`. Even if the contracts type does not exist today, defining it there would allow both the API response types and UI component types to share the same definition.
- **Suggestion**: Check `@slopweaver/contracts` for any contact-related response type. If one exists, import and use it (or derive the component prop type from it). If not, file a contracts ticket to add a canonical `ContactResponse` type.
- **Evidence**: Local `Contact` interface at lines 7-27 with fields matching expected API response shape.

### Finding 6: `SuggestionChip.icon` typed as `React.ReactNode` vs. a component type

- **File**: `apps/app/src/components/contacts/contact-detail-panel.tsx:33-38` and `apps/app/src/components/contacts/contacts-empty-state.tsx:11-17`
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: low
- **Description**: The `icon` field in `SuggestionChip` is typed as `React.ReactNode`, which accepts strings, numbers, arrays, and null. If icon is always a React element (a Lucide icon component), the type should be `React.ReactElement` for stricter checking.
- **Suggestion**: Change `icon: React.ReactNode` to `icon: React.ReactElement` in the shared `SuggestionChip` type created by resolving Finding 1.
- **Evidence**: `icon: React.ReactNode` in both `SuggestionChip` interface definitions.
