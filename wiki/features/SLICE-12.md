# SLICE-12 — Measurement

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | (this finalization commit) |
| frontend | main | 6bf530564f604a8c9d81d93f076dfe41cb0dfa43 |
| backend | main | 5f46cc599365cfcd2b3180776eacaa93f5086fd2 |

## Merged PRs

| Repo | PR | Merge SHA |
|---|---|---|
| frontend | [#14](https://github.com/saadnaveed-amaskit/demo-frontend/pull/14) | 6bf530564f604a8c9d81d93f076dfe41cb0dfa43 |
| backend | [#13](https://github.com/saadnaveed-amaskit/demo-backend/pull/13) | 5f46cc599365cfcd2b3180776eacaa93f5086fd2 |

## Requirements Covered

REQ-MEAS-001…007, 009…011, 014 fully covered. REQ-MEAS-002's cost/time-to-signal figures are surfaced only as a boolean acknowledgment (not separate $ / duration fields). REQ-MEAS-008 (streaming trend + play/pause/restart), REQ-MEAS-012 (closed-loop feedback into Autonomy), and REQ-MEAS-013 (reopen setup) are not implemented — see Known Limitations.

## Acceptance Scenarios Covered

- Go Live gated on balance and cost acknowledgment
- Moving a cluster to control drops its ML price
- Verdict banner on crossing a boundary

Covered on both the frontend (`features/measurement.feature`, UI-driven) and the backend (`tests/features/measurement.feature`, API-driven directly against the real implementation).

## Code Locations

- Backend: `backend/src/measurement/{measurement-types.ts, measurement.controller.ts, measurement.service.ts, measurement.controller.spec.ts}`
- Backend tests: `backend/tests/{features/measurement.feature, steps/measurement.steps.ts, support/app.ts}`
- Frontend: `frontend/src/screens/measurement/{measurement-types.ts, measurement-api.ts, MeasurementScreen.tsx}`
- Frontend tests: `frontend/features/measurement.feature`, `frontend/tests/bdd/steps/measurement.steps.ts`
- Route: `/measurement` in `frontend/src/app/routes.tsx` (previously `ScreenPlaceholder`)

## Frontend Notes

`MeasurementScreen.tsx`: experiment list → detail view. KPI tiles (blocks balanced/total, treatment/control cluster counts, SKUs under test, cost-acknowledged flag). Per-block balance visualized with `@nivo/bar` (`ResponsiveBar`, revenue/grossMargin/velocity match % grouped by block). Blocks/clusters section with per-cluster rationale (BAU/ML price, cross-elasticity, confidence) and a move-arm button per cluster. Cost-of-control acknowledgment via a shadcn `Switch`. Go-Live button disabled with a blocked-reason tooltip until eligible. Live readout section: KPI tiles (probability of winning, incremental margin credible interval, observation day), per-cluster contribution list, verdict banner (gathering/win/kill, color-coded), and scale/kill action buttons. No new pinned-stack dependency was introduced (`Button`/`Switch` from shadcn/ui, `@nivo/bar` already pinned).

## Backend Notes

`MeasurementController`/`MeasurementService`, no constructor dependency on other backend services (self-contained in-memory seed, same pattern as `AgentsService`/`AutonomyService`). Balance is computed on every read from each block's `metricMatch` ratios (min/max ratio >= 0.8 rule) and from whether the block currently has both a treatment and control cluster present (`"Missing-an-arm"` otherwise). Go-Live is gated on all blocks Balanced + cost acknowledged; moving a cluster to `"control"` nulls its displayed `mlPrice` while retaining the underlying recommendation internally so moving back to `"treatment"` restores it. `scale`/`kill` are only valid while `"live"`; `kill` additionally reverts every treatment cluster in the experiment to `"control"`.

## API Contract Notes

`backend/contracts/api-contract.md` updated with 6 new `/measurement/*` endpoints and 8 new schemas (`ExperimentView`, `BlockView`, `ClusterView`, `MetricMatch`, `ReadoutView`, `CredibleInterval`, `ClusterContribution`, `MoveClusterDto`). Per-slice reference contract created (not pre-existing, unlike SLICE-09/10/11 where a draft existed from an earlier preparation pass): `knowledge/contracts/slice-12-measurement/contract.md`, `Stable`.

## Tests

- Backend: `measurement.controller.spec.ts` (Jest, 7 assertions against a mocked service) + `tests/features/measurement.feature` / `tests/steps/measurement.steps.ts` (4 Cucumber scenarios via `supertest` against a real in-process Nest app — this slice's preparation gate also introduced the backend's first-ever Cucumber/BDD tooling: `@cucumber/cucumber`, `ts-node`, `cucumber.js`, `tsconfig.tests.json`, `tests/register.js`, `tests/support/app.ts`)
- Frontend: `features/measurement.feature` / `tests/bdd/steps/measurement.steps.ts` (3 Cucumber scenarios, Playwright-driven)

## Validation Summary

`knowledge/specs/001-platform-baseline/validation/SLICE-12.md` — Shipped. Backend: 133/133 Jest tests passed, 4/4 backend API Cucumber scenarios passed, `npm run check` green. Frontend: 35/35 BDD scenarios passed (3 new + no regressions), 2/2 Vitest, `npm run check` green. 0 patch attempts.

## Known Limitations / Follow-ups

- REQ-MEAS-008 (daily probability-of-winning trend with play/pause/restart) not implemented — the live readout is a static current-state snapshot.
- REQ-MEAS-012 (closed-loop feedback into Autonomy's `ActionClassEntity` accuracy record) unbuilt, per the spec's own `[NEEDS CLARIFICATION]` annotation.
- REQ-MEAS-013 (reopen setup from live/complete, preserving overrides) has no dedicated endpoint.
- Verdict/win-kill boundary day-over-day progression is a v1 deterministic placeholder; only the pre-seeded "already past the win boundary" experiment exercises the win-verdict path.
- REQ-MEAS-002's estimated cost-of-control $ and time-to-signal figures are not separately computed.
