# Tropes Audit: apps/app/src/components/integrations (Slice 85)

Files audited:

- IntegrationHub.tsx
- connection-wizard/ConnectionWizard.tsx
- connection-wizard/ErrorStep.tsx
- connection-wizard/LearnStep.tsx
- connection-wizard/PreConnectStep.tsx
- connection-wizard/StepProgress.tsx
- connection-wizard/SyncStep.tsx
- connection-wizard/connection-wizard-steps.tsx

---

## Violations

### 1. "Beautiful wizard" in a code comment

**File**: `connection-wizard/ConnectionWizard.tsx`
**Line**: 4
**Offending text**:

```
 * Beautiful wizard for connecting integrations with a unified start size + access flow.
```

**Trope**: "Beautiful" as a filler adjective. Code comments are not marketing copy; calling a wizard "beautiful" adds no information and reads as AI-generated padding.
**Fix**: Delete the word "Beautiful" or replace with a factual description of what the wizard does.

---

### 2. "Preparing your inbox" — unnecessary warmth framing

**File**: `connection-wizard/SyncStep.tsx`
**Line**: 148
**Offending text**:

```typescript
: "Preparing your inbox";
```

**Trope**: Vague anthropomorphic progress label. The sync step is importing data from a platform; calling it "Preparing your inbox" hides what is actually happening and uses a cozy AI-product cliche.
**Fix**: Replace with a factual label such as "Importing data" or "Syncing [platform name]".

---

### 3. "Usually a short wait"

**File**: `connection-wizard/SyncStep.tsx`
**Line**: 126
**Offending text**:

```typescript
const timingNote = !effectiveIsComplete && !hasError && !isDeferredSync ? "Usually a short wait" : null;
```

**Trope**: Reassurance filler with no information value. This is the classic AI-product soft-pedaling of wait times. Users can see the progress bar; they do not need to be told it is usually short.
**Fix**: Remove the timing note entirely, or replace with something factual if timing data is available (e.g., an ETA from the backend).

---

### 4. "Learning signals" section label

**File**: `connection-wizard/SyncStep.tsx`
**Line**: 505
**Offending text**:

```tsx
<span className="font-medium text-foreground">Learning signals</span>
```

**Trope**: "Learning signals" is jargon that sounds meaningful but is undefined to the user. It is the AI-startup equivalent of "AI-powered insights." The underlying data (from `learningEvents`) could be described concretely.
**Fix**: Use a label that describes what is shown, e.g., "Extracted patterns" or "What we found" — or remove the section entirely if it does not add user value.

---

### 5. "You can start working right away."

**File**: `connection-wizard/LearnStep.tsx`
**Line**: 67
**Offending text**:

```tsx
<p className="mt-1 text-sm text-muted-foreground">
  Your inbox is ready. {hasError ? "Some items are still syncing." : "You can start working right away."}
</p>
```

**Trope**: "You can start working right away" is a motivational filler sentence common in AI product onboarding. It does not tell the user anything they could not infer from "Setup complete." The phrase pattern is a hallmark of AI-generated copy.
**Fix**: Either drop the second sentence entirely (the completion state is already communicated by the heading and green checkmark) or replace with a concrete next-step prompt.

---

### 6. "Setup complete" heading with redundant reassurance

**File**: `connection-wizard/LearnStep.tsx`
**Lines**: 65-68
**Offending text**:

```tsx
<h2 className="text-xl font-semibold text-foreground">Setup complete</h2>
<p className="mt-1 text-sm text-muted-foreground">
  Your inbox is ready. {hasError ? "Some items are still syncing." : "You can start working right away."}
</p>
```

**Trope**: "Your inbox is ready" immediately following "Setup complete" is redundant. Two consecutive statements of the same completion fact, softened with "right away," is a pattern of AI-generated reassurance padding.
**Fix**: Keep the heading. If a subtitle is needed, make it factual and non-redundant, e.g., display the item count already shown in the summary cards.

---

### 7. "Ensures beautiful UX for small syncs" in a code comment

**File**: `connection-wizard/ConnectionWizard.tsx`
**Line**: 452
**Offending text**:

```typescript
// 2. Minimum display time has elapsed (ensures beautiful UX for small syncs)
```

**Trope**: "Beautiful UX" in a code comment is the same problem as violation 1. Comments should explain why, not praise the result. "Ensures beautiful UX" says nothing a future reader can act on.
**Fix**: Replace with the actual reason: e.g., "prevents the sync step from flashing away immediately for fast syncs, which is disorienting."

