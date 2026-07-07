---
status: Approved
artifact_type: platform-plan
feature_id: 000-platform
approved_by: Saad
approved_at: 07/07/2026
source_spec: knowledge/specs/000-platform/spec.md
---

# Platform Implementation Plan

## Status

`Approved` — approved by Saad on 07/07/2026 (see front matter). `/create-platform-tasks` may now proceed.

> **Speckit note:** This plan follows Speckit planning conventions and structure. It was authored directly rather than via the `/speckit-plan` script (`.specify/scripts/powershell/setup-plan.ps1`), because that script requires an active Speckit feature branch (`Test-FeatureBranch` → `exit 1` otherwise) and no `feature.json` pins the `000-platform` directory. Running it on `main` would fail and/or force an unexpected feature branch, conflicting with this harness's main-branch rule (constitution Principle II; `multi-repo-workspace-safety`) and the mandated fixed output path `knowledge/specs/000-platform/plan.md`. The `/create-platform-plan` command explicitly authorizes direct authoring in this case.

## Source Documents

- `knowledge/specs/000-platform/spec.md` (`status: Approved`, approved by Saad 07/07/2026) — the approved platform spec (120 EARS requirements, 33 Gherkin scenarios, 14 slices).
- `knowledge/.specify/memory/constitution.md` — governance principles, approval gates, quality gates.
- `knowledge/TECHNOLOGY.md` — pinned frontend stack.
- `knowledge/DATA-TAXONOMY.md` — Tier-1/Tier-2 discipline, entity catalog, FK summary, NestJS backend hint.
- `knowledge/reports/platform-requirements-analysis.md` — prior analysis.
- `knowledge/contracts/**` — none present (greenfield).
- `frontend/` — inspected: empty repo (only `README.md`; no `package.json`, source, or test setup).
- `backend/` — inspected: empty repo (only `README.md`; no `package.json`, source, or test setup).

## Planning Summary

The platform is built greenfield across three repos coordinated by `knowledge/`. Because both implementation repos are empty, the plan front-loads a **scaffolding slice** (SLICE-00) that stands up the pinned frontend stack (React 19 + TypeScript + Vite + the mandated libraries), the backend stack (NestJS + TypeScript), the acceptance-test harness (Cucumber + Playwright), unit-test harnesses (Vitest/RTL frontend; NestJS test runner backend), and the `npm run check` / `npm run lint:policy` quality gates required by the constitution. Everything else follows the spec's dependency-ordered slices.

Delivery is **contract-first and acceptance-test-first**, one slice at a time. For each slice: (1) backend defines/updates the API contract in `knowledge/contracts/<slice-id>/` and gets it approved through the gates; (2) executable acceptance tests (Gherkin `.feature` + Playwright step defs) and backend/contract tests are written and confirmed failing; (3) backend is implemented to satisfy the contract; (4) frontend is implemented against the (mockable, then real) contract; (5) end-to-end validation runs against real frontend + real backend; (6) a validation report is written to `knowledge/specs/000-platform/validation/<slice-id>.md`.

The core ML/forecasting "intelligence" is **not** built in v1. Per the spec's Open Questions, Discount Modeling, Price Scenario, and Like-Item Mapping backends will produce their documented output *shapes* via deterministic/placeholder computation behind a stable contract, so the frontend and the whole governed workflow can be built and validated now; real modeling is a later, separately-scoped effort. This is called out per-slice and in Risks.

## Target Architecture

High-level, three-tier, contract-mediated:

