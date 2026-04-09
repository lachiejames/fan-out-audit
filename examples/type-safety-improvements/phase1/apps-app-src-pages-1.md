# Type-Safety Audit: apps-app-src-pages-1

**Files inspected**: 8
**Findings**: 6

## Summary

The knowledge-graph components have the highest concentration of issues: `KnowledgeGraphCanvas.tsx` requires five `as (node: object) => ...` casts because the `react-force-graph-2d` library uses `object` for its callbacks rather than typed interfaces. Three separate components each independently define `categoryIcons: Record<string, React.ReactNode>` when a `KnowledgeCategory`-keyed record would be safer. `PlatformStyleSliders.tsx` uses `Record<string, PlatformStyle | undefined>` when `PlatformId` is already imported. `ai/page.tsx` and `KnowledgeGraphControls.tsx` are clean.

---

## Findings

### 1

**File**: `apps/app/src/pages/ai/components/knowledge-graph/KnowledgeGraphCanvas.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Medium — the casts are required by the library's `object`-typed callback props; upstream `react-force-graph-2d` ships loose types
**Description**: Five props on `<ForceGraph2D>` require casts because the library declares callbacks as `(node: object) => T` instead of using the actual node/link generic. The casts are: `linkColor as (link: object) => string`, `linkWidth as (link: object) => number`, `nodeCanvasObject as (node: object, ...) => void`, `handleNodeClick as (node: object) => void`, and `graphRef as React.RefObject<never>`.
**Suggestion**: Declare a local module augmentation for `react-force-graph-2d` that narrows the callback signatures to `EntityNode` and `EntityLink`, then remove the casts. Alternatively, wrap `<ForceGraph2D>` in a thin typed component that applies the casts in one place rather than at every prop.
**Evidence**:

```typescript
linkColor={linkColor as (link: object) => string}
linkWidth={linkWidth as (link: object) => number}
nodeCanvasObject={nodeCanvasObject as (node: object, ctx: CanvasRenderingContext2D, globalScale: number) => void}
onNodeClick={handleNodeClick as (node: object) => void}
ref={graphRef as React.RefObject<never>}
```

---

### 2

**File**: `apps/app/src/pages/ai/components/knowledge-graph/KnowledgeGraphCanvas.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — runtime behavior is unaffected; but the ref type is redundantly cast
**Description**: `graphRef` is declared as `useRef<{ zoomToFit: ... } | null>` and later used in `handleZoomIn`/`handleZoomOut` as `const fg = graphRef.current as { zoom: ... }`. The `zoom` method is not declared in the ref type, so the cast adds a new shape; the ref declaration should include `zoom`.
**Suggestion**: Add `zoom: (zoom?: number, duration?: number) => void` to the `graphRef` `useRef` type declaration so that `graphRef.current.zoom()` is already typed without a cast.
**Evidence**: `const fg = graphRef.current as { zoom: (zoom?: number, duration?: number) => void }` — lines 158, 163

---

### 3

**File**: `apps/app/src/pages/ai/components/FactsList.tsx`, `apps/app/src/pages/ai/components/knowledge-graph/KnowledgeNodePanel.tsx`, `apps/app/src/pages/ai/components/knowledge-graph/MobileKnowledgeGraphFallback.tsx`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — a misspelled category key would compile silently
**Description**: All three files define `categoryIcons: Record<string, React.ReactNode>` (and `categoryLabels: Record<string, string>` in `KnowledgeNodePanel.tsx`) for a finite set of knowledge categories. `KnowledgeCategory` is a union type available from `@slopweaver/contracts` (or locally from the `knowledgeCategorySchema`).
**Suggestion**: Change to `Record<KnowledgeCategory, React.ReactNode>` (and `Record<KnowledgeCategory, string>` for labels). This makes the compiler enforce exhaustiveness and reject unknown category strings.
**Evidence**:

- `FactsList.tsx` line 57: `categoryIcons: Record<string, React.ReactNode>`
- `KnowledgeNodePanel.tsx`: `categoryIcons: Record<string, React.ReactNode>`, `categoryLabels: Record<string, string>`
- `MobileKnowledgeGraphFallback.tsx`: `categoryIcons: Record<string, React.ReactNode>`

---

### 4

**File**: `apps/app/src/pages/ai/components/PlatformStyleSliders.tsx`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — `PlatformId` is imported from `@slopweaver/contracts` in the same file
**Description**: `platformStyles: Record<string, PlatformStyle | undefined>` at line 12. Since `PlatformId` is already imported, the key type can be narrowed.
**Suggestion**: Change to `Partial<Record<PlatformId, PlatformStyle>>` which is equivalent at runtime but stricter at compile time.
**Evidence**: `platformStyles: Record<string, PlatformStyle | undefined>` — line 12

---

### 5

**File**: `apps/app/src/pages/ai/components/FactsList.tsx`
**Category**: `any` or unsafe `unknown` in production code
**Impact**: Low — the `unknown` return leaks into callers but is not actively unsafe
**Description**: `onAddFact: (content: string) => Promise<unknown>` and `onUpdateFact: (id: string, content: string) => Promise<unknown>` use `unknown` return types. Both handlers are fire-and-forget mutations; `void` is more accurate.
**Suggestion**: Change both to `Promise<void>`.
**Evidence**:

```typescript
onAddFact: (content: string) => Promise<unknown>;
onUpdateFact: (id: string, content: string) => Promise<unknown>;
```

---

### 6

**File**: `apps/app/src/pages/ai/tabs/BehaviorTab.tsx`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low
**Description**: `toneColors: Record<string, string>` at line 308 maps only three tone values. The key set is finite and known.
**Suggestion**: Change to `Record<"balanced" | "casual" | "formal", string>` to prevent typos and get IDE completions.
**Evidence**: `const toneColors: Record<string, string> = { balanced: ..., casual: ..., formal: ... }` — line 308
