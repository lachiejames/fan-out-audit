# Type-Safety Audit: apps-api-src-integrations-14

**Batch 75** — `apps/api/src/integrations/platforms/github/` (oauth, search, sync services, subject-details utils, sync-helpers utils, sync utils), `google-calendar/` (errors, plugin)

**Files inspected**: 8
**Findings**: 4

## Summary

`github-oauth.service.ts` casts Zod-parsed data back to `{ error?: string }` — a minor inconsistency in token error handling. `github-sync.service.ts` has the same `getEmbeddingTexts(parsedItems: unknown[])` base-class cast pattern as Asana. All other files are clean.

## Findings

### Finding 1: (tokenData as { error?: string }) cast on Zod-parsed value

- **File**: `src/integrations/platforms/github/services/github-oauth.service.ts:490`
- **Category**: type-cast
- **Impact**: low
- **Description**: `(tokenData as { error?: string; error_description?: string }).error` — the token response was already parsed but the variable type was widened, requiring a cast to access the error field. The cast is safe but indicates the type narrowing was lost.
- **Suggestion**: Define a union type for the GitHub token response that includes an error variant, so the `.error` field is accessible without casting.
- **Evidence**:

```typescript
const error = (tokenData as { error?: string; error_description?: string }).error;
const errorDesc = (tokenData as { error_description?: string }).error_description;
```

---

### Finding 2: getEmbeddingTexts(parsedItems: unknown[]) in github-sync

- **File**: `src/integrations/platforms/github/services/github-sync.service.ts:578`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `protected override getEmbeddingTexts(parsedItems: unknown[]): string[]` — same `BaseSyncService` base-class `unknown[]` signature pattern as Asana. Forces `parsedItems as ResolvedNotification[]` cast on line 579 and 672.
- **Suggestion**: Make `BaseSyncService` generic on the parsed item type (same fix as asana-sync Finding 5 in batch 72).
- **Evidence**:

```typescript
protected override getEmbeddingTexts(parsedItems: unknown[]): string[] {
  return (parsedItems as ResolvedNotification[]).map((n) => buildEmbeddingText(n));
}
```

---

### Finding 3: github-search.service.ts, github-subject-details.ts, github-sync-helpers.utils.ts, github-sync.utils.ts

- **File**: Multiple
- **Category**: N/A
- **Impact**: none
- **Description**: All four files are clean — well-typed services and utilities with no unsafe casts, no `any`, no `Record` weakening.
- **Suggestion**: No changes needed.

---

### Finding 4: google-calendar errors and plugin

- **File**: `src/integrations/platforms/google-calendar/errors/google-calendar.errors.ts`, `google-calendar.plugin.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Both files are clean. Error factory uses the standard `createPlatformErrors` pattern. Plugin uses `calendar_v3.Schema$Event` SDK types directly rather than duplicating.
- **Suggestion**: No changes needed.
