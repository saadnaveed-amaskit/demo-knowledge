---
status: In Review
artifact_type: slice-preparation
slice_id: SLICE-11
approved_by:
approved_at:
source_spec: knowledge/specs/001-platform-baseline/spec.md
source_plan: knowledge/specs/001-platform-baseline/plan.md
source_tasks: knowledge/specs/001-platform-baseline/tasks.md
frontend_branch: agent/slice-11-pricing-autonomy
backend_branch: agent/slice-11-pricing-autonomy
---

# Slice Preparation: `SLICE-11`

## Status

In Review

## Source Documents

- `knowledge/specs/platform-baseline-context.md`
- `knowledge/specs/001-platform-baseline/spec.md` (status: Approved) — REQ-AUTO-001…010, "Pricing Autonomy" Gherkin block
- `knowledge/specs/001-platform-baseline/plan.md` (status: Approved) — SLICE-11 section
- `knowledge/specs/001-platform-baseline/tasks.md` (status: Approved) — SLICE-11 section (Status: Approved at time of preparation)
- `knowledge/.specify/memory/constitution.md`
- `knowledge/TECHNOLOGY.md`
- `knowledge/DATA-TAXONOMY.md`
- `backend/contracts/api-contract.md` (read for existing conventions; no Autonomy entries exist yet — expected, this slice hasn't been implemented)

## Slice Scope

SLICE-11 — Pricing Autonomy. Trust ladder, promotion gates, live supervision (veto/undo), kill switch, audit. Target repos: frontend, backend. Depends on SLICE-10 (Agent roster), which is `Complete`. Requirements: REQ-AUTO-001…010. Acceptance scenarios: promotion gating (block below reversibility ceiling); kill-switch disables controls and freezes veto countdowns.

REQ-AUTO-010's canonical-version ambiguity (spec Open Q8: "Canonical version for Agents and Autonomy vs. the more-complete V2 fork") resolves the same way it did for SLICE-10 (Agent roster): the Pricing Platform V2 Agentic fork (`/pricing-platform-v2/*`) is not part of this workspace at all (only `knowledge/`, `frontend/`, `backend/` exist here), so there is no competing implementation to reconcile against — this implementation will be canonical by definition. Note: `tasks.md`'s SLICE-11 backend task text says "spec Open Q10" for kill-switch freeze semantics, but the actual numbered Open Question covering both canonical-version and kill-switch-freeze-semantics is **Open Question 8** (§10/§11) — Open Question 10 is about Marketing/Executive scope, unrelated. Flagging this as a minor drift in `tasks.md`'s own cross-reference; it does not block preparation since the two approved Gherkin scenarios are unambiguous about the externally observable behavior to test.

No implementation work was done in this command — no contract file, no production code, in either repo.

## Implementation Branches Created or Reused

| Repo | Branch | State before this command | Action |
|---|---|---|---|
| knowledge | 001-platform-baseline | clean, up to date | Reused (no new branch — per Speckit branch exception) |
| frontend | main (branched from) | clean, up to date, no `agent/slice-11-*` branch existed | Created `agent/slice-11-pricing-autonomy` |
| backend | main (branched from) | clean, up to date, no `agent/slice-11-*` branch existed | Created `agent/slice-11-pricing-autonomy` |

Both implementation branches were newly created (did not previously exist) — no risk of pre-existing unreviewed production code.

## Tests Created or Updated

- `frontend/features/pricing-autonomy.feature` — created. Two scenarios, reusing spec.md's approved Gherkin text verbatim.
- `frontend/tests/bdd/steps/pricing-autonomy.steps.ts` — created. Drives the anticipated `/autonomy` route and anticipated backend endpoints (`GET /autonomy/roster`, `POST /autonomy/kill-switch/engage`) documented inline as "anticipated backend surface — not yet implemented."
- `backend/src/autonomy/autonomy.controller.spec.ts` — created. References not-yet-existing `autonomy.controller.ts`, `autonomy.service.ts`, `autonomy-types.ts` (anticipated `ActionClassEntity`, `AutonomyRosterView`, `KillSwitchState` shapes) to exercise the promotion-gate-blocking and kill-switch-engage behavior ahead of implementation.

No production frontend or backend code, and no `knowledge/contracts/slice-11-autonomy/` contract file, were created — contract authoring is an implementation-phase task per `tasks.md`'s Backend Tasks, not part of this preparation command's allowed scope.

## Expected Failing Results

| Command | Repo | Result | Expected failure reason |
|---|---|---|---|
| `npx jest autonomy` | backend | Failed (TS2307 — cannot find `./autonomy.controller`, `./autonomy.service`, `./autonomy-types`) | Module does not exist yet — correct pre-implementation red |
| `npx jest` (full suite) | backend | 121/121 pre-existing tests passed; 1 new suite failed to compile | No regressions; only the new anticipated module fails |
| `node --import tsx cucumber.js --config cucumber.mjs features/pricing-autonomy.feature` (full suite run) | frontend | 30/32 scenarios passed, 2 failed (the 2 new ones) | "Block promotion..." fails because `GET /autonomy/roster` 404s (`roster.actionClasses` is `undefined`); "Kill switch disables controls" times out waiting for `live-action-row` testid, since `/autonomy` currently renders `ScreenPlaceholder` with no such element |

Both failures are for the correct reason (missing backend endpoint / missing frontend implementation), not for unrelated infrastructure issues. No pre-existing scenario regressed.

## Coverage Map

| Requirement | Scenario | Test file | Step definition | Expected pre-implementation result |
|---|---|---|---|---|
| REQ-AUTO-005, REQ-AUTO-009 | Block promotion below the reversibility ceiling | `frontend/features/pricing-autonomy.feature` | `frontend/tests/bdd/steps/pricing-autonomy.steps.ts` | Fail — `GET /autonomy/roster` does not exist (404) |
| REQ-AUTO-001, REQ-AUTO-008 | Kill switch disables controls | `frontend/features/pricing-autonomy.feature` | `frontend/tests/bdd/steps/pricing-autonomy.steps.ts` | Fail — `/autonomy` route renders `ScreenPlaceholder`, no `live-action-row`/`kill-switch-btn` elements exist |
| REQ-AUTO-005, REQ-AUTO-009 (backend) | Promotion gate blocks at reversibility ceiling | `backend/src/autonomy/autonomy.controller.spec.ts` | N/A (Jest, not Cucumber) | Fail to compile — `autonomy.controller.ts`/`.service.ts`/`autonomy-types.ts` do not exist |
| REQ-AUTO-001, REQ-AUTO-008 (backend) | Kill switch engage/disengage | `backend/src/autonomy/autonomy.controller.spec.ts` | N/A (Jest, not Cucumber) | Fail to compile — same as above |

REQ-AUTO-002, 003, 004, 006, 007, 010 (KPI strip, action-class cards + detail rail, demote, veto/undo, canonical-version/freeze-semantics) are covered by the same `autonomy.controller.spec.ts`'s `demote`/`getRoster` assertions and the contract/entity design implied by the two acceptance scenarios above; dedicated additional Gherkin scenarios for veto/undo and the detail rail were not written in this preparation pass since spec.md's approved Gherkin block for this slice only supplies the two scenarios above — adding further scenarios beyond the approved set would be inventing behavior, which this command must not do. If the human wants explicit BDD coverage for veto/undo/demote/KPI-strip beyond what's implied, that should be added via `/speckit-clarify` or an explicit spec update before implementation.

## Backend API Gherkin Decision

Backend API Gherkin scenarios (e.g. under `backend/tests/features/**`) were **not** created separately from the frontend BDD scenarios. Rationale: the two approved acceptance scenarios in spec.md are both framed as end-user UI interactions ("attempt to promote it", "engage the emergency kill switch"), and the existing repo convention (established in SLICE-02 through SLICE-10) drives backend behavior through the frontend Cucumber/Playwright suite rather than a separate backend-only Gherkin suite — no `backend/tests/features/**` directory exists anywhere in this repo for any prior slice. Backend-side coverage for this slice is instead provided by the Jest contract test (`autonomy.controller.spec.ts`), which exercises the anticipated controller directly (not through the frontend UI), satisfying "externally observable API behavior" coverage without introducing a new, repo-wide-inconsistent test-authoring convention. If the human wants a dedicated backend Gherkin suite for this slice specifically, that should be called out explicitly before implementation.

## Human Review Checklist

- [ ] Implementation branches are correct (`agent/slice-11-pricing-autonomy` in both frontend and backend)
- [ ] Test files match approved requirements and scenarios (REQ-AUTO-001, 005, 008, 009; the two approved Gherkin scenarios)
- [ ] Tests fail for expected missing behavior only (verified: 404/timeout/TS2307, not unrelated errors)
- [ ] No production implementation was added (confirmed: only `.feature`, `.steps.ts`, and `.controller.spec.ts` files created)
- [ ] Backend API Gherkin coverage is present or explicitly not applicable (see decision above — not applicable, contract test used instead, consistent with repo convention)
- [ ] Approved for implementation
