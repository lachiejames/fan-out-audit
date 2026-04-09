# Type-Safety Audit — Batch 96

**Files audited:**

- `apps/api/src/interface/agents/agents.controller.ts`
- `apps/api/src/interface/agents/errors/agent.errors.ts`
- `apps/api/src/interface/agents/guards/tool-safety.guard.ts`
- `apps/api/src/interface/agents/services/agent-context-builder.service.ts`
- `apps/api/src/interface/agents/services/chat-agent.factory.ts`
- `apps/api/src/interface/agents/services/tool-rate-limiter.service.ts`
- `apps/api/src/interface/agents/services/tool-registry.service.ts`
- `apps/api/src/interface/agents/services/tool-registry.validator.ts`

---

## Findings

### 1. tool-registry.service.ts — type-cast

**Line:** 145
**Category:** type-cast
**Severity:** medium
**Description:** `return [id, tool as DomainTool] as const` — cast after `isAiSdkTool(maybeTool)` validation. The type guard `isAiSdkTool` returns `maybeTool is AiSdkTool` but the type narrowing does not automatically produce `DomainTool`. An additional cast is needed to bridge the two types.
**Code:**

```typescript
if (isAiSdkTool(maybeTool)) {
  return [id, tool as DomainTool] as const;
}
```

**Fix:** Align `DomainTool` with the `AiSdkTool` type so that the narrowing from `isAiSdkTool` is sufficient. Alternatively, make `isAiSdkTool` return `maybeTool is DomainTool` directly.

---

### 2. agent-context-builder.service.ts — missing-strict-typing (unusual pattern)

**Line:** 374
**Category:** missing-strict-typing
**Severity:** low
**Description:** Inline dynamic import in a type position: `let enrichedContext: import("@/application/chat/services/context-enrichment.service").EnrichedContext | null = null`. While TypeScript supports inline import types, this is an unusual pattern that makes the type harder to navigate and signals that the import may be missing from the module's static imports.
**Code:**

```typescript
let enrichedContext: import("@/application/chat/services/context-enrichment.service").EnrichedContext | null = null;
```

**Fix:** Add `import type { EnrichedContext } from "@/application/chat/services/context-enrichment.service"` at the top of the file and use `EnrichedContext | null` as the type annotation.

---

## No Findings

- `agents.controller.ts` — Clean ts-rest controller pattern, no findings.
- `agent.errors.ts` — Clean discriminated union error type, no findings.
- `tool-safety.guard.ts` — Clean guard implementation, no findings.
- `chat-agent.factory.ts` — `tools: Record<string, Tool>` is the appropriate heterogeneous tool map type for the AI SDK; no issue.
- `tool-rate-limiter.service.ts` — Clean service with properly typed Redis operations, no findings.
- `tool-registry.validator.ts` — Clean validation with structural type checks, no findings.
