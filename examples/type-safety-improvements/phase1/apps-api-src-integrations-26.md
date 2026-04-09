# Type Safety Audit — apps/api/src/integrations (Batch 26)

**Files inspected**: 8  
**Findings**: 4

## Summary

Batch 26 covers the Microsoft Outlook integration (config/errors/plugin/actions/fetch/id-migration/oauth/search/sync helpers). The config and errors files are clean. The Outlook plugin, fetch service, and id-migration service are well-typed using `@microsoft/microsoft-graph-types` SDK types directly. One finding is the use of `platformMetadata as Record<string, unknown> | null` without validation in `id-migration.service.ts`. The OAuth service and sync helper utils both have structurally necessary cast patterns. Multiple `(response.value ?? []) as OutlookDeltaMessage[]` casts in the sync service are inherent to the untyped `@microsoft/microsoft-graph-client` response shape.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/microsoft-outlook/services/microsoft-outlook-id-migration.service.ts:77`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `integration.platformMetadata` (typed `unknown`) is cast to `Record<string, unknown> | null` to check `immutableIdsMigratedAt`. No validation is performed. If the metadata is malformed, the check silently falls through.

**Suggestion**: Use a Zod schema or type guard that validates the presence of `immutableIdsMigratedAt` before checking it.

**Evidence**:

```typescript
// microsoft-outlook-id-migration.service.ts:77-78
const metadata = integrationResult.value.platformMetadata as Record<string, unknown> | null;
if (metadata?.["immutableIdsMigratedAt"]) {
  return ok(false);
}
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/microsoft-outlook/services/microsoft-outlook-oauth.service.ts:28`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `MicrosoftOutlookPlatformMetadata` has a redundant `extends Record<string, unknown>` and `[key: string]: unknown` index signature, identical to the OneDrive and Teams metadata types (same finding as Batch 25, Finding 4). This is a systematic issue across all Microsoft platform OAuth services.

**Suggestion**: Remove the duplicate index signatures. Track this as a systemic fix across all Microsoft platform metadata interfaces.

**Evidence**:

```typescript
// microsoft-outlook-oauth.service.ts:28-33
interface MicrosoftOutlookPlatformMetadata
  extends MicrosoftPlatformMetadata<"microsoft-outlook">, Record<string, unknown> {
  [key: string]: unknown;
  platform: "microsoft-outlook";
}
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/microsoft-outlook/services/microsoft-outlook-sync.service.ts:257`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `(response.value ?? []) as OutlookDeltaMessage[]` is used in multiple places in the sync service to cast the untyped `@microsoft/microsoft-graph-client` response value. The cast is structurally unavoidable without typed wrappers, but it means malformed responses silently succeed.

**Suggestion**: Define a Zod schema for `OutlookDeltaMessage` (the subset of fields actually used) and validate each message from `response.value` before building the collection. Alternatively, use the `@microsoft/microsoft-graph-types` `Message` type with a mapper.

**Evidence**:

```typescript
// microsoft-outlook-sync.service.ts:257
const messages = (response.value ?? []) as OutlookDeltaMessage[];
// same pattern repeated at ~lines 278, 302, 319
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/microsoft-outlook/services/microsoft-outlook-sync.service.ts:329`  
**Category**: `Record<string, ...>` weakening  
**Impact**: Low

**Description**: `extractSyncConfig` receives `integration: { syncSettings?: unknown; platformMetadata?: unknown }` and discards both fields with `void`. While this is correct for Outlook (no sync config needed), the untyped `unknown` signature leaks into the base class interface. The `@removed?: unknown` field on `OutlookDeltaMessage` is also overly loose — it could be typed as `{ "@removed": { reason: string } }` per the Microsoft Graph Delta API spec.

**Suggestion**: Type `OutlookDeltaMessage["@removed"]` as `{ reason: string } | undefined` to match the actual Graph delta payload shape.

**Evidence**:

```typescript
// microsoft-outlook-sync.service.ts:68
"@removed"?: unknown;  // should be { reason: string } | undefined per Graph delta spec
```
