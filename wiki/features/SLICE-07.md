# SLICE-07 — Price Scenario optimization

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#9](https://github.com/saadnaveed-amaskit/demo-frontend/pull/9) (`60667f6`)
- backend: [#7](https://github.com/saadnaveed-amaskit/demo-backend/pull/7) (`34790ae`)

## Requirements Covered

REQ-SCEN-001…011.

## Acceptance Scenarios Covered

Objective sliders always sum to 100; move the scenario position along the frontier; freeze edits once submitted; resubmit a returned scenario.

## Code Locations

- Frontend: `frontend/src/screens/price-scenarios/` (`PriceScenariosScreen.tsx`, `scenario-types.ts`, `scenarios-api.ts`)
- Backend: `backend/src/price-scenarios/` (`price-scenarios.controller.ts`, `price-scenarios.service.ts`)
- Contract: `knowledge/contracts/slice-07-price-scenario/contract.md` (`Stable`)
- Tests: `frontend/features/price-scenario.feature`, `frontend/tests/bdd/steps/price-scenario.steps.ts`, `backend/src/price-scenarios/*.spec.ts`

## Frontend Notes

Objective-weight redistribution logic (`redistribute()`) keeps revenue/grossMargin/sellThrough summed to 100; `StatusBadge` and risk-band helpers established here are reused conceptually by SLICE-09.

## Backend Notes

`PriceScenariosService` depends on `FocusSetsService` and `GuardrailsService`; state machine is `new → draft (run) → pending (submit) → approved/denied/returned`, with `returned → submit` allowed (resubmit). `updateStatus` accepts any target status "to support BDD and approval workflow" — the exact mechanism SLICE-09 later drives directly.

## API Contract Notes

Base path `/price-scenarios`; 8 endpoints under the `price-scenarios` tag in `backend/contracts/api-contract.md`. The contract's `open_q1_resolution` frontmatter states the contract shape was deliberately locked so SLICE-09 (Approvals) could be built against it without re-shaping — confirmed true in practice.

## Tests

BDD: `price-scenario.feature`. Backend: `price-scenarios.controller.spec.ts`, `price-scenarios.service.spec.ts` (create/run/submit transitions + 400 guard, updateStatus appends changeRequests, deep-dive shape/404s).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-07.md`: `status: Shipped`.

## Known Limitations / Follow-ups

Real ML is out of scope for v1 (spec Open Question 1) — frontier/comparison/recommendations are deterministic placeholders behind a locked contract shape.
