# Audit: apps-api-src-application-6

**Files inspected**: 8
**Findings**: 6

## Summary

These files implement the behavioral fingerprint analysis pipeline (response time, tone, formality, confidence scoring) plus the billing AI cost tracking and token cap enforcement services. The behavioral files are well-typed pure functions. The main issues are: a type cast on `defaultTone` that could be eliminated with a proper DB enum type, a local `CostSummary`/`SavingsBreakdown` interface duplication of types already exported from `@slopweaver/contracts`, weak `Record<string, number>` types where stricter keyed maps exist, and a positional-param violation on `canProceedWithAICall`.

## Findings

### Finding 1: `CostSummary` and `SavingsBreakdown` duplicate contract types

- **File**: `apps/api/src/application/billing/ai-costs/services/cost-tracker.service.ts:32-44`
- **Category**: duplicate-type
- **Impact**: high
- **Description**: `CostSummary` and `SavingsBreakdown` are defined as local private interfaces in the service. The contracts package already exports `CostSummaryResponse` and `SavingsBreakdownResponse` (from `@slopweaver/contracts`) which are inferred from the exact same Zod schemas (`costSummaryResponseSchema` / `savingsBreakdownResponseSchema`) that define what this service actually returns. The local definitions are therefore a second source of truth for the same shape, creating a drift risk if the contract schema ever changes.
- **Suggestion**: Replace the local interfaces with imports of `CostSummaryResponse` and `SavingsBreakdownResponse` from `@slopweaver/contracts`, then update the return types of `getUserCostSummary` and `getSavingsBreakdown` accordingly.
- **Evidence**:

```typescript
// cost-tracker.service.ts (lines 32-44) — local definitions
interface CostSummary {
  totalCost: number;
  totalSavings: number;
  requestCount: number;
  byTaskType: Record<string, number>;
  byModel: Record<string, number>;
}

interface SavingsBreakdown {
  fromCaching: number;
  fromBatching: number;
  fromModelRouting: number;
  total: number;
  percentageReduction: number;
}

// Already exported from @slopweaver/contracts:
// export type CostSummaryResponse = z.infer<typeof costSummaryResponseSchema>;
// export type SavingsBreakdownResponse = z.infer<typeof savingsBreakdownResponseSchema>;
```

---

### Finding 2: `canProceedWithAICall` uses positional param instead of named object param

- **File**: `apps/api/src/application/billing/ai-costs/services/token-cap.service.ts:92`
- **Category**: missing-strict-typing
- **Impact**: medium
- **Description**: `canProceedWithAICall(userId: string)` uses a positional parameter. The codebase's mandatory TypeScript pattern requires named object params for all functions with 1+ parameters (unless they are framework callbacks, type guards, or zero-param). This method is an exported service method, not a framework callback. `getUsageStats` in the same class correctly uses `{ userId }: { userId: string }`, making the inconsistency visible.
- **Suggestion**: Change the signature to `canProceedWithAICall({ userId }: { userId: string }): Promise<Result<void, TokenCapError>>` and update any callers.
- **Evidence**:

```typescript
// token-cap.service.ts line 92 — positional param
async canProceedWithAICall(userId: string): Promise<Result<void, TokenCapError>> {

// Same class, line 142 — named param (correct)
async getUsageStats({ userId }: { userId: string }): Promise<TokenUsageStats> {
```

---

### Finding 3: `ResponseTimeByHour` locally redefined, diverges from the contract type

- **File**: `apps/api/src/application/behavioral/services/response-time-analyzer.service.ts:31`
- **Category**: duplicate-type
- **Impact**: medium
- **Description**: `ResponseTimeByHour` is defined locally as `Record<number, number>` (hour → minutes). The contracts package exports `ResponseTimeByHour` as `Record<string, number>` (derived from `responseTimeByHourSchema = z.record(z.string(), z.number())`), because JSON keys are always strings. These are structurally different types. The local definition using `number` keys silently diverges from the serialized form used by the API contract, which is what the DB stores in the `responseTimeByHour` JSONB column. The cast in `behavioral.utils.ts` line 52 (`as Record<string, number> | null`) is a direct consequence of this divergence.
- **Suggestion**: Remove the local `ResponseTimeByHour` and import the contract type: `import type { ResponseTimeByHour } from "@slopweaver/contracts"`. This collapses the cast at line 52 of `behavioral.utils.ts` automatically, since DB JSONB data already has string keys.
- **Evidence**:

