# Audit: apps-api-src-domain-1

**Files inspected**: 8
**Findings**: 3

## Summary

The error files (`attachment.errors.ts`, `entity.errors.ts`, `network.errors.ts`, `knowledge.errors.ts`, `knowledge-graph.errors.ts`) are well-typed discriminated unions with no unsafe code. The utility files (`entity-mappers.utils.ts`, `reference-matching.utils.ts`, `error-mapper.ts`) are also clean. Three minor issues are noted: inconsistent error factory parameter style (positional vs named), a `Record<KnowledgeCategory, number>` return that could use `Partial`, and a loose `string | null` relationship field that duplicates the contracts type.

## Findings

### Finding 1: `relationship` field typed as `string | null` instead of using the contracts union

- **File**: `apps/api/src/domain/entities/utils/reference-matching.utils.ts:17`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `MatchableEntity.relationship` is typed `string | null`, but the actual values in `calculateReferenceMatch` and `matchRelationship` are compared against literal strings `"manager"`, `"direct_report"`, `"colleague"`. The contracts layer (via `DbResolvedEntity` in `entity-mappers.utils.ts`) already pins these to `"manager" | "direct_report" | "colleague" | "external" | null`. Using `string` here loses exhaustiveness and lets callers pass arbitrary strings that would never match, silently returning `null`.
- **Suggestion**: Change to `"manager" | "direct_report" | "colleague" | "external" | null` (or import that type from contracts) in `MatchableEntity.relationship`.
- **Evidence**:

```typescript
export interface MatchableEntity {
  displayName: string;
  identities: PlatformIdentity[];
  relationship: string | null; // <-- too wide
  isVip: boolean;
}
```

### Finding 2: `AttachmentErrors` factory functions use positional params (violates named-params rule)

- **File**: `apps/api/src/domain/attachments/errors/attachment.errors.ts:80-132`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: All factory functions in `AttachmentErrors` (e.g., `budgetExceeded`, `database`, `fetchFailed`, etc.) use positional parameters. The codebase-wide rule requires named object params for all functions with 1+ parameters. `fetchFailed(platform, message, cause?)` has three positional params, making parameter order errors invisible to callers.
- **Suggestion**: Refactor to named params: `fetchFailed({ platform, message, cause }: { platform: string; message: string; cause?: unknown })`.
- **Evidence**:

```typescript
fetchFailed: (platform: string, message: string, cause?: unknown): AttachmentFetchFailedError => ({
  cause,
  code: "ATTACHMENT_FETCH_FAILED",
  message,
  platform,
}),
```

### Finding 3: `ExtractionResult.itemsByCategory` uses `Record<KnowledgeCategory, number>` (strict, not partial)

- **File**: `apps/api/src/domain/ports/services/knowledge-extraction.port.ts:23-26`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `itemsByCategory: Record<KnowledgeCategory, number>` requires ALL 6 categories to be present in every result object. If a call extracts content that produces no items for some categories, callers must supply `0` for every missing category or TypeScript will complain. `Partial<Record<KnowledgeCategory, number>>` better models the domain (only categories that had extractions are present) and avoids forcing callers to pad with zeros.
- **Suggestion**: Change to `itemsByCategory: Partial<Record<KnowledgeCategory, number>>` and update callers to handle potentially-absent keys.
- **Evidence**:

```typescript
export interface ExtractionResult {
  itemsExtracted: number;
  itemsByCategory: Record<KnowledgeCategory, number>;
}
```
