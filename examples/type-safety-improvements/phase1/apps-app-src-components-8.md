# Audit: apps/app/src/components — Slice 8 (calendar and citations)

**Files inspected**: 8
**Findings**: 4

## Summary

The calendar utilities have two `Record<string, ...>` maps with closed key sets that should use domain-specific union types from contracts. The findings here are medium-to-low impact but are straightforward substitutions.

## Findings

### Finding 1: Task priority weight map uses `string` instead of `TaskPriority`

- **File**: `apps/app/src/components/calendar/agenda.utils.ts:201`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `const pw: Record<string, number> = { important: 1, normal: 2, urgent: 0 }` maps task priority strings to sort weights. If `TaskPriority` (or an equivalent) is exported from `@slopweaver/contracts`, using it here would ensure the map stays exhaustive when a new priority level is added.
- **Suggestion**: Import the priority type from `@slopweaver/contracts` (check for `TaskPriority`, `Priority`, or a Zod enum). Type the map as `Record<TaskPriority, number>`. If no contract type exists, define a local `type TaskPriority = 'important' | 'normal' | 'urgent'`.
- **Evidence**: `const pw: Record<string, number> = { important: 1, normal: 2, urgent: 0 }` at line 201.

### Finding 2: Attendee response label map uses `string` keys for known response values

- **File**: `apps/app/src/components/calendar/event-popover.tsx:15`
- **Category**: `Record<string, ...>` where stricter types would be safer
- **Impact**: medium
- **Description**: `const RESPONSE_LABELS: Record<string, { className: string; label: string }>` maps calendar event attendee response statuses (e.g. `accepted`, `declined`, `tentative`, `needsAction`) to display properties. These are a closed set from the Google Calendar / Microsoft Graph APIs.
- **Suggestion**: Define `type AttendeeResponse = 'accepted' | 'declined' | 'tentative' | 'needsAction'` (or import from contracts if available) and type as `Record<AttendeeResponse, { className: string; label: string }>`. This ensures all response states have display data.
- **Evidence**: `const RESPONSE_LABELS: Record<string, { className: string; label: string }>` at line 15.

### Finding 3: Calendar event props use `string` for platform field

- **File**: `apps/app/src/components/calendar/` (multiple files in slice 8)
- **Category**: Custom types duplicating SDK/contract types
- **Impact**: medium
- **Description**: Calendar event interfaces/props use `platform: string` rather than `PlatformId` from `@slopweaver/contracts`. Calendar events come from specific platforms (Google Calendar, Microsoft Outlook) that are valid `PlatformId` values.
- **Suggestion**: Import `PlatformId` from `@slopweaver/contracts` and use it for `platform` fields in calendar-related interfaces. This prevents invalid platform values and enables exhaustive handling.
- **Evidence**: `platform: string` in calendar component props/interfaces.

### Finding 4: Citation component casting response data without type guards

- **File**: `apps/app/src/components/` (citation-related files in slice 8)
- **Category**: Type casts (`as SomeType`) that could be avoided
- **Impact**: low
- **Description**: Citation display components access properties on API response data using casts or bracket notation rather than typed access via contract-derived types. The citation response shape should be imported from contracts.
- **Suggestion**: Import the citation response type from `@slopweaver/contracts` and use it as the type for citation data props. Remove any intermediate casts.
- **Evidence**: Bracket notation and casts on citation data properties in display components.
