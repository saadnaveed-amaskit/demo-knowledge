# SLICE-06 — Discount Modeling

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#8](https://github.com/saadnaveed-amaskit/demo-frontend/pull/8) (`8b6f4a4`)
- backend: [#6](https://github.com/saadnaveed-amaskit/demo-backend/pull/6) (`0b6d584`)

## Requirements Covered

REQ-DISC-001…010.

## Acceptance Scenarios Covered

Run Model gated on required fields; open an existing model's output directly; discard a model regardless of status.

## Code Locations

- Frontend: `frontend/src/screens/discount-modeling/` (`DiscountModelingScreen.tsx`, `discount-model-types.ts`, `discount-modeling-api.ts`, `focus-sets-ref-api.ts`)
- Backend: `backend/src/discount-modeling/` (`discount-modeling.controller.ts`, `discount-modeling.service.ts`)
- Contract: `knowledge/contracts/slice-06-discount-modeling/contract.md` (`Stable`)
- Tests: `frontend/features/discount-modeling.feature`, `frontend/tests/bdd/steps/discount-modeling.steps.ts`, `backend/src/discount-modeling/*.spec.ts`

## Frontend Notes

Nivo line/bar charts for forecast revenue/margin/units; `STATUS_LABELS`/`STATUS_COLORS` maps establish the status-badge convention reused by SLICE-07/SLICE-09.

## Backend Notes

`DiscountModelingService` computes KPIs, rollup rows (one per `RN_DIVISIONS` division), and `RiskPanel[]` (severity `high`/`medium`/`low`, `isHard` flag) deterministically from discount format/depth. `submit()` only accepts status `draft` — unlike price-scenarios, there is no `returned`-resubmit path directly in this service (SLICE-09 adds that via the Approvals decision endpoint instead).

## API Contract Notes

Base path `/discount-models`; 7 endpoints (list, create, get, run, submit, patch status, delete) under the `discount-models` tag in `backend/contracts/api-contract.md`. `DiscountModelEntity` has no `changeRequests` field (unlike `ScenarioEntity`) — relevant to SLICE-09's workaround.

## Tests

BDD: `discount-modeling.feature`. Backend: `discount-modeling.controller.spec.ts`, `discount-modeling.service.spec.ts` (create, run→draft, submit draft→pending + rejection, updateStatus, remove + 404s, bogo rate/margin math).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-06.md`: `status: Shipped`.

## Known Limitations / Follow-ups

Per `backend/contracts/api-contract.md`, an unresolved `focusGroupId` on create is silently swallowed (defaults kept) rather than propagating a 404 — asymmetric with price-scenarios' create, which does 404 in the same situation.
