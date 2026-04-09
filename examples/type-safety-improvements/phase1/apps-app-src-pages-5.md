# Audit: apps-app-src-pages-5

**Files inspected**: 8
**Findings**: 7

## Summary

This slice covers the signup page, tasks views, today-page panels, and the triage page. The dominant issues are unsafe `as SomeType` casts used to silence TS when handling API response union bodies, `Record<string, unknown>` / loosely typed inline interfaces used in place of contract-derived types, and a non-narrowed `any`-adjacent `PerformanceEventTiming` cast in sentry monitoring code inside morning-brief.

---

## Findings

### Finding 1: `as { message?: string }` type cast on error response body

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/signup/page.tsx:149`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The `else` branch of the signup handler casts `response.body` to `{ message?: string }`. The ts-rest client already encodes the union of all `responses` from the contract, so this cast bypasses the narrowed union and trusts an unverified shape.
- **Suggestion**: Use the ts-rest `strictStatusCodes` union — at status !== 200 the body type should already be the error schema. Access `errorResponseSchema`-typed body via the contract's inferred type rather than `as`.
- **Evidence**: `const errorBody = response.body as { message?: string };`

### Finding 2: `as TaskCategory | "all"` cast when reading from `<select>` value

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/tasks/page.tsx:291,307,334`
- **Category**: type-cast
- **Impact**: low
- **Description**: `event.target.value` is widened to `string` by the browser DOM and then cast to `TaskCategory | "all"` (or `TaskPriority | "all"`, `TaskSortMode`) with `as`. If the `<option>` values ever drift from the type union, this cast will silently accept invalid values.
- **Suggestion**: Add a type guard or a Zod parse step — or derive the `<option>` list from the union type itself so the values are guaranteed to be in the union. Alternatively, narrow with an explicit inclusion check before setting state.
- **Evidence**: `setCategoryFilter(event.target.value as TaskCategory | "all")`, repeated for priority and sort.

### Finding 3: Inline proposal shape duplicates contract `Todo` type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/tasks/tasks-proposals-banner.tsx:31-38`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `TasksProposalsBanner` accepts `proposals` as an inline array type that redefines `id`, `title`, `priority`, `source`, `aiReasoning`, and `sourceContentId`. This shadow type is structurally similar to (but not derived from) the `Todo`/proposal type in `@slopweaver/contracts`. If the contract evolves, this inline type will silently diverge.
- **Suggestion**: Import the proposal/todo type from `@slopweaver/contracts` (or from `@/hooks/useTodosData`) and use `Pick<Task, ...>` rather than a freehand inline type. At minimum annotate `priority` as the actual `TaskPriority` union rather than `string`.
- **Evidence**: `priority: string` — should be `TaskPriority`; the full inline type is at lines 31–38.

### Finding 4: Non-narrowed `as TriageAction` cast for AI suggestion

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/triage/page.tsx:282`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `item.aiSuggestion` is typed as `string | null` or `unknown` in the session item shape, and is cast directly to `TriageAction` without validation. If the backend returns an unexpected string, the availability check and keyboard shortcut handling will silently accept an invalid action.
- **Suggestion**: Add a type guard `isTriageAction(value: unknown): value is TriageAction` that checks membership in the `TriageAction` union, and use it before casting. This matches the project's `typescript-patterns.md` guidance on type predicates.
- **Evidence**: `const suggested = item.aiSuggestion as TriageAction;`

### Finding 5: `BriefNotification` interface with `Record<string, unknown>` metadata

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/today/morning-brief-panel.tsx:32-37`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The local `BriefNotification` interface declares `metadata: Record<string, unknown>`. The code immediately Zod-parses this via `morningBriefMetadataSchema`, which is the correct pattern — but the interface itself is a local duplicate of a notification shape that should come from `@slopweaver/contracts`. If the notification response type in the contract has a more specific metadata type, this creates a divergence.
- **Suggestion**: Import the notification type from the contracts package and use the contract type directly. Keep the Zod parse for runtime safety, but let TypeScript track the broader origin type rather than a local freehand interface.
- **Evidence**: `interface BriefNotification { ... metadata: Record<string, unknown>; }`

### Finding 6: `(separator?.index as number)` non-null assertion equivalent via `!`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/pages/today/morning-brief-panel.tsx:362`
- **Category**: type-cast
- **Impact**: low
- **Description**: The expression `separator.index!` uses a non-null assertion on a `RegExpMatchArray["index"]` which is `number | undefined`. This silently drops the undefined case. The surrounding condition `separator?.index != null` already guards this, but the assertion repeats the narrowing unsafely with `!`.
- **Suggestion**: Since `separator.index != null` is checked in the condition guard just before, use the narrowed value directly (TypeScript flow analysis should narrow it). Alternatively, restructure into an explicit `if` guard so the `!` is not needed.
- **Evidence**: `end = separator.index!;` (line 364 in the file as read).

### Finding 7: `PerformanceEventTiming` cast on observer entries

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/sentry.ts:44-45`
- **Category**: type-cast
- **Impact**: low
- **Description**: `list.getEntries()` returns `PerformanceEntry[]`. The code casts this to `PerformanceEventTiming[]` and then immediately re-casts each entry to `{ name?: string; duration?: number; interactionId?: number }` before extracting fields. The double cast (first to `PerformanceEventTiming`, then to a freehand object) is redundant — one or the other should be sufficient, and the freehand object cast defeats the benefit of the named `PerformanceEventTiming` type.
- **Suggestion**: Cast once to `PerformanceEventTiming` and access `entry.duration`, `entry.name`, and `entry.interactionId` directly (these are standard `PerformanceEventTiming` fields in modern browsers). Add `lib: ["DOM"]` / `"ES2019"` if needed to get the built-in type.
- **Evidence**: `for (const entry of list.getEntries() as PerformanceEventTiming[]) { const anyEntry = entry as { name?: string; duration?: number; interactionId?: number };`
