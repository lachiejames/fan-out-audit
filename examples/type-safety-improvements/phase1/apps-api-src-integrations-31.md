# Type-Safety Audit — Batch 92

**Files audited:**

- `apps/api/src/integrations/platforms/notion/services/notion-sync.service.ts`
- `apps/api/src/integrations/platforms/notion/utils/notion-api-error.utils.ts`
- `apps/api/src/integrations/platforms/notion/utils/notion-parent.utils.ts`
- `apps/api/src/integrations/platforms/notion/utils/notion-sync-relevance.utils.ts`
- `apps/api/src/integrations/platforms/notion/utils/notion-type-mapper.ts`
- `apps/api/src/integrations/platforms/slack/errors/slack.errors.ts`
- `apps/api/src/integrations/platforms/slack/mappers/slack-content.mappers.ts`
- `apps/api/src/integrations/platforms/slack/mappers/slack-message.mappers.ts`

---

## Findings

### 1. notion-sync.service.ts — type-cast (SDK gap)

**Line:** 465
**Category:** type-cast
**Severity:** medium
**Description:** `(await this.notionClient.pages.retrieve({ page_id })) as PageObjectResponse` — the Notion SDK `pages.retrieve()` returns `GetPageResponse` which is a union of `PageObjectResponse | PartialPageObjectResponse`. The cast skips checking which variant was returned.
**Code:**

```typescript
const page = (await client.pages.retrieve({ page_id: pageId })) as PageObjectResponse;
```

**Fix:** Add a type guard `isFullPage(r: GetPageResponse): r is PageObjectResponse` that checks `r.object === "page" && "properties" in r`, and use it instead of the cast.

---

### 2. notion-sync.service.ts — type-cast (SDK gap, repeated)

**Lines:** 491–498
**Category:** type-cast
**Severity:** medium
**Description:** `database as { description?: RichTextItemResponse[]; icon?: PageObjectResponse["icon"]; ... }` — same SDK gap as in `notion-fetch.service.ts`. The `databases.retrieve()` return type does not surface `description` and `icon` without narrowing.
**Code:**

```typescript
const dbAny = database as {
  description?: RichTextItemResponse[];
  icon?: PageObjectResponse["icon"];
  title?: RichTextItemResponse[];
};
```

**Fix:** Same fix as batch 91, finding #3 — introduce a `isDatabaseObjectResponse` type guard and use it to narrow.

---

### 3. notion-sync.service.ts — type-cast (base class override)

**Line:** 626
**Category:** type-cast
**Severity:** medium
**Description:** `parsedItems as ParsedNotionItem[]` — `BaseSyncService.parsedItems` is typed `unknown[]` because the template method pattern uses a common hook. The override must cast at the usage site.
**Code:**

```typescript
const items = parsedItems as ParsedNotionItem[];
```

**Fix:** Make `BaseSyncService` generic: `class BaseSyncService<TItem>` with `parsedItems: TItem[]`, so the override type is inferred without casting.

---

### 4. notion-sync.service.ts — missing-strict-typing

**Line:** 644
**Category:** missing-strict-typing
**Severity:** medium
**Description:** `fetchThread` method return type is `Promise<unknown>`. The method fetches full Notion page content and should return `Promise<NotionPageContent | null>` or a `Result` type.
**Code:**

```typescript
private async fetchThread(pageId: string): Promise<unknown> {
```

**Fix:** Declare the return type explicitly as `Promise<Result<NotionPageContent, NotionError>>` and propagate typed errors through the call chain.

---

### 5. notion-type-mapper.ts — type-cast

**Line:** 94
**Category:** type-cast
**Severity:** medium
**Description:** `...(block as Record<string, unknown>)` — spread to preserve block-type-specific fields beyond the `BlockObjectResponse` base union. Block subtypes carry extra keys that are not in the union discriminant.
**Code:**

```typescript
return {
  ...baseBlock,
  ...(block as Record<string, unknown>),
};
```

**Fix:** Use `Object.assign` with explicit type assertion only over the extra fields, or define union-specific mappers per block type that return typed objects.

---

### 6. notion-type-mapper.ts — record-weakening

**Line:** 227
**Category:** record-weakening
**Severity:** medium
**Description:** `mapProperties` return type `Record<string, { id?: string; type: string }>` loses property-type-specific shape (e.g., title vs. select vs. relation).
**Code:**

```typescript
function mapProperties(properties: PageObjectResponse["properties"]): Record<string, { id?: string; type: string }> {
```

**Fix:** Define a `MappedProperty` discriminated union that captures known property types and return `Record<string, MappedProperty>`.

---

### 7. notion-type-mapper.ts — type-cast

**Lines:** 229–235
**Category:** type-cast
**Severity:** medium
**Description:** `result[key] = { ...(value as Record<string, unknown>) }` — property spread during property mapping to preserve extra fields from each Notion property subtype.
**Code:**

```typescript
result[key] = { id: value.id, type: value.type, ...(value as Record<string, unknown>) };
```

**Fix:** Per-subtype mapped types (from fix #6) would eliminate the spread-as-unknown pattern.

---

## No Findings

- `notion-api-error.utils.ts` — Clean error extraction logic, no findings.
- `notion-parent.utils.ts` — Custom `NotionParent` interface used instead of SDK type; appropriate and more readable.
- `notion-sync-relevance.utils.ts` — Uses `Reflect.get` on `unknown` to read block-type fields dynamically; defensive and correct.
- `slack.errors.ts` — Clean discriminated union, no findings.
- `slack-content.mappers.ts` — Clean typed mappers, no findings.
- `slack-message.mappers.ts` — Uses `NonNullable<ConversationsHistoryResponse["messages"]>[number]` SDK type extraction; clean pattern, no findings.
