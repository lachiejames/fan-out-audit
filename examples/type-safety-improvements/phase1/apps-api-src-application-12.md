# Audit: apps-api-src-application-12

**Files inspected**: 8
**Findings**: 6

## Summary

These files implement content read/resolve services and supporting utilities (decoding, filtering, metadata normalization, preview generation, and source validation). The main issues are repeated `unknown` casts to `ContentResponse["metadata"]` that bypass the type system, `FilterableQuery` locally redefining fields already typed in the contract layer, an inline logger type in `content.resolve.ts` that omits the full `ILoggerPort` interface, and several `as` casts in `content-metadata.utils.ts` that could be eliminated with better narrowing helpers.

## Findings

### Finding 1: Repeated `unknown` cast to `ContentResponse["metadata"]`

- **File**: `apps/api/src/application/content/services/content.read.ts:107` and `:132`
- **Category**: type-cast
- **Impact**: high
- **Description**: `decodedMetadata` is typed as `unknown` (line 69) and is cast to `ContentResponse["metadata"]` in two places. The value comes from `decodeContentMetadataStrict`, which already returns the correct shape. Widening the intermediate variable to `unknown` and then force-casting it throws away the guarantee that `decodeContentMetadataStrict` provides. If the return type of that function ever changes, the cast silently passes the wrong type through.
- **Suggestion**: Type `decodedMetadata` with the return type of `decodeContentMetadataStrict` (import and use `ContentMetadata` or whatever type that function returns from `@slopweaver/contracts`) so no cast is needed at the call sites.
- **Evidence**:

```typescript
let decodedMetadata: unknown;    // line 69
// ...
decodedMetadata = decodeContentMetadataStrict({ ... });   // returns typed value

// line 107 — unsafe cast
metadata: decodedMetadata as ContentResponse["metadata"],

// line 132 — same cast repeated
metadata: decodedMetadata as ContentResponse["metadata"],
```

---

### Finding 2: Inline logger type in `ResolveContentByExternalRefDeps` duplicates `ILoggerPort`

- **File**: `apps/api/src/application/content/services/content.resolve.ts:36`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `ResolveContentByExternalRefDeps` defines a hand-rolled `logger` shape with only `warn` and `error` methods. The codebase already has `ILoggerPort` (imported in `content.read.ts` from `@/domain/ports/logging/logger.port`). This bespoke slice diverges from the real port: if `ILoggerPort` gains a new required method, this type becomes stale. It also makes the caller's type narrower than the actual injected object.
- **Suggestion**: Replace the inline `{ warn: ...; error: ... }` with `ILoggerPort` from `@/domain/ports/logging/logger.port`.
- **Evidence**:

```typescript
// content.resolve.ts lines 36-39
logger: {
  warn: (meta: unknown) => void;
  error: (meta: unknown) => void;
};
```

---

### Finding 3: `FilterableQuery` locally redefines fields already in `ListContentQuery`

- **File**: `apps/api/src/application/content/utils/content-filters.utils.ts:7`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `FilterableQuery` lists nine fields typed as `unknown` that overlap exactly with fields in `ListContentQuery` (derived from the contract via `ServerInferRequest`). The local interface is a looser duplicate — any change to the contract query schema (e.g. renaming `minPriority` to `priorityMin`) would silently leave this interface out of sync. Using `Pick<ListContentQuery, ...>` or `Partial<ListContentQuery>` directly would keep the two in lockstep.
- **Suggestion**: Replace `FilterableQuery` with `Partial<Pick<ListContentQuery, 'type' | 'source' | 'integrationId' | 'hasWorkItem' | 'isRead' | 'isArchived' | 'minPriority' | 'snoozed' | 'query'>>` (importing `ListContentQuery` from `../services/content.service.types`), so field names track the contract automatically.
- **Evidence**:

```typescript
// content-filters.utils.ts lines 7-17
interface FilterableQuery {
  type?: unknown;
  source?: unknown;
  integrationId?: unknown;
  hasWorkItem?: unknown;
  isRead?: unknown;
  isArchived?: unknown;
  minPriority?: unknown;
  snoozed?: unknown;
  query?: unknown;
}
```

---

### Finding 4: Double `as` casts in `asRecord` could be eliminated

