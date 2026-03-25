# Tropes Audit: apps/app/src/components/layout

Files audited:

- `app-content-loader.tsx`
- `app-layout.tsx`

## Findings

No AI writing tropes found.

Both files are structural/layout components with no user-facing copy. `ContentLoader` renders a skeleton loading state with no text. `AppLayout` composes a sidebar and routed outlet with no visible copy beyond what child routes produce. The comment "Global SSE listener for pattern learning events (toasts + query invalidation)" is a developer comment, not user-facing text.
