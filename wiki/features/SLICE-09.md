# SLICE-09 — Approvals queue

## Status

Complete

Finalized 2026-07-09 via `/finalize-knowledge-after-merge SLICE-09`. Both the frontend PR ([#11](https://github.com/saadnaveed-amaskit/demo-frontend/pull/11)) and backend PR ([#9](https://github.com/saadnaveed-amaskit/demo-backend/pull/9)) are merged to `main`; `knowledge/specs/001-platform-baseline/tasks.md`'s `Status:` field and the validation report's front matter (`knowledge/specs/001-platform-baseline/validation/SLICE-09.md`, now `status: Shipped`) have been updated to match, and the per-slice contract reference (`knowledge/contracts/slice-09-approvals/contract.md`) was promoted from `Draft` to `Stable`.

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#11](https://github.com/saadnaveed-amaskit/demo-frontend/pull/11) (`d234cdb`)
- backend: [#9](https://github.com/saadnaveed-amaskit/demo-backend/pull/9) (`a6df300`) — includes commit `f192964` (Approvals endpoints added to `api-contract.yaml`) and `3abcf7c` (generated `api-contract.md`); a separate, unrelated PR ([#10](https://github.com/saadnaveed-amaskit/demo-backend/pull/10), branch `agent/slice-09-baseline-update`) merged the original reverse-engineered `api-contract.yaml` baseline just before this slice's own commits landed.

## Requirements Covered

REQ-APPR-001…012. REQ-APPR-007's wording was updated 2026-07-09 (was "optional" comment for scenario request-changes; now mandatory) to resolve spec Open Question 5. **REQ-APPR-005 and REQ-APPR-008 are explicitly deferred** — see Known Limitations.

## Acceptance Scenarios Covered

Deny requires a reason; returning a discount model for changes requires a comment; decided items move to a separate table; viewing a pending discount model opens its review drawer with a risk banner.

## Code Locations

- Frontend: `frontend/src/screens/approvals/` (`ApprovalsScreen.tsx`, `approval-types.ts`, `approvals-api.ts`); wired into `frontend/src/app/routes.tsx`
- Backend: `backend/src/approvals/` (`approvals.controller.ts`, `approvals.service.ts`, `approval-types.ts`, `approvals.controller.spec.ts`); wired into `backend/src/app.module.ts`
- Contract: `knowledge/contracts/slice-09-approvals/contract.md` (`Stable`); `backend/contracts/api-contract.md` (Approvals paths/schemas); `backend/contracts/api-contract.yaml` — **no longer present on `main`** (deleted in commit `bb1a951`)
- Tests: `frontend/features/approvals-queue.feature`, `frontend/tests/bdd/steps/approvals-queue.steps.ts`

## Frontend Notes

Two-tab queue (Price Scenarios / Discounts) with pending-count badges, risk badges, and Approve/Deny/Request Changes actions. Deny and Request Changes both require a non-empty reason/comment, enforced client-side (disabled confirm button) and server-side. Decided items render in a read-only table. Scenario review reuses `ScenarioOutput` data (narrative, comparison table, guardrail results) rather than reusing the `PriceScenariosScreen` component instance directly. Discount review shows a risk banner (hard/advisory counts), impact, and constraint warnings — deliberately without a guardrail list or SKU breakdown, per REQ-APPR-009. No shadcn `Sheet`/`Dialog` existed in the repo at implementation time; review "drawers" are in-panel view swaps, not true overlays.

## Backend Notes

`ApprovalsService` has no entity store of its own — it aggregates `PriceScenariosService.findAll()` and `DiscountModelingService.findAll()` on every read, filtering by status. `submitter`/`team`/`brand`/`division` are v1 deterministic placeholders (a 3-entry roster rotated by `id % 3`), not real submitter identity. Risk is derived per REQ-APPR-011: discount risk from `RiskPanel.isHard`/severity; scenario risk falls back to the existing "Inventory Risk" comparison-row value (no dedicated $ inventory-risk-reduction metric exists in the v1 placeholder engine). Because `DiscountModelEntity` has no `changeRequests` field (unlike `ScenarioEntity`), discount change-request history is derived from `ApprovalsService`'s own in-memory decision log rather than the underlying entity — this log is lost on process restart. The `action` field on `DecisionDto` has zero runtime enum validation; any value other than exactly `"approve"` or `"deny"` (including a typo) maps to status `"returned"`.

## API Contract Notes

Base path `/approvals`; 6 endpoints (`/queue`, `/decided`, `/scenarios/:id/review`, `/discounts/:id/review`, `/scenarios/:id/decision`, `/discounts/:id/decision`) under the `approvals` tag — see [Contract Summary](../api/contract-summary.md). Comment validation runs *before* the underlying id lookup, so a missing comment combined with an invalid id yields 400, not 404 (documented in `backend/contracts/api-contract.md`).

## Tests

BDD: `features/approvals-queue.feature` (4 scenarios) / `tests/bdd/steps/approvals-queue.steps.ts`. Backend: `approvals.controller.spec.ts` (8 tests: queue/decided aggregation, review delegation, decision routing, and a dedicated `describe` block asserting `BadRequestException` when deny/request_changes is missing a comment).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-09.md` (front matter `status: Shipped`): backend 113/113 tests pass, `npm run check` passes in both repos; frontend 28/28 BDD scenarios pass (4 new + no regressions), `npx vitest run` and `npm run check` pass. REQ-APPR-001…004, 006, 007, 009…012 pass; REQ-APPR-005/008 are recorded as **deferred**, not failing.

## Known Limitations / Follow-ups

- **REQ-APPR-005 / REQ-APPR-008 deferred**: no `BusinessIssueEntity`/action-tracker subsystem exists anywhere in the codebase, and spec Open Question 9 (closed-loop automation scope) remains unresolved. A follow-up slice is recommended once that entity design is settled.
- `submitter`/`team`/`brand`/`division` remain v1 placeholders pending real identity capture (out of scope per spec's identity-provider note).
- `backend/contracts/api-contract.yaml` was deleted from `main` after this slice's work added Approvals entries to it — only `api-contract.md` currently exists.
