# Type-Safety Audit: apps-api-src-integrations-9

**Batch 70** — `apps/api/src/integrations/core/utils/` (search-ranking, search-result-mapping, sync-helpers)

## Summary

3 files reviewed. All are well-typed pure utility modules. The only notable finding is an intentional `_original: unknown` field on reranked documents in `search-result-mapping.utils.ts` which requires a runtime cast at the consumption site. `search-ranking.utils.ts` and `sync-helpers.utils.ts` are clean with no type-safety issues.

---

## Findings

### 1

- **File**: `apps/api/src/integrations/core/utils/search-result-mapping.utils.ts`
- **Line**: 75
- **Category**: Unsafe `unknown`
- **Impact**: Low–Medium
- **Description**: `rerankedDocuments: ReadonlyArray<{ document: { _original: unknown }; relevanceScore: number }>` — the `_original` field holds a `CandidateResult` but is typed as `unknown` to allow `filterAndMapRerankedResults` to be called from `semantic-search.service.ts` without importing `CandidateResult` in both directions. At line 79 the caller (`scoreAndFilterRerankedResults`) must cast `_original` back to `CandidateResult`.
- **Suggestion**: Import `CandidateResult` directly in `search-result-mapping.utils.ts` and type `_original` as `CandidateResult`. These utils files are co-located and the circular concern does not apply here.
- **Evidence**: `rerankedDocuments: ReadonlyArray<{ document: { _original: unknown }; relevanceScore: number }>`

---

### 2

- **File**: `apps/api/src/integrations/core/utils/sync-helpers.utils.ts`
- **Line**: 165–167
- **Category**: Missing strict typing
- **Impact**: Low
- **Description**: `const [singleResult] = results; return singleResult as SyncResultResponse;` — the non-null assertion `singleResult as SyncResultResponse` is redundant because the `results.length === 1` guard already ensures the element exists. TypeScript's control-flow does not narrow array destructuring in this case, but a direct `return results[0]!` would be cleaner.
- **Suggestion**: Replace with `return results[0]!` or restructure to avoid the assertion entirely by using `.reduce()` only for the multi-element case.
- **Evidence**: `const [singleResult] = results; return singleResult as SyncResultResponse;`

---
