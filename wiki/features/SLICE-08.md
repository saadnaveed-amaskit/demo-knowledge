# SLICE-08 — Deep Dive

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#10](https://github.com/saadnaveed-amaskit/demo-frontend/pull/10) (`bcedd79`)
- backend: [#8](https://github.com/saadnaveed-amaskit/demo-backend/pull/8) (`506d508`)

## Requirements Covered

REQ-DEEP-001…005.

## Acceptance Scenarios Covered

Explain a recommended price; progressive unlock of tiles by optimization level.

## Code Locations

- Frontend: `frontend/src/screens/price-scenarios/DeepDiveSection.tsx` (renders inside `PriceScenariosScreen.tsx`)
- Backend: `GET /price-scenarios/:id/deep-dive` in `backend/src/price-scenarios/price-scenarios.controller.ts` / `.service.ts`
- Contract: `knowledge/contracts/slice-08-deep-dive/contract.md` (`Stable`)
- Tests: `frontend/features/deep-dive.feature`, `frontend/tests/bdd/steps/deep-dive.steps.ts`

## Frontend Notes

`DeepDiveSection.tsx` is an AG Grid-based price-adjustments explorer with a row "Explain" modal and export, plus marketing/discount tiles that progressively unlock by optimization level (`UNLOCK_LEVELS = [0,20,40,60,80]`).

## Backend Notes

`getDeepDive()` requires the scenario to have been run first (`output` populated) — returns 404 both when the scenario id is unknown and when it exists but hasn't been run, distinguished only by message text. `priceAdjustments.length === min(20, skuCount)`; `marketingTiles`/`discountTiles` always have exactly 5 entries each.

## API Contract Notes

`GET /price-scenarios/{id}/deep-dive`, documented under the `price-scenarios` tag in `backend/contracts/api-contract.md`, returning `DeepDiveOutput` (`priceAdjustments`, `marketingTiles`, `discountTiles`).

## Tests

BDD: `deep-dive.feature` / `tests/bdd/steps/deep-dive.steps.ts`.

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-08.md`: `status: Passed` — 105 backend tests, 24/24 BDD scenarios, `npm run check` + build green on both repos; 1 patch attempt recorded (promotions ESLint fix + guardrails BDD timing fix).

## Known Limitations / Follow-ups

None recorded beyond the deterministic-placeholder-engine caveat inherited from SLICE-07.
