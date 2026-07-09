# SLICE-10 — Agent roster

## Status

Complete

Finalized 2026-07-09 via `/finalize-knowledge-after-merge SLICE-10`. Both the frontend PR ([#12](https://github.com/saadnaveed-amaskit/demo-frontend/pull/12)) and backend PR ([#11](https://github.com/saadnaveed-amaskit/demo-backend/pull/11)) are merged to `main`; `knowledge/specs/001-platform-baseline/tasks.md`'s `Status:` field and the validation report's front matter (`knowledge/specs/001-platform-baseline/validation/SLICE-10.md`, now `status: Shipped`) have been updated to match, and the per-slice contract reference (`knowledge/contracts/slice-10-agents/contract.md`) was promoted from `Draft` to `Stable`.

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | a4e1d373afcda411ba6adcf8868e1aed258aa4b5 |
| frontend | main | 05d5371ef41b813ad369cd598d8ca982a40afa55 |
| backend | main | a18bdff033c9dac4a90c365c547446057da3ae9b |

## Merged PRs

- frontend: [#12](https://github.com/saadnaveed-amaskit/demo-frontend/pull/12) (`05d5371`)
- backend: [#11](https://github.com/saadnaveed-amaskit/demo-backend/pull/11) (`a18bdff`) — includes the `backend/contracts/api-contract.md` update (6 new `/agents/*` endpoints, 7 new schemas)

## Requirements Covered

REQ-AGENT-001…007. REQ-AGENT-007 (one canonical Agents implementation vs. the Pricing Platform V2 Agentic fork, spec Open Q8) resolves by definition — the V2 fork isn't part of this workspace at all, so there's no competing implementation to reconcile against.

## Acceptance Scenarios Covered

Pause and resume a monitor (reuses spec.md's Gherkin verbatim); hire a standing agent from the catalog.

## Code Locations

- Frontend: `frontend/src/screens/agents/` (`AgentsScreen.tsx`, `agent-types.ts`, `agents-api.ts`); wired into `frontend/src/app/routes.tsx`
- Backend: `backend/src/agents/` (`agents.controller.ts`, `agents.service.ts`, `agent-types.ts`, `agents.controller.spec.ts`); wired into `backend/src/app.module.ts`
- Contract: `knowledge/contracts/slice-10-agents/contract.md` (`Stable`); `backend/contracts/api-contract.md` (Agents paths/schemas)
- Tests: `frontend/features/agent-roster.feature`, `frontend/tests/bdd/steps/agent-roster.steps.ts`

## Frontend Notes

`/agents` screen: a 5-tile KPI strip (agents on team, signals today, acting autonomously, evidence-backed X/Y, task agents running) followed by three card sections — Monitors, Operators, Task Agents. Monitor cards toggle pause/resume via a single button whose label flips ("Pause"/"Resume"). Operator cards are clickable and navigate to `/autonomy` (a `ScreenPlaceholder` today, since SLICE-11 doesn't exist yet). Task-agent cards have an "Open" button (navigates to `openLink`, e.g. `/scenario`) and a "Retire" button that disappears once already retired. A "Hire Agent" toggle reveals a kind/subtype picker (plain `<select>` elements, consistent with the `focus-group-select` precedent) that calls the hire endpoint and refreshes the roster. No new Zustand store — local `useState` only, matching the established per-screen pattern.

## Backend Notes

`AgentsService` seeds 3 monitors, 2 operators, and 2 task agents in-memory at process start; all data resets on restart. Two new Tier-1 entities were introduced — `MonitorEntity` and `TaskAgentEntity` — while `OperatorView` is explicitly Tier-2/derived per `plan.md` ("Operators are derived"), since no Autonomy system (SLICE-11) exists yet to supply real trust-ladder data; hired operators always get placeholder `trustLevel: "Low"`/`evidenceStatus: "unproven"` regardless of subtype. `POST /agents/hire`'s `kind` field has no runtime enum validation — the service branches on `dto.kind === "monitor"` with an implicit `else` for anything else (including typos), so an invalid `kind` string silently hires an operator. Pause/resume/retire are all idempotent (no error on redundant calls). Task agents have no creation endpoint in v1 — they're seeded, not spawned by real platform events.

## API Contract Notes

Base path `/agents`; 6 endpoints (`GET /roster`, `GET /catalog`, `POST /monitors/:id/pause`, `POST /monitors/:id/resume`, `POST /hire`, `POST /task-agents/:id/retire`) under the `agents` tag — see [Contract Summary](../api/contract-summary.md). Because `backend/contracts/api-contract.yaml` no longer exists on `main` (deleted in an earlier commit), these entries were hand-authored directly into `api-contract.md` rather than generated from YAML.

## Tests

BDD: `features/agent-roster.feature` (2 scenarios) / `tests/bdd/steps/agent-roster.steps.ts`. Backend: `agents.controller.spec.ts` (8 tests: roster/catalog reads, pause/resume/hire/retire delegation, and a dedicated `describe` block asserting `BadRequestException` for an invalid hire subtype and `NotFoundException` for an unknown monitor id).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-10.md` (front matter `status: Shipped`): backend 121/121 tests pass, `npm run check` passes in both repos; frontend 30/30 BDD scenarios pass (2 new + no regressions), `npx vitest run` and `npm run check` pass. All REQ-AGENT-* requirements pass (REQ-AGENT-004/005 verified manually — no dedicated BDD scenario for operator/task-agent card behavior yet).

## Known Limitations / Follow-ups

- Operator `trustLevel`/`evidenceStatus`/`trackRecord` are deterministic placeholders pending SLICE-11 (Pricing Autonomy), which would supply real trust-ladder data.
- Task agents are seeded, not spawned by live platform events (e.g. a scenario submission) — a future slice wiring real spawning should not need to change this contract's read/retire shape.
- REQ-AGENT-004's "navigate to Autonomy" click currently lands on a `ScreenPlaceholder` until SLICE-11 is implemented — expected, not a defect.
- `backend/contracts/api-contract.md`'s own embedded Source table still shows the placeholder text "pending commit on this branch" rather than the final merge SHA `a18bdff` — not corrected here per the "do not edit api-contract.md" constraint on this finalization command.
