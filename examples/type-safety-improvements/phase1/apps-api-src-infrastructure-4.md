# Audit: apps-api-src-infrastructure-4

**Files inspected**: 8
**Findings**: 6

## Summary

This batch covers document-intelligence prompts, the contract-analysis, document-intelligence, entity-extraction, OCR, and summary services. The files are generally well-structured. Key findings: (1) several services use `result as { usage?: { ... } }` casts to extract token usage from an opaque `unknown` return type; (2) `ContractAnalysisService` and `EntityExtractionService` cast the validated Zod result back to the domain type (`as ContractAnalysis`, `as ExtractedEntities`) when the Zod schema already guarantees the correct shape; (3) `OCRService` re-declares its own local `VisionMediaType` that duplicates the one in `vision-content.utils.ts`.

## Findings

### Finding 1: Repeated `result as { usage?: { ... } }` cast in `getTokenUsage` callbacks

- **File**: `apps/api/src/infrastructure/document-intelligence/services/contract-analysis.service.ts:175`, `entity-extraction.service.ts:163`, `ocr.service.ts:135`, `ocr.service.ts:358`, `summary.service.ts:120`
- **Category**: type-cast
- **Impact**: medium
- **Description**: Every service that calls `executeWithReconciliation` has a `getTokenUsage` callback that casts the result to `{ usage?: { input_tokens?: number; output_tokens?: number } }`. This identical pattern is copy-pasted in five places across four files. If the Anthropic SDK changes the shape of the usage object, all five casts silently return 0.
- **Suggestion**: Extract a typed helper `extractAnthropicTokenUsage(result: unknown)` into a shared utility (e.g. `shared/utils/ai-sdk-error.utils.ts` or a new `anthropic-token-usage.utils.ts`) and replace all five inline casts with a single call.
- **Evidence**:

```typescript
// Identical block in contract-analysis.service.ts:175, entity-extraction.service.ts:163, etc.
getTokenUsage: (result) => {
  const r = result as { usage?: { input_tokens?: number; output_tokens?: number } };
  const input = r?.usage?.input_tokens ?? 0;
  const output = r?.usage?.output_tokens ?? 0;
  return input + output > 0
    ? { inputTokens: input, outputTokens: output, totalTokens: input + output }
    : null;
},
```

### Finding 2: Redundant `as ContractAnalysis` cast after Zod `.safeParse()`

- **File**: `apps/api/src/infrastructure/document-intelligence/services/contract-analysis.service.ts:267`
- **Category**: type-cast
- **Impact**: low
- **Description**: After `contractAnalysisSchema.safeParse(parsed)` succeeds, `validatedResult.data` is typed as the inferred Zod output. The final `return ok(validated as ContractAnalysis)` cast is unnecessary if `ContractAnalysis` matches the Zod schema's inferred type. The cast hides any actual mismatch between the Zod schema and the domain type.
- **Suggestion**: Replace the cast with `satisfies ContractAnalysis` or, better, derive `ContractAnalysis` from the schema with `z.infer<typeof contractAnalysisSchema>` so they are structurally identical by definition and the cast can be removed.
- **Evidence**:

```typescript
return ok(validated as ContractAnalysis);
```

### Finding 3: Redundant `as ExtractedEntities` cast after Zod `.parse()`

- **File**: `apps/api/src/infrastructure/document-intelligence/services/entity-extraction.service.ts:236`
- **Category**: type-cast
- **Impact**: low
- **Description**: Same pattern as Finding 2 but in `EntityExtractionService`. After `parseJson(... schema: extractedEntitiesSchema)` succeeds, the value is already typed as `z.infer<typeof extractedEntitiesSchema>`. The `as ExtractedEntities` cast suppresses any structural mismatch.
- **Suggestion**: Derive `ExtractedEntities` from `z.infer<typeof extractedEntitiesSchema>` and remove the cast.
- **Evidence**:

```typescript
return ok(validated as ExtractedEntities);
```

### Finding 4: Redundant `as DocumentSummary` cast after Zod `.parse()`

- **File**: `apps/api/src/infrastructure/document-intelligence/services/summary.service.ts:187`
- **Category**: type-cast
- **Impact**: low
- **Description**: Same pattern as Findings 2 and 3 in `SummaryService`.
- **Suggestion**: Derive `DocumentSummary` from `z.infer<typeof summarySchema>` or use `satisfies DocumentSummary` after the parse.
- **Evidence**:

```typescript
return ok(validated as DocumentSummary);
```

### Finding 5: Local `VisionMediaType` re-declaration in `ocr.service.ts` duplicates `vision-content.utils.ts`

- **File**: `apps/api/src/infrastructure/document-intelligence/services/ocr.service.ts:34`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `ocr.service.ts` declares `type VisionMediaType = "image/jpeg" | "image/png" | "image/gif" | "image/webp"` privately. The exact same type (with the same four values) is exported as `VisionMediaType` from `vision-content.utils.ts:23`. If a new image type is added to `vision-content.utils.ts`, the OCR service will not benefit from it, and the OCR normalizer will silently reject images that are otherwise supported.
- **Suggestion**: Remove the private declaration and import `VisionMediaType` from `vision-content.utils.ts`.
- **Evidence**:

```typescript
// ocr.service.ts:34 - private duplicate
type VisionMediaType = "image/jpeg" | "image/png" | "image/gif" | "image/webp";

// vision-content.utils.ts:23 - canonical exported definition
export type VisionMediaType = "image/jpeg" | "image/png" | "image/gif" | "image/webp";
```

### Finding 6: `result as unknown` pattern for OCR BullMQ token extraction (same as Finding 1 in ocr.service.ts)

- **File**: `apps/api/src/infrastructure/document-intelligence/services/ocr.service.ts:135,358`
- **Category**: type-cast
- **Impact**: low
- **Description**: OCR service has two separate `getTokenUsage` lambdas (one in `extractFromImage`, one in `extractFromPDFViaVision`) with the identical unsafe cast. This is a second instance of the pattern described in Finding 1.
- **Suggestion**: Same as Finding 1 — use a shared `extractAnthropicTokenUsage` helper.
- **Evidence**:

```typescript
// ocr.service.ts:135
getTokenUsage: (result) => {
  const r = result as { usage?: { input_tokens?: number; output_tokens?: number } };
  ...
},
// ocr.service.ts:358 — identical
```