- **File**: `apps/api/src/application/content/utils/content-metadata.utils.ts:19` and `:25`
- **Category**: type-cast
- **Impact**: low
- **Description**: `asRecord` casts `value as Record<string, unknown>` after a structural guard (not-null, not-array, typeof object), and then does the same for a parsed JSON value. The first guard is sufficient — TypeScript can narrow directly if the return type annotation is used with `satisfies` or via a helper, avoiding the need for `as`. As written, the cast hides a potential edge case: `value` might still be a `Date` or other object type that passes the `typeof === 'object'` check but isn't a plain record.
- **Suggestion**: Use `Object.prototype.toString.call(value) === '[object Object]'` (or a stricter `isPlainObject` predicate) so the guard precisely targets plain records and `as` is genuinely safe. Alternatively, introduce a type predicate `isRecord(value: unknown): value is Record<string, unknown>` and call it here instead of inline casting.
- **Evidence**:

```typescript
// lines 18-20
if (typeof value === "object" && value !== null && !Array.isArray(value)) {
  return value as Record<string, unknown>; // Date, Map, Set also pass this guard
}
// lines 23-26
const parsed = JSON.parse(value) as unknown;
if (typeof parsed === "object" && parsed !== null && !Array.isArray(parsed)) {
  return parsed as Record<string, unknown>; // same issue
}
```

---

### Finding 5: `source` parameter typed as `string` instead of `ContentSource` union

- **File**: `apps/api/src/application/content/utils/content-metadata.utils.ts:51`, `content-preview.utils.ts:55` and `:75`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `normalizeMetadataForDecode`, `buildInboxPreview`, and `buildStoredContentPreview` all accept `source: string`. These functions branch on specific platform strings (`"slack"`, `"google-gmail"`). Using the plain `string` type means a misspelled platform name silently reaches the fallback branch without any compile-time warning. The contracts package already exports a `PlatformId` / `SupportedPlatform` type and `BuiltinContentSource` is defined locally in `content-source.utils.ts`; a union of both would provide exhaustive narrowing.
- **Suggestion**: Change `source: string` to `source: PlatformId | BuiltinContentSource` (importing `PlatformId` from `@slopweaver/contracts` and `BuiltinContentSource` from `./content-source.utils`). This makes impossible source values a compile error.
- **Evidence**:

```typescript
// content-metadata.utils.ts line 51-52
}: {
  source: string;   // should be PlatformId | BuiltinContentSource

// content-preview.utils.ts line 55
  source: string;   // same

// content-preview.utils.ts line 75
  source: string;   // same
```

---

### Finding 6: `parsedIdentifiers` accessed via string index literals instead of typed properties

- **File**: `apps/api/src/application/content/services/content.resolve.ts:106–179`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: Throughout the platform-identifier resolution block, identifiers returned by `parseWorkspaceUrl` are accessed via bracket notation with string literals (`parsedIdentifiers["channelId"]`, `parsedIdentifiers["threadId"]`, etc.) combined with inline `typeof ... === "string"` guards. If `parseWorkspaceUrl` returns a typed object (e.g. `Record<string, string>` or a discriminated union per platform), these repeated manual checks duplicate the contract that type already provides. The pattern is also error-prone: a renamed key in `parseWorkspaceUrl`'s return type would compile silently.
- **Suggestion**: Check the return type of `parseWorkspaceUrl`. If it already discriminates by platform (e.g. `{ platform: 'slack'; identifiers: { channelId: string; messageTs: string; ... } }`), destructure the narrowed type inside each `if (source === "slack")` branch instead of re-guarding with `typeof`. If it returns `Record<string, string>`, update its return type to a discriminated union so callers get exhaustive typed access.
- **Evidence**:

```typescript
// lines 104-108
if (
  source === "slack" &&
  parsedIdentifiers &&
  typeof parsedIdentifiers["channelId"] === "string" &&  // re-narrowing what the type should already guarantee
  typeof parsedIdentifiers["messageTs"] === "string"
) {
  const messageTs = normalizeSlackPermalinkMessageTs({ messageTs: parsedIdentifiers["messageTs"] });
  const threadTsRaw =
    parsedIdentifiers && typeof parsedIdentifiers["threadTs"] === "string" ? parsedIdentifiers["threadTs"] : null;
```
