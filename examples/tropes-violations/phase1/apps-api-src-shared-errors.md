# Tropes Audit: apps/api/src/shared/errors

## Files Reviewed

- `apps/api/src/shared/errors/embedding.errors.ts`
- `apps/api/src/shared/errors/message.errors.ts`

## Findings

Error message strings in these files are surfaced to API consumers and potentially displayed in the UI via error responses. Several contain AI writing tropes.

---

### embedding.errors.ts

**Line 53** — `embeddingTimeout` factory:

```
`Embedding generation timeout after ${timeoutMs}ms (Voyage AI not responding)`
```

Violation: parenthetical editorializing ("Voyage AI not responding") is an AI-style aside that reads as the system explaining itself. The parenthetical adds no actionable information for the caller.
Suggested fix: `Embedding generation timed out after ${timeoutMs}ms`

---

### message.errors.ts

**Line 203** — `calendarAccessRequired`:

```
"Calendar access required. Please reconnect Google in Settings to enable meeting scheduling."
```

Violation: "Please" is a politeness filler common in AI-generated copy. The instruction is sound but the phrasing is overly deferential.
Suggested fix: `Calendar access required. Reconnect Google in Settings to enable meeting scheduling.`

**Line 248** — `integrationNotConnected`:

```
`${platform} integration not found. Please connect your account in Settings.`
```

Violation: "Please" politeness filler.
Suggested fix: `${platform} integration not connected. Connect your account in Settings.`

**Line 312** — `trialFeatureLocked`:

```
"This feature is not available during the trial. Upgrade to unlock it."
```

Violation: "Upgrade to unlock it" is a stock upsell phrase common in AI-generated product copy. "Unlock" is a soft-sell cliche.
Suggested fix: `This feature requires a paid plan. Upgrade in Settings.`

---

## Summary

| File                  | Location                          | Issue                                                |
| --------------------- | --------------------------------- | ---------------------------------------------------- |
| `embedding.errors.ts` | `embeddingTimeout` message        | Parenthetical aside editorializing on provider state |
| `message.errors.ts`   | `calendarAccessRequired` message  | "Please" politeness filler                           |
| `message.errors.ts`   | `integrationNotConnected` message | "Please" politeness filler                           |
| `message.errors.ts`   | `trialFeatureLocked` message      | "Upgrade to unlock it" upsell cliche                 |
