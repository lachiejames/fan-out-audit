# Audit: apps-api-src-interface-23

**Findings count**: 2
**Summary**: Two issues in webhook utility files. A narrowed `object` body is cast to a structural inline type in `webhooks-http.helpers.ts` instead of using a type guard or Zod parse. In `webhook-job.utils.ts`, the local `WebhookJob` union includes `| Record<string, unknown>` as a catch-all that widens the union, and the local job data interfaces duplicate those already defined in the processor file.

---

## Finding 1

**File**: `apps/api/src/interface/webhooks/utils/webhooks-http.helpers.ts`
**Line**: 64
**Category**: Type cast (`as`)
**Impact**: Low — after the guard `body && typeof body === "object" && "signedPayload" in body`, the code casts `body as { signedPayload?: unknown }` to access the field. The `in` check already confirms the property exists; the cast is technically redundant but harmless. The `unknown` type on `signedPayload` is then guarded correctly with a `typeof payload === "string"` check.

**Description**: Inline structural cast on a pre-narrowed object. The pattern is safe here because the `typeof === "string"` check follows immediately, but the cast is not the idiomatic way to read a property already narrowed by `in`.

**Suggestion**: Declare a minimal Zod schema or use `Object.hasOwn` + direct property access without the cast:

```typescript
const signedPayload = "signedPayload" in body ? (body as { signedPayload?: unknown }).signedPayload : undefined;
```

…or simply use the existing `signedPayloadSchema.safeParse(body)` path that already handles this correctly lower in the same function.

**Evidence**:

```typescript
if (body && typeof body === "object" && "signedPayload" in body) {
  const payload = (body as { signedPayload?: unknown }).signedPayload;
```

---

## Finding 2

**File**: `apps/api/src/interface/webhooks/utils/webhook-job.utils.ts`
**Lines**: 13–40
**Category**: Duplicate type definition
**Impact**: Medium — the file declares `SlackWebhookJob`, `GmailWebhookJob`, `LinearWebhookJob`, and `LemonSqueezyWebhookJob` as local private interfaces with a comment saying they are "copied from processor for type safety". These are duplicates of the definitions in `webhook.processor.ts`. Additionally, the `WebhookJob` union here includes `| Record<string, unknown>` as a fallback, which widens the union and effectively makes it `Record<string, unknown>` for callers because that is the widest member.

**Description**: Job data type definitions duplicated from the processor instead of being shared from a single canonical location. The `| Record<string, unknown>` escape hatch in the local `WebhookJob` union defeats the discriminated union.

**Suggestion**: Extract all job data interfaces to `apps/api/src/interface/webhooks/types/webhook-job-data.ts`, export them, and import from both the processor and this utils file. Remove the `| Record<string, unknown>` from the union; use `| { [key: string]: unknown }` with a dedicated `UnknownWebhookJob` named type if a catch-all is genuinely needed.

**Evidence**:

```typescript
// "copied from processor for type safety"
interface SlackWebhookJob { ... }
// ...
type WebhookJob =
  | SlackWebhookJob
  | GmailWebhookJob
  | LinearWebhookJob
  | LemonSqueezyWebhookJob
  | Record<string, unknown>; // widens entire union
```
