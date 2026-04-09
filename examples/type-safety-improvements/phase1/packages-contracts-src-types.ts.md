# Audit: packages-contracts-src-types.ts

**Files inspected**: 1
**Findings**: 2

## Summary

`types.ts` is the root type aggregator that exports all contract types in both namespaced form (e.g., `TodoTypes`) and as flat re-exports for convenience. The file is well-structured and follows the dual-export pattern cleanly. The main concerns are: several contract domains with known type issues (Knowledge, Triage, SyncProgress) are exposed here via flat re-exports, potentially surfacing conflicting type names to consumers, and some type-only imports are missing `import type` syntax.

## Findings

### Finding 1: Missing coverage for several contract domains in the type aggregator

- **File**: `packages/contracts/src/types.ts`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: Several contract domains with exported types are not represented in `types.ts` as namespaced exports or flat re-exports: `SuggestedActionsTypes`, `WorkItemTypes`, `PredictionTypes`, `ProactiveTypes`, `PersonaTypes`, `TrainingTypes`, `TriageTypes`, `MemoryTypes`. This means consumers who want namespaced access (e.g., `WorkItemTypes.WorkItemStatus`) must import directly from the feature module path, making the `types.ts` aggregator incomplete.
- **Suggestion**: Add the missing namespaced exports:
  ```ts
  export * as SuggestedActionsTypes from "@/contracts/suggested-actions/types.ts";
  export * as WorkItemTypes from "@/contracts/work-items/types.ts";
  export * as PredictionTypes from "@/contracts/predictions/types.ts";
  export * as TriageTypes from "@/contracts/triage/constants.ts";
  export * as MemoryTypes from "@/contracts/memory/types.ts";
  ```
- **Evidence**: The namespaced exports section (line 17-32) does not include `WorkItemTypes`, `SuggestedActionsTypes`, `PredictionTypes`, `TriageTypes`, or `MemoryTypes`.

### Finding 2: Flat re-exports from `types.ts` do not include `WorkItem`, `WorkItemStatus`, etc.

- **File**: `packages/contracts/src/types.ts`
- **Category**: missing-strict-typing
- **Impact**: low
- **Description**: The "COMMONLY USED TYPE EXPORTS" section re-exports frequently used types for convenience, but `WorkItem`, `WorkItemStatus`, `WorkItemType`, and `CreateWorkItemBody` are absent. These are among the most commonly used types in the frontend (the queue feature uses them extensively). Consumers currently must import from `@slopweaver/contracts/work-items/types` or the feature path directly.
- **Suggestion**: Add flat convenience exports for work-item types:
  ```ts
  export type { WorkItem, WorkItemStatus, WorkItemType, CreateWorkItemBody } from "@/contracts/work-items/types.ts";
  ```
- **Evidence**: The flat re-exports section ends with `UserTypes` re-exports (line 165-177) and does not include work-items, suggested-actions, or prediction types.
