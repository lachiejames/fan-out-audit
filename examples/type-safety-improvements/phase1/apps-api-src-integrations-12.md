# Type-Safety Audit: apps-api-src-integrations-12

**Batch 73** — `apps/api/src/integrations/platforms/atlassian/`, `facebook-messenger/` (errors, plugin, mappers, client provider, fetch, oauth, sync services)

**Files inspected**: 8
**Findings**: 8

## Summary

The Facebook Messenger OAuth service has the most significant issues: `page_id`/`page_name` fields are smuggled through a `FacebookTokenResponse` type via `as FacebookTokenResponse`, and three `tokenRecord` casts (`as string`) on fields not declared in the response schema. Multiple services cast `integration.platformMetadata` to a narrow `{ pageId?: string } | null` type because the DB column is `unknown`. Dynamic `require("node:crypto")` in the webhook utility bypasses static type analysis.

## Findings

### Finding 1: platformMetadata cast in facebook-messenger.plugin.ts

- **File**: `src/integrations/platforms/facebook-messenger/facebook-messenger.plugin.ts:238`
- **Category**: type-cast
- **Impact**: low
- **Description**: `integration.platformMetadata as { pageId?: string } | null` — recurring cast because `platformMetadata` is `unknown` in the DB schema. The same pattern appears in fetch, oauth, and sync services.
- **Suggestion**: Centralise a Zod `facebookMessengerPlatformMetadataSchema` and call `.parse()` once; eliminate repeated inline casts.
- **Evidence**:

```typescript
const metadata = integration.platformMetadata as { pageId?: string } | null;
```

---

### Finding 2: page_id/page_name smuggled through as FacebookTokenResponse

- **File**: `src/integrations/platforms/facebook-messenger/services/facebook-messenger-oauth.service.ts:252`
- **Category**: type-cast
- **Impact**: high
- **Description**: `page_id` and `page_name` are added to the token object during OAuth (line 214), but `FacebookTokenResponse` does not declare these fields. They are accessed via `tokenRecord as Record<string, unknown>` followed by `as string` casts. This is a structural mismatch — fields added at runtime are not part of the type.
- **Suggestion**: Extend `FacebookTokenResponse` (or define a new `FacebookPageTokenResponse`) to include `page_id?: string` and `page_name?: string` so the fields are accessible without casts.
- **Evidence**:

```typescript
const tokenRecord = tokenData as Record<string, unknown>;
const pageId = tokenRecord["page_id"] as string;
const pageName = tokenRecord["page_name"] as string;
```

---

### Finding 3: as FacebookTokenResponse cast in facebook-messenger-oauth

- **File**: `src/integrations/platforms/facebook-messenger/services/facebook-messenger-oauth.service.ts:309`
- **Category**: type-cast
- **Impact**: low
- **Description**: `tokenData as FacebookTokenResponse` cast on a value already stored as `FacebookTokenResponse` — redundant or used because the variable type was widened to `unknown` for the `page_id` smuggling above.
- **Suggestion**: Resolved together with Finding 2 by adding the fields to the type.
- **Evidence**:

```typescript
return ok(tokenData as FacebookTokenResponse);
```

---

### Finding 4: as FacebookMessengerPlatformMetadata | null cast

- **File**: `src/integrations/platforms/facebook-messenger/services/facebook-messenger-oauth.service.ts:303`
- **Category**: type-cast
- **Impact**: low
- **Description**: `integration.platformMetadata as FacebookMessengerPlatformMetadata | null` — same `unknown` DB column cast pattern as Finding 1.
- **Suggestion**: Centralise with a Zod parser.
- **Evidence**:

```typescript
const existingMetadata = integration.platformMetadata as FacebookMessengerPlatformMetadata | null;
```

---

### Finding 5: (await response.json()) as { data: Array<Record<string, unknown>> }

- **File**: `src/integrations/platforms/facebook-messenger/services/facebook-messenger-oauth.service.ts:382`
- **Category**: type-cast
- **Impact**: low
- **Description**: JSON response for page list deserialized as `{ data: Array<Record<string, unknown>> }`. The `Record<string, unknown>` value type is overly wide for what is a well-documented Facebook page object.
- **Suggestion**: Define a `FacebookPage` interface with known fields and validate with a Zod schema.
- **Evidence**:

```typescript
const pagesData = (await response.json()) as { data: Array<Record<string, unknown>> };
```

---

### Finding 6: Dynamic require("node:crypto") in webhook utils

- **File**: `src/integrations/platforms/facebook-messenger/utils/facebook-messenger-webhook.utils.ts:79`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: `require("node:crypto").timingSafeEqual(...)` uses a dynamic `require` instead of a static import, bypassing TypeScript type checking for the `timingSafeEqual` call.
- **Suggestion**: Replace with `import { timingSafeEqual } from "node:crypto"` at the top of the file.
- **Evidence**:

```typescript
const equal = require("node:crypto").timingSafeEqual(hmac, sig) as boolean;
```

---

### Finding 7: facebook-messenger client provider JSON cast

- **File**: `src/integrations/platforms/facebook-messenger/providers/facebook-messenger-client.provider.ts:167`
- **Category**: type-cast
- **Impact**: low
- **Description**: `return ok((await response.json()) as T)` — unavoidable cast at the generic HTTP client JSON deserialization boundary. Acceptable.
- **Suggestion**: No change required; consider adding a Zod schema parameter long-term.

---

### Finding 8: atlassian-api.utils.ts, facebook-messenger mappers

- **File**: `src/integrations/platforms/atlassian/atlassian-api.utils.ts`, `src/integrations/platforms/facebook-messenger/mappers/facebook-messenger-message.mappers.ts`
- **Category**: N/A
- **Impact**: none
- **Description**: Both files are clean — well-typed pure utility functions with no unsafe casts.
- **Suggestion**: No changes needed.