- **Frontend (SPA):** React 19 + TypeScript + Vite. React Router v7 routes for each screen in the spec's navigation map. Zustand stores (`state.ts`, `useXxxState`) for UI state; nuqs for URL-synced filters. shadcn/ui components, Nivo charts, AG Grid for dense tables, Framer Motion transitions, RHF + Zod forms, date-fns. Consumes backend contracts via a typed API client; may mock contracts during acceptance-test-first work.
- **Backend (API):** NestJS + TypeScript (indicated by `DATA-TAXONOMY.md`'s "NestJS `@Entity()` classes"). Owns all API contracts and Tier-1 `*Entity` domain records. Implements Focus Set CRUD + the canonical **resolve-to-SKUs** operation, guardrail CRUD + live compliance evaluation, approval state machine, and the run/simulation endpoints (placeholder computation behind stable output contracts in v1).
- **Knowledge repo:** source of truth for spec/plan/tasks, approved contracts (`knowledge/contracts/<slice-id>/`), validation reports, wiki, logs, and shipped-feature history.
- **API/contract boundary:** backend owns request/response shapes; frontend consumes them. Frontend request/response projections are Tier-2 (`*View`/`*Row`/`*Form`/…), never treated as canonical entities unless the backend contract defines them so.
- **Validation/reporting:** per-slice validation reports map EARS requirements and Gherkin scenarios to pass/fail results, stored in `knowledge/`. A patcher agent (≤3 attempts) addresses mismatches.

## Repository Responsibilities

### knowledge/

- Owns `specs/000-platform/{spec,plan,tasks}.md` and per-slice `validation/<slice-id>.md` reports.
- Owns approved API contracts under `knowledge/contracts/<slice-id>/` (draft → stable lifecycle; promoted to stable only post-merge).
- Owns wiki, `log.md`, shipped-feature history, PR links, commit SHAs.
- No implementation code. Finalization files updated only after implementation PRs merge.

### frontend/

- React 19 + TS + Vite SPA implementing the screens/routes in the spec.
- UI-rendering files PascalCase; non-UI modules kebab-case; stores `state.ts`; hooks `useXxxState`; avoid generic names; cross-app shared code in `src/shared/` only when domain-neutral.
- Zustand + nuqs state; shadcn/ui, Nivo, AG Grid, Framer Motion, RHF + Zod, date-fns.
- Cucumber `.feature` files + Playwright-backed step definitions (`features/**`, `tests/bdd/**`, `test:bdd` script); Vitest + React Testing Library unit tests (accessible-role queries preferred).
- `npm run check` (lint + `lint:policy` + typecheck + build) must pass. Consumes backend contracts; may mock during acceptance-test-first.

### backend/

- NestJS + TypeScript API implementing the owned contracts and Tier-1 `*Entity` records.
- Domain logic: Focus Set resolution, guardrail evaluation, approval/lifecycle state machines, run/simulation endpoints (placeholder computation behind stable contracts in v1).
- Backend unit tests + integration tests + contract tests (fail-first before implementation).
- `npm run check` / build must pass. Owns and satisfies the approved API contract; must not drift from it.
- ORM/database choice `[NEEDS CLARIFICATION]` — see Risks; may start with an in-memory/seed-backed repository layer to satisfy contracts before a DB is chosen.

## Platform Slices

Ordered; each independently testable, reviewable, PR-sized, and (except SLICE-00 tooling) cross-repo. Requirements/scenarios reference the approved spec. Contracts live in `knowledge/contracts/<slice-id>/`.

### SLICE-00 — Repository scaffolding & quality gates
- **Purpose:** Stand up both empty repos with the pinned stacks, test harnesses, and `npm run check`/`lint:policy` gates so every later slice can follow acceptance-test-first.
- **Target repos:** frontend, backend (knowledge for CI/gate docs).
- **Dependencies:** none.
- **Requirements covered:** enabling foundation for all (no product EARS directly; supports REQ-SHELL-* onward).
- **Acceptance scenarios covered:** none directly; establishes the runnable BDD setup the acceptance-test-first rule depends on.
- **Contracts needed:** none.
- **Validation expectations:** `npm run check` runs and passes in both repos; a smoke Playwright/Cucumber run executes; unit-test runner executes.
- **PR expectations:** one PR per repo (frontend scaffold, backend scaffold).

### SLICE-01 — Platform shell & navigation
- **Purpose:** App shell, sidebar sections, routing, global brand/channel filters.
- **Target repos:** frontend.
- **Dependencies:** SLICE-00.
- **Requirements:** REQ-SHELL-001, REQ-SHELL-002. (REQ-SHELL-003 "Ask anything" deferred — see spec Open Q7.)
- **Acceptance scenarios:** navigation renders; routes resolve; global brand/channel filters present.
- **Contracts needed:** none (static shell); brand/channel are client state.
- **Validation:** BDD nav test passes; `npm run check`.
- **PR:** one frontend PR.

### SLICE-02 — Focus Set management
- **Purpose:** Focus Builder CRUD + condition tree + live preview + export; backend Focus Set resource + canonical resolve-to-SKUs + catalog attribute enumeration.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-01.
- **Requirements:** REQ-FOCUS-001…012.
- **Acceptance scenarios:** create/preview/no-match; search/sort; export.
- **Contracts:** `contracts/slice-02-focus-sets/` — Focus Set CRUD, resolve-to-SKUs, attribute enumeration, matched-SKU export payload.
- **Validation:** BDD + unit + backend/contract tests; live vs snapshot resolution decision recorded (spec Open Q3).
- **PR:** separate frontend + backend PRs.

### SLICE-03 — Product Grid
- **Purpose:** SKU curation, product/SKU rollup toggle, soft-delete/restore, export, read-only criteria bar.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-02.
- **Requirements:** REQ-GRID-001…007.
- **Acceptance scenarios:** soft-delete/restore; product/SKU toggle.
- **Contracts:** `contracts/slice-03-product-grid/` — SKU resolution for a Focus Set; persisted exclusions (if chosen).
- **Validation:** BDD + unit; qty-color scale reconciliation recorded (REQ-GRID-007).
- **PR:** separate frontend + backend PRs.

### SLICE-04 — Guardrails management
- **Purpose:** Guardrail CRUD + toggles + live evaluation service.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-01.
- **Requirements:** REQ-GUARD-001…007.
- **Acceptance scenarios:** inline edit; non-overridable enforced.
- **Contracts:** `contracts/slice-04-guardrails/` — guardrail CRUD; live compliance evaluation endpoint.
- **Validation:** BDD + unit + backend tests; versioning/audit decision recorded (REQ-GUARD-007).
- **PR:** separate frontend + backend PRs.

### SLICE-05 — Promotions calendar
- **Purpose:** Promo CRUD, calendar/list, detail drawer, per-SKU promo price.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-02.
- **Requirements:** REQ-PROMO-001…008.
- **Acceptance scenarios:** date validation; status filter counts; view products.
- **Contracts:** `contracts/slice-05-promotions/` — promo CRUD; per-SKU promo price; canonical Promotion/Discount entity decision (spec Open Q2).
- **Validation:** BDD + unit + backend tests.
- **PR:** separate frontend + backend PRs.

### SLICE-06 — Discount Modeling
- **Purpose:** Model create/run/output/lifecycle, risk panels, CSV export.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-02.
- **Requirements:** REQ-DISC-001…010.
- **Acceptance scenarios:** run gating; open output directly; discard regardless of status.
- **Contracts:** `contracts/slice-06-discount-modeling/` — run-model → `DiscountModelOutput`; model persistence + status. **v1 output is placeholder computation behind the stable shape** (spec Open Q1).
- **Validation:** BDD + unit + backend/contract tests.
- **PR:** separate frontend + backend PRs.

### SLICE-07 — Price Scenario optimization
- **Purpose:** Scenario create/run, profit-vs-revenue frontier + optimization slider, comparison table, narrative; live guardrail pre-fill.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-02, SLICE-04.
- **Requirements:** REQ-SCEN-001…011.
- **Acceptance scenarios:** weight-sum-100; slider bands; freeze when submitted; resubmit returned.
- **Contracts:** `contracts/slice-07-price-scenario/` — scenario run (6 phases) → frontier + uplift + per-SKU recs; live guardrail evaluation. **v1 placeholder computation** (spec Open Q1).
- **Validation:** BDD + unit + backend/contract tests.
- **PR:** separate frontend + backend PRs.

### SLICE-08 — Deep Dive
- **Purpose:** SKU-level drill, Explain modal, progressive unlock, marketing/discount tiles.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-07.
- **Requirements:** REQ-DEEP-001…005.
- **Acceptance scenarios:** Explain modal; progressive unlock by optimization level.
- **Contracts:** `contracts/slice-08-deep-dive/` — per-SKU recommendation detail + explainability trace (placeholder in v1).
- **Validation:** BDD + unit.
- **PR:** separate frontend + backend PRs.

### SLICE-09 — Approvals queue
- **Purpose:** Two-tab queue, review drawers, decisions, cross-domain linkage, reason rules.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-06, SLICE-07, SLICE-04.
- **Requirements:** REQ-APPR-001…012.
- **Acceptance scenarios:** deny-reason mandatory; return-comment mandatory; decided table.
- **Contracts:** `contracts/slice-09-approvals/` — approval state machine; live guardrail-compliance record; action-tracker linkback; one reason-required policy (spec Open Q5).
- **Validation:** BDD + unit + backend tests.
- **PR:** separate frontend + backend PRs.

### SLICE-10 — Agent roster
- **Purpose:** Monitors/operators/task agents, hire/pause/retire, KPI strip.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-01.
- **Requirements:** REQ-AGENT-001…007.
- **Acceptance scenarios:** pause/resume monitor; hire agent.
- **Contracts:** `contracts/slice-10-agents/` — roster + lifecycle + signals; canonical-version decision vs V2 fork (spec Open Q8).
- **Validation:** BDD + unit; new Tier-1 entities defined (spec Open Q12).
- **PR:** separate frontend + backend PRs.

### SLICE-11 — Pricing Autonomy
- **Purpose:** Trust ladder, promotion gates, live supervision (veto/undo), kill switch, audit.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-10.
- **Requirements:** REQ-AUTO-001…010.
- **Acceptance scenarios:** promotion gating; kill-switch disables controls + freezes countdowns.
- **Contracts:** `contracts/slice-11-autonomy/` — action-class state, promotion-gate evaluation, live-action state machine, kill-switch semantics, audit (spec Open Q8/Q12).
- **Validation:** BDD + unit + backend tests.
- **PR:** separate frontend + backend PRs.

### SLICE-12 — Measurement
- **Purpose:** Setup (arm assignment, balance checks, go-live gate) + live readout (probability trend, verdict, per-cluster contribution).
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-07.
- **Requirements:** REQ-MEAS-001…014.
- **Acceptance scenarios:** go-live gate; control drops ML price; verdict banner.
- **Contracts:** `contracts/slice-12-measurement/` — cluster/block/balance, sequential estimator, outcome feedback. **v1 uses placeholder trajectories behind the stable shape** (spec Open Q1/Q9).
- **Validation:** BDD + unit + backend tests.
- **PR:** separate frontend + backend PRs.

### SLICE-13 — Like-Item Mapping (Cold Start)
- **Purpose:** Cold-start queue, ranked candidates, confidence breakdown, validate/override with audit.
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-02.
- **Requirements:** REQ-COLD-001…012.
- **Acceptance scenarios:** validate top pick; override with audit; close-call flag.
- **Contracts:** `contracts/slice-13-like-items/` — matching + confidence scoring + mapping persistence + audit. **v1 placeholder matching** (spec Open Q1); PIM/DAM/sales feeds deferred.
- **Validation:** BDD + unit + backend tests.
- **PR:** separate frontend + backend PRs.

### SLICE-14 — Configuration
- **Purpose:** Read-only user access list (+ enforcement only if access control is brought into scope).
- **Target repos:** frontend, backend.
- **Dependencies:** SLICE-01.
- **Requirements:** REQ-CONFIG-001…002.
- **Acceptance scenarios:** user list renders read-only.
- **Contracts:** `contracts/slice-14-config/` — user list read model; CRUD/IdP/authorization only if in scope (spec Open Q6).
- **Validation:** BDD + unit.
- **PR:** separate frontend + backend PRs.

## Contract Strategy

- **Backend owns every API contract.** Each full-stack slice defines/updates its contract under `knowledge/contracts/<slice-id>/` and takes it through the spec/plan/tasks approval gates before implementation.
- **Frontend consumes contracts.** During acceptance-test-first work the frontend may mock the contract; final slice validation must run against the **real backend**.
- **Contract lifecycle:** draft in `knowledge/` → approved before implementation → promoted to stable only after implementation PRs merge (post-merge finalization).
- **No drift:** if an implementation reveals a needed contract change, stop and get human approval unless already covered by approved tasks. Frontend request/response projections stay Tier-2 unless the contract defines them as canonical.
- **Shared cross-cutting contracts:** the Focus Set resolve-to-SKUs contract (SLICE-02) is the canonical shared operation reused by SLICE-03/05/06/07; it must not be re-derived per screen.

## Data Taxonomy Plan

- **Tier-1 `*Entity`** records are backend-owned (NestJS `@Entity()` targets), each with `id` and FK relationships per `DATA-TAXONOMY.md`. Never used directly as UI state.
- **Tier-2 constructs** are derived at render time in the frontend using the correct suffix: `*View` (display projection), `*Row` (grid/table), `*Form` (RHF shape), `*Filters` (filter/URL state), `*State` (local UI state), `*Option` (dropdown). Existing catalogued examples (e.g. `FocusSetView`, `ProductView`, `DeepDiveView`, `RollupRow`, `ScenarioForm`, `ScenarioGraphState`) are reused as-is.
- **New Tier-1 entities** for Autonomy, Agents, Measurement, Cold Start, and the GuardrailCheck tier are defined during their slice contracts (spec Open Q12) — as canonical entities, not by promoting a `*View`.
- **Reconciliation items** carried from the spec: canonical Promotion/Discount entity (SLICE-05) and one Focus Set resolution source (SLICE-02).

## Testing Strategy

- **Acceptance-test-first (mandatory for user-facing behavior):** for each slice, add/adjust Gherkin scenarios in `spec.md`, add executable `.feature` files + Playwright step defs in `frontend/`, run and confirm expected failure, implement, rerun to green. No implemented scenario stays `@wip`.
- **Unit testing:** frontend Vitest + React Testing Library (accessible-role queries preferred); backend NestJS unit tests. Unit tests supplement — never replace — acceptance coverage.
- **Contract testing:** backend contract tests validate responses against the approved `knowledge/contracts/<slice-id>/` shapes; written fail-first before backend implementation.
- **Integration testing:** backend integration tests for CRUD/state-machine flows (Focus Set resolve, guardrail evaluation, approval transitions).
- **E2E testing:** final per-slice validation runs Playwright against real frontend + real backend (no mocks).
- **Expected-failure-before-implementation:** every new/changed test is run and confirmed failing for the right reason before implementation begins.
- **Validation report:** per slice at `knowledge/specs/000-platform/validation/<slice-id>.md`, mapping EARS requirements ↔ scenarios/tests ↔ pass/fail, including `npm run check`, build, and stack/taxonomy/naming/design-system compliance. Commands that did not run are marked `Not available`.

## Technology Compliance Plan

- **shadcn/ui:** all primitives (Button, Dialog, Select, Table, Tabs, Drawer) via shadcn; add missing via `npx shadcn@latest add`. No hand-rolled primitives; no direct Radix imports outside the approved UI layer.
- **Nivo:** all charts (forecast charts, profit/revenue frontier, probability-of-winning trend, balance visuals). Inline SVG only for decorative visuals with no Nivo equivalent — the frontier curve and gross-margin waterfall verified against Nivo capabilities during SLICE-06/07.
- **Zustand:** UI state in `state.ts` via `useXxxState`. No Redux/Jotai/Recoil/Context-as-global-state.
- **nuqs:** URL-synced filters (Focus Set library search/sort, Product Grid search, Like-Item filters, Approvals tab, global brand/channel).
- **React Router v7:** all routing per the spec navigation map.
- **React Hook Form + Zod:** all create/edit forms with required-field, date-order, weight-sum-100, numeric-range validation.
- **Framer Motion:** standard page transition (`opacity/y`, 0.18s easeOut) and run-sequence animations.
- **date-fns:** all date logic (promo status derivation, calendar grid, durations, measurement day-by-day).
- **Design system:** Zinc + teal (`#14b8a6`); semantic green/amber/red/blue; Geist Sans/Mono via `RN_TYPOGRAPHY` tokens (no raw `text-*`); Lucide 1.5px icons; AG Grid `.ag-root-wrapper { height: 100% }` for large tables.
- **TypeScript:** strict mode respected; no `any` escape hatches without justification.
- **`npm run check`:** lint + `lint:policy` (pinned-stack enforcement) + typecheck + build must pass per repo per slice. Any approved deviation carries `// tech-policy-exception: <id> ticket:<TICK> reason:<why>`.

## Risk and Mitigation

| Risk | Type | Mitigation |
|---|---|---|
| Both implementation repos are empty; no stack/scripts/BDD harness exist | Workflow | SLICE-00 scaffolds pinned stacks, test harnesses, and `npm run check` before any product slice. |
| Backend framework/ORM/DB not fully pinned (only NestJS hinted by DATA-TAXONOMY) | Technical | Adopt NestJS + TS; start with an in-memory/seed-backed repository layer to satisfy contracts; defer ORM/DB choice and record it. `[NEEDS CLARIFICATION]` |
| No real ML/forecasting engine exists | Product | v1 backends emit documented output *shapes* via placeholder computation behind stable contracts; real modeling is separately scoped later. |
| Fragmented discount/promo model (three shapes) | Product | Decide the canonical Promotion/Discount entity in SLICE-05 before Promotions/Discount contracts finalize (spec Open Q2). |
| Focus Set resolution must be one canonical service | Technical | Single resolve-to-SKUs contract in SLICE-02, reused by all downstream slices; decide live vs snapshot (spec Open Q3). |
| Guardrails advisory vs blocking undecided per screen | Product | Resolve block-vs-warn policy per screen during SLICE-04/07/09 (spec Open Q4). |
| Agents/Autonomy diverged from V2 fork | Product | Pick canonical version before SLICE-10/11 (spec Open Q8). |
| Speckit scripts assume feature branches; harness is main-based | Workflow | Author artifacts directly following Speckit conventions to fixed paths on `main`; no feature branches. |
| Cross-repo slices can drift | Workflow | Contract-first with separate per-repo PRs; final E2E validation against real frontend + real backend. |

## Rollback and Recovery

- **Per-slice isolation:** each slice ships as separate, reviewable per-repo PRs; an unmerged slice can be abandoned without affecting merged slices.
- **Revert:** a merged slice is rolled back by reverting its PR(s) in the affected repo(s); knowledge finalization for that slice is reverted correspondingly (slice status back to In Progress/Blocked).
- **Contract safety:** contracts stay `draft` in knowledge until implementation merges; a reverted slice keeps its contract in draft (not promoted to stable).
- **Data:** while backend uses an in-memory/seed repository, rollback has no migration impact; once a DB is introduced, each slice's schema changes must be reversible migrations (recorded in that slice's plan/tasks).
- **Patcher:** validation mismatches get ≤3 scoped patch attempts; after 3, stop and write a blocker report rather than force-merging.

