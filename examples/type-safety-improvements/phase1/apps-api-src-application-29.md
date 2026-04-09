# Audit: apps-api-src-application-29

**Files inspected**: 8
**Findings**: 4

## Summary

These files implement the push notification subsystem: domain errors, APNs/FCM provider services (using raw HTTP with JWT/OAuth tokens rather than vendor SDKs), a desktop SSE service, a notification router/orchestrator, a history query service, shared provider types, and quiet-hours suppression logic. The main type-safety issues are: a `PushNotificationHistoryItem` interface that duplicates DB-derived string literals instead of reusing the Drizzle-exported enum types; an unsafe `as Record<string, unknown>` cast in FCM config parsing that bypasses proper narrowing; a loose `{ type: string; data?: unknown }` SSE event type that allows any event shape; and a `success: true as const` / `false as const` pattern that could be replaced by a discriminated union already defined in `push-provider.types.ts`.

## Findings

### Finding 1: `PushNotificationHistoryItem` duplicates enum string literals instead of reusing Drizzle types

- **File**: `apps/api/src/application/push-notifications/services/push-notification-history.service.ts:20-29`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `PushNotificationHistoryItem` hard-codes `channel: "desktop" | "mobile" | "in_app"`, `urgency: "low" | "medium" | "high" | "critical"`, and `deliveryStatus: "pending" | "sent" | "delivered" | "failed"` as inline string unions. The exact same values are already exported from `enums.ts` as `PushChannel`, `PushUrgencyTier`, and `PushDeliveryStatus` (derived from the Drizzle `pgEnum` definitions). If an enum value is ever added or renamed, the interface silently drifts.
- **Suggestion**: Replace the three inline unions with the existing Drizzle-derived types:
  ```typescript
  import { PushChannel, PushDeliveryStatus, PushUrgencyTier } from "@/shared/db/tables/enums";
  // then in the interface:
  channel: PushChannel;
  urgency: PushUrgencyTier;
  deliveryStatus: PushDeliveryStatus;
  ```
- **Evidence**:

```typescript
export interface PushNotificationHistoryItem {
  id: string;
  title: string;
  body: string;
  channel: "desktop" | "mobile" | "in_app"; // duplicates PushChannel
  urgency: "low" | "medium" | "high" | "critical"; // duplicates PushUrgencyTier
  contentId: string | null;
  sentAt: string;
  deliveredAt: string | null;
  clickedAt: string | null;
  dismissedAt: string | null;
  deliveryStatus: "pending" | "sent" | "delivered" | "failed"; // duplicates PushDeliveryStatus
}
```

### Finding 2: Unsafe `as Record<string, unknown>` cast after narrowing in FCM config parsing

- **File**: `apps/api/src/application/push-notifications/services/fcm-push.service.ts:219`
- **Category**: type-cast
- **Impact**: medium
- **Description**: After correctly narrowing `parsed` to `object` (line 215), the code immediately widens it back with `as Record<string, unknown>`. This cast bypasses TypeScript's structural checking—if field names change or the JSON shape is unexpected, the cast silently accepts it. The narrowed `object` type already permits `in` checks and `typeof` property guards, so the cast is unnecessary.
- **Suggestion**: Remove the cast and access properties directly with `typeof` guards on the narrowed `parsed` value, or use a minimal Zod schema (`z.object({ project_id: z.string(), client_email: z.string(), private_key: z.string() })`) to parse the service account payload in one step. At minimum, replace the cast with `const value = parsed as { [key: string]: unknown }` and add an `as const` satisfaction check, but the Zod approach is cleanest.
- **Evidence**:

```typescript
if (!parsed || typeof parsed !== "object") {
  return err(PushNotificationErrors.providerAuthFailed("fcm", "Invalid service account JSON payload"));
}

const value = parsed as Record<string, unknown>; // <-- unsafe widening cast
const projectId = typeof value["project_id"] === "string" ? value["project_id"].trim() : "";
const clientEmail = typeof value["client_email"] === "string" ? value["client_email"].trim() : "";
const privateKeyRaw = typeof value["private_key"] === "string" ? value["private_key"].trim() : "";
```

### Finding 3: SSE event parameter typed as `{ type: string; data?: unknown }` allows unconstrained event shapes

- **File**: `apps/api/src/application/push-notifications/services/desktop-notification.service.ts:134`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The private `sendEvent` helper accepts `event: { type: string; data?: unknown }`. The `type` field admits any string, but the service only ever emits `"connected"` and `"notification"` events. The `data` field is typed as `unknown`, which means any value could be serialized without type verification. This lets future callers pass an unrecognized event type or a mismatched payload silently.
- **Suggestion**: Define a discriminated union for the two known event shapes and narrow `sendEvent`'s parameter to it:

  ```typescript
  type SseEvent =
    | { type: "connected" }
    | { type: "notification"; data: DesktopNotificationPayload };

  private sendEvent({ res, event }: { res: Response; event: SseEvent }): void { ... }
  ```

  This makes the `data` field required for `"notification"` events and disallows unknown event types.

- **Evidence**:

```typescript
private sendEvent({ res, event }: { res: Response; event: { type: string; data?: unknown } }): void {
  const eventData = event.data ? JSON.stringify(event.data) : "";
  res.write(`event: ${event.type}\n`);
  if (eventData) {
    res.write(`data: ${eventData}\n`);
  }
  res.write("\n");
}
```

### Finding 4: APNs response body parsed with an unsafe `as` cast instead of safe narrowing

- **File**: `apps/api/src/application/push-notifications/services/apns-push.service.ts:188`
- **Category**: type-cast
- **Impact**: low
- **Description**: The APNs error body is parsed with `(await response.json().catch(() => null)) as { reason?: string } | null`. The `as` cast asserts the shape without verifying it—if the APNs API changes its error schema, `body?.reason` will silently be `undefined` rather than surfacing an unexpected shape. The same pattern also appears in `fcm-push.service.ts:96` (`as FcmV1ResponseBody | null`), though `FcmV1ResponseBody` at least documents the expected shape as an interface.
- **Suggestion**: Replace the `as` cast with a minimal type guard or inline check:
  ```typescript
  const raw: unknown = await response.json().catch(() => null);
  const body = raw != null && typeof raw === "object" && "reason" in raw ? (raw as { reason?: string }) : null;
  ```
  Or introduce a small `isApnsErrorBody` predicate that validates the shape before use. The FCM variant is lower-risk because `FcmV1ResponseBody` is declared as a local interface, but it would benefit from the same treatment.
- **Evidence**:

```typescript
const body = (await response.json().catch(() => null)) as { reason?: string } | null;
const reason = body?.reason;
```
