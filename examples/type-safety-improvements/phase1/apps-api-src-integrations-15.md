# Type-Safety Audit: apps-api-src-integrations-15

**Batch 76** — `apps/api/src/integrations/platforms/google-calendar/` (oauth, search, sync, service, criteria utils), `google-docs/` (errors, plugin, fetch services)

**Files inspected**: 8
**Findings**: 5

## Summary

`google-calendar-oauth.service.ts` has the recurring `platformMetadata as GoogleCalendarPlatformMetadata | null` cast. `google-calendar-sync.service.ts` has a `filter(Boolean) as CachedGoogleCalendarEvent[]` cast. `google-calendar.service.ts` uses `Record<string, unknown>` for the event update request body. `google-docs-fetch.service.ts` handles the export response type safely via a runtime `typeof` check. `google-docs` errors and plugin are clean.

## Findings

### Finding 1: platformMetadata as GoogleCalendarPlatformMetadata | null in oauth

- **File**: `src/integrations/platforms/google-calendar/services/google-calendar-oauth.service.ts:186`
- **Category**: type-cast
- **Impact**: low
- **Description**: `integration.platformMetadata as GoogleCalendarPlatformMetadata | null` — recurring pattern across all OAuth services because `platformMetadata` column is typed `unknown`.
- **Suggestion**: Centralise a Zod schema for Google Calendar platform metadata and parse at the OAuth boundary.
- **Evidence**:

```typescript
const metadata = integration.platformMetadata as GoogleCalendarPlatformMetadata | null;
```

---

### Finding 2: filter(Boolean) as CachedGoogleCalendarEvent[] in sync

- **File**: `src/integrations/platforms/google-calendar/services/google-calendar-sync.service.ts:476`
- **Category**: type-cast
- **Impact**: low
- **Description**: `ctx.eventIds.map(id => ctx.eventCache.get(id)).filter(Boolean) as CachedGoogleCalendarEvent[]` — the `filter(Boolean)` narrows away `undefined` but TypeScript needs a cast.
- **Suggestion**: Use `.filter((e): e is CachedGoogleCalendarEvent => e != null)` type predicate.
- **Evidence**:

```typescript
ctx.allEvents = ctx.eventIds.map((id) => ctx.eventCache.get(id)).filter(Boolean) as CachedGoogleCalendarEvent[];
```

---

### Finding 3: requestBody: Record<string, unknown> for event update

- **File**: `src/integrations/platforms/google-calendar/services/google-calendar.service.ts:557`
- **Category**: record-weakening
- **Impact**: medium
- **Description**: `const requestBody: Record<string, unknown> = {}` used to build a calendar event update body. The `calendar_v3.Schema$Event` type from the SDK already defines the correct shape.
- **Suggestion**: Type as `Partial<calendar_v3.Schema$Event>` to get SDK-type safety for all field assignments.
- **Evidence**:

```typescript
const requestBody: Record<string, unknown> = {};
if (params.title !== undefined) requestBody["summary"] = params.title;
if (params.description !== undefined) requestBody["description"] = params.description;
```

---

### Finding 4: google-calendar-criteria.utils.ts — local Zod-inferred type alias

- **File**: `src/integrations/platforms/google-calendar/utils/google-calendar-criteria.utils.ts:302`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: `type GoogleCalendarEventMetadata = z.infer<typeof googleCalendarEventMetadataSchema>` — a local type alias derived from a Zod schema. This is an acceptable pattern (type follows schema). No duplication of SDK types; `calendar_v3.Schema$Event` is used correctly throughout.
- **Suggestion**: No changes needed — this follows the established Zod schema → inferred type pattern.

---

### Finding 5: google-calendar-search, google-docs errors, plugin, fetch

- **File**: Multiple
- **Category**: N/A
- **Impact**: none
- **Description**: `google-calendar-search.service.ts` is clean. `google-docs/errors/` uses standard factory pattern. `google-docs.plugin.ts` is clean. `google-docs-fetch.service.ts` handles the export response via `typeof exportResponse.data === "string" ? ... : JSON.stringify(...)` which is proper runtime narrowing, not a cast.
- **Suggestion**: No changes needed.
