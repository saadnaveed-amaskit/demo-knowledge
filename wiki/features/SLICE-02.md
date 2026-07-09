# SLICE-02 — Focus Set management

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#3](https://github.com/saadnaveed-amaskit/demo-frontend/pull/3) (`9a2fa8c`)
- backend: [#2](https://github.com/saadnaveed-amaskit/demo-backend/pull/2) (`aab22b9`)

## Requirements Covered

REQ-FOCUS-001…012.

## Acceptance Scenarios Covered

Create a Focus Set with live preview; warn on a no-match filter; search/sort the Focus Set library; export a Focus Set to CSV.

## Code Locations

- Frontend: `frontend/src/screens/focus-builder/` (`FocusBuilder.tsx`, `ConditionTree.tsx`, `focus-types.ts`, `focus-api.ts`, `csv.ts`, `state.ts`)
- Backend: `backend/src/focus-sets/` (`focus-sets.controller.ts`, `focus-sets.service.ts`, condition-tree evaluation in `condition.ts`)
- Contract: `knowledge/contracts/slice-02-focus-sets/contract.md` (`Stable`)
- Tests: `frontend/features/focus-set-management.feature`, `frontend/tests/bdd/steps/focus-set-management.steps.ts`, `backend/src/focus-sets/*.spec.ts`

## Frontend Notes

First full-stack, contract-first slice. Introduces the recursive AND/OR condition-tree editor (`ConditionTree.tsx`) and a dedicated `useFocusBuilderStore` Zustand store for create/edit form state.

## Backend Notes

`FocusSetsService` is `Map`-backed (not an array like most other services) and resolves condition trees live against catalog data. Depended on by `PriceScenariosService` and the discount-modeling focus-group selector.

## API Contract Notes

Base path `/focus-sets`; see `backend/contracts/api-contract.md` for the full endpoint/schema list under the `focus-sets` tag (9 endpoints: list, create, resolve preview, get/update/delete by id, get SKUs, duplicate).

## Tests

BDD: `focus-set-management.feature`. Backend: `focus-sets.controller.spec.ts`, `focus-sets.service.spec.ts` (filter resolution, auto-incremented id, edit-in-place, duplicate, live productCount, delete).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-02.md`: status `PASS — shipped`, finalized 2026-07-09.

## Known Limitations / Follow-ups

None recorded beyond platform-level Open Question 3 (Focus Set resolution: live evaluation vs. persisted snapshot), which spec.md records as resolved in favor of live evaluation.
