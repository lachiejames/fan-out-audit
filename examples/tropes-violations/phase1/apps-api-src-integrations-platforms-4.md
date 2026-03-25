# Tropes Audit: apps/api/src/integrations/platforms (batch 4)

Files audited:

- `microsoft-outlook/errors/microsoft-outlook.errors.ts`
- `microsoft-outlook/utils/microsoft-outlook-error-mapper.ts`
- `microsoft-teams/errors/microsoft-teams.errors.ts`
- `microsoft-teams/utils/microsoft-teams-error-mapper.ts`
- `monday/errors/monday.errors.ts`
- `monday/utils/monday-error-mapper.ts`
- `notion/errors/notion.errors.ts`
- `notion/utils/notion-error-mapper.ts`

## Findings

### microsoft-teams/errors/microsoft-teams.errors.ts

**Line 129** — `apiError` factory:

```
message: "MicrosoftTeams API returned an error",
```

The brand name is run together as "MicrosoftTeams" instead of "Microsoft Teams". This is a display string seen in error responses, not an internal identifier.

**Line 148** — `integrationInactive` factory:

```
message: `MicrosoftTeams integration is not active (status: ${status}). Please reconnect your MicrosoftTeams account.`
```

Same run-together "MicrosoftTeams" in two places within a user-facing message.

**Line 154** — `integrationNotFound` factory:

```
message: "MicrosoftTeams integration not found. Please connect your MicrosoftTeams account."
```

Same issue.

**Line 171** — `oauthFailed` factory:

```
message: "MicrosoftTeams OAuth authentication failed",
```

Same.

**Line 181** — `syncFailed` factory:

```
message: "MicrosoftTeams sync failed",
```

Same.

**Line 187** — `tokenExpired` factory:

```
message: "MicrosoftTeams access token expired. Please reconnect your MicrosoftTeams account."
```

Same issue in two places.

**Line 190** — `unauthorized` factory default:

```
message = "Please reconnect your MicrosoftTeams account"
```

Same.

**Line 177** — `rateLimited` factory:

```
message: "Rate limited by Microsoft Teams API. Please try again later.",
```

"Please try again later" is a common AI writing filler phrase. The message gives the user no actionable information beyond waiting. Acceptable for a rate-limit error but worth noting.

### monday/errors/monday.errors.ts

**Line 150** — `apiError` factory:

```
message: "Monday.com API returned an error.",
```

No violations beyond what is already structural to the error-factory pattern. Clean.

**Line 165** — `budgetExceeded` factory:

```
message: reason ?? "Monday.com action budget exceeded",
```

Clean.

**Line 175** — `graphqlError` factory:

```
message: ... : "GraphQL error",
```

Terse fallback but not a trope violation.

No findings.

### notion/errors/notion.errors.ts

**Line 206** — `permissionDenied` factory fallback message:

```
"Permission denied. The integration may not have access to this resource."
```

"may not have access" is hedged, vague language. The system either has access or it doesn't. A clearer phrasing: "Permission denied. The Notion integration does not have access to this resource."

**Line 213** — `rateLimited` fallback:

```
"Notion API rate limited. Please try again later."
```

Same "please try again later" filler as noted above for Teams. Borderline.

### microsoft-outlook/errors/microsoft-outlook.errors.ts

No trope violations. Error messages are terse and factual.

### Error mapper files (all four)

No user-facing strings. Status code mappings only. No findings.

## Summary

| File                                                        | Violations                                                                       |
| ----------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `microsoft-outlook/errors/microsoft-outlook.errors.ts`      | None                                                                             |
| `microsoft-outlook/utils/microsoft-outlook-error-mapper.ts` | None                                                                             |
| `microsoft-teams/errors/microsoft-teams.errors.ts`          | Brand name typo ("MicrosoftTeams" run together) in 7 user-facing message strings |
| `microsoft-teams/utils/microsoft-teams-error-mapper.ts`     | None                                                                             |
| `monday/errors/monday.errors.ts`                            | None                                                                             |
| `monday/utils/monday-error-mapper.ts`                       | None                                                                             |
| `notion/errors/notion.errors.ts`                            | Hedged language in `permissionDenied` fallback                                   |
| `notion/utils/notion-error-mapper.ts`                       | None                                                                             |

## Recommended Fixes

**microsoft-teams/errors/microsoft-teams.errors.ts** — Replace all instances of `"MicrosoftTeams"` in message strings with `"Microsoft Teams"` (with space). Affected factories: `apiError`, `integrationInactive`, `integrationNotFound`, `oauthFailed`, `syncFailed`, `tokenExpired`, `unauthorized`.

**notion/errors/notion.errors.ts** — Line 206: change `"The integration may not have access to this resource."` to `"The Notion integration does not have access to this resource."` to remove hedging.
