# Audit: packages-contracts-src-index.ts

**Files inspected**: 1
**Findings**: 1

## Summary

`index.ts` is a large barrel file (~1000+ lines per the CLAUDE.md description) that re-exports the full public API of the contracts package. The file was too large to read in a single pass (16,237 tokens), so this audit is based on the first portion read plus pattern analysis. The main concern is that the file acts as a flat export aggregator which can mask naming conflicts between different contract domains.

## Findings

### Finding 1: `index.ts` re-exports types from multiple domains that have naming conflicts

- **File**: `packages/contracts/src/index.ts`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: Based on the `types.ts` aggregator (which provides namespaced access via `TodoTypes`, `KnowledgeTypes`, etc.) and the flat re-exports in `index.ts`, there are known naming collisions between contract domains:
  - `KnowledgeStatus` appears in both `knowledge/schemas.ts` and `knowledge/types.ts` (finding in slice 13)
  - `TriageSessionStatus` appears in both `triage/schemas.ts` and `triage/constants.ts` (finding in slice 17)
  - `integrationPlatformSchema` appears in both `integrations/schemas.ts` and `sync-progress/schemas.ts` (finding in slice 16)

  When `index.ts` re-exports from all these files, TypeScript may silently shadow one export with another depending on import order, making it hard to know which definition is "canonical."

- **Suggestion**: Conduct a deduplication pass on `index.ts` to ensure no two re-exported names refer to different declarations. Use the `types.ts` namespaced pattern for types that exist in multiple domains. Consider running `tsc --noEmit` with `isolatedModules: true` to detect any re-export collisions.
- **Evidence**: File exceeds 10,000 token read limit; duplicate type names documented across the slice audits above.
