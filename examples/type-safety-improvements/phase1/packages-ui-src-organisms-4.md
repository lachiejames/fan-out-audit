# Audit: packages/ui/src/organisms (slice 4)

**Files inspected**

- `packages/ui/src/organisms/inbox/SnoozeMenu.tsx`
- `packages/ui/src/organisms/Search/SearchResultItem.tsx`
- `packages/ui/src/organisms/table.tsx`
- `packages/ui/src/organisms/table/column.ts`
- `packages/ui/src/organisms/table/row.ts`
- `packages/ui/src/organisms/table/sorting.ts`
- `packages/ui/src/organisms/table/use-table.tsx`
- `packages/ui/src/organisms/table/utils.ts`
- `packages/ui/src/organisms/unified-logo-mark.tsx`
- `packages/ui/src/organisms/Workflows/workflow-node.tsx`

**Findings**: 4 findings

---

## Summary

`SearchResultItem.tsx` has a redundant `as PlatformId` cast after an `isPlatformId` guard. `unified-logo-mark.tsx` uses `Record<string, string>` for a finite badge color map and repeats the `T | null` optional prop pattern. `WorkflowNode` uses a loose fallback `|| Zap` that bypasses exhaustive key checking. The table layer is well-typed.

---

## Findings

### Finding 1: `result.platform as PlatformId` cast after `isPlatformId` guard in SearchResultItem

- **File**: `packages/ui/src/organisms/Search/SearchResultItem.tsx` (line 79)
- **Category**: Type cast (`as SomeType`) avoidable with better typing
- **Impact**: Low — same pattern as `MessageCard`; the cast would be unnecessary if `isPlatformId` is declared as a proper type predicate
- **Description**: `const knownPlatformId = isPlatformId(result.platform) ? (result.platform as PlatformId) : null`. The explicit cast is needed only if `isPlatformId` doesn't narrow to `PlatformId`. If it is a type predicate, TypeScript narrows automatically.
- **Suggestion**: Verify `isPlatformId` is a type predicate in `@slopweaver/contracts`. If it is, remove the cast: `const knownPlatformId = isPlatformId(result.platform) ? result.platform : null`.
- **Evidence**: `const knownPlatformId = isPlatformId(result.platform) ? (result.platform as PlatformId) : null;`

---

### Finding 2: `CONTEXT_BADGE_COLORS: Record<string, string>` in unified-logo-mark.tsx

- **File**: `packages/ui/src/organisms/unified-logo-mark.tsx` (line 21)
- **Category**: `Record<string, ...>` where stricter union keys would be safer
- **Impact**: Low — the map has a finite set of keys; typos fail silently as the fallback is `"bg-primary"`
- **Description**: `CONTEXT_BADGE_COLORS` is typed as `Record<string, string>` but contains a known set of platform IDs plus UI context keys (`"draft"`, `"task"`, `"search"`). The platform keys could be validated against `PlatformId`.
- **Suggestion**: Type as `Partial<Record<PlatformId | "draft" | "task" | "search" | "github", string>>` — or extract a `BadgeContext = PlatformId | "draft" | "task" | "search" | "github"` union.
- **Evidence**: `const CONTEXT_BADGE_COLORS: Record<string, string> = { "google-gmail": ..., slack: ..., draft: ..., ... }`

---

### Finding 3: `UnifiedLogoMarkProps` uses `T | null` for optional props

- **File**: `packages/ui/src/organisms/unified-logo-mark.tsx` (lines 7–18)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Medium — fifth occurrence of the `T | null` optional prop pattern (alongside `AiPresenceLogo`, `CalendarStrip`, `InboxFilters`); should be standardised to optional `?` props
- **Description**: `size`, `state`, `className`, `onClick`, and `contextPlatform` are all typed as `T | null`. This forces callers to pass `null` for every prop they want to omit.
- **Suggestion**: Convert to standard optional props (`size?: LogoSize`) with default parameter values.
- **Evidence**: `size?: LogoSize | null;`, `state?: LogoState | null;`

---

### Finding 4: `|| Zap` fallback in WorkflowNode bypasses exhaustive key checking

- **File**: `packages/ui/src/organisms/Workflows/workflow-node.tsx` (line 66)
- **Category**: Type cast (`as SomeType`)
- **Impact**: Low — the `icons` map is `Record<WorkflowNodeIcon, LucideIcon>`, so an unknown key returning `Zap` at runtime indicates a bug, not a legitimate default
- **Description**: `const IconComponent = icons[node.icon] || Zap;` — since `icons` is typed as `Record<WorkflowNodeIcon, LucideIcon>` (all keys required), `icons[node.icon]` should never be `undefined` if `node.icon` is a valid `WorkflowNodeIcon`. The `|| Zap` fallback masks cases where `node.icon` is an invalid value at runtime.
- **Suggestion**: Remove the fallback and rely on the type system: `const IconComponent = icons[node.icon];`. If a runtime fallback is genuinely needed (e.g., for data from an API), add an explicit comment.
- **Evidence**: `const IconComponent = icons[node.icon] || Zap;`
