# Audit: apps-app-src-shared-1

**Files inspected**: 8
**Findings**: 6

## Summary

This slice covers shared config files, context providers, and the Supabase constants. The dominant issues are `Record<string, ...>` lookup maps where a stricter key type is available, unsafe `as` casts for priority/config lookups that silently swallow bad input, a local `MessagePriority` type that duplicates information already in `@slopweaver/contracts`, and a loosely-typed custom event handler in `PaywallContext` that uses `Record<string, unknown>` for the event detail.

---

## Findings

### Finding 1: `getMessagePriorityConfig` casts `string` to `MessagePriority`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/config/priority-config.ts:62`
- **Category**: type-cast
- **Impact**: medium
- **Description**: The function accepts `priority: MessagePriority | string` and immediately casts the value to `MessagePriority` for the `Record` lookup. If called with an unknown string (e.g. an API value that no longer maps), the cast silently returns `undefined` which is not reflected in the return type `PriorityUIConfig`.
- **Suggestion**: Return `PriorityUIConfig | undefined` (or add a fallback) and remove the `as MessagePriority` cast. Alternatively, restrict the parameter to `MessagePriority` and have callers validate before calling.
- **Evidence**: `return MESSAGE_PRIORITY_CONFIG[priority as MessagePriority];`

### Finding 2: Local `MessagePriority` type may duplicate a contracts type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/config/priority-config.ts:11`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `export type MessagePriority = "urgent" | "important" | "normal"` is defined locally. The contracts package already contains priority-related types for todos and messages (`TodoPriority`, etc.). If the contracts type expands (e.g. adds a new priority tier), the local type will silently diverge.
- **Suggestion**: Check whether `@slopweaver/contracts` exports a compatible priority union. If so, import and re-use it (or derive from it with `Pick`/utility types). If the local type genuinely covers a different domain, add a comment explaining the distinction.
- **Evidence**: `export type MessagePriority = "urgent" | "important" | "normal";`

### Finding 3: Loose `CustomEvent` detail typing in `PaywallContext`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/context/PaywallContext.tsx:118-127`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: The `slopweaver:payment-required` event handler casts the event to `CustomEvent<{ details?: unknown; message?: unknown; statusCode?: unknown }>` and then further accesses `.detail.details` as a loosely typed object. All subsequent checks (`typeof details?.requiredActions === "number"` etc.) are manually re-validated. A typed custom event interface would provide static safety and remove the defensive runtime checks.
- **Suggestion**: Define a typed `SlopweaverPaymentRequiredEvent` interface (or import it if already defined elsewhere) that exactly describes the event detail shape, and type the `addEventListener` call with that interface. This eliminates the nested `as` casts for `details`, `code`, `requiredActions`, etc.
- **Evidence**: `const custom = event as CustomEvent<{ details?: unknown; message?: unknown; statusCode?: unknown }>;`

### Finding 4: `normalizeTaskCategory` casts `string` to `TaskCategory`

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/config/task-category-config.ts:18-23`
- **Category**: type-cast
- **Impact**: low
- **Description**: `normalizeTaskCategory` checks `category in TASK_CATEGORY_CONFIG` but then returns `category as TaskCategory`. TypeScript cannot narrow a string to a key type via `in` without a type guard. This is correct at runtime, but the `as` cast is still required because the `in` check doesn't narrow the type in TS.
- **Suggestion**: Use a proper type predicate: `function isTaskCategory(value: string): value is TaskCategory { return value in TASK_CATEGORY_CONFIG; }` and return the narrowed type. This replaces the `as` cast with a statically proved narrowing.
- **Evidence**: `return category as TaskCategory;`

### Finding 5: `platform-config.ts` — `platformId as PlatformId` cast without guard

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/config/platform-config.ts:82`
- **Category**: type-cast
- **Impact**: low
- **Description**: `PLATFORMS[platformId as PlatformId]` casts an arbitrary `string` parameter to `PlatformId` for the record lookup. The function has a `if (!platform)` null check after, which handles the undefined case at runtime. But TypeScript doesn't know the input is invalid — the cast pretends it is always a `PlatformId`.
- **Suggestion**: Use `isPlatformId(platformId)` (already imported from `@slopweaver/contracts` in `platform-metadata-display.ts`) before the lookup, or use `PLATFORMS[platformId as keyof typeof PLATFORMS]` with an explicit `undefined` check. The current runtime behavior is fine; the issue is the cast masking a type mismatch.
- **Evidence**: `const platform = PLATFORMS[platformId as PlatformId];`

### Finding 6: `SyncLimitContext` — `tier` typed as `SubscriptionTier | "free"` extends contract type

- **File**: `/Users/lachiejames/dev/slopweaver/apps/app/src/shared/context/SyncLimitContext.tsx:26`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `SyncBudget.tier` is typed as `SubscriptionTier | "free"`. If `SubscriptionTier` from contracts already includes `"free"`, this union is redundant and hints that the local and contract types may have drifted. If `"free"` is genuinely not in the contract's union, this is a separate concern that should be reflected in the contracts package.
- **Suggestion**: Verify whether `SubscriptionTier` in `@slopweaver/contracts` includes `"free"`. If yes, remove the `| "free"` suffix. If not, add `"free"` to the contract type rather than extending it locally.
- **Evidence**: `tier: SubscriptionTier | "free";`
