# Architecture Overview

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | (this finalization commit) |
| frontend | main | 6bf530564f604a8c9d81d93f076dfe41cb0dfa43 |
| backend | main | 5f46cc599365cfcd2b3180776eacaa93f5086fd2 |

## System Summary

Retail Nucleus is a pricing-intelligence platform for a retail organization, covering Focus Set curation, product-grid management, guardrail constraints, promotions, discount modeling, price scenario optimization, deep-dive analysis, an approvals workflow, an agent roster, pricing autonomy (trust-ladder governance), and matched-cluster experiment measurement. The platform is a three-repo workspace: `knowledge` (specs/plans/tasks/contracts references/validation/wiki), `frontend` (React SPA), and `backend` (NestJS API). Governance follows an approval-gated Speckit workflow (spec → plan → tasks → implementation) recorded in `knowledge/.specify/memory/constitution.md`. The platform baseline is tracked as feature `001-platform-baseline`, broken into 15 implementation slices (SLICE-00 through SLICE-14); 13 are `Complete` (SLICE-00…12), and 2 (SLICE-13…14) are `Approved` but not yet implemented.

## Repository Responsibilities

| Repo | Responsibility | Notes |
|---|---|---|
| knowledge | Specs, plans, tasks, validation reports, per-slice contract references (`knowledge/contracts/<slice>/contract.md`), constitution/technology/data-taxonomy governance docs, wiki | Source of truth for requirements and governance; does not own the canonical API contract file |
| frontend | React 19 + TypeScript + Vite SPA; Cucumber/Playwright BDD tests; Vitest unit tests | Consumes the backend API via per-screen fetch-wrapper modules |
| backend | NestJS 11 REST API; Jest unit/controller tests; owns the canonical API contract | All persistence is in-memory (no ORM/database) — see [Backend Structure](../codebase/backend-structure.md) |

## Frontend Overview

Pinned stack (per `knowledge/TECHNOLOGY.md` and `knowledge/.specify/memory/constitution.md`): React 19, TypeScript, Vite, Tailwind CSS + shadcn/ui, Framer Motion, React Router v7, Zustand (UI state) + nuqs (URL state), React Hook Form + Zod, AG Grid (large tables), Nivo (charts), date-fns. The app is a single-page app with one router (`src/app/routes.tsx`) built from a persona-grouped nav model (`src/app/nav.ts`: Pricing Team, Pricing Strategist, Admin). One screen folder exists per feature slice under `src/screens/`. See [Frontend Structure](../codebase/frontend-structure.md) for the full directory, route, and test inventory.

## Backend Overview

NestJS 11 API (`@nestjs/common`, `@nestjs/core`, `@nestjs/platform-express`). One flat feature module per slice under `src/` (catalog, focus-sets, product-grid, guardrails, promotions, discount-modeling, price-scenarios, approvals, agents, autonomy, measurement), plus `health` and a `shared/in-memory-repository.ts` scaffold. **No ORM or database is used anywhere** — every module holds its records in an in-memory array or `Map`, seeded at process start and reset on restart; this was verified directly in source (e.g. `private store: ScenarioEntity[] = []` in `price-scenarios.service.ts`, `private readonly store = new Map<string, FocusSetEntity>()` in `focus-sets.service.ts`). No global route prefix or global `ValidationPipe` is registered (`src/main.ts`); most request bodies are plain TypeScript interfaces with no runtime shape validation beyond what individual services check explicitly. As of SLICE-12, the backend also has Cucumber/BDD tooling (`@cucumber/cucumber`, `ts-node`) for API-level acceptance tests, run via an in-process NestJS test application (`tests/support/app.ts`) rather than the long-running dev server. See [Backend Structure](../codebase/backend-structure.md) for the full module, route, and test inventory.

## API / Contract Boundary

