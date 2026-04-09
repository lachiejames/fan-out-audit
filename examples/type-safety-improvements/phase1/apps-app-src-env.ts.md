# Type-Safety Audit: apps-app-src-env.ts

**Files audited:**

- `apps/app/src/env.ts`

---

## Findings

None. The file is clean.

`env.ts` uses a Zod schema to validate `import.meta.env.MODE` and exports a strongly-typed `env` object. The validation is straightforward (`z.enum(["development", "production", "test"])`), the schema is inlined (appropriate for a single-field schema), and there are no type casts, `any` usages, or duplicate type definitions.
