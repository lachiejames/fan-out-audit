# AI Writing Tropes Audit: apps/app/src (App.tsx, main.tsx)

Files audited:

- `/Users/lachiejames/dev/slopweaver/apps/app/src/App.tsx`
- `/Users/lachiejames/dev/slopweaver/apps/app/src/main.tsx`

## Findings

No violations.

Both files are infrastructure-only. `App.tsx` contains React Router route definitions with no JSX text content, error messages, or loading state strings. `main.tsx` contains provider wiring, QueryClient configuration, and app bootstrap logic with no user-facing copy.

The one string that could surface to a user is the thrown `Error` at `main.tsx:112`:

```
"Root element not found. Check index.html has <div id='root'></div>"
```

This fires before React mounts and is only visible in the browser console as an uncaught exception. It is a developer error message, not user-facing UI copy, and does not require changes.
