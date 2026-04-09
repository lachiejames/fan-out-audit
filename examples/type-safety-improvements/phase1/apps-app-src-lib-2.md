# Type-Safety Audit: apps-app-src-lib-2

**Files inspected**: 8
**Findings**: 5

## Summary

The `billing/native-iap*.ts` pair has the highest concentration of issues: a locally defined `StorePlatform` type is duplicated across two files, `purchaseOptions` uses a loose `Record<string, unknown>` when a concrete `PurchaseOptions` type already exists in scope, and several response-shape casts are necessary because the native Capacitor plugin returns untyped data. The citation utilities are clean; `auth-context.tsx`, `calendar-panel-store.ts`, and `store-billing.ts` have no findings.

---

## Findings

### 1

**File**: `apps/app/src/lib/billing/native-iap-extractors.ts`
**Category**: Custom types duplicating SDK types / `Record<string, unknown>` where stricter types exist
**Impact**: Low — runtime behavior unchanged; misleads readers about the data contract
**Description**: `StoreProduct = Record<string, unknown>` (line 14) is a weak alias for the Capacitor `Product` type. The entire file works around typed Capacitor Store APIs using this alias.
**Suggestion**: Import `Product` from `@capacitor-community/in-app-purchases` (or the Tauri IAP plugin) and replace the alias. If the plugin does not ship typings, declare a local interface that mirrors the actual shape returned by the SDK.
**Evidence**: `type StoreProduct = Record<string, unknown>;` — line 14

---

### 2

**File**: `apps/app/src/lib/billing/native-iap-extractors.ts`
**Category**: Type casts (`as SomeType`) that could be avoided
**Impact**: Low — the casts are guarded by property checks, so runtime safety is acceptable; but the cast pattern obscures that Capacitor returns a plain object
**Description**: Lines 158–163 cast the raw plugin response to `{ products?: unknown }` and line 176 casts to `{ products?: unknown }` / `{ purchases?: unknown }`. These shapes could be encoded as a discriminated Zod schema or a type guard function to avoid scattering `as` casts throughout the extractors.
**Suggestion**: Write a `isProductsResponse(v: unknown): v is { products: StoreProduct[] }` type guard and use it instead of the inline `as` casts.
**Evidence**:

```typescript
(response as { products?: unknown }).products(
  // line 158
  response as { purchases?: unknown },
).purchases; // line 176
```

---

### 3

**File**: `apps/app/src/lib/billing/native-iap-extractors.ts` and `apps/app/src/lib/billing/native-iap.ts`
**Category**: Duplicate type definitions
**Impact**: Medium — two canonical definitions of the same type; if they drift they silently break one call-site
**Description**: `StorePlatform` is defined as a string union in `native-iap-extractors.ts` (line 303) and again independently in `native-iap.ts` (line 35). There is no import relationship between them; both define `"ios" | "android" | "macos"`.
**Suggestion**: Define `StorePlatform` once in `native-iap.ts` (or a shared `native-iap.types.ts`) and re-export or import it in `native-iap-extractors.ts`.
**Evidence**:

- `native-iap-extractors.ts` line 303: `type StorePlatform = "ios" | "android" | "macos";`
- `native-iap.ts` line 35: `type StorePlatform = "ios" | "android" | "macos";`

---

### 4

**File**: `apps/app/src/lib/billing/native-iap.ts`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — `PurchaseOptions` is declared three lines above the usage
**Description**: `purchaseOptions: Record<string, unknown>` at line 530 is the parameter type for an internal function. The `PurchaseOptions` interface is already defined at lines 39–43 of the same file and describes exactly these fields.
**Suggestion**: Replace `Record<string, unknown>` with `PurchaseOptions`.
**Evidence**: `purchaseOptions: Record<string, unknown>` — line 530 (while `interface PurchaseOptions { ... }` exists at lines 39–43)

---

### 5

**File**: `apps/app/src/lib/citations/queue-citation-normalizer.ts`
**Category**: `Record<string, ...>` where stricter types exist
**Impact**: Low — no runtime impact; slightly weakens IDE autocomplete
**Description**: `LEGACY_SOURCE_TYPE_MAP: Record<string, CitationSourceType>` (line 8) maps only two fixed strings. The key set is static and known.
**Suggestion**: Replace with `Record<"related_context" | "source_message", CitationSourceType>` so TypeScript enforces that both keys are present and rejects unknown keys.
**Evidence**: `const LEGACY_SOURCE_TYPE_MAP: Record<string, CitationSourceType> = { related_context: ..., source_message: ... };`