## Constitution Compliance Check

| Area | Required? | Plan |
|---|---|---|
| Platform requirements source of truth | Yes | Spec/plan derived from `PLATFORM-REQUIREMENTS.md` + governance docs; no code copied from reference repos. |
| Artifact approval gates | Yes | spec (Approved) → this plan (Draft, stops for approval) → tasks → slice implementation; each slice gated by `Status: Approved`. |
| EARS/Gherkin traceability | Yes | Slices reference REQ-* IDs and spec Gherkin; validation reports map requirements ↔ tests. |
| Acceptance-test-first | Yes | SLICE-00 provides BDD harness; every user-facing slice writes failing `.feature`/step defs before implementation. |
| Contract-first, if needed | Yes | Backend owns contracts in `knowledge/contracts/<slice-id>/`, approved before implementation; frontend consumes/mocks then validates real. |
| Retail Nucleus pinned stack | Yes | Technology Compliance Plan enforces shadcn/Nivo/Zustand/nuqs/RR7/RHF+Zod/Framer/date-fns/AG Grid + `lint:policy`. |
| Data taxonomy | Yes | Tier-1 `*Entity` backend-owned; Tier-2 constructs frontend-derived; no `*Entity` as UI state; new entities defined in slice contracts. |
| Validation report | Yes | Per-slice report at `knowledge/specs/000-platform/validation/<slice-id>.md` mapping requirements to results. |
| Patcher limit | Yes | ≤3 scoped patch attempts per slice, then blocker report; no hiding/deleting failing tests. |

## Plan rules compliance

This plan describes HOW at a high level only. It creates no tasks, no code, and no tests, and does not skip human approval.

---

_It is `Approved` (by Saad on 07/07/2026); `/create-platform-tasks` may now proceed._
