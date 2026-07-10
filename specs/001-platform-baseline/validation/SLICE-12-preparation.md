---
status: Approved
artifact_type: slice-preparation
slice_id: SLICE-12
approved_by: Saad
approved_at: 10-07-2026
source_spec: knowledge/specs/001-platform-baseline/spec.md
source_plan: knowledge/specs/001-platform-baseline/plan.md
source_tasks: knowledge/specs/001-platform-baseline/tasks.md
frontend_branch: agent/slice-12-measurement
backend_branch: agent/slice-12-measurement
---

# Slice Preparation: `SLICE-12`

## Status

Approved

## Source Documents

- `knowledge/specs/platform-baseline-context.md`
- `knowledge/specs/001-platform-baseline/spec.md` (status: Approved) — REQ-MEAS-001…014, "Measurement" Gherkin block
- `knowledge/specs/001-platform-baseline/plan.md` (status: Approved) — SLICE-12 section
- `knowledge/specs/001-platform-baseline/tasks.md` (status: Approved) — SLICE-12 section (Status: Approved at time of preparation)
- `knowledge/.specify/memory/constitution.md`
- `knowledge/TECHNOLOGY.md`
- `knowledge/DATA-TAXONOMY.md`
- `backend/contracts/api-contract.md` (read for existing conventions; no Measurement entries exist yet — expected, this slice hasn't been implemented)

## Slice Scope

SLICE-12 — Measurement. Setup (AI-proposed arm assignment, per-block balance checks, cost-of-control acknowledgment, Go-Live gate) plus live readout (probability-of-winning trend, KPI tiles, per-cluster contribution, verdict banner with scale/kill actions). Target repos: frontend, backend. Depends on SLICE-07 (Price Scenario optimization), which is `Complete`. Requirements: REQ-MEAS-001…014. Acceptance scenarios: Go Live gated on balance and cost acknowledgment; moving a cluster to control drops its ML price; verdict banner on crossing a boundary.

**REQ-MEAS-012** ("feed the causal result into the relevant pricing action class's accuracy record") is explicitly annotated in the approved spec itself as `[NEEDS CLARIFICATION: closed loop unbuilt today]` — this closed-loop integration with SLICE-11 (Pricing Autonomy)'s `ActionClassEntity` accuracy tracking is **out of scope for this preparation and for the three approved acceptance scenarios**, consistent with the spec's own annotation and the deferred-scope precedent established for SLICE-09 (REQ-APPR-005/008). Spec Open Question 9 (automatic vs. manual, in scope now?) remains unresolved and is not required to write the three approved Gherkin scenarios' tests.

No implementation work was done in this command — no contract file, no production code, in either repo.

## Implementation Branches Created or Reused

| Repo | Branch | State before this command | Action |
|---|---|---|---|
| knowledge | 001-platform-baseline | clean, up to date | Reused (no new branch — per Speckit branch exception) |
| frontend | main (branched from) | clean, up to date, no `agent/slice-12-*` branch existed | Created `agent/slice-12-measurement` |
| backend | main (branched from) | clean, up to date, no `agent/slice-12-*` branch existed | Created `agent/slice-12-measurement` |

Both implementation branches were newly created (did not previously exist) — no risk of pre-existing unreviewed production code.

## Tests Created or Updated

- `frontend/features/measurement.feature` — created. Three scenarios, reusing spec.md's approved Gherkin text verbatim.
- `frontend/tests/bdd/steps/measurement.steps.ts` — created. Drives the anticipated `/measurement` route and anticipated backend endpoints (`GET /measurement/experiments/:id`, `POST .../clusters/:clusterId/move`, `POST .../acknowledge-cost`), documented inline as "anticipated backend surface — not yet implemented." Assumes 3 seeded experiments at distinct lifecycle stages (imbalanced/unacknowledged; balanced/unacknowledged with a treatment cluster; live with a "win" verdict) since the 3 approved scenarios each require a different precondition state and there's no requirement for an in-scenario state-mutation path from imbalanced → balanced (balance is a derived property of cluster arm assignments, not a directly settable flag).
- `backend/src/measurement/measurement.controller.spec.ts` — created. References not-yet-existing `measurement.controller.ts`, `measurement.service.ts`, `measurement-types.ts` (anticipated `ExperimentView`, `ClusterView`, `BlockView`, `ReadoutView` shapes) to exercise listExperiments/getExperiment/moveCluster/acknowledgeCost/goLive/scale ahead of implementation.
- `backend/tests/features/measurement.feature` — created. 4 scenarios phrased as observable API request/response behavior (status codes, response body fields), exercising the backend directly — not reusing the frontend's UI-phrased Gherkin text verbatim, per the rule that backend scenarios must exercise the API directly, not through the UI.
- `backend/tests/steps/measurement.steps.ts` — created. Drives the anticipated `/measurement/*` endpoints via `supertest` against an in-process Nest application (no frontend involved, no port bound).
- `backend/tests/support/app.ts` — created. Boots (once, via `@nestjs/testing`) an in-process `AppModule` instance and exposes it to step definitions; this is the backend's first Cucumber-driven API test, so this support utility is new infrastructure, not a reuse of an existing pattern.
- `backend/cucumber.js`, `backend/tsconfig.tests.json`, `backend/tests/register.js` — created. Backend BDD/Cucumber setup did not exist in this repo before this slice (only Jest existed). `ts-node` (real TypeScript compiler) is used instead of the frontend's `tsx` (esbuild-based), because NestJS's dependency injection relies on `emitDecoratorMetadata`, which esbuild does not emit — verified this was necessary by confirming the in-process app boots and resolves providers correctly (clean 404s from the router, not a DI/module-resolution crash).
- `backend/package.json` — updated. Added `@cucumber/cucumber` and `ts-node` as devDependencies, and a `test:bdd` script (`cucumber-js`), mirroring the frontend's script name for consistency. `npm install` was run to install them.

No production frontend or backend code, and no `knowledge/contracts/slice-12-measurement/` contract file, were created — contract authoring is an implementation-phase task per `tasks.md`'s Backend Tasks, not part of this preparation command's allowed scope. The backend BDD tooling additions (`ts-node`, `@cucumber/cucumber`, config/support files) are test infrastructure, not production application code.

## Expected Failing Results

| Command | Repo | Result | Expected failure reason |
|---|---|---|---|
| `npx jest measurement` | backend | Failed (TS2307 — cannot find `./measurement.controller`, `./measurement.service`, `./measurement-types`) | Module does not exist yet — correct pre-implementation red |
| `npx jest` (full suite) | backend | 126/126 pre-existing tests passed; 1 new suite failed to compile | No regressions; only the new anticipated module fails |
| `node --import tsx cucumber.js --config cucumber.mjs features/measurement.feature` | frontend | 32/35 scenarios passed, 3 failed (the 3 new ones) | "Go Live gated..." fails because `GET /measurement/experiments/1` 404s (`experiment.blocks` is `undefined`); "Moving a cluster..." fails the same way against experiment 2; "Verdict banner..." fails its own precondition check because experiment 3 doesn't exist (status/verdict undefined, not `"live"`/`"win"`) |
| `npm run test:bdd` (`cucumber-js`) | backend | Failed (4/4 scenarios failed) | Every scenario's first API call (`GET /measurement/experiments/:id` or a POST action) returns a clean `404` from the in-process Nest app — the router has no `/measurement/*` routes registered yet. Confirms the Cucumber+ts-node+supertest infrastructure itself works correctly (the app boots, DI resolves, routing works) — the failures are purely "route not found," the correct pre-implementation red. |
| `npm run check` (lint + lint:policy + typecheck + build) | backend | Failed at typecheck — same pre-existing `measurement.controller.spec.ts` TS2307 errors as before adding the BDD infra | Confirms the new `tests/` files (excluded from `tsconfig.json`'s `include: ["src/**/*"]`) do not interfere with the existing check pipeline; the only failure is the already-expected Jest contract test compile error |

All backend and frontend failures are for the correct reason (missing backend endpoint / router), not for unrelated infrastructure issues. No pre-existing scenario regressed (32/32 pre-existing frontend BDD scenarios and 126/126 pre-existing backend Jest tests still pass).

## Coverage Map

| Requirement | Scenario | Test file | Step definition | Expected pre-implementation result |
|---|---|---|---|---|
| REQ-MEAS-005, REQ-MEAS-006 | Go Live gated on balance and cost acknowledgment | `frontend/features/measurement.feature` | `frontend/tests/bdd/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/1` does not exist (404) |
| REQ-MEAS-007 | Moving a cluster to control drops its ML price | `frontend/features/measurement.feature` | `frontend/tests/bdd/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/2` does not exist (404) |
| REQ-MEAS-011, REQ-MEAS-014 | Verdict banner on crossing a boundary | `frontend/features/measurement.feature` | `frontend/tests/bdd/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/3` does not exist (404) |
| REQ-MEAS-005/006 (backend, contract test) | Go-live blocked when ineligible | `backend/src/measurement/measurement.controller.spec.ts` | N/A (Jest, not Cucumber) | Fail to compile — module does not exist |
| REQ-MEAS-007 (backend, contract test) | Move cluster clears mlPrice | `backend/src/measurement/measurement.controller.spec.ts` | N/A (Jest, not Cucumber) | Fail to compile — same as above |
| REQ-MEAS-011 (backend, contract test) | Live experiment win verdict | `backend/src/measurement/measurement.controller.spec.ts` | N/A (Jest, not Cucumber) | Fail to compile — same as above |
| REQ-MEAS-005, REQ-MEAS-006 (backend API) | Go-live is blocked when a block is imbalanced | `backend/tests/features/measurement.feature` | `backend/tests/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/1` returns 404 |
| REQ-MEAS-005, REQ-MEAS-006 (backend API) | Go-live succeeds once all blocks are balanced and cost is acknowledged | `backend/tests/features/measurement.feature` | `backend/tests/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/2` returns 404 |
| REQ-MEAS-007 (backend API) | Moving a cluster to control clears its ML price recommendation | `backend/tests/features/measurement.feature` | `backend/tests/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/2` returns 404 |
| REQ-MEAS-011, REQ-MEAS-014 (backend API) | A live experiment reports a win verdict once its win-probability crosses the boundary | `backend/tests/features/measurement.feature` | `backend/tests/steps/measurement.steps.ts` | Fail — `GET /measurement/experiments/3` returns 404 |

REQ-MEAS-001, 002, 003, 004, 008, 009, 010, 013 (arm-assignment proposal, KPI strip, per-block balance visualization, probability trend streaming, KPI tiles, per-cluster contribution, reopen-setup-preserving-overrides) are covered by the same `measurement.controller.spec.ts`'s `listExperiments`/`getExperiment` assertions and the entity/contract design implied by the three acceptance scenarios above; dedicated additional Gherkin scenarios for these were not written in this preparation pass since spec.md's approved Gherkin block for this slice only supplies the three scenarios above — adding further scenarios beyond the approved set would be inventing behavior, which this command must not do. **REQ-MEAS-012 is explicitly out of scope** (see Slice Scope above). If the human wants explicit BDD coverage for arm-assignment/balance-visualization/live-streaming/reopen-setup beyond what's implied, that should be added via `/speckit-clarify` or an explicit spec update before implementation.

## Backend API Gherkin Decision

**Backend API Gherkin required and created.**

This slice introduces new, externally observable backend/API behavior (6 new `/measurement/*` endpoints: list/get experiments, move cluster, acknowledge cost, go-live, scale/kill), so a Jest contract test alone is not sufficient — backend API Gherkin/Cucumber scenarios exercising the API directly are required per the acceptance-test-first and contract-first rules, and contract/unit tests supplement but do not replace them.

Created:
- `backend/tests/features/measurement.feature` — 4 scenarios, phrased as observable request/response behavior (status codes, response body fields), covering REQ-MEAS-005, 006, 007, 011, 014.
- `backend/tests/steps/measurement.steps.ts` — step definitions using `supertest` against an in-process Nest application (`backend/tests/support/app.ts`), calling the backend API directly — no frontend UI involved, per the requirement that backend Gherkin scenarios must not drive the frontend.
- Backend Cucumber/BDD tooling did not exist in this repo before this slice (only Jest existed for any prior slice). Added `@cucumber/cucumber` and `ts-node` as devDependencies, a `cucumber.js` config, a dedicated `tsconfig.tests.json`, and a `tests/register.js` ts-node bootstrap. `ts-node` (not the frontend's `tsx`) was chosen because NestJS's DI relies on `emitDecoratorMetadata`, which esbuild-based `tsx` does not emit — verified necessary by confirming the app boots and resolves providers cleanly under `ts-node` (failures are clean 404s, not DI/module-resolution crashes).
- `npm run test:bdd` (`cucumber-js`) was run and all 4 scenarios failed for the expected reason (404 — no `/measurement/*` routes registered yet), confirming the harness itself works correctly and isolates the real gap.

The Jest contract test (`measurement.controller.spec.ts`) and the backend Cucumber scenarios both exist and are complementary, consistent with the rule that contract/unit/integration tests may supplement but must not replace backend API Gherkin coverage for externally observable API behavior.

## Human Review Checklist

- [x] Implementation branches are correct (`agent/slice-12-measurement` in both frontend and backend)
- [x] Test files match approved requirements and scenarios (REQ-MEAS-005, 006, 007, 011, 014; the three approved Gherkin scenarios)
- [x] Tests fail for expected missing behavior only (verified: 404s / undefined-property errors, not unrelated errors)
- [x] No production implementation was added (confirmed: only `.feature`, `.steps.ts`, `.controller.spec.ts` files, and backend BDD test-tooling config/support files were created; `backend/package.json` gained only devDependencies + a `test:bdd` script)
- [x] Backend API Gherkin coverage is present or explicitly not applicable (see decision above — **required and created**: `backend/tests/features/measurement.feature` + `backend/tests/steps/measurement.steps.ts`, run and confirmed failing for the expected reason)
- [x] Approved for implementation
