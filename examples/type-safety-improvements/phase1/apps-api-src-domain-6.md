# Audit: apps-api-src-domain-6

**Files inspected**: 8
**Findings**: 5

## Summary

This batch covers service ports for integrations lookup, knowledge extraction, LLM context, memory, OAuth flow store, OCR, rate limiting, and plugin registry. Key findings: `ILLMContextPort` methods use positional params; `SyncParams.body` and `OAuthHandler` query/request params are `unknown`/`Record<string, unknown>`; the `PluginInfo` interface partially duplicates the integrations layer's `IntegrationPlugin` type; and `IIntegrationsLookupPort` uses positional params in abstract methods.

## Findings

### Finding 1: `ILLMContextPort` abstract methods use positional params (violates named-params rule)

- **File**: `apps/api/src/domain/ports/services/llm-context.port.ts:45-64`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Three abstract methods — `getContext(userId: string)`, `buildSystemPromptWithContext(basePrompt: string, userId: string)`, and `buildSystemPrompt(basePrompt: string, userId: string)` — use positional parameters. The codebase rule requires named object params for all functions with 1+ parameters. `buildSystemPromptWithContext` has two positional params (`basePrompt` and `userId`) which could be passed in wrong order.
- **Suggestion**: Refactor all three to named params:
  - `getContext({ userId }: { userId: string }): Promise<LLMContext>`
  - `buildSystemPromptWithContext({ basePrompt, userId }: { basePrompt: string; userId: string }): Promise<{ prompt: string }>`
  - `buildSystemPrompt({ basePrompt, userId }: { basePrompt: string; userId: string }): Promise<string>`
- **Evidence**:

```typescript
abstract getContext(userId: string): Promise<LLMContext>;
abstract buildSystemPromptWithContext(basePrompt: string, userId: string): Promise<{ prompt: string }>;
abstract buildSystemPrompt(basePrompt: string, userId: string): Promise<string>;
```

### Finding 2: `MemoryPort` read/list/delete use positional params (violates named-params rule)

- **File**: `apps/api/src/domain/ports/services/memory.port.ts:47-73`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `read(userId: string, path: string)`, `list(userId: string, directory: string)`, and `delete(userId: string, path: string)` all use positional parameters. These have 2 string params each, making it easy to accidentally swap `userId` and `path`.
- **Suggestion**: Refactor to named params following the project convention:
  - `read({ userId, path }: { userId: string; path: string })`
  - `list({ userId, directory }: { userId: string; directory: string })`
  - `delete({ userId, path }: { userId: string; path: string })`
- **Evidence**:

```typescript
abstract read(userId: string, path: string): Promise<Result<string | null, MemoryPortError>>;
abstract list(userId: string, directory: string): Promise<Result<string[], MemoryPortError>>;
abstract delete(userId: string, path: string): Promise<Result<void, MemoryPortError>>;
```

### Finding 3: `OAuthHandler.handleInit/handleAuthorize/handleCallbackWithWizard` use `query: unknown` and `request/response: unknown`

- **File**: `apps/api/src/domain/ports/services/plugin-registry.port.ts:133-146`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `OAuthHandler` methods accept `query: unknown`, `request: unknown`, and `response: unknown`. The comment explains these are kept opaque to avoid importing NestJS/Express types into the domain layer, which is architecturally valid. However, `unknown` accepts literally any value (including `null`, numbers, etc.). `Record<string, unknown>` for `query` would at least enforce it's a key-value map.
- **Suggestion**: Change `query: unknown` to `query: Record<string, unknown>` in both `handleInit` and `handleAuthorize`. For `request` and `response` in `handleCallbackWithWizard`, `unknown` is acceptable since these are NestJS/Express objects. Add JSDoc comments explaining the expected types.
- **Evidence**:

```typescript
handleInit: (params: { userId: string; query: unknown }) => Promise<Result<{ authUrl: string }, PluginOperationError>>;
```

### Finding 4: `SyncParams.body` typed as `Record<string, unknown>`

- **File**: `apps/api/src/domain/ports/services/plugin-registry.port.ts:50`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `SyncParams.body: Record<string, unknown>` is the webhook/sync body payload. The comment notes "Optional hooks are deliberately typed as unknown at this boundary." For `body`, `Record<string, unknown>` is appropriate since it must be JSON-serializable. The `onProgress`, `resumeState`, and `onCheckpoint` fields typed as `unknown` are more concerning — they're callback functions/state objects but typed as `unknown`.
- **Suggestion**: Type the callback fields more specifically: `onProgress?: (event: unknown) => void | Promise<void>`, `onCheckpoint?: (state: unknown) => void | Promise<void>`, and `resumeState?: Record<string, unknown>`. This at least communicates the expected shapes.
- **Evidence**:

```typescript
export interface SyncParams {
  integrationId: string;
  userId: string;
  body: Record<string, unknown>;
  // Optional hooks (streaming/checkpointing) are deliberately typed as unknown at this boundary.
  onProgress?: unknown;
  resumeState?: unknown;
  onCheckpoint?: unknown;
}
```

### Finding 5: `IIntegrationsLookupPort` abstract methods use positional params via `input` object but inconsistently

- **File**: `apps/api/src/domain/ports/services/integrations-lookup.port.ts:58-78`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `findById(input: { integrationId: string; userId: string })` and `findAllForUser(input: { userId: string })` use the object param pattern correctly with the name `input`. However, `findByPlatformForUser(input: { userId: string; platform: string })` also uses `input`. The inconsistency is that `input` is a non-descriptive param name — the convention in this codebase uses destructured names (e.g., `{ userId, platform }` directly in the parameter list). While not a type-safety bug, it adds friction when reading call sites.
- **Suggestion**: Use destructured named params throughout for consistency with the codebase style: `findById({ integrationId, userId }: { integrationId: string; userId: string })`.
- **Evidence**:

```typescript
abstract findById(input: {
  integrationId: string;
  userId: string;
}): Promise<Result<IntegrationRecord | null, IntegrationsLookupPortError>>;
```
