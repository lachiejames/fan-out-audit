# Audit: apps-api-src-domain-2

**Files inspected**: 8
**Findings**: 4

## Summary

The domain port files are well-structured. Several findings relate to intentional architecture-boundary trade-offs (`DomainTool.execute` accepts `unknown`, `DomainTools` is `Record<string, DomainTool>`) but a few have concrete improvement opportunities: `AgentStreamResult.toolCalls/toolResults` use `unknown` for args/result types where a narrower bound is possible, `ActionDetectionContext.metadata` is `Record<string, unknown> | null` where a documented union would be clearer, and the `IPostHogPort.capture/identify` properties use `Record<string, unknown>` where PostHog's own `Properties` type could be leveraged.

## Findings

### Finding 1: `AgentStreamResult.toolCalls` args typed as `unknown`

- **File**: `apps/api/src/domain/ports/ai/chat-agent-factory.port.ts:60-61`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `toolCalls: Promise<{ toolCallId: string; toolName: string; args: unknown }[]>` and `toolResults: Promise<{ toolCallId: string; toolName: string; result: unknown }[]>` use `unknown` for the actual tool inputs and outputs. While the domain intentionally avoids importing from the `ai` SDK (comment says so), even within the domain layer these could be typed as `Record<string, unknown>` rather than bare `unknown` since all AI SDK tool args are JSON-serializable objects. `unknown` forces every consumer to perform their own unsafe narrowing without guidance.
- **Suggestion**: Change `args: unknown` to `args: Record<string, unknown>` and `result: unknown` to `result: Record<string, unknown> | string | null` to give callers a narrower, documented contract without importing SDK types.
- **Evidence**:

```typescript
readonly toolCalls: Promise<{ toolCallId: string; toolName: string; args: unknown }[]>;
readonly toolResults: Promise<{ toolCallId: string; toolName: string; result: unknown }[]>;
```

### Finding 2: `DomainTool.execute` accepts and returns `unknown`

- **File**: `apps/api/src/domain/ports/ai/domain-tool.types.ts:21`
- **Category**: any-usage
- **Impact**: low
- **Description**: `execute: (args: unknown) => unknown | Promise<unknown>` is intentionally loose to avoid SDK type leakage (documented in file header). However, `unknown` for args forces every `execute` implementation to perform runtime narrowing. A generic signature `execute<TArgs = unknown, TResult = unknown>(args: TArgs): TResult | Promise<TResult>` or constraining args to `Record<string, unknown>` would add a small amount of safety without pulling in SDK types.
- **Suggestion**: Consider `execute: (args: Record<string, unknown>) => unknown | Promise<unknown>` for args, since all AI SDK tool arguments are JSON objects. The return type `unknown` is acceptable.
- **Evidence**:

```typescript
execute: (args: unknown) => unknown | Promise<unknown>;
```

### Finding 3: `ActionDetectionContext.metadata` typed as `Record<string, unknown> | null`

- **File**: `apps/api/src/domain/ports/ai/action-detector.port.ts:29`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `metadata: Record<string, unknown> | null` on `ActionDetectionContext` is very generic. The comment `source: string` nearby suggests this is platform-specific metadata (Slack message metadata, Gmail headers, etc.). A discriminated union per source, or at minimum a documented pattern, would prevent accidental misuse where callers pass arbitrary key-value pairs.
- **Suggestion**: If the actual keys are known per platform, define typed metadata variants. Otherwise, add a JSDoc comment listing what keys are expected so callers have guidance.
- **Evidence**:

```typescript
export interface ActionDetectionContext {
  // ...
  metadata: Record<string, unknown> | null;
  // ...
}
```

### Finding 4: `IPostHogPort` uses `Record<string, unknown>` for event properties instead of PostHog SDK types

- **File**: `apps/api/src/domain/ports/analytics/posthog.port.ts:23-32`
- **Category**: sdk-type-duplication
- **Impact**: low
- **Description**: `properties?: Record<string, unknown>` in `capture()` and `properties: Record<string, unknown>` in `identify()` are generic dictionaries. The PostHog Node.js SDK exports a `Properties` type (`Record<string, unknown>` in practice, but also a `CaptureMessage` / `IdentifyMessage` struct). Since the domain port intentionally abstracts PostHog away, the current `Record<string, unknown>` is appropriate — but the `identify` method's `properties` should be `Record<string, unknown>` with a JSDoc note that it maps to PostHog person properties, which is already present. This finding is low priority as the current typing matches the SDK's underlying type anyway.
- **Suggestion**: No code change required; add a JSDoc link to PostHog person-properties documentation so implementors know the expected keys.
- **Evidence**:

```typescript
abstract identify(params: { distinctId: string; properties: Record<string, unknown> }): void;
```
