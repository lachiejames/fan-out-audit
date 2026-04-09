# Type-Safety Audit — Batch 100

**Files audited:**

- `apps/api/src/interface/agents/workspace-agent.service.ts`
- `apps/api/src/interface/attachment-embedding/errors/attachment-embedding.errors.ts`
- `apps/api/src/interface/auth/auth.module.ts`
- `apps/api/src/interface/auth/guards/auth.guard.ts`
- `apps/api/src/interface/auth/guards/user-rate-limit.guard.ts`
- `apps/api/src/interface/common/decorators/concurrency-limit.decorator.ts`
- `apps/api/src/interface/common/interceptors/concurrency-cleanup.interceptor.ts`
- `apps/api/src/interface/http/account-deletion/account-deletion.controller.ts`

---

## Findings

### 1. workspace-agent.service.ts — type-cast

**Lines:** 441–449
**Category:** type-cast
**Severity:** medium
**Description:** `getTokenUsage: (r: unknown) => { ... }` with `const res = r as { usage?: { inputTokens?: number; ... } }` — the `executeWithReconciliation` callback takes `unknown` as the result and must cast to access token usage. The `generateText` return type from the AI SDK is well-known and could be used here.
**Code:**

```typescript
getTokenUsage: (r: unknown) => {
  const res = r as { usage?: { inputTokens?: number; outputTokens?: number; totalTokens?: number } };
  if (!res?.usage) return null;
  return { ... };
},
```

**Fix:** Type the `executeWithReconciliation` callback generically: `executeWithReconciliation<T>({ fn, getTokenUsage: (r: T) => ... })` so the actual AI SDK return type flows through without casting.

---

### 2. workspace-agent.service.ts — missing-strict-typing

**Lines:** 466–470
**Category:** missing-strict-typing
**Severity:** low
**Description:** `const usage = typeof result === "object" && result !== null && "usage" in result ? (result as { usage?: unknown }).usage : undefined` — defensive access of the `usage` field on the AI SDK result object. This suggests `executeWithReconciliation` returns `unknown` rather than a typed result.
**Code:**

```typescript
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
```

**Fix:** Once `executeWithReconciliation` is made generic (see finding #1), `result.usage` is directly accessible with the full AI SDK `GenerateTextResult` type.

---

### 3. workspace-agent.service.ts — missing-strict-typing (tools map)

**Line:** 213
**Category:** missing-strict-typing
**Severity:** low
**Description:** `const tools: Record<string, Tool> = {}` — `Tool` here is `import("ai").Tool` without generics, which is equivalent to `Tool<any, any>`. The tools map loses specific input/output types for each tool.
**Code:**

```typescript
const tools: Record<string, Tool> = {};
```

**Fix:** Use `Record<string, AnyTool>` where `AnyTool = Tool<z.ZodTypeAny, z.ZodTypeAny>` (once the type is strengthened per batch 97 finding #1), or keep `Tool` if the AI SDK requires `Record<string, Tool>` for its `generateText` `tools` parameter.

---

### 4. concurrency-cleanup.interceptor.ts — type-cast

**Line:** 31
**Category:** type-cast
**Severity:** low
**Description:** `const key = request.concurrencyKey as string | undefined` — accesses a non-standard `concurrencyKey` field added to the request by an upstream guard. The Express `Request` type doesn't include `concurrencyKey`, requiring the cast.
**Code:**

```typescript
const key = request.concurrencyKey as string | undefined;
```

**Fix:** Extend the Express `Request` interface with an ambient declaration: `declare global { namespace Express { interface Request { concurrencyKey?: string } } }` in a `.d.ts` file, eliminating the cast.

---

## No Findings

- `attachment-embedding.errors.ts` — Clean discriminated union with `retriable` flag; no findings.
- `auth.module.ts` — Minimal module definition; no findings.
- `auth.guard.ts` — Comprehensive, well-typed guard. `AuthUser` is a properly bounded type derived from `DomainAuthUser`. Token cache uses explicit `CachedAuthResult` type. No type-safety findings.
- `user-rate-limit.guard.ts` — Rate limit configuration uses typed `Record<RateLimitCategory, Record<EffectiveTier, number>>`; no findings.
- `concurrency-limit.decorator.ts` — Clean typed decorator metadata; no findings.
- `account-deletion.controller.ts` — Clean ts-rest controller; no findings.