```typescript
// response-time-analyzer.service.ts line 31 — local definition with number keys
export type ResponseTimeByHour = Record<number, number>; // hour (0-23) -> avg response time in minutes

// packages/contracts/src/contracts/behavioral/schemas.ts line 18 — contract definition with string keys
export const responseTimeByHourSchema = z.record(z.string(), z.number().min(0));
// → type ResponseTimeByHour = Record<string, number>

// behavioral.utils.ts line 52 — cast required to bridge the gap
responseTimeByHour: dbFingerprint.responseTimeByHour as Record<string, number> | null,
```

---

### Finding 4: `defaultTone` cast in `toContractFingerprint`

- **File**: `apps/api/src/application/behavioral/services/behavioral.utils.ts:43`
- **Category**: type-cast
- **Impact**: medium
- **Description**: `dbFingerprint.defaultTone` is cast to `ContractBehavioralFingerprint["defaultTone"]` with `as`. This is necessary because the DB column's inferred type doesn't match the contract's `"formal" | "casual" | "balanced" | null`. The cast is safe today, but it bypasses compile-time verification. If the DB enum gains a new value not in the contract (or vice versa), the cast silently allows an invalid value to pass through.
- **Suggestion**: Check the DB table definition for `defaultTone`. If it uses a Drizzle `pgEnum` with the same values as `channelToneSchema`, import `channelToneSchema.options` into the table definition so Drizzle infers the correct literal union. This would make the types structurally compatible and eliminate the cast. If the column is plain `text`, consider narrowing it with a Drizzle `.$type<ToneType>()` annotation.
- **Evidence**:

```typescript
// behavioral.utils.ts line 43
defaultTone: dbFingerprint.defaultTone as ContractBehavioralFingerprint["defaultTone"],
```

---

### Finding 5: `RELATIONSHIP_FORMALITY_MAP` uses `Record<string, number>` instead of a stricter mapped type

- **File**: `apps/api/src/application/behavioral/utils/formality-analyzer.utils.ts:53`
- **Category**: record-weakening
- **Impact**: low
- **Description**: `RELATIONSHIP_FORMALITY_MAP` is typed `Record<string, number>`. The keys are a fixed, known set of relationship strings (`"client"`, `"colleague"`, `"direct_report"`, etc.). Using `Record<string, number>` means TypeScript won't catch typos in lookup keys, and the `?? 50` fallback in `getRelationshipFormality` is needed to paper over the lack of exhaustiveness. A stricter type would prevent lookups for keys not in the map.
- **Suggestion**: Define a `RelationshipType` union type for the keys and type the map as `Record<RelationshipType | "unknown", number>`. The `getRelationshipFormality` function's `relationship: string | null` parameter would stay as-is, but narrowing via `relationship in RELATIONSHIP_FORMALITY_MAP` would give exhaustive key checking within the map itself.
- **Evidence**:

```typescript
// formality-analyzer.utils.ts lines 53-64
export const RELATIONSHIP_FORMALITY_MAP: Record<string, number> = {
  client: 85,
  colleague: 50,
  direct_report: 55,
  executive: 90,
  external: 75,
  friend: 25,
  manager: 75,
  mentor: 60,
  team: 45,
  unknown: 50,
};
```

---

### Finding 6: `ResponseSpeedByRecipient` and `FormalityByRecipient` locally redefined vs contract exports

- **File**: `apps/api/src/application/behavioral/services/response-time-analyzer.service.ts:44` and `apps/api/src/application/behavioral/utils/formality-analyzer.utils.ts:11`
- **Category**: duplicate-type
- **Impact**: low
- **Description**: Both `ResponseSpeedByRecipient` (`Record<string, number>`) and `FormalityByRecipient` (`Record<string, number>`) are defined locally in the application layer. The contracts package exports identically-shaped types with the same names (`ResponseSpeedByRecipient` and `FormalityByRecipient` in `@slopweaver/contracts`). `fingerprint-confidence-calculator.ts` already imports the local versions; if the contract types ever add validation constraints (e.g., value ranges), the local copies would lag behind. This is a lower-impact duplication because the shapes are currently identical `Record<string, number>`, but it is still a split source of truth.
- **Suggestion**: Remove both local type aliases and import them from `@slopweaver/contracts` instead. Update the importing files (`fingerprint-confidence-calculator.ts` and downstream) accordingly.
- **Evidence**:

```typescript
// response-time-analyzer.service.ts line 44 — local definition
export type ResponseSpeedByRecipient = Record<string, number>; // entityId -> avg response time in minutes

// formality-analyzer.utils.ts line 11 — local definition
export type FormalityByRecipient = Record<string, number>;

// packages/contracts/src/contracts/behavioral/types.ts lines 19-20 — contract exports
export type FormalityByRecipient = z.infer<typeof formalityByRecipientSchema>;
export type ResponseSpeedByRecipient = z.infer<typeof responseSpeedByRecipientSchema>;
```
