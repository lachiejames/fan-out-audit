# Audit: apps-api-src-interface-19

**Findings count**: 2
**Summary**: Two minor issues in webhook controllers. A redundant post-Zod cast on an already-typed payload in the Facebook Messenger controller, and unsafe Express header casts in the GitHub Marketplace controller (headers can be `string[]`, not just `string`).

---

## Finding 1

**File**: `apps/api/src/interface/http/webhooks/facebook-messenger-webhook.controller.ts`
**Line**: 101
**Category**: Type cast (`as`)
**Impact**: Low — the cast is redundant. After Zod validation the variable is already typed as `FacebookMessengerWebhookPayload`, which declares `entry` as `Array<FacebookWebhookEntry>`. The `as FacebookWebhookEntry[]` cast adds noise without adding safety.

**Description**: `payload.entry as FacebookWebhookEntry[]` is written immediately after `const payload = facebookMessengerWebhookPayloadSchema.parse(body)`. The Zod parse already types `payload.entry` correctly; the cast is unnecessary.

**Suggestion**: Remove the cast and use `payload.entry` directly.

**Evidence**:

```typescript
const entries = payload.entry as FacebookWebhookEntry[];
```

---

## Finding 2

**File**: `apps/api/src/interface/webhooks/github-marketplace-webhook.controller.ts`
**Lines**: 43–44
**Category**: Type cast (`as`)
**Impact**: Medium — Express headers are typed `string | string[] | undefined`. The casts `as string | undefined` silently drop the `string[]` case. If GitHub ever sends a multi-value header the value would be silently truncated to the first string in an array or misread.

**Description**: Two header accesses cast array-capable header values to `string | undefined`:

```typescript
req.headers["x-github-delivery"] as string | undefined;
req.headers["x-github-event"] as string | undefined;
```

In practice these headers are single-valued, but the cast bypasses the type system's `string[]` possibility.

**Suggestion**: Normalise the header value explicitly: `const val = Array.isArray(h) ? h[0] : h` before use, then no cast is needed.

**Evidence**:

```typescript
const deliveryId = req.headers["x-github-delivery"] as string | undefined;
const eventType = req.headers["x-github-event"] as string | undefined;
```
