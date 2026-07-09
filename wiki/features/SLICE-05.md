# SLICE-05 — Promotions calendar

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#7](https://github.com/saadnaveed-amaskit/demo-frontend/pull/7) (`30ebbcc`)
- backend: [#5](https://github.com/saadnaveed-amaskit/demo-backend/pull/5) (`1ec5787`)

## Requirements Covered

REQ-PROMO-001…008.

## Acceptance Scenarios Covered

Prevent an end date before the start date; filter by status with counts; view per-SKU promo price.

## Code Locations

- Frontend: `frontend/src/screens/promotions/` (`PromotionsScreen.tsx`, `promotion-types.ts`, `promotions-api.ts`)
- Backend: `backend/src/promotions/` (`promotions.controller.ts`, `promotions.service.ts`)
- Contract: `knowledge/contracts/slice-05-promotions/contract.md` (`Stable`)
- Tests: `frontend/features/promotions-calendar.feature`, `frontend/tests/bdd/steps/promotions-calendar.steps.ts`, `backend/src/promotions/*.spec.ts`

## Frontend Notes

Calendar + list view for time-boxed promotions; uses `date-fns` for date handling, RHF + Zod for the create/edit form.

## Backend Notes

`PromotionsService` derives `status` (`scheduled`/`active`/`expired`) on every read from `startDate`/`endDate` vs. today's date, rather than storing it; `getProducts` caps results at the first 20 matching SKUs.

## API Contract Notes

Base path `/promotions`; 5 endpoints (list, create, get products, patch, delete) under the `promotions` tag in `backend/contracts/api-contract.md`.

## Tests

BDD: `promotions-calendar.feature`. Backend: `promotions.controller.spec.ts`, `promotions.service.spec.ts` (status derivation, date validation, update, 404s, remove, getProducts 404s).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-05.md`: `status: Shipped`.

## Known Limitations / Follow-ups

Spec Open Question 2 (canonical Promotion/Discount entity reconciling three fragmented shapes) is noted as "Yes, reconcile" but the reconciliation itself is tracked against SLICE-05/SLICE-06 collectively rather than fully completed within this slice alone — see `knowledge/specs/001-platform-baseline/plan.md`.
