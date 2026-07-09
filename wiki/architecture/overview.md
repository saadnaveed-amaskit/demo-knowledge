# Architecture Overview

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## System Summary

Retail Nucleus is a pricing-intelligence platform for a retail organization, covering Focus Set curation, product-grid management, guardrail constraints, promotions, discount modeling, price scenario optimization, deep-dive analysis, and an approvals workflow. The platform is a three-repo workspace: `knowledge` (specs/plans/tasks/contracts references/validation/wiki), `frontend` (React SPA), and `backend` (NestJS API). Governance follows an approval-gated Speckit workflow (spec → plan → tasks → implementation) recorded in `knowledge/.specify/memory/constitution.md`. The platform baseline is tracked as feature `001-platform-baseline`, broken into 15 implementation slices (SLICE-00 through SLICE-14); 10 are `Complete` (SLICE-00…09), and 5 (SLICE-10…14) are `Approved` but not yet implemented.

## Repository Responsibilities

| Repo | Responsibility | Notes |
|---|---|---|
| knowledge | Specs, plans, tasks, validation reports, per-slice contract references (`knowledge/contracts/<slice>/contract.md`), constitution/technology/data-taxonomy governance docs, wiki | Source of truth for requirements and governance; does not own the canonical API contract file |
| frontend | React 19 + TypeScript + Vite SPA; Cucumber/Playwright BDD tests; Vitest unit tests | Consumes the backend API via per-screen fetch-wrapper modules |
| backend | NestJS 11 REST API; Jest unit/controller tests; owns the canonical API contract | All persistence is in-memory (no ORM/database) — see [Backend Structure](../codebase/backend-structure.md) |

## Frontend Overview

Pinned stack (per `knowledge/TECHNOLOGY.md` and `knowledge/.specify/memory/constitution.md`): React 19, TypeScript, Vite, Tailwind CSS + shadcn/ui, Framer Motion, React Router v7, Zustand (UI state) + nuqs (URL state), React Hook Form + Zod, AG Grid (large tables), Nivo (charts), date-fns. The app is a single-page app with one router (`src/app/routes.tsx`) built from a persona-grouped nav model (`src/app/nav.ts`: Pricing Team, Pricing Strategist, Admin). One screen folder exists per feature slice under `src/screens/`. See [Frontend Structure](../codebase/frontend-structure.md) for the full directory, route, and test inventory.

## Backend Overview

NestJS 11 API (`@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express`). One flat feature module per slice under `src/` (catalog, focus-sets, product-grid, guardrails, promotions, discount-modeling, price-scenarios, approvals), plus `health` and a `shared/in-memory-repository.ts` scaffold. **No ORM or database is used anywhere** — every module holds its records in an in-memory array or `Map`, seeded at process start and reset on restart; this was verified directly in source (e.g. `private store: ScenarioEntity[] = []` in `price-scenarios.service.ts`, `private readonly store = new Map<string, FocusSetEntity>()` in `focus-sets.service.ts`). No global route prefix or global `ValidationPipe` is registered (`src/main.ts`); most request bodies are plain TypeScript interfaces with no runtime shape validation beyond what individual services check explicitly. See [Backend Structure](../codebase/backend-structure.md) for the full module, route, and test inventory.

## API / Contract Boundary

The canonical, human-readable API contract is `backend/contracts/api-contract.md` (47 endpoints, 57 schemas, reverse-engineered from the actual `main` branch source and tests, not from design docs). **`backend/contracts/api-contract.yaml`, an OpenAPI 3.0.3 structured version of the same contract, existed at one point on this branch but was deleted from `main` in backend commit `bb1a951` ("deleted redundant file") prior to the SLICE-09 merge — it does not exist in the current backend tree.** Per-slice contract references also exist under `knowledge/contracts/<slice-id>/contract.md` (8 folders, `slice-02` through `slice-09`; `slice-09-approvals` is `Draft`, the rest are `Stable`) — these are supporting/design-time references, not the canonical contract. See [Contract Summary](../api/contract-summary.md).

## Data Flow

Frontend screens call small, hand-written `fetch`-wrapper modules (one per screen, e.g. `screens/approvals/approvals-api.ts`) against `import.meta.env.VITE_API_URL ?? "http://localhost:3000"`. The backend has no request validation pipe and no persistence layer, so every request is served directly from an in-memory store recreated on process start; nothing survives a backend restart. `ApprovalsService` (SLICE-09) is the first cross-module composition point — it aggregates read models from `PriceScenariosService` and `DiscountModelingService` rather than owning its own entity store.

## Validation and Testing Overview

- **Backend:** Jest (`ts-jest`), one `*.controller.spec.ts` and one `*.service.spec.ts` per module (the `catalog` module and the `approvals` service have no dedicated spec file — approvals' service-level behavior is tested inside `approvals.controller.spec.ts`). `npm run check` = lint → lint:policy → typecheck → build.
- **Frontend:** Vitest for unit tests (currently one file, `src/App.test.tsx`), Cucumber + Playwright for BDD acceptance tests (`features/*.feature` + `tests/bdd/steps/*.steps.ts`, one pair per implemented slice). `npm run check` = lint → lint:policy → typecheck → build.
- **Knowledge:** a validation report exists per implemented slice under `knowledge/specs/001-platform-baseline/validation/SLICE-0N.md` for SLICE-00 through SLICE-09; none exist yet for SLICE-10…14 (not implemented).

## Known Gaps

- `backend/contracts/api-contract.yaml` was deleted from `main` after being added; only the Markdown contract currently exists in the repo.
- SLICE-10 (Agent roster), SLICE-11 (Pricing Autonomy), SLICE-12 (Measurement), SLICE-13 (Like-Item Mapping / Cold Start), and SLICE-14 (Configuration) are `Approved` in `tasks.md` but have no implementation, contract, or validation report yet.
- REQ-APPR-005 and REQ-APPR-008 (cross-domain sub-action drawer and action-tracker linkback) are explicitly deferred within SLICE-09 — no `BusinessIssueEntity`/action-tracker subsystem exists anywhere in the codebase (per `knowledge/specs/001-platform-baseline/validation/SLICE-09.md`).
- Several platform spec Open Questions remain only partially resolved (e.g. Marketing/Executive scope, the "Ask anything" stub, Agents/Autonomy canonical-version reconciliation with the Pricing Platform V2 fork) — see `knowledge/specs/001-platform-baseline/spec.md` Open Questions section for the full, current list.