---

### 8. "No worries. You can try again whenever you're ready."

**File**: `connection-wizard/ErrorStep.tsx`
**Lines**: 35-36
**Offending text**:

```typescript
access_denied: {
  message: "No worries. You can try again whenever you're ready.",
```

**Trope**: "No worries" is a classic AI-product empathy phrase that reads as insincere and condescending. It is the textual equivalent of an unnecessary loading spinner smile.
**Fix**: Replace with a plain factual statement: "Connection was cancelled. You can try again from the integrations page."

---

### 9. "We couldn't finish connecting [platform]. This is usually temporary."

**File**: `connection-wizard/ErrorStep.tsx`
**Lines**: 77-81
**Offending text**:

```typescript
unknown: {
  message: `We couldn't finish connecting ${platformName}. This is usually temporary.`,
```

**Trope**: "This is usually temporary" is a filler reassurance with no information value and no action guidance. It is a hallmark of AI-copy error states.
**Fix**: Drop "This is usually temporary." The retry CTA already implies the user should try again. If there is diagnostic information available, surface it instead.

---

### 10. "We'll start syncing automatically" / "We'll start automatically"

**File**: `connection-wizard/SyncStep.tsx`
**Lines**: 232, 321
**Offending text**:

```tsx
<p className="font-medium text-foreground">We&apos;ll start syncing automatically</p>
...
<p className="mt-1">{etaNote ?? "We will start automatically as soon as capacity opens."}</p>
```

**Trope**: Both phrasings use first-person "We" for a system, a common AI-product anthropomorphism trope. "We'll start syncing automatically" and "We will start automatically" attribute agency to the product in a way that sounds hollow when the underlying reality is a queue wait.
**Fix**: Use passive or system-focused language: "Sync will start automatically when capacity opens" / "Queued. Sync starts when capacity is available."

---

### 11. "Connected successfully" loading state label

**File**: `connection-wizard/connection-wizard-steps.tsx`
**Line**: 247
**Offending text**:

```tsx
<p className="mt-4 text-muted-foreground">Connected successfully</p>
<p className="text-sm text-muted-foreground/70">Preparing your connection...</p>
```

**Trope**: "Preparing your connection" repeats the "Preparing your X" pattern from violation 2. It is a vague progress label that hides what is actually happening (waiting for backend to confirm the pending OAuth record).
**Fix**: Replace with a factual label, e.g., "Confirming connection..." or "Waiting for backend..."

---

### 12. "Connecting to [platform]..." loading label

**File**: `connection-wizard/connection-wizard-steps.tsx`
**Line**: 239
**Offending text**:

```tsx
<p className="mt-4 text-muted-foreground">Connecting to {platformConfig.name}...</p>
```

**Trope**: Not a severe trope, but this appears during the OAuth redirect initiation — the browser is about to navigate away. The label is shown for only a fraction of a second and the "Connecting to..." framing implies a live connection rather than a redirect. Minor.
**Fix**: Replace with "Redirecting to [platform]..." to accurately describe what is happening.

---

## Summary

| #   | File                        | Line(s)  | Issue                                                      |
| --- | --------------------------- | -------- | ---------------------------------------------------------- |
| 1   | ConnectionWizard.tsx        | 4        | "Beautiful wizard" in JSDoc comment                        |
| 2   | SyncStep.tsx                | 148      | "Preparing your inbox" — vague cliche progress label       |
| 3   | SyncStep.tsx                | 126      | "Usually a short wait" — filler reassurance                |
| 4   | SyncStep.tsx                | 505      | "Learning signals" — undefined jargon label                |
| 5   | LearnStep.tsx               | 67       | "You can start working right away" — motivational filler   |
| 6   | LearnStep.tsx               | 65-68    | "Your inbox is ready" — redundant completion confirmation  |
| 7   | ConnectionWizard.tsx        | 452      | "Ensures beautiful UX" in code comment                     |
| 8   | ErrorStep.tsx               | 35-36    | "No worries. You can try again whenever you're ready."     |
| 9   | ErrorStep.tsx               | 77-81    | "This is usually temporary." — filler reassurance in error |
| 10  | SyncStep.tsx                | 232, 321 | "We'll start syncing automatically" — anthropomorphic "We" |
| 11  | connection-wizard-steps.tsx | 247      | "Preparing your connection..." — vague progress label      |
| 12  | connection-wizard-steps.tsx | 239      | "Connecting to..." when redirect is the actual action      |