The canonical, human-readable API contract is `backend/contracts/api-contract.md` (67 endpoints, 78 schemas — originally reverse-engineered from `main` source/tests, with SLICE-10/11/12's Agents/Autonomy/Measurement entries hand-authored directly since the YAML source no longer exists). **`backend/contracts/api-contract.yaml`, an OpenAPI 3.0.3 structured version of the same contract, existed at one point but was deleted from `main` in backend commit `bb1a951` ("deleted redundant file") prior to the SLICE-09 merge — it does not exist in the current backend tree.** Per-slice contract references also exist under `knowledge/contracts/<slice-id>/contract.md` (11 folders, `slice-02` through `slice-12`, all `Stable`) — these are supporting/design-time references, not the canonical contract. See [Contract Summary](../api/contract-summary.md).

## Data Flow

Frontend screens call small, hand-written `fetch`-wrapper modules (one per screen, e.g. `screens/approvals/approvals-api.ts`) against `import.meta.env.VITE_API_URL ?? "http://localhost:3000"`. The backend has no request validation pipe and no persistence layer, so every request is served directly from an in-memory store recreated on process start; nothing survives a backend restart. `ApprovalsService` (SLICE-09) is the first cross-module composition point — it aggregates read models from `PriceScenariosService` and `DiscountModelingService` rather than owning its own entity store.

## Validation and Testing Overview

- **Backend:** Jest (`ts-jest`), one `*.controller.spec.ts` and one `*.service.spec.ts` per module (the `catalog` module and the `approvals` service have no dedicated spec file — approvals' service-level behavior is tested inside `approvals.controller.spec.ts`). As of SLICE-12, the backend also runs Cucumber (`cucumber-js`, via `npm run test:bdd`) for API-level acceptance scenarios (`tests/features/*.feature` + `tests/steps/*.steps.ts`), exercised against a real in-process Nest app, not the dev server. `npm run check` = lint → lint:policy → typecheck → build.
- **Frontend:** Vitest for unit tests (currently one file, `src/App.test.tsx`), Cucumber + Playwright for BDD acceptance tests (`features/*.feature` + `tests/bdd/steps/*.steps.ts`, one pair per implemented slice). `npm run check` = lint → lint:policy → typecheck → build.
- **Knowledge:** a validation report exists per implemented slice under `knowledge/specs/001-platform-baseline/validation/SLICE-0N.md` for SLICE-00 through SLICE-12; none exist yet for SLICE-13…14 (not implemented).

## Known Gaps

- `backend/contracts/api-contract.yaml` was deleted from `main` after being added; only the Markdown contract currently exists in the repo.
- SLICE-13 (Like-Item Mapping / Cold Start) and SLICE-14 (Configuration) are `Approved` in `tasks.md` but have no implementation, contract, or validation report yet.
- SLICE-12 (Measurement)'s REQ-MEAS-008 (streaming trend), REQ-MEAS-012 (closed-loop Autonomy feedback), and REQ-MEAS-013 (reopen setup) are not implemented — not covered by the approved preparation-gate test scaffolding.
- REQ-APPR-005 and REQ-APPR-008 (cross-domain sub-action drawer and action-tracker linkback) are explicitly deferred within SLICE-09 — no `BusinessIssueEntity`/action-tracker subsystem exists anywhere in the codebase (per `knowledge/specs/001-platform-baseline/validation/SLICE-09.md`).
- SLICE-10's Operator trust data (`trustLevel`/`evidenceStatus`/`trackRecord`) remains a deterministic placeholder even after SLICE-11 (Pricing Autonomy) shipped — SLICE-11's own `ActionClassEntity` trust-ladder data was not wired back into SLICE-10's `OperatorView`, since Operators and Action Classes are different domain concepts (agent-hire-time trust vs. action-class promotion gates). Revisit if a future slice needs to unify them.
- SLICE-11's promotion-gate thresholds (sample count, accuracy, acceptance rate) are v1 placeholders with no historical dataset behind them; kill-switch "freeze" is client-side only (no backend timer concept).
- Several platform spec Open Questions remain only partially resolved (e.g. Marketing/Executive scope, the "Ask anything" stub, Agents/Autonomy canonical-version reconciliation with the Pricing Platform V2 fork) — see `knowledge/specs/001-platform-baseline/spec.md` Open Questions section for the full, current list.
