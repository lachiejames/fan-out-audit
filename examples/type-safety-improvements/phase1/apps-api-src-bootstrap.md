# Audit: apps-api-src-bootstrap

**Files inspected**: 6
**Findings**: 7

## Summary

The bootstrap layer is well-structured overall. The most impactful findings are: (1) a stale Sentry `tracesSampler` bracket-access pattern that references a removed `transactionContext` key instead of the current top-level `name` field, silently returning `undefined` for all sampling decisions; (2) repeated `as PluginOperationError` casts across every error-mapping closure in `plugin-registry.adapter.ts` that could be replaced with a single typed helper; (3) a `BudgetConfig` type alias in `integration-plugin.interface.ts` that is pure duplication of `PlatformBudgetConfig`; and (4) two `configService.get<string>()` calls that return `string | undefined` but are consumed without narrowing, relying on the caller to check for `undefined`.

## Findings

### Finding 1: Stale Sentry `tracesSampler` bracket-access silently returns `undefined`

- **File**: `apps/api/src/bootstrap/sentry.ts:88`
- **Category**: `missing-strict-typing`
- **Impact**: high
- **Description**: The `tracesSampler` callback reads `samplingContext["transactionContext"]?.name` using a bracket-access on the index-typed `CustomSamplingContext` base (which allows `[key: string]: any`). In Sentry SDK v10 the `transactionContext` wrapper was removed; `name` is now a **direct top-level property** on `TracesSamplerSamplingContext`. The current code always evaluates `name` as `undefined`, so every sampling guard (`name?.includes(...)`) is silently falsy and the function always falls through to the `0.1` default — health checks are never suppressed, auth routes are never always-sampled, and sync/embed routes are never down-sampled.
- **Suggestion**: Replace `samplingContext["transactionContext"]?.name` with `samplingContext.name`. Because `TracesSamplerSamplingContext` declares `name: string` as a required direct property, TypeScript will surface any future API drift at compile time.
- **Evidence**:

```typescript
// sentry.ts:88 — current (broken)
const name = samplingContext["transactionContext"]?.name;

// Sentry SDK v10 type (samplingcontext.d.ts)
export interface SamplingContext extends CustomSamplingContext {
  /** The name of the span being sampled. */
  name: string;
  // ...
}

// Fix
const name = samplingContext.name; // typed as string, no bracket-access needed
```

---

### Finding 2: Repeated `as PluginOperationError` casts in error-mapping closures

- **File**: `apps/api/src/bootstrap/adapters/plugin-registry.adapter.ts:118,127,136,150,176,198,246,258,268`
- **Category**: `type-cast`
- **Impact**: medium
- **Description**: Every `.mapErr()` call builds a `{ message, code? }` object literal and immediately casts it `as PluginOperationError`. The cast is used because the spread conditional `{ ...(e.code !== undefined && { code: e.code }) }` produces an object type that TypeScript cannot automatically narrow to `PluginOperationError`. Since `PluginOperationError` is `{ code?: string; message: string }`, a small private helper that constructs the value with explicit typing would eliminate all nine casts without losing any safety.
- **Suggestion**: Extract a private `toPortError` helper typed as `(e: { code?: unknown; message: string }) => PluginOperationError`. This removes the nine casts and centralises the transformation logic.
- **Evidence**:

```typescript
// Repeated 9 times — e.g. line 118
(e) => ({ ...(e.code !== undefined && { code: e.code }), message: e.message }) as PluginOperationError

// Proposed helper (no cast needed)
private toPortError(e: { code?: unknown; message: string }): PluginOperationError {
  return {
    message: e.message,
    ...(typeof e.code === "string" && { code: e.code }),
  };
}
```

---

### Finding 3: `BudgetConfig` type alias duplicates `PlatformBudgetConfig`

- **File**: `apps/api/src/integrations/core/interfaces/integration-plugin.interface.ts:29`
- **Category**: `duplicate-type`
- **Impact**: low
- **Description**: `BudgetConfig` is declared as `export type BudgetConfig = PlatformBudgetConfig;` — it is a pure alias with no added constraints. The adapter's `toBudgetConfig` method accepts `BudgetConfig` and returns `PlatformBudgetConfig`, but since the two types are structurally identical the mapping is a no-op (lines 281–288 of the adapter). Any future divergence between the alias and its source would silently break the contract.
- **Suggestion**: Remove the `BudgetConfig` alias from `integration-plugin.interface.ts` and use `PlatformBudgetConfig` directly throughout `IntegrationPlugin` and the adapter. The `toBudgetConfig` mapping method can then be deleted since `BudgetConfig === PlatformBudgetConfig`.
- **Evidence**:

```typescript
// integration-plugin.interface.ts:29
export type BudgetConfig = PlatformBudgetConfig; // pure alias, zero added semantics

// plugin-registry.adapter.ts:280–288 — no-op mapping due to identical shapes
private toBudgetConfig({ config }: { config: BudgetConfig }): PlatformBudgetConfig {
  return {
    category: config.category,
    defaultLimit: config.defaultLimit,
    displayName: config.displayName,
    platform: config.platform,
    unit: config.unit,
  };
}
```

---

### Finding 4: `configService.get<string>()` returns `string | undefined` but is used without narrowing

