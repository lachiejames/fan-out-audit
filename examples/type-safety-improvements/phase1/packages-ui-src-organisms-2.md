# Audit: packages/ui/src/organisms (slice 2)

**Files inspected**

- `packages/ui/src/organisms/chat/response.tsx`
- `packages/ui/src/organisms/chat/shimmer.tsx`
- `packages/ui/src/organisms/chat/tool-content.tsx`
- `packages/ui/src/organisms/chat/tool-header.tsx`
- `packages/ui/src/organisms/chat/tool.tsx`
- `packages/ui/src/organisms/inbox/AIResponseBubble.tsx`
- `packages/ui/src/organisms/inbox/CalendarStrip.tsx`

**Findings**: 2 findings

---

## Summary

`tool-header.tsx` uses `Record<ToolUIPart["state"], ...>` correctly — keyed by a union from an external library, which is good practice. `CalendarStrip.tsx` repeats the `T | null` prop pattern seen in `AiPresenceLogo` for optional props. `AIResponseBubble.tsx` and the chat files are clean.

---

## Findings

### Finding 1: `CalendarStripProps` uses `T | null` for optional props

- **File**: `packages/ui/src/organisms/inbox/CalendarStrip.tsx` (lines 14–23)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Medium — same issue as `AiPresenceLogoProps`: consumers must pass `null` rather than omitting props; inconsistent with the rest of the design system
- **Description**: `events: CalendarEvent[] | null`, `onEventClick: ((event: CalendarEvent) => void) | null`, `defaultCollapsed: boolean | null`, and `className: string | null` are all typed as `T | null` instead of optional `T?`. Consumers must explicitly pass `null` for each prop they don't need.
- **Suggestion**: Convert to optional props (`events?: CalendarEvent[]`, etc.) with default parameter values in the destructuring. This is consistent with every other organism in this package.
- **Evidence**:
  ```typescript
  export interface CalendarStripProps {
    events: CalendarEvent[] | null;
    onEventClick: ((event: CalendarEvent) => void) | null;
    defaultCollapsed: boolean | null;
    className: string | null;
  }
  ```

---

### Finding 2: `ToolHeader` — `getStatusBadge` exported without explicit return type

- **File**: `packages/ui/src/organisms/chat/tool-header.tsx` (line 17)
- **Category**: Missing explicit prop types / inferred-any leaking
- **Impact**: Low — TypeScript infers `JSX.Element` correctly, but the project rule requires explicit return types on exported functions
- **Description**: `getStatusBadge` is an exported function but has no explicit `: React.JSX.Element` return type annotation.
- **Suggestion**: Add `: React.JSX.Element` to the `getStatusBadge` function signature.
- **Evidence**: `export const getStatusBadge = ({ status }: { status: ToolUIPart["state"] }) => {`
