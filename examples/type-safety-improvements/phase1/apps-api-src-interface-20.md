# Audit: apps-api-src-interface-20

**Findings count**: 2
**Summary**: One high-impact duplicate type definition and one recurring unsafe Express header cast pattern. The `RawBodyRequest` interface extending `Request` with `rawBody?: Buffer` is defined identically in all five webhook signature guard files. Express header accesses in the same guards cast `string | string[] | undefined` to `string | undefined` without normalising the array case.

---

## Finding 1

**Files**:

- `apps/api/src/interface/webhooks/facebook-messenger-signature.guard.ts`
- `apps/api/src/interface/webhooks/github-marketplace-signature.guard.ts`
- `apps/api/src/interface/webhooks/lemon-squeezy-signature.guard.ts`
- `apps/api/src/interface/webhooks/linear-signature.guard.ts`
- `apps/api/src/interface/webhooks/slack-signature.guard.ts`
  **Lines**: Near top of each file
  **Category**: Duplicate type definition
  **Impact**: High — five identical local interface declarations. Any change to the `rawBody` field (e.g. accepting `Buffer | string`) requires updating five files. A shared definition ensures consistency and is the single source of truth.

**Description**: Every guard file contains:

```typescript
interface RawBodyRequest extends Request {
  rawBody?: Buffer;
}
```

This is a textbook case of the duplicate type anti-pattern. The interface is used only to access `req.rawBody` for HMAC verification, which is the same operation across all guards.

**Suggestion**: Extract to `apps/api/src/interface/webhooks/types/raw-body-request.ts` and import in each guard. Note: `webhooks-http.helpers.ts` uses an inline intersection `(req as Request & { rawBody?: Buffer | string })` which should also be unified with this shared type.

**Evidence** (representative, same in all 5 files):

```typescript
// In slack-signature.guard.ts
interface RawBodyRequest extends Request {
  rawBody?: Buffer;
}
// ...
const rawBodyReq = request as RawBodyRequest;
```

---

## Finding 2

**Files**: All five guard files listed above
**Lines**: Signature header extraction in each guard's `canActivate` method
**Category**: Type cast (`as`)
**Impact**: Medium — Express header values can be `string | string[] | undefined`. Casting directly to `string | undefined` silently discards the multi-value case. For timestamp and signature headers this is practically safe (single-value by spec), but the cast bypasses the type system.

**Description**: Each guard reads one or two headers (e.g. `x-slack-request-timestamp`, `x-slack-signature`) with a cast to `string | undefined`. If a proxy ever forwards duplicate headers as an array, the runtime value would be `string[]` and the subsequent HMAC comparison would fail silently.

**Suggestion**: Use a helper `getFirstHeader(headers: IncomingHttpHeaders, name: string): string | undefined` that normalises `string | string[]` to `string`, and call it consistently across all guards.

**Evidence** (representative, from slack-signature.guard.ts):

```typescript
const timestamp = request.headers["x-slack-request-timestamp"] as string | undefined;
const signature = request.headers["x-slack-signature"] as string | undefined;
```
