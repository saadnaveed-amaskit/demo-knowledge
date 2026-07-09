---
slice_id: SLICE-05
status: In Review
validated_at: 2026-07-09
---

# Validation Report — SLICE-05: Promotions Calendar

## Repository State

| Repo       | Branch                       | Commit SHA                               | State |
| ---------- | ---------------------------- | ---------------------------------------- | ----- |
| knowledge  | 001-platform-baseline        | 29902cabe225b84ca196738487f2dea848118392 | clean |
| frontend   | agent/slice-05-promotions    | 0f354dbc6e4fa7af8654eedf12b32975535ac184 | clean |
| backend    | agent/slice-05-promotions    | 159965bdb4a39d09bb59ab4b72542f104ad121a6 | clean |

## Artifact Paths

| Artifact        | Path                                                     |
| --------------- | -------------------------------------------------------- |
| Platform spec   | knowledge/specs/001-platform-baseline/spec.md            |
| Platform plan   | knowledge/specs/001-platform-baseline/plan.md            |
| Platform tasks  | knowledge/specs/001-platform-baseline/tasks.md           |
| Contract        | knowledge/contracts/slice-05-promotions/contract.md      |

## Commands Run

| Repo     | Command              | Result  |
| -------- | -------------------- | ------- |
| backend  | npm run test         | Passed  |
| frontend | npm run check        | Passed  |
| frontend | npm run build        | Passed  |
| frontend | npm run test:bdd     | Passed  |

## Backend Test Results

```
Test Suites: 9 passed, 9 total
Tests:       59 passed, 59 total
```

New suites added:
- `src/promotions/promotions.service.spec.ts` — 9 tests
- `src/promotions/promotions.controller.spec.ts` — 7 tests

## Frontend Check Results

```
npm run lint       ✓ (1 pre-existing warning on button.tsx — not in SLICE-05 scope)
npm run lint:policy ✓ pinned-stack compliance verified
npm run typecheck  ✓
npm run build      ✓ built in 15s
```

## BDD Results

```
15 scenarios (15 passed)
58 steps (58 passed)
0m22.374s
```

New scenarios added (promotions-calendar.feature):
1. Prevent an end date before the start date — PASSED
2. Filter promotions by status with counts — PASSED
3. View per-SKU promo price — PASSED

All pre-existing scenarios (focus-set, product-grid, guardrails) continued to pass.

## Requirements Coverage

| Requirement | Scenario / Test                                        | Result | Notes                                                |
| ----------- | ------------------------------------------------------ | ------ | ---------------------------------------------------- |
| REQ-PROMO-1 | Prevent an end date before the start date              | Passed | Zod refinement + backend BadRequestException         |
| REQ-PROMO-2 | Filter promotions by status with counts                | Passed | Active/Scheduled/Expired tabs with live counts       |
| REQ-PROMO-3 | View per-SKU promo price                               | Passed | PromoDetail drawer → View Products → promo-price cells |
| REQ-PROMO-4 | promotions.service.spec: create with status derivation | Passed | active / scheduled / expired from date comparison    |
| REQ-PROMO-5 | promotions.service.spec: BadRequestException on bad dates | Passed | endDate <= startDate throws                        |
| REQ-PROMO-6 | promotions.service.spec: getProducts with focus set    | Passed | ≤ 20 SKUs, promo prices computed                    |

## Pinned-Stack Compliance

| Library        | Used               | Compliant |
| -------------- | ------------------ | --------- |
| shadcn/ui      | Button, Input      | Yes       |
| React Hook Form| PromoForm          | Yes       |
| Zod            | promoSchema + refine | Yes     |
| date-fns       | Calendar utilities | Yes       |
| Framer Motion  | Not needed for SLICE-05 | N/A  |
| Nivo           | Not needed for SLICE-05 | N/A  |

## Data Taxonomy Compliance

| Entity / View          | Tier | File                                           | Compliant |
| ---------------------- | ---- | ---------------------------------------------- | --------- |
| PromotionEntity        | 1    | backend/src/promotions/promotion-types.ts      | Yes       |
| CreatePromotionDto     | 2    | backend/src/promotions/promotion-types.ts      | Yes       |
| UpdatePromotionDto     | 2    | backend/src/promotions/promotion-types.ts      | Yes       |
| PromoProductRow        | 2    | backend/src/promotions/promotion-types.ts      | Yes       |
| PromoProductsView      | 2    | backend/src/promotions/promotion-types.ts      | Yes       |
| PromoStatusFilter      | 2    | frontend/src/screens/promotions/promotion-types.ts | Yes   |

## Naming & Structure Compliance

- UI file: `PromotionsScreen.tsx` (PascalCase) ✓
- Non-UI: `promotions-api.ts`, `promotion-types.ts` (kebab-case) ✓
- Directories: `src/screens/promotions/` (kebab-case) ✓
- No generic names used ✓

## Design System Compliance

- Zinc grays + teal accent used for status badges ✓
- Lucide icons (Plus, X, ChevronLeft, ChevronRight, Calendar, List) ✓
- shadcn Button / Input used ✓

## Mismatches Found

None.

## Patch Attempts

None required.

## Remaining Blockers

None. Ready for PR review.
