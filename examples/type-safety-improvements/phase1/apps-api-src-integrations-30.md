# Type-Safety Audit — Batch 91

**Files audited:**

- `apps/api/src/integrations/platforms/monday/services/monday-write.service.ts`
- `apps/api/src/integrations/platforms/monday/services/utils/monday-sync-helpers.utils.ts`
- `apps/api/src/integrations/platforms/monday/utils/monday-sync.utils.ts`
- `apps/api/src/integrations/platforms/notion/errors/notion.errors.ts`
- `apps/api/src/integrations/platforms/notion/notion.plugin.ts`
- `apps/api/src/integrations/platforms/notion/services/notion-fetch.service.ts`
- `apps/api/src/integrations/platforms/notion/services/notion-oauth.service.ts`
- `apps/api/src/integrations/platforms/notion/services/notion-search.service.ts`

---

## Findings

### 1. monday-write.service.ts — record-weakening

**Line:** 27, 38
**Category:** record-weakening
**Severity:** medium
**Description:** `columnValues?: Record<string, unknown>` in both `CreateItemParams` and `UpdateItemParams`. Monday column values are heterogeneous by design (text, number, status, date, etc.) but each column type has a documented structure. A discriminated union or per-column-type interface would enable compile-time validation.
**Code:**

```typescript
columnValues?: Record<string, unknown>
```

**Fix:** Define a `MondayColumnValue` union type covering the known Monday column value shapes, and replace `Record<string, unknown>` with `Record<string, MondayColumnValue>`.

---

### 2. notion.plugin.ts — type-cast

**Line:** 100–101
**Category:** type-cast
**Severity:** medium
**Description:** `(titleProp as Record<string, unknown>)["title"]` — double cast to access `title` array from a Notion property union. The Notion SDK `PropertyItemObjectResponse` union does not narrow cleanly without a type guard.
**Code:**

```typescript
const titleProp = properties["title"] ?? properties["Name"];
const titleArr = (titleProp as Record<string, unknown>)["title"];
```

**Fix:** Add a type guard `function isTitleProperty(p: unknown): p is { title: RichTextItemResponse[] }` and replace the cast.

---

### 3. notion-fetch.service.ts — type-cast (SDK gap)

**Line:** 188
**Category:** type-cast
**Severity:** medium
**Description:** `const dbAny = database as { description?: RichTextItemResponse[]; icon?: PageObjectResponse["icon"]; ... }` — the Notion SDK's `databases.retrieve()` return type is `GetDatabaseResponse` which is a union and does not expose `description` or `icon` at the top level.
**Code:**

```typescript
const dbAny = database as {
  description?: RichTextItemResponse[];
  icon?: PageObjectResponse["icon"];
  ...
};
```

**Fix:** Introduce a local `DatabaseObjectResponse` type guard that checks `database.object === "database"` and narrows to the full database shape, eliminating the cast.

---

### 4. notion-fetch.service.ts — type-cast

**Line:** 119
**Category:** type-cast
**Severity:** medium
**Description:** `return error as NotionError` in a `.mapErr` callback without validation. If the upstream `Result` error type is `unknown` or wider, the cast is unsafe.
**Code:**

```typescript
.mapErr((error) => error as NotionError)
```

**Fix:** Use a proper error-mapping function that checks `error.code` discriminant before casting, or restructure the `Result` chain to propagate typed errors.

---

### 5. notion-oauth.service.ts — type-cast

**Lines:** 346, 369
**Category:** type-cast
**Severity:** medium
**Description:** `integration.platformMetadata as NotionPlatformMetadata | null` — platform metadata is stored as `unknown` in the database. This cast is necessary but unvalidated.
**Code:**

```typescript
const metadata = integration.platformMetadata as NotionPlatformMetadata | null;
```

**Fix:** Add a runtime validation function `parseNotionPlatformMetadata(raw: unknown): NotionPlatformMetadata | null` using Zod or manual checks, replacing the raw cast.

---

### 6. notion-search.service.ts — record-weakening

**Line:** 256
**Category:** record-weakening
**Severity:** medium
**Description:** `filter?: Record<string, unknown>` in the `queryDatabase` method signature. The Notion SDK exports a precise filter type `QueryDatabaseParameters["filter"]` that should be used here.
**Code:**

```typescript
filter?: Record<string, unknown>
```

**Fix:** Replace with `filter?: QueryDatabaseParameters["filter"]` imported from `@notionhq/client`.

---

### 7. notion-search.service.ts — type-cast

**Line:** 288
**Category:** type-cast
**Severity:** medium
**Description:** `filter as Parameters<typeof client.dataSources.query>[0]["filter"]` — cast from `Record<string, unknown>` to the SDK filter type to bridge the loose input signature.
**Code:**

```typescript
filter: filter as Parameters<typeof client.dataSources.query>[0]["filter"];
```

**Fix:** Flows naturally from fixing finding #6 — once the param type is the SDK filter type, the cast is no longer needed.

---

### 8. notion-search.service.ts — type-cast

**Line:** 201
**Category:** type-cast
**Severity:** medium
**Description:** `result as { id: string; icon?: ...; ... }` in `searchNative` to narrow database search results from the SDK union response type.
**Code:**

```typescript
const db = result as { id: string; icon?: PageObjectResponse["icon"]; title: RichTextItemResponse[] };
```

**Fix:** Add a type guard `isDatabaseSearchResult(r: SearchResponseResult): r is DatabaseSearchResult` and use it instead.

---

## No Findings

- `monday-sync-helpers.utils.ts` — `parseColumnValues` returns `Record<string, unknown>` which is appropriate for heterogeneous Monday column values; no actionable finding.
- `monday-sync.utils.ts` — `columnValues: Record<string, unknown>` in embedding text builder matches the write service pattern; no finding beyond what is tracked in finding #1.
- `notion.errors.ts` — Clean discriminated union, no findings.