- **File**: `apps/api/src/bootstrap/app.ts:35`, `apps/api/src/bootstrap/app.ts:143`
- **Category**: `missing-strict-typing`
- **Impact**: medium
- **Description**: `ConfigService.get<T>()` (without `WasValidated = true`) returns `T | undefined` (see `ValidatedResult<false, T>`). Both calls assign the result to a local variable typed as `string | undefined`, but:
  - Line 35: `NODE_ENV` is passed to `helmet({ hsts: { preload: NODE_ENV === "production" } })` — the comparison is safe (returns `false` when `undefined`) but the type is wider than intended and `NODE_ENV` is a known-required env var validated by `apiEnvSchema`.
  - Line 143: `csrfSecret` is checked via `if (!csrfSecret)` before use, which is correct, but the variable's type is `string | undefined` rather than `string`, so the narrowing is implicit rather than declared.
  - Both calls could use `configService.getOrThrow<string>()` for `NODE_ENV` (always required) or remain as-is with an explicit `string | undefined` annotation where `undefined` is genuinely possible.
- **Suggestion**: Use `configService.getOrThrow<string>("NODE_ENV")` for `NODE_ENV` since it is validated by `apiEnvSchema` and crashing early is the correct behaviour. For `CSRF_SECRET`, the existing `if (!csrfSecret)` guard is correct; adding an explicit `string | undefined` type annotation makes the intent clear.
- **Evidence**:

```typescript
// app.ts:35
const NODE_ENV = configService.get<string>("NODE_ENV"); // string | undefined

// app.ts:143
const csrfSecret = configService.get<string>("CSRF_SECRET"); // string | undefined
// followed by: if (!csrfSecret) { throw new Error(...) }

// Fix for NODE_ENV (validated, always present)
const NODE_ENV = configService.getOrThrow<string>("NODE_ENV"); // string
```

---

### Finding 5: `expressApp.use(...)` error handler cast `err as JsonParseError` follows an unchecked type assertion

- **File**: `apps/api/src/bootstrap/app.ts:291`
- **Category**: `type-cast`
- **Impact**: low
- **Description**: The Express 4-argument error handler receives `err: Error` and immediately re-widens it via `const jsonError = err as JsonParseError`. The local `JsonParseError` interface (defined inline at line 282) adds three optional properties. The cast is safe in practice because the properties are optional and the code does check them before use (`jsonError.type === "entity.parse.failed"`), but the cast bypasses TypeScript's structural checks. A narrowing type-guard would be safer and equally concise.
- **Suggestion**: Replace the cast with a type-predicate guard `function isJsonParseError(e: Error): e is JsonParseError { return e instanceof SyntaxError && ... }`. Alternatively, keep the inline `instanceof SyntaxError` check that already exists on line 295 and remove the cast — `jsonError.type` can be accessed via `(err as JsonParseError).type` only at the point of use, or by checking `"type" in err`.
- **Evidence**:

```typescript
// app.ts:290–298
expressApp.use((err: Error, _req: Request, res: Response, _next: NextFunction) => {
  const jsonError = err as JsonParseError; // cast — bypasses structural check

  const isJsonParseError =
    err instanceof SyntaxError &&
    (jsonError.type === "entity.parse.failed" || ...);
```

---

### Finding 6: `syncMessages` closure uses `as PluginSyncParams` to construct a partial object

- **File**: `apps/api/src/bootstrap/adapters/plugin-registry.adapter.ts:188–204`
- **Category**: `type-cast`
- **Impact**: medium
- **Description**: The `syncMessages` wrapper constructs `pluginParams` as `{} as PluginSyncParams` (partial) and then assigns optional properties conditionally. The initial object satisfies only three of the required fields before the cast, so TypeScript cannot verify structural completeness at the cast site. If `PluginSyncParams` gains a new required field the cast will silently mask the missing assignment.
- **Suggestion**: Build the full required-fields object inline and extract optional properties in a second step, using `Partial<Pick<PluginSyncParams, "onCheckpoint" | "onProgress" | "resumeState">>` spread rather than the upfront cast.
- **Evidence**:

```typescript
// plugin-registry.adapter.ts:188–204
type PluginSyncParams = Parameters<typeof plugin.syncMessages>[0];
const pluginParams = {
  body: params.body,
  integrationId: params.integrationId,
  userId: params.userId,
} as PluginSyncParams; // cast on incomplete object
// optional fields assigned imperatively after cast
if (params.onCheckpoint !== undefined) {
  pluginParams.onCheckpoint = params.onCheckpoint as NonNullable<PluginSyncParams["onCheckpoint"]>;
}
```

---

### Finding 7: `exceptionFactory` in `ValidationPipe` declares `unknown` return type instead of `BadRequestException`

- **File**: `apps/api/src/bootstrap/app.ts:62`
- **Category**: `missing-strict-typing`
- **Impact**: low
- **Description**: The `exceptionFactory` callback is typed as returning `unknown` (the NestJS `ValidationPipe` option signature uses `(errors: ValidationError[]) => unknown`). The implementation always returns `new BadRequestException(...)` which is a concrete `HttpException`. While harmless at runtime, the `unknown` return type prevents any type-level guarantee that the factory always produces an `HttpException` subclass. If the contract changes (e.g., a different exception class is mistakenly returned), TypeScript will not catch it.
- **Suggestion**: Annotate the return type explicitly as `BadRequestException` (or `HttpException`) to add compile-time safety: `exceptionFactory(errors: ValidationError[]): BadRequestException { ... }`.
- **Evidence**:

```typescript
// app.ts:62–64
new ValidationPipe({
  exceptionFactory(errors: ValidationError[]): unknown {
    return new BadRequestException({ errors });
  },
```
