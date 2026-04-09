# Type-Safety Audit — Batch 94

**Files audited:**

- `apps/api/src/integrations/platforms/slack/services/slack-sync.fetch.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-sync.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-sync.types.ts`
- `apps/api/src/integrations/platforms/slack/services/utils/slack-error.utils.ts`
- `apps/api/src/integrations/platforms/slack/slack.plugin.ts`
- `apps/api/src/integrations/platforms/slack/utils/slack-linking.utils.ts`
- `apps/api/src/integrations/platforms/slack/utils/slack-parsing.utils.ts`
- `apps/api/src/integrations/platforms/slack/utils/slack-resume-state.utils.ts`

---

## Findings

### 1. slack-sync.fetch.ts — duplicate-type / missing-strict-typing

**Line:** 43
**Category:** duplicate-type, missing-strict-typing
**Severity:** medium
**Description:** Return type `Array<{ text: string; thread_ts?: string; ts?: string; user?: string }> | null` duplicates a shape already defined (or derivable from) `MappedSlackMessage` in the Slack mappers. The inline shape is also less precise than the existing type.
**Code:**

```typescript
async fetchSlackThread(...): Promise<Array<{ text: string; thread_ts?: string; ts?: string; user?: string }> | null>
```

**Fix:** Return `Promise<MappedSlackMessage[] | null>` using the existing mapper type.

---

### 2. slack-sync.service.ts — type-cast (base class override)

**Lines:** 495–497
**Category:** type-cast
**Severity:** medium
**Description:** `parsedItems: unknown[]` parameter in `syncAttachments` override is then cast to `MappedSlackMessage[]`. Same root cause as the Notion sync service — `BaseSyncService` uses `unknown[]` for the hook parameter.
**Code:**

```typescript
protected syncAttachments(parsedItems: unknown[]): void {
  const messages = parsedItems as MappedSlackMessage[];
```

**Fix:** Make `BaseSyncService` generic: `class BaseSyncService<TItem>` with typed `syncAttachments(parsedItems: TItem[])`.

---

### 3. slack-sync.service.ts — type-cast (SDK gap)

**Line:** 799
**Category:** type-cast
**Severity:** medium
**Description:** `(a as { latest?: { ts?: string } }).latest?.ts` — accesses `latest.ts` on a Slack channel object. The `@slack/web-api` `ConversationsListResponse["channels"][number]` type does not include `latest` even though the API may return it.
**Code:**

```typescript
const tsA = (a as { latest?: { ts?: string } }).latest?.ts ?? "";
```

**Fix:** Define `interface SlackChannelWithLatest extends SlackChannel { latest?: { ts?: string } }` and cast to that typed extension.

---

### 4. slack-sync.service.ts — missing-strict-typing (initial type annotation)

**Line:** 159
**Category:** missing-strict-typing
**Severity:** low
**Description:** `users: {} as Record<string, { real_name?: string; avatar_url?: string }>` — initial value annotation uses a narrower inline shape than the `SlackUser` type used for the actual entries.
**Code:**

```typescript
const context = {
  users: {} as Record<string, { real_name?: string; avatar_url?: string }>,
```

**Fix:** Use `{} as Record<string, SlackUser>` (or the type from `slack-sync.types.ts`) to keep the initial annotation consistent with the entry type.

---

### 5. slack.plugin.ts — type-cast

**Line:** 172
**Category:** type-cast
**Severity:** medium
**Description:** `slackIdsResult.value as Extract<ActionTargetIdentifiers, { platform: "slack" }>` — cast after platform string check, but the type system does not narrow `ActionTargetIdentifiers` from the platform check alone.
**Code:**

```typescript
const slackIdentifiers = slackIdsResult.value as Extract<ActionTargetIdentifiers, { platform: "slack" }>;
```

**Fix:** Add a type guard `isSlackTargetIdentifiers(ids: ActionTargetIdentifiers): ids is Extract<ActionTargetIdentifiers, { platform: "slack" }>` that checks `ids.platform === "slack"`.

---

### 6. slack.plugin.ts — type-cast

**Line:** 360
**Category:** type-cast
**Severity:** medium
**Description:** `(body as { syncRunId?: string }).syncRunId` — the webhook body type from the ts-rest contract doesn't include `syncRunId`, so it must be cast to access it.
**Code:**

```typescript
const syncRunId = (body as { syncRunId?: string }).syncRunId;
```

**Fix:** Add `syncRunId?: string` to the relevant webhook body contract schema in `@slopweaver/contracts`, or use a separate webhook payload type that extends the base type.

---

## No Findings

- `slack-sync.types.ts` — `users: Record<string, SlackUser>` is properly typed; no findings.
- `slack-error.utils.ts` — `const maybeError = error as { message?: string; ... }` on `unknown` error is a necessary low-impact cast for error message extraction from unexpected error shapes.
- `slack-linking.utils.ts` — Clean, no findings.
- `slack-parsing.utils.ts` — `parseSlackUser` uses a structural inline type appropriate for partial SDK user shapes; no findings.
- `slack-resume-state.utils.ts` — Clean, no findings.
