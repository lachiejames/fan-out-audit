# Type Safety Audit — apps/api/src/integrations (Batch 24)

**Files inspected**: 8  
**Findings**: 6

## Summary

Batch 24 covers the LinkedIn write service and the Microsoft Calendar integration. The most impactful finding is in `microsoft-calendar.service.ts` where `mapGraphEventToCalendarEvent` uses `Record<string, any>` as both the parameter type and for nested property access — `any` in production code bypasses all type checking. The sync service uses multiple casts from Graph API responses and a complex inline cursor cast. The LinkedIn write service has a minor safe post-narrowing cast.

## Findings

---

### Finding 1

**File**: `apps/api/src/integrations/platforms/microsoft-calendar/services/microsoft-calendar.service.ts:75`  
**Category**: `any` in production code  
**Impact**: High

**Description**: `mapGraphEventToCalendarEvent` declares its `event` parameter as `Record<string, any>`. Using `any` disables all type checking within the function body — there is no protection against misspelled property names, incorrect nested access, or unexpected value types. This affects attendees, organizer, location, and online meeting fields.

**Suggestion**: Replace `Record<string, any>` with `Record<string, unknown>` for the parameter type. Then use type-checked narrowing (e.g., `typeof event["id"] === "string"`) for each property access, which is already partially done in the function body. Alternatively, define a typed `GraphCalendarEvent` interface matching the expected Microsoft Graph response shape.

**Evidence**:

```typescript
// microsoft-calendar.service.ts:75
function mapGraphEventToCalendarEvent({ event }: { event: Record<string, any> }): MicrosoftCalendarEvent | null {

// microsoft-calendar.service.ts:106
const organizer = event["organizer"] as Record<string, any> | undefined;

// microsoft-calendar.service.ts:122
const location = event["location"] as Record<string, any> | undefined;

// microsoft-calendar.service.ts:123
const onlineMeeting = event["onlineMeeting"] as Record<string, any> | undefined;
```

---

### Finding 2

**File**: `apps/api/src/integrations/platforms/microsoft-calendar/services/microsoft-calendar-sync.service.ts:388`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `(response.value ?? []) as MicrosoftCalendarEvent[]` casts the Microsoft Graph API response array without validating that each element matches `MicrosoftCalendarEvent`. The `response.value` type from `@microsoft/microsoft-graph-client` is `unknown[]` or `any[]`, making the cast unchecked.

**Suggestion**: Validate each event with a Zod schema or the `mapGraphEventToCalendarEvent` mapper (which already handles malformed events by returning `null`) before collecting them into the typed array.

**Evidence**:

```typescript
// microsoft-calendar-sync.service.ts:388
const events = (response.value ?? []) as MicrosoftCalendarEvent[];
```

---

### Finding 3

**File**: `apps/api/src/integrations/platforms/microsoft-calendar/services/microsoft-calendar-sync.service.ts:459`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `ctx.allEvents` is built from a `.filter(Boolean) as MicrosoftCalendarEvent[]` pattern on `ctx.eventIds.map(id => ctx.eventCache.get(id))`. A type predicate filter is safer.

**Suggestion**: Use `.filter((x): x is MicrosoftCalendarEvent => x !== undefined)` instead of the Boolean cast.

**Evidence**:

```typescript
// microsoft-calendar-sync.service.ts:459
ctx.allEvents = ctx.eventIds.map((id) => ctx.eventCache.get(id)).filter(Boolean) as MicrosoftCalendarEvent[];
```

---

### Finding 4

**File**: `apps/api/src/integrations/platforms/microsoft-calendar/services/microsoft-calendar-sync.service.ts:351`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: The cursor is cast inline to a complex shape `{ calendarIndex?: number; nextLink?: string; pastDays?: number; futureDays?: number }` from `unknown`. This inline cast is applied every time the cursor is read, meaning there is no single point of validation.

**Suggestion**: Define a `MicrosoftCalendarCursor` type or interface and a corresponding Zod schema. Parse the cursor once at the start of the fetch function and use the typed result throughout.

**Evidence**:

```typescript
// microsoft-calendar-sync.service.ts:351
const state = (cursor as {
  calendarIndex?: number;
  nextLink?: string;
  pastDays?: number;
  futureDays?: number;
}) ?? { ... };
```

---

### Finding 5

**File**: `apps/api/src/integrations/platforms/microsoft-calendar/microsoft-calendar.plugin.ts:306`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `parseCursorState({ value: params.resumeState }) as MicrosoftCalendarCursorState` casts without validation. The same pattern exists in `jira.plugin.ts` and `linear.plugin.ts`.

**Suggestion**: Add a Zod schema for `MicrosoftCalendarCursorState` and validate before use.

**Evidence**:

```typescript
// microsoft-calendar.plugin.ts:306
syncParams.resumeState = parseCursorState({ value: params.resumeState }) as MicrosoftCalendarCursorState;
```

---

### Finding 6

**File**: `apps/api/src/integrations/platforms/linkedin/services/linkedin-write.service.ts:22`  
**Category**: Type cast (`as T`)  
**Impact**: Low

**Description**: `extractRestliId` checks `"restliId" in value` then casts to `{ restliId: string }`. This is a safe post-narrowing cast but could be replaced with a proper type guard function that TypeScript can flow-type through.

**Suggestion**: Extract a type guard `function hasRestliId(value: unknown): value is { restliId: string }` so TypeScript narrows automatically without a cast.

**Evidence**:

```typescript
// linkedin-write.service.ts:22 (approximate)
if ("restliId" in value) {
  return (value as { restliId: string }).restliId;
}
```
