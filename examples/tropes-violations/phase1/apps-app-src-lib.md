# AI Writing Tropes Audit — apps/app/src/lib

**Files audited** (4 exist, 2 not found):

| File                          | Status            |
| ----------------------------- | ----------------- |
| `ai-chat-controller.tsx`      | Exists — audited  |
| `auth-context.tsx`            | Exists — audited  |
| `command-palette-context.tsx` | Exists — audited  |
| `onboarding-context.tsx`      | Exists — audited  |
| `sidebar-context.tsx`         | Not found in repo |
| `toast-context.tsx`           | Not found in repo |

---

## ai-chat-controller.tsx

**Line 146-148:**

```tsx
toast.error("Failed to get AI response", {
  description: error.message || "Please try again",
});
```

**Trope: "Please try again" as a fallback error description.**
This is a generic AI-era filler phrase used when no real information is available. It adds no value and slightly reads like an automated system. If the error message is empty, omit the description entirely rather than printing a platitude.

Suggested fix: Remove the `|| "Please try again"` fallback; show the description only when `error.message` is non-empty.

**Line 475:**

```tsx
toast.error("Can't defer this action", { description: "Missing approval ID" });
```

No violation — this is an accurate technical error message.

**Lines 516:**

```tsx
toast.success("Added to Queue", { description: "Review and edit it before executing" });
```

No violation — specific and actionable.

**Lines 552-554:**

```tsx
toast.error("Failed to send message", {
  description: error instanceof Error ? error.message : "Please try again",
});
```

**Trope: "Please try again" as a fallback error description (repeated).**
Same pattern as line 148. The fallback "Please try again" appears in two separate catch blocks in this file. Both should drop the fallback when no real message is available.

**Lines 588-590:**

```tsx
toast.error("Failed to send message", {
  description: error instanceof Error ? error.message : "Please try again",
});
```

**Trope: "Please try again" as a fallback error description (third occurrence).**
Same issue in the pending-message send path.

---

## auth-context.tsx

**Line 271:**

```tsx
toast.error("Session expired", { description: "Please sign in again" });
```

**Trope: "Please sign in again" as a stock session-expiry message.**
This is the standard AI/SaaS boilerplate for session expiry. It is not wrong, but it is hollow. The user knows they need to sign in — the page will redirect them. Consider whether this toast is needed at all, or whether it could be more specific: "Your session expired. Signing you out."

Borderline — flagged for review, not a hard required fix.

**Lines 220-222:**

```tsx
toast.error("Logout failed", {
  description: "You have been logged out locally, but the server may still have your session",
});
```

No violation — this is an accurate, specific error description.

---

## command-palette-context.tsx

**Line 157-158:**

```tsx
const message = "Command not available";
toast.error(message, { description: "This action isn't wired yet." });
```

**Trope: "This action isn't wired yet" is internal developer language surfaced to users.**
If this toast can be triggered in production (not just dev/debug), a user seeing "This action isn't wired yet" is a rough experience. This reads like a developer placeholder that leaked into production copy. Either gate it behind a dev-only condition or replace with user-facing copy such as "This feature isn't available yet."

**Line 177:**

```tsx
toast.loading(command.toast?.loading ?? "Running command...");
```

**Trope: "Running command..." as a generic loading label.**
"Command" is technical jargon. If the caller provides `command.toast.loading`, it will be used — but the fallback is exposed to users whenever a command author omits it. Consider a better default: "Working..." or just remove the fallback loading toast entirely.

**Lines 180-181:**

```tsx
const successMessage = actionResult?.successMessage ?? command.toast?.success ?? "Command complete";
```

**Trope: "Command complete" as a generic success message.**
Same issue — "Command complete" is internal developer language that users should never see. If no success message is provided, consider suppressing the toast rather than showing a generic confirmation.

**Lines 200-201:**

```tsx
: "Please try again.";
```

**Trope: "Please try again" as a catch-all error fallback.**
Appears in two error paths inside `executeCommand` (one for async failures, one for sync throws). Same issue as the pattern flagged in `ai-chat-controller.tsx`.

---

## onboarding-context.tsx

No violations found. The file contains no user-visible copy — it is a pure state-management context with no rendered strings.

---

## Files Not Found

`sidebar-context.tsx` and `toast-context.tsx` do not exist at the listed paths. They were not audited.
