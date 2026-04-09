# Type-Safety Audit — Batch 93

**Files audited:**

- `apps/api/src/integrations/platforms/slack/services/slack-fetch-actions.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-fetch.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-huddle.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-message-actions.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-message-read.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-oauth.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-search.service.ts`
- `apps/api/src/integrations/platforms/slack/services/slack-sync-cache.ts`

---

## Findings

### 1. slack-fetch-actions.service.ts — missing-strict-typing

**Lines:** 42, 56
**Category:** missing-strict-typing
**Severity:** low
**Description:** `sendMessage` return type includes `message?: unknown` for the Slack API's returned message object. The `@slack/web-api` SDK exports `ChatPostMessageResponse["message"]` which is a proper type.
**Code:**

```typescript
message?: unknown;
```

**Fix:** Replace `message?: unknown` with `message?: NonNullable<ChatPostMessageResponse["message"]>` imported from `@slack/web-api`.

---

### 2. slack-fetch.service.ts — sdk-type-duplication

**Lines:** 29–30
**Category:** sdk-type-duplication
**Severity:** medium
**Description:** Local type aliases `type SlackUser = ...` and `type SlackChannel = ...` are defined inline and duplicate the shapes already derivable from `UsersInfoResponse["user"]` and `ConversationsListResponse["channels"][number]` from the `@slack/web-api` SDK. The same shapes are also defined in `slack-message.mappers.ts`.
**Code:**

```typescript
type SlackUser = { id?: string; real_name?: string; profile?: { image_48?: string } };
type SlackChannel = { id?: string; name?: string; is_member?: boolean; ... };
```

**Fix:** Import the canonical SDK-derived types from `slack-message.mappers.ts` (or define them once there) and use those instead of the local aliases.

---

### 3. slack-fetch.service.ts — type-cast

**Line:** 647
**Category:** type-cast
**Severity:** medium
**Description:** `(slack as { token?: unknown }).token` — accesses the private `token` field on the Slack `WebClient` instance to log a token preview for debugging. The SDK does not expose `token` in its public interface.
**Code:**

```typescript
const tokenPreview = (slack as { token?: unknown }).token;
```

**Fix:** Remove the token logging (security-sensitive) or use the WebClient constructor's `token` parameter captured at creation time.

---

### 4. slack-huddle.service.ts — type-cast

**Line:** 152
**Category:** type-cast
**Severity:** medium
**Description:** `if ("code" in error) return error as SlackError` in `.mapErr` — casts an error to `SlackError` based only on presence of a `code` property without verifying the full discriminant.
**Code:**

```typescript
.mapErr((error) => {
  if ("code" in error) return error as SlackError;
  return SlackErrors.unknownError(error.message);
})
```

**Fix:** Use a `isSlackError(e: unknown): e is SlackError` type guard that validates `e.type` against the `SlackError` union discriminant.

---

### 5. slack-message-actions.service.ts — type-cast

**Line:** 110
**Category:** type-cast
**Severity:** low
**Description:** `{ message: checkedResponse.message as unknown }` — intentionally widens the SDK message response type back to `unknown` for storage. Widens to unknown rather than keeping the SDK type.
**Code:**

```typescript
return ok({ message: checkedResponse.message as unknown });
```

**Fix:** Keep as `ChatPostMessageResponse["message"]` throughout; only widen to `unknown` at the storage boundary where schema validation occurs.

---

### 6. slack-message-actions.service.ts — type-cast (repeated pattern)

**Lines:** 113–114
**Category:** type-cast
**Severity:** medium
**Description:** `if ("code" in error) return error as SlackError` — same unsafe cast pattern as finding #4.
**Fix:** Same fix as finding #4 — use `isSlackError` type guard.

---

### 7. slack-message-read.service.ts — type-cast (repeated pattern)

**Lines:** 175, 276
**Category:** type-cast
**Severity:** medium
**Description:** `if ("code" in error) return error as SlackError` — same pattern repeated twice in mapErr callbacks.
**Fix:** Same fix as finding #4 — centralize in a shared `isSlackError` type guard.

---

### 8. slack-oauth.service.ts — type-cast

**Lines:** 287, 308
**Category:** type-cast
**Severity:** medium
**Description:** `integration.platformMetadata as SlackPlatformMetadata | null` — platform metadata stored as `unknown` in the DB; same pattern as Notion OAuth service.
**Code:**

```typescript
const metadata = integration.platformMetadata as SlackPlatformMetadata | null;
```

**Fix:** Add a runtime `parseSlackPlatformMetadata(raw: unknown): SlackPlatformMetadata | null` validation function.

---

### 9. slack-search.service.ts — type-cast (repeated pattern)

**Line:** 236
**Category:** type-cast
**Severity:** medium
**Description:** `if ("code" in error) return error as SlackError` — fourth occurrence of the same unsafe cast in mapErr.
**Fix:** Same fix as finding #4.

---

## No Findings

- `slack-sync-cache.ts` — Clean, properly typed cache utilities, no findings.
