# Type-Safety Audit — Batch 97

**Files audited:**

- `apps/api/src/interface/agents/tools/_shared/ai-tool.types.ts`
- `apps/api/src/interface/agents/tools/_shared/builders/platform-search.builders.ts`
- `apps/api/src/interface/agents/tools/_shared/builders/work-item-tool.builder.ts`
- `apps/api/src/interface/agents/tools/_shared/registry/tool-manifest.types.ts`
- `apps/api/src/interface/agents/tools/_shared/review-edit/tool-review-edit-mapper.registry.ts`
- `apps/api/src/interface/agents/tools/_shared/review-edit/tool-review-edit-mapper.types.ts`
- `apps/api/src/interface/agents/tools/calendar/calendar-tool.utils.ts`
- `apps/api/src/interface/agents/tools/google-drive/get-google-drive-file.tool.ts`

---

## Findings

### 1. ai-tool.types.ts — any-usage

**Line:** 8
**Category:** any-usage
**Severity:** medium
**Description:** `export type AnyTool = Tool<any, any>` — uses `any` for both input and output generics of the AI SDK `Tool` type. The JSDoc states this is intentional to avoid TS2742 errors at tool factory return sites, but `any` disables all type checking on tool input/output shapes.
**Code:**

```typescript
export type AnyTool = Tool<any, any>;
```

**Fix:** Use `Tool<z.ZodType, z.ZodType>` or a bounded wildcard `Tool<z.ZodTypeAny, z.ZodTypeAny>` to retain some type constraint while still allowing heterogeneous tool collections. If TS2742 is the only blocker, it can be suppressed with `// @ts-expect-error TS2742` at specific call sites rather than degrading the entire type.

---

### 2. tool-manifest.types.ts — any-usage (propagated from AnyTool)

**Line:** 20
**Category:** any-usage
**Severity:** medium
**Description:** `type AnyTool = import("ai").Tool<any, any>` — inline re-declaration of the `AnyTool` type with `any` generics. This is a local duplicate of the type in `ai-tool.types.ts` and propagates the `any` issue into the manifest registry.
**Code:**

```typescript
type AnyTool = import("ai").Tool<any, any>;
export interface ToolManifestEntry {
  factory: (context: ToolFactoryContext, useCases: ToolUseCases) => AnyTool | Promise<AnyTool>;
}
```

**Fix:** Import `AnyTool` from `@/interface/agents/tools/_shared/ai-tool.types` instead of declaring inline, then apply the same fix as finding #1.

---

### 3. work-item-tool.builder.ts — missing-strict-typing

**Line:** 35
**Category:** missing-strict-typing
**Severity:** low
**Description:** `execute: (input: z.output<InputSchema>) => Promise<unknown> | unknown` — the `execute` callback return type is `unknown`. The output is always parsed through `workItemToolOutputSchema`, so the callback could return any shape, but the `unknown` type forces a Zod parse on every call which silently drops fields.
**Code:**

```typescript
execute: (input: z.output<InputSchema>) => Promise<unknown> | unknown;
```

**Fix:** Change the callback type to `execute: (input: z.output<InputSchema>) => Promise<WorkItemToolOutput> | WorkItemToolOutput` and remove the redundant `workItemToolOutputSchema.parse()` call, or keep the parse but document why `unknown` is intentional.

---

### 4. tool-review-edit-mapper.types.ts — record-weakening

**Line:** 18
**Category:** record-weakening
**Severity:** medium
**Description:** `export type ToolReviewEditMapper = (toolInput: Record<string, unknown>) => ToolReviewEditMapperResult` — the mapper input is `Record<string, unknown>`, which loses the tool-specific input shape that was validated by Zod at the call site.
**Code:**

```typescript
export type ToolReviewEditMapper = (toolInput: Record<string, unknown>) => ToolReviewEditMapperResult;
```

**Fix:** Make the mapper generic: `export type ToolReviewEditMapper<TInput extends Record<string, unknown> = Record<string, unknown>> = (toolInput: TInput) => ToolReviewEditMapperResult`. Callers with known tool input types can then provide the specific type parameter.

---

## No Findings

- `platform-search.builders.ts` — `metadata?: Record<string, unknown>` in `buildNativeSearchResult` is appropriate since search result metadata is platform-heterogeneous. `normalizeSearchMetadata` handles typed field access; no findings.
- `tool-review-edit-mapper.registry.ts` — `Record<string, ToolReviewEditMapper>` for the registry is appropriate for a keyed heterogeneous map; no findings.
- `calendar-tool.utils.ts` — Clean utilities with explicit typed interfaces; no findings.
- `get-google-drive-file.tool.ts` — Returns `AnyTool` (tracking from finding #1) but the tool implementation itself is clean with typed schemas; no additional findings.
