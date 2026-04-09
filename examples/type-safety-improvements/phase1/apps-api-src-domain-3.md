# Audit: apps-api-src-domain-3

**Files inspected**: 8
**Findings**: 4

## Summary

The integration port files are mostly well-typed. Key findings: `ThreadData` in `message-sync.port.ts` is `unknown` where a documented union would help; `FetchThreadResult` and `SendReplyResult` in `thread-context.port.ts` use `unknown` and an overly-wide intersection type; `SlackThreadResult.users` uses `Record<string, SlackUserProfile>` which is appropriate but noteworthy; and `GmailDraftResult` duplicates the Gmail SDK's draft shape.

## Findings

### Finding 1: `ThreadData` typed as `unknown`

- **File**: `apps/api/src/domain/ports/integrations/message-sync.port.ts:57`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `export type ThreadData = unknown` is used as the return type of `fetchThread()`. The comment explains "The actual structure varies by platform — slack/gmail/linear each return different formats", but `unknown` forces every consumer of `fetchThread` to perform unsafe narrowing with no guidance. A discriminated union per platform, or at least `Record<string, unknown>`, would give callers something to work with.
- **Suggestion**: Define a discriminated union: `type ThreadData = SlackThreadData | GmailThreadData | LinearThreadData | Record<string, unknown>` where the known platforms have typed shapes and a fallback covers future platforms. Alternatively, make `fetchThread` generic: `fetchThread<T = unknown>(params: FetchThreadParams): Promise<T>`.
- **Evidence**:

```typescript
/**
 * Thread data (platform-specific)
 * The actual structure varies by platform - slack/gmail/linear each return different formats
 */
export type ThreadData = unknown;
```

### Finding 2: `FetchThreadResult` typed as `unknown` and `SendReplyResult` as a weak intersection

- **File**: `apps/api/src/domain/ports/integrations/thread-context.port.ts:55-56`
- **Category**: any-usage
- **Impact**: medium
- **Description**: `export type FetchThreadResult = unknown` and `export type SendReplyResult = { platform: string } & Record<string, unknown>` are the return types for the two main port methods. `FetchThreadResult = unknown` is especially problematic — callers of `fetchThread()` get a `Result<unknown, ThreadContextError>` and must unsafely cast to use the result. `SendReplyResult`'s intersection is better but the `Record<string, unknown>` suffix still allows any key.
- **Suggestion**: For `FetchThreadResult`, define at minimum a `{ messages: unknown[]; metadata?: Record<string, unknown> }` shape, or use a discriminated union. For `SendReplyResult`, if `platform` is the only guaranteed field, document the extension contract more explicitly.
- **Evidence**:

```typescript
export type FetchThreadResult = unknown;
export type SendReplyResult = { platform: string } & Record<string, unknown>;
```

### Finding 3: `SlackThreadResult.users` uses `Record<string, SlackUserProfile>`

- **File**: `apps/api/src/domain/ports/integrations/content-fetch.port.ts:116`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `users: Record<string, SlackUserProfile>` maps Slack user IDs (strings) to profiles. The key is semantically a Slack user ID. Using a branded type `SlackUserId = string & { readonly __brand: 'SlackUserId' }` would catch mixing of user IDs with other string identifiers. This is a low-priority improvement.
- **Suggestion**: Consider branding the key: `users: Record<SlackUserId, SlackUserProfile>` or at minimum add a JSDoc comment clarifying the key is a Slack user ID.
- **Evidence**:

```typescript
export interface SlackThreadResult {
  // ...
  users: Record<string, SlackUserProfile>;
  // ...
}
```

### Finding 4: `GmailDraftResult` duplicates Gmail SDK draft shape

- **File**: `apps/api/src/domain/ports/integrations/draft-sync.port.ts:39-45`
- **Category**: sdk-type-duplication
- **Impact**: low
- **Description**: `GmailDraftResult` with `id` (draft ID) and `message: { id: string; threadId: string }` mirrors the `gmail_v1.Schema$Draft` type from the `googleapis` package (`Draft.id`, `Draft.message.id`, `Draft.message.threadId`). Since this is a domain port that intentionally hides SDK types, the duplication is architecturally justified, but the field names should stay in sync with the SDK to avoid confusion.
- **Suggestion**: Add a JSDoc comment linking to `gmail_v1.Schema$Draft` so future maintainers know this mirrors the Gmail API response shape. No code change required.
- **Evidence**:

```typescript
export interface GmailDraftResult {
  id: string; // Draft ID
  message: {
    id: string; // Message ID
    threadId: string; // Thread ID
  };
}
```
