# Cross-Cutting Patterns (Group 16)

**Slices analyzed**: apps-app-src-lib-2, apps-app-src-lib-3, apps-app-src-lib-4, apps-app-src-lib-5, apps-app-src-lib-6, apps-app-src-lib-7, apps-app-src-pages-1, apps-app-src-pages-2, apps-app-src-pages-3, apps-app-src-pages-4, apps-app-src-pages-5, apps-app-src-pages-6

## Pattern 1: `Record<string, T>` used where a known union key type exists

- **Seen in**: apps-app-src-lib-2 (findings 1, 4, 5), apps-app-src-lib-3 (finding 3), apps-app-src-lib-6 (finding 1), apps-app-src-pages-1 (findings 3, 4, 6), apps-app-src-pages-2 (finding 3 indirectly), apps-app-src-pages-3 (findings 1, 3), apps-app-src-pages-4 (finding 2), apps-app-src-pages-5 (finding 5)
- **Category**: record-weakening
- **Combined impact**: high -- this is the single most pervasive type-safety issue in the frontend codebase, spanning at least 15 distinct instances across 10+ files
- **What's happening**: Lookup maps, accumulators, and interface fields use `Record<string, T>` when the key set is a finite, already-defined union type (`PlatformId`, `KnowledgeCategory`, `CallProvider`, `StorePlatform`, `CitationSourceType`, tone values, etc.). This defeats exhaustiveness checking, permits typos in keys, and weakens downstream consumers who receive the overly-wide type.
- **Suggestion**: Systematically replace with `Record<UnionType, T>` (or `Partial<Record<UnionType, T>>` when not all keys are guaranteed present). A codebase-wide grep for `Record<string,` in `apps/app/src/` would likely surface additional instances beyond those audited. Consider an ESLint rule or code review checklist item to prevent regression.
- **Estimated scope**: 15-20 files, 20-25 instances

## Pattern 2: `as SomeType` casts on URL search params and API error response bodies

- **Seen in**: apps-app-src-pages-2 (finding 4), apps-app-src-pages-4 (finding 3), apps-app-src-pages-5 (findings 1, 2, 4), apps-app-src-pages-6 (findings 2, 3)
- **Category**: type-cast
- **Combined impact**: high -- these casts bypass the type system at trust boundaries (user input from URLs, API responses) where validation matters most
- **What's happening**: Two related sub-patterns: (a) `searchParams.get("x") as SomeUnion | null` is cast before a membership check validates it -- the cast precedes the guard, so TS trusts the value before it is proven. This appears in settings, inbox filters, and tasks pages. (b) `response.body as { message?: string }` bypasses ts-rest's discriminated union narrowing on non-200 status codes. This appears identically in forgot-password, login, reset-password, and signup pages (4 files, same code).
- **Suggestion**: (a) Create a shared `castIfValid<T extends string>(value: string | null, allowed: readonly T[]): T | null` helper and use it across all URL param parsing. (b) For API error bodies, narrow via `response.status !== 200` which should give the typed error body from the ts-rest contract. If the contract error type does not include `message`, fix the contract rather than casting.
- **Estimated scope**: 10-12 files, 15-18 instances

## Pattern 3: `react-force-graph-2d` callback casts duplicated across graph components

- **Seen in**: apps-app-src-pages-1 (findings 1, 2), apps-app-src-pages-3 (finding 2)
- **Category**: type-cast
- **Combined impact**: medium -- ~11 identical casts across 2 components, caused by the library shipping `object`-typed callbacks
- **What's happening**: Both `KnowledgeGraphCanvas.tsx` and `EntityGraphView.tsx` use `<ForceGraph2D>` and must cast every callback prop (`linkColor`, `linkWidth`, `nodeCanvasObject`, `onNodeClick`, `ref`) because the library types its callbacks as `(node: object) => T`. The casts are identical in structure but independently written in each component.
- **Suggestion**: Create a single module augmentation file (`react-force-graph-2d.d.ts`) that narrows the callback generics to the actual node/link types. Alternatively, create a thin `TypedForceGraph2D` wrapper component that applies the casts once internally. Either approach eliminates ~11 casts.
- **Estimated scope**: 2 files, 11 casts; fix is a single `.d.ts` or wrapper component

## Pattern 4: Duplicate type definitions that drift from canonical sources

