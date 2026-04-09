# Audit: apps-api-src-infrastructure-2

**Files inspected**: 8
**Findings**: 5

## Summary

This batch covers MIME registry infrastructure and analytics user-property building. Most files are well-typed. Findings center on three themes: (1) `as` casts used to work around the `includes()` limitation on `readonly` arrays; (2) `Record<string, MimeRegistryEntry>` where the key could be narrowed to a union; (3) `Record<string, unknown>` return type on `buildPostHogPersonProperties` which is unavoidably wide but could be annotated more intentionally; and (4) a `Record<string, number>` accumulator in the PostHog builder used where an explicit shape would be safer.

## Findings

### Finding 1: `as SomeType` cast in every `isSupportedX` type-guard (includes limitation)

- **File**: `apps/api/src/infrastructure/ai/utils/pptx-content.utils.ts:52`
- **Category**: type-cast
- **Impact**: low
- **Description**: `SUPPORTED_PPTX_TYPES.includes(mimeType as PresentationMediaType)` uses a cast to satisfy the `readonly` array's strict element type. The same pattern appears in `rtf-content.utils.ts:43`, `spreadsheet-content.utils.ts:51`, `vision-content.utils.ts:65`, and `zip-content.utils.ts:95`. Each file has its own copy of this cast.
- **Suggestion**: Use a shared helper `function isMember<T>(arr: readonly T[], value: string): value is T { return (arr as readonly string[]).includes(value); }` to centralise the one unsafe cast and make all type-guards cast-free at the call site.
- **Evidence**:

```typescript
// pptx-content.utils.ts:52
export function isSupportedPptxType(mimeType: string): mimeType is PresentationMediaType {
  return SUPPORTED_PPTX_TYPES.includes(mimeType as PresentationMediaType);
}
```

### Finding 2: `MIME_REGISTRY` key type is `string` instead of a narrower union

- **File**: `apps/api/src/infrastructure/ai/utils/mime-registry.ts:48`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `MIME_REGISTRY` is typed as `Record<string, MimeRegistryEntry>`. Because the object is declared `as const`, TypeScript knows the exact set of string keys. A type alias for the union of those keys (or using `keyof typeof MIME_REGISTRY`) would prevent accidental lookups with arbitrary strings and enable exhaustive checks.
- **Suggestion**: Derive a `MimeType = keyof typeof MIME_REGISTRY` alias and tighten `getMimePolicy` / `getMimeTypesForPolicy` to accept `MimeType` rather than `string`.
- **Evidence**:

```typescript
export const MIME_REGISTRY: Record<string, MimeRegistryEntry> = { ... } as const;
// getMimePolicy accepts any string:
export function getMimePolicy({ mimeType, surface }: { mimeType: string; surface: ExtractionSurface }): MimePolicy {
```

### Finding 3: Duplicate `isSupportedXType` type-guard pattern across five files

- **File**: `apps/api/src/infrastructure/ai/utils/pptx-content.utils.ts:51`, `rtf-content.utils.ts:41`, `spreadsheet-content.utils.ts:50`, `vision-content.utils.ts:64`, `zip-content.utils.ts:94`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Each file independently defines a `readonly string[]` constant, a media-type union, and a type-guard that casts to that union to satisfy `includes()`. The guard logic is identical across all five files; only the array constant differs. This is not a type-safety problem per se, but the duplication means any future fix or improvement must be applied in five places.
- **Suggestion**: Extract a generic `createMimeTypeGuard<T extends string>(supported: readonly T[])` factory into a shared util, eliminating the repeated cast and the repeated guard boilerplate.
- **Evidence**:

```typescript
// Same structure in every file:
export const SUPPORTED_X_TYPES: readonly XMediaType[] = [...] as const;
export function isSupportedXType(mimeType: string): mimeType is XMediaType {
  return SUPPORTED_X_TYPES.includes(mimeType as XMediaType);
}
```

### Finding 4: `Record<string, unknown>` return type on `buildPostHogPersonProperties`

- **File**: `apps/api/src/infrastructure/analytics/posthog-user-properties.ts:172`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `buildPostHogPersonProperties` returns `Record<string, unknown>`. The function always sets a predictable set of top-level keys (e.g. `integration_count`, `connected_platforms`, `billing_period`). The type is intentionally open because PostHog properties are unstructured, but there is no documented reason the return type cannot be at least partially typed (e.g. with an intersection of a known-keys interface and `Record<string, unknown>`). Callers receive no compile-time guidance about which properties are guaranteed to be present.
- **Suggestion**: Introduce a `PostHogPersonProperties` interface for the guaranteed keys and intersect it with `Record<string, unknown>` in the return type.
- **Evidence**:

```typescript
export function buildPostHogPersonProperties({ ... }: PostHogUserEnrichmentInput): Record<string, unknown> {
  const props: Record<string, unknown> = {};
  // ...
  props["integration_count"] = nonDemoIntegrations.length;
  props["connected_platforms"] = platforms;
  // ...
  return props;
}
```

### Finding 5: `Record<string, number>` accumulators for dynamic status/health counts

- **File**: `apps/api/src/infrastructure/analytics/posthog-user-properties.ts:215`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `statusCounts` and `healthCounts` are declared as `Record<string, number>`. The `status` field on an integration is a typed enum value, so `Record<IntegrationStatus, number>` (using the correct enum type) would prevent typos and make the accumulator exhaustive.
- **Suggestion**: Import the integration status/health type from the shared contracts and use it as the key type: `Partial<Record<IntegrationStatus, number>>`.
- **Evidence**:

```typescript
const statusCounts: Record<string, number> = {};
for (const i of nonDemoIntegrations) {
  statusCounts[i.status] = (statusCounts[i.status] ?? 0) + 1;
}
```
