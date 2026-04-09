# Type Safety Audit — apps/api/src/integrations (Batch 25)

**Files inspected**: 8  
**Findings**: 4

## Summary

Batch 25 covers `microsoft-calendar-relevance.utils.ts` and the Microsoft OneDrive integration. The relevance utils and error types are clean. The OneDrive plugin has a `parseCursorState as OneDriveCursorState` cast. The content-extraction module has a `fetchOnDemandFile` that returns `Promise<unknown>` and uses bracket access on a `Record<string, unknown>` cast from the Graph API response. The sync service has the recurring `.filter(Boolean) as T[]` and `nextResumeState as CursorState` patterns. The OAuth service repeats the `[key: string]: unknown` index signature on the platform metadata interface.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/microsoft-onedrive/microsoft-onedrive.plugin.ts:190`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `parseCursorState({ value: params.resumeState }) as OneDriveCursorState` casts without validation. This is the same pattern found in jira/linear/microsoft-calendar plugins.

**Suggestion**: Validate `parseCursorState` output against a `OneDriveCursorState` Zod schema before assigning.

**Evidence**:

```typescript
// microsoft-onedrive.plugin.ts:190
syncParams.resumeState = parseCursorState({ value: params.resumeState }) as OneDriveCursorState;
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/microsoft-onedrive/services/microsoft-onedrive-content-extraction.ts:150`  
**Category**: Missing strict typing  
**Impact**: Medium

**Description**: `fetchOnDemandFile` returns `Promise<unknown>`. Callers must either cast the result or write runtime type guards at every call site. The function body builds a known object shape but does not type it. This forces type unsafety upstream.

**Suggestion**: Define a `OnDemandFile` interface with the exact properties the function returns (`id`, `name`, `mimeType`, `content`, etc.) and make the return type `Promise<OnDemandFile | null>`.

**Evidence**:

```typescript
// microsoft-onedrive-content-extraction.ts:150
}): Promise<unknown> {
  // ...
  return {
    content: contentText,
    contentExtracted,
    id: file["id"],  // bracket access on Record<string, unknown> cast
    mimeType,
    modifiedTime: file["lastModifiedDateTime"],
    name: file["name"],
    size: file["size"],
    webUrl: file["webUrl"],
  };
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/microsoft-onedrive/services/microsoft-onedrive-sync.service.ts:449`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `ctx.files = selection.selected.map((item) => fileCache.get(item.id)).filter(Boolean) as MicrosoftOneDriveSyncContext["files"]`. The `.filter(Boolean) as T` pattern recurs across sync services. A typed predicate is safer.

**Suggestion**: Use `.filter((x): x is NonNullable<typeof x> => x !== undefined)`.

**Evidence**:

```typescript
// microsoft-onedrive-sync.service.ts:449
ctx.files = selection.selected
  .map((item) => fileCache.get(item.id))
  .filter(Boolean) as MicrosoftOneDriveSyncContext["files"];
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/microsoft-onedrive/services/microsoft-onedrive-oauth.service.ts:30`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `MicrosoftOneDrivePlatformMetadata` extends `MicrosoftPlatformMetadata<"microsoft-onedrive">` but adds both `extends Record<string, unknown>` and `[key: string]: unknown` — a duplicate open index signature. This prevents TypeScript from enforcing that known properties are accessed correctly. The same pattern appears in `microsoft-outlook-oauth.service.ts` and `microsoft-teams-oauth.service.ts`.

**Suggestion**: Remove the duplicate `extends Record<string, unknown>` and `[key: string]: unknown`. If `MicrosoftPlatformMetadata` itself already has an index signature (through `PlatformMetadata`), the extension is redundant.

**Evidence**:

```typescript
// microsoft-onedrive-oauth.service.ts:28-33
interface MicrosoftOneDrivePlatformMetadata
  extends MicrosoftPlatformMetadata<"microsoft-onedrive">, Record<string, unknown> {
  [key: string]: unknown; // duplicate - also declared via Record<string, unknown> extend
  platform: "microsoft-onedrive";
}
```
