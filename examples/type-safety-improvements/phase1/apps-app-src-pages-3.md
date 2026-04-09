# Type-Safety Audit: apps-app-src-pages-3

**Files inspected**: 8
**Findings**: 3

## Summary

`calls-page.utils.ts` uses `Record<string, string>` (and `Record<string, string>`) for lookup maps keyed on provider IDs; a finite string union would be safer. `contacts/EntityGraphView.tsx` has the same `react-force-graph-2d` cast pattern found in the ai-page knowledge graph. The remaining files (`calls-empty-state.tsx`, `calls-error-state.tsx`, `calls.utils.ts`, `contacts/contacts-page.utils.ts`, `contacts/page.tsx`, `forgot-password/page.tsx`) are clean or have issues already captured in pages-2 (auth page error-body cast).

---

## Findings

### 1

**File**: `apps/app/src/pages/calls/calls-page.utils.ts`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — a misspelled provider key compiles silently and returns the fallback value rather than a compile error
**Description**: `getProviderLabel`, `getProviderColor`, and `getProviderBorderColor` each define a local `Record<string, string>` lookup map keyed by provider ID strings: `fathom | grain | otter | slack_huddle | teams | zoom`. These keys are a finite, stable set.
**Suggestion**: Define a `CallProvider` union type (`"fathom" | "grain" | "otter" | "slack_huddle" | "teams" | "zoom"`) and type each lookup map as `Record<CallProvider, string>` with the parameter types changed to `CallProvider`. This makes missing keys a compile error.
**Evidence**:

```typescript
const labels: Record<string, string> = { fathom: ..., grain: ..., otter: ..., ... }  // getProviderLabel
const colors: Record<string, string> = { fathom: ..., ... }  // getProviderColor
const colors: Record<string, string> = { fathom: ..., ... }  // getProviderBorderColor
```

---

### 2

**File**: `apps/app/src/pages/contacts/EntityGraphView.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Medium — same library limitation as `KnowledgeGraphCanvas.tsx`; five casts required by `react-force-graph-2d`'s loose callback types
**Description**: Props on `<ForceGraph2D>` require casts: `linkColor as (link: object) => string`, `linkLineDash as (link: object) => number[] | null`, `linkWidth as (link: object) => number`, `nodeCanvasObject as (node: object, ...) => void`, `handleNodeClick as (node: object) => void`, and `graphRef as React.RefObject<never>`.
**Suggestion**: Same as `KnowledgeGraphCanvas.tsx` (pages-1, finding 1): add a module augmentation for `react-force-graph-2d` that narrows the callback generics, or consolidate the casts into a shared typed wrapper component.
**Evidence**:

```typescript
linkColor={linkColor as (link: object) => string}
linkLineDash={linkLineDash as (link: object) => number[] | null}
linkWidth={linkWidth as (link: object) => number}
nodeCanvasObject={nodeCanvasObject as (node: object, ctx: CanvasRenderingContext2D, globalScale: number) => void}
onNodeClick={handleNodeClick as (node: object) => void}
ref={graphRef as React.RefObject<never>}
```

---

### 3

**File**: `apps/app/src/pages/contacts/contacts-page.utils.ts`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — the return type leaks to callers who use platform IDs as keys
**Description**: `countPlatforms` returns `Record<string, number>`. The keys are derived from `contact.platforms` which comes from the `Contact` type. If `Contact.platforms: PlatformId[]`, the return type could be `Partial<Record<PlatformId, number>>`.
**Suggestion**: Check the `Contact` type definition. If `platforms: PlatformId[]`, change the return type to `Partial<Record<PlatformId, number>>`. This tightens all downstream callers that iterate over the result.
**Evidence**: `export function countPlatforms(...): Record<string, number>` — line 22
