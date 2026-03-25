# Tropes Audit: apps/api/src/domain/attachments

Files audited:

- `apps/api/src/domain/attachments/errors/attachment.errors.ts`
- `apps/api/src/domain/attachments/utils/error-mapper.ts`

## Findings

No findings.

Both files contain only internal domain error types, factory functions, and an HTTP error mapper. The user-facing message strings are terse and factual:

- `"Daily attachment budget exceeded (${currentUsage}/${dailyLimit} bytes). Resets at ${resetAt.toISOString()}"`
- `"File size ${fileSize} bytes exceeds maximum ${maxSize} bytes"`
- `"Attachment with ID \"${attachmentId}\" not found"`
- `"MIME type \"${mimeType}\" is not supported. Supported types: ${supportedTypes.join(", ")}"`

None of the checked categories apply: no magic adverbs, no delve/leverage/robust/streamline, no ornate nouns, no serves-as dodge, no negative parallelism, no stakes inflation, no em-dash addiction, no bold-first bullets, no pedagogical voice, no false suspense.
