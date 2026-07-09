# Backend Structure

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| backend | main | a18bdff033c9dac4a90c365c547446057da3ae9b |

## Package Scripts

| Script | Command | Purpose |
|---|---|---|
| build | `nest build` | Compile the NestJS app via Nest CLI |
| check | `npm run lint && npm run lint:policy && npm run typecheck && npm run build` | Composite CI-style gate: lint → policy lint → typecheck → build |
| lint | `eslint "{src,test}/**/*.ts"` | Lint TypeScript source/test files |
| lint:policy | `node scripts/lint-policy.mjs` | Run the project's dependency/policy lint script |
| start | `nest start` | Run the app (non-watch) via Nest CLI |
| start:dev | `nest start --watch` | Run the app in watch/dev mode |
| test | `jest` | Run the Jest test suite |
| typecheck | `tsc --noEmit -p tsconfig.json` | Type-check without emitting output |

Jest config is embedded in `package.json` (`rootDir: "src"`, `testRegex: ".*\\.spec\\.ts$"`, `ts-jest` transform).

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| @nestjs/cli | ^11.0.0 | Nest CLI — build/start/schematics commands (dev) |
| @nestjs/common | ^11.0.0 | Core NestJS decorators/utilities (Controller, Injectable, pipes, exceptions) |
| @nestjs/core | ^11.0.0 | NestJS framework runtime (DI container, app bootstrap) |
| @nestjs/platform-express | ^11.0.0 | Express HTTP adapter for NestJS |
| @nestjs/schematics | ^11.0.0 | Code-generation schematics used by Nest CLI (dev) |
| @nestjs/testing | ^11.0.0 | NestJS testing utilities (TestingModule, etc.) (dev) |
| @types/express | ^5.0.0 | TypeScript types for Express (dev) |
| @types/jest | ^29.5.14 | TypeScript types for Jest (dev) |
| @types/node | ^22.10.5 | TypeScript types for Node.js (dev) |
| @types/supertest | ^6.0.2 | TypeScript types for Supertest (dev) |
| eslint | ^9.17.0 | Linter (dev) |
| globals | ^15.14.0 | Predefined global-variable sets for ESLint config (dev) |
| jest | ^29.7.0 | Test runner/framework (dev) |
| reflect-metadata | ^0.2.2 | Metadata reflection polyfill required by NestJS decorators |
| rxjs | ^7.8.1 | Reactive Extensions library used internally by NestJS |
| supertest | ^7.0.0 | HTTP assertion library for testing Nest controllers/endpoints (dev) |
| ts-jest | ^29.2.5 | TypeScript transformer for Jest (dev) |
| typescript | ^5.7.2 | TypeScript compiler (dev) |
| typescript-eslint | ^8.19.0 | TypeScript-aware ESLint rules/parser (dev) |

## Main Directories

| Path | Purpose | Notes |
|---|---|---|
| src/agents/ | Agent roster: monitors, derived operators, task agents; hire/pause/resume/retire | SLICE-10; no dependency on other services |
| src/catalog/ | Exposes catalog-derived filterable attributes | `catalog-data.ts` static SKU dataset + service/controller; no dedicated spec file |
| src/discount-modeling/ | CRUD + run/submit lifecycle for discount models | SLICE-06 |
| src/focus-sets/ | CRUD for Focus Sets (named SKU filters) | SLICE-02; condition-tree evaluation in `condition.ts` |
| src/guardrails/ | CRUD + evaluate for pricing guardrail rules | SLICE-04; hard/advisory severity |
| src/health/ | Health-check endpoint | SLICE-00 scaffolding |
| src/price-scenarios/ | CRUD + run/submit/status lifecycle for price scenarios, deep-dive analytics | SLICE-07 / SLICE-08 |
| src/product-grid/ | Per-Focus-Set product/SKU grid view with SKU exclusion/restoration | SLICE-03 |
| src/promotions/ | CRUD for promotions with derived status | SLICE-05 |
| src/approvals/ | Aggregated approvals queue/decided views across price-scenarios and discount-modeling, with decision workflow | SLICE-09 |
| src/shared/ | `in-memory-repository.ts` — generic `InMemoryRepository<T>` scaffold (Map-backed), explicitly a placeholder pending a persistent ORM/DB choice | Not directly used by any module's own service |

Root-level `src/` files: `app.module.ts` (wires all controllers/providers), `main.ts` (bootstrap; `app.enableCors()` with no options; no global route prefix).

## Routes / Controllers

