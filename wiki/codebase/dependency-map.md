# Dependency Map

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | a4e1d373afcda411ba6adcf8868e1aed258aa4b5 |
| frontend | main | 05d5371ef41b813ad369cd598d8ca982a40afa55 |
| backend | main | a18bdff033c9dac4a90c365c547446057da3ae9b |

## Frontend Dependencies

See the full table in [Frontend Structure](frontend-structure.md#dependencies). Key production dependencies: `react`/`react-dom` (^19.0.0), `react-router-dom` (^7.1.1), `zustand` (^5.0.2), `nuqs` (^2.4.0), `react-hook-form`/`zod`/`@hookform/resolvers` (forms/validation), `framer-motion` (^12.0.0), `@nivo/core`/`@nivo/line`/`@nivo/bar` (charts), `ag-grid-community`/`ag-grid-react` (^33.0.0), `date-fns` (^4.1.0), `@radix-ui/*` + `class-variance-authority` + `clsx` + `tailwind-merge` (shadcn/ui primitives and styling utilities), `lucide-react` (icons).

## Backend Dependencies

See the full table in [Backend Structure](backend-structure.md#dependencies). Key production dependencies: `@nestjs/common`/`@nestjs/core`/`@nestjs/platform-express` (^11.0.0), `reflect-metadata` (^0.2.2), `rxjs` (^7.8.1). No database driver or ORM package is present in either dependencies or devDependencies.

## Internal Dependencies

| From | To | Evidence |
|---|---|---|
| backend: PriceScenariosService | FocusSetsService | Constructor injection in `price-scenarios.service.ts` |
| backend: PriceScenariosService | GuardrailsService | Constructor injection in `price-scenarios.service.ts` (used to build `guardrailResults`) |
| backend: ApprovalsService | PriceScenariosService | Constructor injection in `approvals.service.ts` (queue/decided aggregation, decision routing) |
| backend: ApprovalsService | DiscountModelingService | Constructor injection in `approvals.service.ts` (queue/decided aggregation, decision routing) |
| frontend: screens/approvals/approvals-api.ts | backend `/approvals/*` | `fetch` calls to `queue`, `decided`, `scenarios/:id/review`, `discounts/:id/review`, `scenarios/:id/decision`, `discounts/:id/decision` |
| frontend: screens/price-scenarios/scenarios-api.ts | backend `/price-scenarios/*` | `fetch` calls to list/create/run/submit/delete/deep-dive endpoints |
| frontend: screens/discount-modeling/discount-modeling-api.ts | backend `/discount-models/*` | `fetch` calls to list/create/run/submit/delete endpoints |
| frontend: screens/discount-modeling/focus-sets-ref-api.ts | backend `/focus-sets` | `fetch` calls used to populate the focus-group selector in the discount-modeling form |
| frontend: screens/agents/agents-api.ts | backend `/agents/*` | `fetch` calls to `roster`, `catalog`, `monitors/:id/pause`, `monitors/:id/resume`, `hire`, `task-agents/:id/retire` |
| frontend: all screens | backend base URL | `import.meta.env.VITE_API_URL ?? "http://localhost:3000"`, per-screen fetch-wrapper module |

Note: `AgentsService` (SLICE-10) has no constructor dependencies on other backend services — it seeds its own monitors/operators/task-agents in-memory, unlike `PriceScenariosService`/`ApprovalsService`.

## Frontend ↔ Backend Boundary

The frontend has no shared HTTP client or React Query layer — each screen folder owns a small, hand-written `fetch`-wrapper module (e.g. `approvals-api.ts`, `scenarios-api.ts`) that calls the backend directly and throws a plain `Error` on a non-OK response. There is no generated API client; frontend TypeScript types for request/response shapes (e.g. `frontend/src/screens/approvals/approval-types.ts`) are hand-maintained to mirror the backend's types, not generated from `backend/contracts/api-contract.md` or `api-contract.yaml`.

## Contract Dependencies

- **Canonical contract:** `backend/contracts/api-contract.md` (53 endpoints, 64 schemas) — see [Contract Summary](../api/contract-summary.md).
- **`backend/contracts/api-contract.yaml`** (OpenAPI 3.0.3 structured version) existed at one point but was deleted from `main` in backend commit `bb1a951` — it is not present in the current tree. SLICE-10's Agents endpoints/schemas were hand-authored directly into the Markdown for this reason.
- **Per-slice contract references** under `knowledge/contracts/<slice-id>/contract.md` exist for SLICE-02 through SLICE-10 (9 folders, all `Stable`).

## Known Gaps

- Fetch-wrapper filenames for the remaining pre-SLICE-09 screens (focus-builder, guardrails, product-grid, promotions) were not individually re-verified in this pass; they follow the same per-screen `*-api.ts` convention confirmed for price-scenarios, discount-modeling, and approvals.
- No dependency-graph tool output was available; the Internal Dependencies table above is derived from direct constructor-injection and `fetch`-call evidence only, not a generated import graph.
