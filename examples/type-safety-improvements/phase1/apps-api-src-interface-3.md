# Type-Safety Audit — Batch 98

**Files audited:**

- `apps/api/src/interface/agents/tools/google-gmail/search-google-gmail.tool.ts`
- `apps/api/src/interface/agents/tools/google-gmail/send-google-gmail-reply.review-edit.ts`
- `apps/api/src/interface/agents/tools/jira/create-jira-issue.tool.ts`
- `apps/api/src/interface/agents/tools/jira/update-jira-issue.tool.ts`
- `apps/api/src/interface/agents/tools/microsoft-outlook/send-microsoft-outlook-reply.review-edit.ts`
- `apps/api/src/interface/agents/tools/microsoft-teams/send-teams-message.review-edit.ts`
- `apps/api/src/interface/agents/tools/notion/get-notion-page.tool.ts`
- `apps/api/src/interface/agents/tools/notion/update-notion-page.tool.ts`

---

## Findings

### 1. send-google-gmail-reply.review-edit.ts — record-weakening (interface boundary)

**Line:** 30
**Category:** record-weakening
**Severity:** medium
**Description:** `mapSendGmailReplyToolToWorkItem: ToolReviewEditMapper = (toolInput)` — accepts `toolInput: Record<string, unknown>` because `ToolReviewEditMapper` is typed with that loose input (tracked in batch 97 finding #4). Fields are extracted with runtime `typeof` checks inside the function. The specific Gmail reply tool input type is available from the Zod schema but is not surfaced here.
**Code:**

```typescript
export const mapSendGmailReplyToolToWorkItem: ToolReviewEditMapper = (toolInput) => {
  const threadId = requireString({ input: toolInput, key: "threadId", toolName });
```

**Fix:** Once `ToolReviewEditMapper` is made generic (batch 97 finding #4 fix), narrow to the specific input type: `ToolReviewEditMapper<z.infer<typeof sendGmailReplySchema>>`.

---

### 2. create-jira-issue.tool.ts — missing-strict-typing (metadata typing)

**Line:** 144–149
**Category:** missing-strict-typing
**Severity:** low
**Description:** `metadata: { description, title: JSON.stringify(jiraMetadata) }` — stores Jira-specific metadata as a JSON string in `title`. This is a known workaround for the work item schema, but the `metadata` object type is `Record<string, unknown>` in the work item contract, so the implicit stringification is untracked by the type system.
**Code:**

```typescript
metadata: {
  description,
  title: JSON.stringify(jiraMetadata),
},
```

**Fix:** Define a `JiraCreateWorkItemMetadata` interface with `description?: string` and `title: string` (JSON), and assert to it explicitly: `metadata: { description, title: JSON.stringify(jiraMetadata) } satisfies JiraCreateWorkItemMetadata`.

---

### 3. get-notion-page.tool.ts — type-cast

**Lines:** 58–68
**Category:** type-cast
**Severity:** medium
**Description:** Multiple inline casts on Notion page properties to extract title text: `(titleProp as { title?: unknown }).title` and `(entry as { plain_text?: unknown }).plain_text`. The Notion page properties type is a large union and doesn't narrow automatically.
**Code:**

```typescript
const maybeTitle = (titleProp as { title?: unknown }).title;
// ...
const plainText = (entry as { plain_text?: unknown }).plain_text;
```

**Fix:** Add type guards `hasTitleArray(p: unknown): p is { title: { plain_text?: string }[] }` and use them to narrow instead of casting. The Notion SDK exports `RichTextItemResponse` which can be used for the `plain_text` field.

---

### 4. get-notion-page.tool.ts — type-cast

**Line:** 85
**Category:** type-cast
**Severity:** low
**Description:** `(block as { hasChildren?: boolean }).hasChildren` — accesses `hasChildren` on a `BlockObjectResponse` which may not be in the SDK's base union type but is returned by the API.
**Code:**

```typescript
hasChildren: (block as { hasChildren?: boolean }).hasChildren,
```

**Fix:** Extend the local block type or use `"hasChildren" in block ? block.hasChildren : undefined` for safer access.

---

### 5. update-notion-page.tool.ts — record-weakening

**Line:** 33
**Category:** record-weakening
**Severity:** medium
**Description:** `properties: z.record(z.string(), z.unknown())` in the `updateNotionPageSchema` — Notion page properties are heterogeneous by type, but the Notion SDK exports `UpdatePageParameters["properties"]` which is the precise accepted type.
**Code:**

```typescript
properties: z.record(z.string(), z.unknown()).optional();
```

**Fix:** While keeping the Zod schema for runtime parsing, add a refinement or use `z.record(z.string(), z.union([...known property types...]))` to constrain the values more tightly.

---

### 6. update-notion-page.tool.ts — type-cast (formatPropertyValue)

**Lines:** 226–250
**Category:** type-cast
**Severity:** low
**Description:** Multiple `as Record<string, unknown>` casts in `formatPropertyValue` to access property-type-specific shapes (title array, rich_text array, select object, etc.) when the input is `value: unknown`.
**Code:**

```typescript
const obj = value as Record<string, unknown>;
if ("title" in obj && Array.isArray(obj["title"])) {
  const titleArray = obj["title"] as { text?: { content?: string } }[];
```

**Fix:** Use runtime `typeof` / `Array.isArray` / `"prop" in obj` checks with intermediate type predicates to avoid the nested casts. The `formatPropertyValue` function is already doing structural checks — completing the narrowing with type guards eliminates the casts.

---

## No Findings

- `search-google-gmail.tool.ts` — Clean hybrid search tool using the shared builder; all types flow from the builder generics; no findings.
- `send-microsoft-outlook-reply.review-edit.ts` — Same pattern as Gmail reply mapper; tracked by finding #1 (ToolReviewEditMapper); no additional findings.
- `send-teams-message.review-edit.ts` — Same pattern; tracked by finding #1; no additional findings.
- `update-jira-issue.tool.ts` — Same metadata stringification pattern as finding #2; tracking there covers it.