- **Seen in**: apps-app-src-lib-2 (finding 3), apps-app-src-pages-1 (finding 3 implicitly), apps-app-src-pages-2 (findings 1, 3), apps-app-src-pages-5 (findings 3, 5), apps-app-src-pages-6 (finding 1)
- **Category**: duplicate-type
- **Combined impact**: medium -- each duplicate is a silent divergence risk when the canonical type evolves
- **What's happening**: Types are defined locally instead of imported from their canonical source: `StorePlatform` defined in 2 billing files; `KnowledgeCategory` redefined locally in `KnowledgeTab.tsx` when the Zod schema is already imported; `SelectedMessageOrigin` defined identically in 2 inbox hooks; `NotificationItem` defined locally instead of derived from the ts-rest contract; `BriefNotification` with a freehand metadata shape instead of using the contract type; inline task proposal shape in `tasks-proposals-banner.tsx` instead of using the contract `Todo` type.
- **Suggestion**: For each duplicate: import from the canonical source (`@slopweaver/contracts`, a shared types file, or derive via `z.infer`). For `StorePlatform`, define once and re-export. For `SelectedMessageOrigin`, export from `useInboxDeepLink.ts`. Establish a convention: if a type exists in contracts, never redefine it locally.
- **Estimated scope**: 8-10 files, 6-8 distinct duplicate types

## Pattern 5: `as SomeType` casts on parsed/deserialized data instead of type guards

- **Seen in**: apps-app-src-lib-2 (finding 2), apps-app-src-lib-4 (finding 3), apps-app-src-lib-5 (finding 1), apps-app-src-lib-6 (findings 2, 3), apps-app-src-pages-4 (finding 1), apps-app-src-pages-5 (finding 4), apps-app-src-pages-6 (finding 4)
- **Category**: type-cast
- **Combined impact**: medium -- casts on deserialized data (JSON.parse, localStorage, API cache, native plugin responses) are the most dangerous cast category because the runtime shape is genuinely unknown
- **What's happening**: When code receives data from an external boundary (JSON.parse of localStorage, Capacitor plugin responses, React Query cache, Zod-parsed but loosely-typed API fields), it uses `as SomeType` to assert the shape rather than validating it. Examples: `parsed as { state?: unknown; version?: number }` for localStorage envelopes, `response as { products?: unknown }` for native IAP responses, `cached as Content` for query cache data, `item.aiSuggestion as TriageAction` for AI-generated values.
- **Suggestion**: Create type guard functions (e.g., `isStorageEnvelope`, `isProductsResponse`, `isTriageAction`) or use Zod `.safeParse()` at these boundaries. The project already uses Zod extensively; extending it to these deserialization points is consistent with existing patterns.
- **Estimated scope**: 8-10 files, 10-12 casts

## Pattern 6: `window` property access via repeated inline casts

- **Seen in**: apps-app-src-lib-3 (findings 1, 2)
- **Category**: type-cast
- **Combined impact**: low -- contained to the diagnostics subsystem but the fix is trivial and eliminates 5 casts
- **What's happening**: Four diagnostics files and one `requestIdleCallback` usage all cast `window` with inline type intersections (`window as typeof window & Record<string, boolean>`) to access custom properties. The same cast structure is repeated in every file.
- **Suggestion**: Add a single `Window` interface augmentation in a `.d.ts` file declaring all custom `__SLOPWEAVER_*` properties and `requestIdleCallback`. This eliminates all 5 inline casts.
- **Estimated scope**: 4-5 files, 5 casts; fix is a single `.d.ts` file

## Pattern 7: `categoryIcons` / `categoryLabels` maps duplicated across knowledge UI components

- **Seen in**: apps-app-src-pages-1 (finding 3)
- **Category**: duplicate-type + record-weakening
- **Combined impact**: low -- 3 components define the same `Record<string, ReactNode>` icon map for knowledge categories
- **What's happening**: `FactsList.tsx`, `KnowledgeNodePanel.tsx`, and `MobileKnowledgeGraphFallback.tsx` each independently define `categoryIcons: Record<string, React.ReactNode>` with the same icon mappings. This is both a record-weakening issue (should be `Record<KnowledgeCategory, ReactNode>`) and a code duplication issue (same data defined 3 times).
- **Suggestion**: Extract a shared `KNOWLEDGE_CATEGORY_ICONS: Record<KnowledgeCategory, ReactNode>` constant (and similarly for labels) into a shared file under the knowledge-graph components directory, then import in all three components.
- **Estimated scope**: 3 files; fix is extracting 1 shared constant file
