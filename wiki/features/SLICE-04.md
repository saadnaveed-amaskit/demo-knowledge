# SLICE-04 — Guardrails management

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#5](https://github.com/saadnaveed-amaskit/demo-frontend/pull/5) and [#6](https://github.com/saadnaveed-amaskit/demo-frontend/pull/6) (`59d77c2`, `0d2143d`) — both merged under branch `agent/slice-04-guardrails`
- backend: [#4](https://github.com/saadnaveed-amaskit/demo-backend/pull/4) (`bda55a9`)

## Requirements Covered

REQ-GUARD-001…007.

## Acceptance Scenarios Covered

Inline-edit a guardrail threshold; non-overridable guardrails are enforced as hard constraints.

## Code Locations

- Frontend: `frontend/src/screens/guardrails/` (`GuardrailsScreen.tsx`, `guardrail-types.ts`, `guardrails-api.ts`)
- Backend: `backend/src/guardrails/` (`guardrails.controller.ts`, `guardrails.service.ts`)
- Contract: `knowledge/contracts/slice-04-guardrails/contract.md` (`Stable`)
- Tests: `frontend/features/guardrails-management.feature`, `frontend/tests/bdd/steps/guardrails-management.steps.ts`, `backend/src/guardrails/*.spec.ts`

## Frontend Notes

Form built with React Hook Form + Zod, matching the platform's pinned-stack form convention.

## Backend Notes

`GuardrailsService` is array-backed and seeded at startup; `evaluate()` computes pass/fail per guardrail with `hard`/`advisory` severity (`advisory` if `isOverridable`, else `hard`) — this severity/derivation is reused later by SLICE-07 (scenario guardrail checks) and SLICE-09 (approvals risk banners).

## API Contract Notes

Base path `/guardrails`; 7 endpoints (list, create, evaluate, patch, toggle active, toggle overridable, delete) under the `guardrails` tag in `backend/contracts/api-contract.md`.

## Tests

BDD: `guardrails-management.feature`. Backend: `guardrails.controller.spec.ts`, `guardrails.service.spec.ts` (seed data, create defaults, update, toggle active/overridable, remove + 404s, evaluate pass/fail with severity).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-04.md`: `status: Shipped` — 42 backend tests, 12 BDD scenarios, `npm run check` green on both repos (per `knowledge/log.md`).

## Known Limitations / Follow-ups

Spec Open Question 4 (screen-by-screen hard-block vs. warn behavior for guardrail violations) is referenced across later slices (e.g. SLICE-07, SLICE-09) rather than fully centralized here.
