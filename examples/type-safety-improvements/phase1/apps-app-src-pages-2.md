# Type-Safety Audit: apps-app-src-pages-2

**Files inspected**: 8
**Findings**: 4

## Summary

`KnowledgeTab.tsx` is the most impactful file: it defines a local `KnowledgeCategory` type that duplicates the one from `@slopweaver/contracts`, and uses two `as` casts that could be replaced with safe property access. `ObservationsTab.tsx` casts the API response body to a local interface that likely duplicates a contract type. The auth-page files, analytics page, and assistant pages are clean.

---

## Findings

### 1

**File**: `apps/app/src/pages/ai/tabs/KnowledgeTab.tsx`
**Category**: Duplicate type definitions
**Impact**: Medium — if the canonical type in contracts gains new members, the local copy silently diverges
**Description**: `KnowledgeCategory` is defined as a local type alias at line 18 (`type KnowledgeCategory = "fact" | "preference" | "style" | "skill" | "context" | "relationship"`). The same type is derivable from `knowledgeCategorySchema` which is already imported from `@slopweaver/contracts` at the top of the file.
**Suggestion**: Replace the local definition with `type KnowledgeCategory = z.infer<typeof knowledgeCategorySchema>;` or import the type directly from contracts if it is exported as `KnowledgeCategory`.
**Evidence**: Line 18: `type KnowledgeCategory = "fact" | "preference" | "style" | "skill" | "context" | "relationship";` while `knowledgeCategorySchema` is imported from `@slopweaver/contracts`

---

### 2

**File**: `apps/app/src/pages/ai/tabs/KnowledgeTab.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — both casts access optional properties that could be read safely
**Description**: Two casts: `(l.source as { id: string }).id` at line 200 and `item.metadata?.["source"] as string` at line 148. The first could use optional chaining on a typed `source` property; the second could use a type guard or `String(...)`.
**Suggestion**: For line 200, check whether `l.source` has a typed interface and use it. For line 148, if the contract metadata schema defines `source` as `string`, derive the type from the schema rather than casting.
**Evidence**:

- Line 148: `item.metadata?.["source"] as string`
- Line 200: `(l.source as { id: string }).id`

---

### 3

**File**: `apps/app/src/pages/ai/tabs/ObservationsTab.tsx`
**Category**: Custom types duplicating SDK types / Duplicate type definitions
**Impact**: Low — the local `NotificationItem` interface may diverge from the contract
**Description**: `res.body.items as NotificationItem[]` at line 46 casts the API response body to a locally defined `NotificationItem` interface. The API response type for this endpoint is already fully typed via the ts-rest contract; the cast bypasses that type.
**Suggestion**: Remove the local `NotificationItem` interface and derive the type from the contract response type using `Extract<Awaited<ReturnType<...>>, { status: 200 }>["body"]["items"][number]`. Then remove the `as` cast.
**Evidence**: `res.body.items as NotificationItem[]` — line 46 (while `NotificationItem` is a locally defined interface)

---

### 4

**File**: `apps/app/src/pages/forgot-password/page.tsx` and `apps/app/src/pages/login/page.tsx` and `apps/app/src/pages/reset-password/page.tsx`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — pattern is consistent across all three auth pages
**Description**: All three auth pages cast `response.body as { message?: string }` in the non-200 branch. The ts-rest contract response type for error status codes (400, 422, etc.) already includes `message` as a typed field via `StandardErrorSchema`. The cast could be replaced by using the typed error response body directly.
**Suggestion**: Check the exact non-200 response type from the contract. If `response.status === 400` narrows `response.body` to `StandardErrorSchema`, destructure `message` directly without casting.
**Evidence**:

- `forgot-password/page.tsx` line 64: `const errorBody = response.body as { message?: string }`
- `login/page.tsx` line 135: `const errorBody = response.body as { message?: string }`
- `reset-password/page.tsx` line 72: `const errorBody = response.body as { message?: string }`
