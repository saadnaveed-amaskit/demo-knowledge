# SLICE-03 — Product Grid

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#4](https://github.com/saadnaveed-amaskit/demo-frontend/pull/4) (`36d559c`)
- backend: [#3](https://github.com/saadnaveed-amaskit/demo-backend/pull/3) (`67df4a3`)

## Requirements Covered

REQ-GRID-001…007.

## Acceptance Scenarios Covered

Soft-delete and restore a SKU (or all SKUs); toggle between product and SKU grid views.

## Code Locations

- Frontend: `frontend/src/screens/product-grid/` (`ProductGrid.tsx`, `product-grid-types.ts`, `product-grid-api.ts`)
- Backend: `backend/src/product-grid/` (`product-grid.controller.ts`, `product-grid.service.ts`)
- Contract: `knowledge/contracts/slice-03-product-grid/contract.md` (`Stable`)
- Tests: `frontend/features/product-grid.feature`, `frontend/tests/bdd/steps/product-grid.steps.ts`, `backend/src/product-grid/*.spec.ts`

## Frontend Notes

Uses AG Grid to browse/curate SKUs within a Focus Set; routed but not listed in the sidebar (`UNLISTED_SCREENS` in `nav.ts`).

## Backend Notes

`ProductGridService` builds grid/product/SKU views on read and tracks per-focus-set SKU exclusions in a `Map<string, Set<string>>` (session-local, not persisted).

## API Contract Notes

Base path `/product-grid`; 4 endpoints (get grid, exclude a SKU, restore one SKU, restore all SKUs) under the `product-grid` tag in `backend/contracts/api-contract.md`.

## Tests

BDD: `product-grid.feature`. Backend: `product-grid.controller.spec.ts`, `product-grid.service.spec.ts` (grid grouping, 404 for unknown focus set, exclude/restore/restoreAll semantics).

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-03.md`: `status: Shipped` (validated 2026-07-09).

## Known Limitations / Follow-ups

Per `backend/contracts/api-contract.md`, the single-SKU and all-SKU restore endpoints silently no-op on an unknown focus set/SKU id rather than returning 404, which is inconsistent with the rest of the API's delete/removal endpoints — a documented backend quirk, not yet reconciled.
