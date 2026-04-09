# Audit: apps-api-src-infrastructure-1

**Files inspected**: 8
**Findings**: 6

## Summary

The most impactful issues are the three identical unsafe `usage` extraction patterns across
the AI adapters (Findings 1–3): each adapter wraps `result` — already typed as
`GenerateTextResult<…>` (which has `usage: LanguageModelUsage`) — in a redundant runtime
type-guard and casts it to `{ usage?: unknown }`, converting a well-typed SDK value into
`unknown` before immediately passing it to `extractInputOutputTokensFromUsage`. Eliminating
these casts allows TypeScript to statically verify the field access and removes dead defensive
code. Two secondary issues are an inline `mimeType as AudioMediaType` widening cast that can
be replaced by a proper type-predicate (Finding 4), a repeated inline subscription-status
union literal (Finding 5), and a minor unsafe cast in `is-ai-sdk-tool.ts` (Finding 6).

## Findings

### Finding 1: Redundant `usage` extraction cast in `ActionDetectorAdapter`

- **File**: `apps/api/src/infrastructure/ai/adapters/action-detector.adapter.ts:137`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `result` is the resolved value of `generateText(…)`, whose return type is
  `GenerateTextResult<TOOLS, OUTPUT>`. That interface declares `readonly usage: LanguageModelUsage`,
  so `result.usage` is already well-typed. The code instead performs a redundant runtime shape-check
  (`typeof result === "object" && "usage" in result`) and then casts to `{ usage?: unknown }`,
  downgrading a known `LanguageModelUsage` to `unknown`. The cast is both unnecessary and unsafe:
  it silently discards all static type information and moves correctness guarantees into the
  runtime-only `extractInputOutputTokensFromUsage` implementation.
- **Suggestion**: Replace the three lines with `const usage = result.usage;` and rely on the
  SDK's typed `LanguageModelUsage` directly. The downstream `extractInputOutputTokensFromUsage`
  already accepts `unknown` so the call-site signature does not need to change — but the cast
  can be dropped entirely because `result.usage` satisfies `unknown`.
- **Evidence**:

```typescript
// current (lines 137-140)
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
```

---

### Finding 2: Redundant `usage` extraction cast in `AIWorkItemGenerationAdapter`

- **File**: `apps/api/src/infrastructure/ai/adapters/ai-work-item-generation.adapter.ts:127`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Identical pattern to Finding 1. `result` is the resolved value of
  `generateText(…)` and therefore carries `usage: LanguageModelUsage` at compile time.
  The defensive `typeof result === "object" && "usage" in result` guard plus the
  `as { usage?: unknown }` cast are entirely redundant.
- **Suggestion**: Replace with `const usage = result.usage;`.
- **Evidence**:

```typescript
// current (lines 127-130)
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
```

---

### Finding 3: Redundant `usage` extraction cast in `TriageAIAdapter`

- **File**: `apps/api/src/infrastructure/ai/adapters/triage-ai.adapter.ts:166`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Same pattern as Findings 1 and 2. The cast appears in all three adapters,
  likely copy-pasted. Since `safeApiCall` preserves the generic type `T` of the wrapped
  `generateText` call, `apiResult.value` is typed as `GenerateTextResult<…>` with a known
  `usage` field.
- **Suggestion**: Replace with `const usage = result.usage;`. Consider extracting a shared
  `recordAiUsage` helper so this pattern cannot diverge again.
- **Evidence**:

```typescript
// current (lines 166-169)
const usage =
  typeof result === "object" && result !== null && "usage" in result
    ? (result as { usage?: unknown }).usage
    : undefined;
```

---

### Finding 4: `mimeType as AudioMediaType` widening cast in `isSupportedAudioType`

- **File**: `apps/api/src/infrastructure/ai/utils/audio-content.utils.ts:30`
- **Category**: type-cast
- **Impact**: low
- **Description**: `SUPPORTED_AUDIO_TYPES.includes(mimeType as AudioMediaType)` casts an
  arbitrary `string` to `AudioMediaType` purely to satisfy the typed `includes` overload.
  TypeScript's `ReadonlyArray<T>.includes` only accepts `T`, so callers must cast. The same
  file has a parallel function `isSupportedEpubType` in `epub-content.utils.ts` that is
  correctly declared as a type predicate (`mimeType is EpubMediaType`), yet
  `isSupportedAudioType` lacks that predicate and uses `as` instead. The missing predicate
  means callers receive only `boolean` and must cast again at the usage site.
- **Suggestion**: Declare a proper type predicate and use the idiomatic widening approach:
  ```typescript
  export function isSupportedAudioType({ mimeType }: { mimeType: string }): mimeType is AudioMediaType {
    return (SUPPORTED_AUDIO_TYPES as readonly string[]).includes(mimeType);
  }
  ```
  This removes the `as AudioMediaType` cast and gives callers a narrowed type after the
  check, consistent with how `isSupportedEpubType` works in the same codebase.
- **Evidence**:

```typescript
// current (line 30)
return SUPPORTED_AUDIO_TYPES.includes(mimeType as AudioMediaType);
```

---

### Finding 5: Duplicate subscription-status union literal in `ai-model.utils.ts`

- **File**: `apps/api/src/infrastructure/ai/model/ai-model.utils.ts:44,101,123,219`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: The inline union `"trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused"`
  appears four times in `ai-model.utils.ts` alone, and eleven times across the `apps/api/src/`
  tree. The canonical type `SubscriptionStatus` already exists in `@slopweaver/contracts`
  (derived from `subscriptionStatusSchema`) and is already imported elsewhere in the API.
  `ai-model.port.ts` (domain layer) also inlines the same union twice. The duplication means
  that adding or renaming a status value requires touching eleven files.
- **Suggestion**: Import and use `SubscriptionStatus` from `@slopweaver/contracts` wherever
  the inline union is used in the infrastructure and domain layers. `ai-model.port.ts` is a
  domain file that should not import from `@slopweaver/contracts` if that would violate layer
  rules; in that case, define a single alias in `shared/constants` and re-export it from
  the port file.
- **Evidence**:

```typescript
// repeated inline union at lines 44, 101, 123, 219 of ai-model.utils.ts
subscriptionStatus?: "trialing" | "active" | "past_due" | "cancelled" | "expired" | "paused";
// canonical type already in @slopweaver/contracts:
export type SubscriptionStatus = z.infer<typeof subscriptionStatusSchema>;
```

---

### Finding 6: `as Record<string, unknown>` cast in `describeAiSdkToolShape`

- **File**: `apps/api/src/infrastructure/ai/utils/is-ai-sdk-tool.ts:32`
- **Category**: type-cast
- **Impact**: low
- **Description**: `Object.keys(value as Record<string, unknown>)` casts `value` (already
  narrowed to a non-null object by the guard above) to `Record<string, unknown>` purely to
  satisfy `Object.keys`. `Object.keys` accepts `object`, so the cast is not needed for
  correctness and introduces a mild suppression of TypeScript's index-signature checking.
- **Suggestion**: Pass `value` directly: `Object.keys(value)`. TypeScript accepts any
  `object` (or `{}`) for `Object.keys`. Alternatively type the narrowed parameter as
  `object` at the top of the branch.
- **Evidence**:

```typescript
// current (line 32)
const keys = Object.keys(value as Record<string, unknown>).slice(0, 50);
```
