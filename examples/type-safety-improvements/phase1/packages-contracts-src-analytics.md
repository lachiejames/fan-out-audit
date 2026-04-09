# Audit: packages-contracts-src-analytics

**Files inspected**: 1
**Findings**: 4

## Summary

The event taxonomy file is well-structured with strong use of `as const` and derived union types. The main weaknesses are loose `string` fields in event property interfaces where narrower types exist, and one `Record<string, ...>` where a union key is available.

## Findings

### Finding 1: `platform` field typed as `string` in IntegrationEventProperties

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/analytics/event-taxonomy.ts:137`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `IntegrationEventProperties.platform` is typed as `string`, but the codebase has a `PlatformId` union available in `integrations/platforms.ts`. Any platform string will pass, including typos.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and type this as `platform: PlatformId | string` or `platform: PlatformId` (if analytics only tracks known integrations).
- **Evidence**: `platform: string;` at line 137.

### Finding 2: `platform` field typed as `string` in SyncEventProperties

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/analytics/event-taxonomy.ts:145`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: Same as Finding 1 — `SyncEventProperties.platform` is a bare `string` with no narrowing to known platform IDs.
- **Suggestion**: Use `PlatformId` (or `PlatformId | string` for forward-compat) to catch typos at call sites.
- **Evidence**: `platform: string;` at line 145.

### Finding 3: `work_item_kind` and `origin` typed as loose `string` in WorkItemEventProperties

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/analytics/event-taxonomy.ts:155-157`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `work_item_kind` and `origin` are `string | undefined`. There are likely known enums for work item kinds (e.g., `WorkItemType`) and origins, but these are not referenced here.
- **Suggestion**: If `WorkItemType` (from work-items contract) is the canonical set, type `work_item_kind` as `WorkItemType | undefined`. Similarly consider a union for `origin`.
- **Evidence**: `work_item_kind?: string;` at line 156; `origin?: string;` at line 157.

### Finding 4: `error_type` and `error_code` typed as loose `string` in ErrorEventProperties

- **File**: `/Users/lachiejames/dev/slopweaver/packages/contracts/src/analytics/event-taxonomy.ts:183-184`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `error_code` and `error_type` are loose `string | null`. A canonical error code enum (e.g., `PaywallErrorCode`) exists for billing errors. Keeping these as `string` means no compile-time enforcement of consistent error identifiers.
- **Suggestion**: Accept that analytics properties are intentionally broad, but add a JSDoc listing the known `error_type` values to act as soft documentation. Alternatively, use a union of known type strings with a fallback: `"api" | "billing" | "auth" | (string & {})`.
- **Evidence**: `error_code?: string | null;` and `error_type?: string | null;` at lines 183-184.