| Path | Purpose | Notes |
|---|---|---|
| GET /agents/roster | Full roster (KPIs, monitors, operators, task agents) | `agents.controller.ts`, SLICE-10 |
| GET /agents/catalog | Provisionable monitor/operator types | SLICE-10 |
| POST /agents/monitors/:id/pause | Pause a monitor | HttpCode 200, SLICE-10 |
| POST /agents/monitors/:id/resume | Resume a monitor | HttpCode 200, SLICE-10 |
| POST /agents/hire | Hire a monitor or operator from the catalog | SLICE-10 |
| POST /agents/task-agents/:id/retire | Retire a task agent | HttpCode 200, SLICE-10 |
| GET /health | Liveness check | `health.controller.ts` |
| GET /catalog/attributes | List filterable catalog attributes and observed values | `catalog.controller.ts` |
| GET /focus-sets | List all focus sets | `focus-sets.controller.ts` |
| POST /focus-sets | Create a focus set | |
| POST /focus-sets/resolve | Preview SKUs matched by an ad-hoc filter tree | HttpCode 200 |
| GET /focus-sets/:id | Get a focus set by id | |
| PUT /focus-sets/:id | Update a focus set | |
| DELETE /focus-sets/:id | Delete a focus set | HttpCode 204 |
| GET /focus-sets/:id/skus | Resolve SKUs matched by a saved focus set's filter | |
| POST /focus-sets/:id/duplicate | Duplicate a focus set under a new name | |
| GET /product-grid/:focusSetId | Get the product grid for a focus set | `product-grid.controller.ts` |
| POST /product-grid/:focusSetId/exclude | Mark a SKU as excluded | HttpCode 200 |
| DELETE /product-grid/:focusSetId/exclusions/:skuId | Restore a single excluded SKU | HttpCode 204 |
| DELETE /product-grid/:focusSetId/exclusions | Restore all excluded SKUs | HttpCode 204 |
| GET /guardrails | List all guardrails | `guardrails.controller.ts` |
| POST /guardrails | Create a guardrail | |
| POST /guardrails/evaluate | Evaluate metrics against active guardrails | HttpCode 200 |
| PATCH /guardrails/:id | Partially update a guardrail | |
| PATCH /guardrails/:id/active | Toggle active flag | HttpCode 200 |
| PATCH /guardrails/:id/overridable | Toggle isOverridable flag | HttpCode 200 |
| DELETE /guardrails/:id | Delete a guardrail | HttpCode 204 |
| GET /promotions | List all promotions | `promotions.controller.ts` |
| POST /promotions | Create a promotion | |
| GET /promotions/:id/products | Get discounted product rows for a promotion | |
| PATCH /promotions/:id | Partially update a promotion | |
| DELETE /promotions/:id | Delete a promotion | HttpCode 204 |
| GET /discount-models | List all discount models | `discount-modeling.controller.ts` |
| POST /discount-models | Create a discount model | |
| GET /discount-models/:id | Get a discount model by id | |
| POST /discount-models/:id/run | Run the model, populate output | HttpCode 200 |
| POST /discount-models/:id/submit | Submit for approval | HttpCode 200 |
| PATCH /discount-models/:id/status | Force-set status | HttpCode 200 |
| DELETE /discount-models/:id | Delete a discount model | HttpCode 204 |
| GET /price-scenarios | List all price scenarios | `price-scenarios.controller.ts` |
| POST /price-scenarios | Create a price scenario | |
| GET /price-scenarios/:id | Get a price scenario by id | |
| GET /price-scenarios/:id/deep-dive | Get deep-dive breakdown | SLICE-08 |
| POST /price-scenarios/:id/run | Run, populate output | HttpCode 200 |
| POST /price-scenarios/:id/submit | Submit for approval | HttpCode 200 |
| PATCH /price-scenarios/:id/status | Force-set status | HttpCode 200 |
| DELETE /price-scenarios/:id | Delete a price scenario | HttpCode 204 |
| GET /approvals/queue | List pending scenarios and discount models | `approvals.controller.ts`, SLICE-09 |
| GET /approvals/decided | List decided scenarios and discount models | SLICE-09 |
| GET /approvals/scenarios/:id/review | Full scenario output for approval-mode review | SLICE-09 |
| GET /approvals/discounts/:id/review | Discount risk banner/impact for approval-mode review | SLICE-09 |
| POST /approvals/scenarios/:id/decision | Approve/deny/request-changes on a scenario | HttpCode 200, SLICE-09 |
| POST /approvals/discounts/:id/decision | Approve/deny/request-changes on a discount model | HttpCode 200, SLICE-09 |

No global path prefix is set in `main.ts`, so the above are the final, literal route paths.

## Services / Modules

