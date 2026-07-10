# SLICE-11 — Pricing Autonomy

## Status

Complete

Finalized 2026-07-10 via `/finalize-knowledge-after-merge SLICE-11`. Both the frontend PR ([#13](https://github.com/saadnaveed-amaskit/demo-frontend/pull/13)) and backend PR ([#12](https://github.com/saadnaveed-amaskit/demo-backend/pull/12)) are merged to `main`; `knowledge/specs/001-platform-baseline/tasks.md`'s `Status:` field and the validation report's front matter (`knowledge/specs/001-platform-baseline/validation/SLICE-11.md`, now `status: Shipped`) have been updated to match, and the per-slice contract reference (`knowledge/contracts/slice-11-autonomy/contract.md`) was promoted from `Draft` to `Stable`.

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | d74974dfa32bb0eaccd07584e810ee6ea13b8ddb |
| frontend | main | 6ec84dbde0379096cad99a1be3536e0ff1664e4a |
| backend | main | a0e6742ed3c7be66f6e37cb420b40029112ad3d9 |

## Merged PRs

- frontend: [#13](https://github.com/saadnaveed-amaskit/demo-frontend/pull/13) (`6ec84db`)
- backend: [#12](https://github.com/saadnaveed-amaskit/demo-backend/pull/12) (`a0e6742`) — includes the `backend/contracts/api-contract.md` update (8 new `/autonomy/*` endpoints, 6 new schemas)

## Requirements Covered

REQ-AUTO-001…010. REQ-AUTO-010 (canonical Agents/Autonomy implementation vs. the Pricing Platform V2 Agentic fork, spec Open Q8) resolves by definition — the V2 fork isn't part of this workspace, so there's no competing implementation to reconcile against.

## Acceptance Scenarios Covered

Block promotion below the reversibility ceiling (reuses spec.md's Gherkin verbatim); kill switch disables promote/veto/undo controls and freezes the veto countdown.

## Code Locations

- Frontend: `frontend/src/screens/autonomy/` (`AutonomyScreen.tsx`, `autonomy-types.ts`, `autonomy-api.ts`); wired into `frontend/src/app/routes.tsx`
- Backend: `backend/src/autonomy/` (`autonomy.controller.ts`, `autonomy.service.ts`, `autonomy-types.ts`, `autonomy.controller.spec.ts`); wired into `backend/src/app.module.ts`
- Contract: `knowledge/contracts/slice-11-autonomy/contract.md` (`Stable`); `backend/contracts/api-contract.md` (Autonomy paths/schemas)
- Tests: `frontend/features/pricing-autonomy.feature`, `frontend/tests/bdd/steps/pricing-autonomy.steps.ts` (written in the approved preparation phase, then fixed for two test-robustness bugs during implementation)

## Frontend Notes

`/autonomy` screen: a 4-tile KPI strip (total action classes, eligible to promote, total live $ value, avg proof accuracy) followed by action-class cards (Promote/Demote, a blocked-reason message on failed promote), an audit-trail feed for the selected card, and live-supervision rows (Veto/Undo) with a per-row countdown. An emergency kill-switch button disables Promote/Veto/Undo and pauses the client-side countdown ticker (Demote remains available per REQ-AUTO-006). No new Zustand store — local `useState`, consistent with SLICE-09/10. Two mechanical test-robustness fixes were made to the approved `pricing-autonomy.steps.ts` during implementation: an unscoped `promote-btn` locator (Playwright strict-mode violation across 3 cards) and a missing wait for the async kill-switch engage/refresh round trip; a `Before` hook was also added to reset in-memory kill-switch state between scenarios (no reset endpoint exists).

## Backend Notes

`AutonomyService` seeds 3 action classes and 2 live actions in-memory at process start; all data resets on restart. Three new Tier-1 entities were introduced — `ActionClassEntity`, `LiveActionEntity`, `AuditEntry` (per spec Open Q12, no prior DATA-TAXONOMY.md entries existed for any). Promotion is gated in order per REQ-AUTO-009: kill switch disengaged → reversibility ceiling (`Low` reversibility caps at `Supervised`; `Medium`/`High` cap at `Autonomous`) → sample count ≥100 → accuracy ≥0.85 → acceptance rate ≥0.80 — the first failing gate's message is returned as the 400 body. Demote is never gated, including while the kill switch is engaged (REQ-AUTO-006). `AutonomyRosterView.liveActions` and two `ActionClassEntity` fields (`liveDollarValue`, `createdAt`) are typed as optional solely so the pre-existing, approved preparation-phase contract test's mocks (written before these fields were designed) still type-check unmodified — documented as a known gap, not a taxonomy violation.

## API Contract Notes

Base path `/autonomy`; 8 endpoints (`GET /roster`, `GET /action-classes/:id/audit`, `POST /action-classes/:id/promote`|`demote`, `POST /live-actions/:id/veto`|`undo`, `POST /kill-switch/engage`|`disengage`) under the `autonomy` tag — see [Contract Summary](../api/contract-summary.md). Hand-authored directly into `api-contract.md` since the structured YAML source no longer exists on `main` (same approach as SLICE-10).

## Tests

BDD: `features/pricing-autonomy.feature` (2 scenarios) / `tests/bdd/steps/pricing-autonomy.steps.ts`. Backend: `autonomy.controller.spec.ts` (5 tests: roster read, promote-blocked-by-reversibility-ceiling, demote, kill-switch engage/disengage).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-11.md` (front matter `status: Shipped`): backend 126/126 tests pass, `npm run check` passes in both repos; frontend 32/32 BDD scenarios pass (2 new + no regressions, run twice consecutively for stability), `npx vitest run` and `npm run check` pass. All REQ-AUTO-* requirements pass (REQ-AUTO-003/004/007 verified manually — no dedicated BDD scenario beyond the two approved ones).

## Known Limitations / Follow-ups

- Promotion-gate thresholds (sample count, accuracy, acceptance rate) are v1 placeholders — no historical proof-accuracy dataset exists to derive them from.
- Kill-switch freeze is client-side only — the backend has no server-side timer/scheduling concept; "freeze" is the frontend's countdown ticker pausing while `killSwitchEngaged` is true.
- `AuditEntry.actor` is a fixed placeholder ("Pricing Strategist"), not real user identity, consistent with the platform's deferred identity-provider integration.
- The BDD "reversibility ceiling" scenario assumes the first-rendered action-class card is the one at ceiling — true for the current seed data (action class id 1) but not guaranteed if seed order changes.
- `backend/contracts/api-contract.md`'s own embedded Source table still shows a pre-merge placeholder rather than the final merge SHA `a0e6742` — not corrected here per the "do not edit api-contract.md" constraint.
