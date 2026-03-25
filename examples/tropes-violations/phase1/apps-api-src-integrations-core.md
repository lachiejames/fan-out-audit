# Tropes Audit: apps/api/src/integrations/core/errors

## Files Audited

- `apps/api/src/integrations/core/errors/plugin-registry.errors.ts`

## Findings

No findings.

All string content in this file is developer-facing exception messages (thrown as programming errors, not user-visible). The single message in `PluginNotFoundError` is a DI/module wiring diagnostic addressed to developers, not end users. No AI writing tropes apply.