| Path | Purpose | Notes |
|---|---|---|
| src/agents/agents.service.ts | In-memory array CRUD-lite (seeded monitors/operators/task agents); hire, pause/resume, retire | No constructor dependencies on other services |
| src/catalog/catalog.service.ts | Derives distinct filterable attribute/value options from static `CATALOG_SKUS` data | Read-only, no store |
| src/focus-sets/focus-sets.service.ts | In-memory `Map`-backed CRUD + duplicate/resolve for Focus Sets | Evaluates condition trees against the catalog |
| src/product-grid/product-grid.service.ts | Builds grid/product/SKU views for a Focus Set; tracks per-focus-set SKU exclusions | `Map<string, Set<string>>` |
| src/guardrails/guardrails.service.ts | In-memory array CRUD (seeded) for guardrail rules + `evaluate()` compliance check | |
| src/promotions/promotions.service.ts | In-memory array CRUD for promotions; derives status from dates; computes affected-product views | |
| src/discount-modeling/discount-modeling.service.ts | In-memory array CRUD + run/submit/status lifecycle; computes KPIs, rollups, risk panels, chart datasets | |
| src/price-scenarios/price-scenarios.service.ts | In-memory array CRUD + run/submit/status state machine; generates deep-dive analytics | Depends on `FocusSetsService`, `GuardrailsService` |
| src/approvals/approvals.service.ts | Aggregates pending/decided items across `PriceScenariosService` and `DiscountModelingService`; enforces a mandatory-comment decision policy for deny/request-changes | No entity store of its own; keeps an in-memory decision log |
| src/shared/in-memory-repository.ts | Generic `InMemoryRepository<T>` scaffold | Not used directly by any module's service; a placeholder pending ORM/DB choice |

## Tests

| Test file | Type | Notes |
|---|---|---|
| src/agents/agents.controller.spec.ts | Controller + service | Roster/catalog reads, pause/resume/hire/retire delegation; second `describe` block tests `BadRequestException` (invalid hire subtype) and `NotFoundException` (unknown monitor id) directly against `AgentsService` |
| src/health/health.controller.spec.ts | Controller | Returns `{status: "ok"}` |
| src/focus-sets/focus-sets.controller.spec.ts | Controller | Catalog attributes, resolve preview, 400 on missing name, full CRUD+duplicate+delete lifecycle |
| src/focus-sets/focus-sets.service.spec.ts | Service | Filter resolution, create with auto-incremented id, edit-in-place, duplicate, live productCount, delete |
| src/product-grid/product-grid.controller.spec.ts | Controller | Get grid (200/404), exclude SKU, restore one/all SKUs |
| src/product-grid/product-grid.service.spec.ts | Service | Grid grouping, 404 for unknown focus set, exclude/restore semantics |
| src/guardrails/guardrails.controller.spec.ts | Controller | List seed guardrails, create, patch, toggle active/overridable, delete, evaluate |
| src/guardrails/guardrails.service.spec.ts | Service | findAll seed data, create defaults, update, toggle, remove + 404s, evaluate pass/fail |
| src/discount-modeling/discount-modeling.controller.spec.ts | Controller | Empty list, create, run output, submit-requires-draft, status transitions, remove, 404 |
| src/discount-modeling/discount-modeling.service.spec.ts | Service | Create, run→draft, submit draft→pending + rejection, updateStatus, remove + 404s, math checks |
| src/price-scenarios/price-scenarios.controller.spec.ts | Controller | findAll/findOne, create, run, submit, updateStatus, remove, getDeepDive |
| src/price-scenarios/price-scenarios.service.spec.ts | Service | Create/run/submit transitions + guard, updateStatus appends changeRequests, deep-dive shape/404s |
| src/promotions/promotions.controller.spec.ts | Controller | Empty list, create with derived status, 400 on invalid dates, update, 404, remove, getProducts 404 |
| src/promotions/promotions.service.spec.ts | Service | Create + status derivation, date validation, update, 404s, remove, getProducts 404s |
| src/approvals/approvals.controller.spec.ts | Controller + service | getQueue/getDecided, getScenarioReview/getDiscountReview delegation, decideScenario/decideDiscount; second `describe` block tests the mandatory-comment reason policy directly against `ApprovalsService` |

`src/catalog/` has no dedicated spec file (only exercised indirectly via the `GET /catalog/attributes` assertion inside `focus-sets.controller.spec.ts`). `src/approvals/approvals.service.ts` has no separate `*.service.spec.ts` — its behavior is tested inside `approvals.controller.spec.ts`.

## Known Gaps

- No ORM or database — confirmed by source inspection (no `@Entity`, `@Column`, `TypeOrmModule`, `PrismaClient`, or DB driver import anywhere in `src/`). All data resets on process restart.
- No global `ValidationPipe`/class-validator is registered; request bodies are plain TypeScript interfaces with no runtime shape enforcement beyond what individual services check explicitly (per `backend/contracts/api-contract.md`'s `x-global-behavior` note).
- `src/shared/in-memory-repository.ts` is an unused generic scaffold, explicitly commented as a placeholder pending a persistent ORM/DB decision.
