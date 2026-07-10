---
status: Approved
artifact_type: platform-tasks
platform_baseline: true
feature_id: 001-platform-baseline
approved_by: Saad
approved_at: 07/07/2026
source_spec: knowledge/specs/001-platform-baseline/spec.md
source_plan: knowledge/specs/001-platform-baseline/plan.md
---

<!-- Speckit metadata (preserved for compatibility) -->
**Branch**: `001-platform-baseline` | **Date**: 2026-07-07
**Input**: spec.md + plan.md from `specs/001-platform-baseline/`

# Platform Task Roadmap

## Status

`Approved` — approved by Saad on 07/07/2026 (see front matter). Individual slices must be set to `Status: Approved` before that slice may be implemented. Analysis passed with no blocking issues (see **Speckit Analysis Findings**).

> **Speckit note:** Task context was resolved via native Speckit infrastructure (`.specify/scripts/powershell/setup-tasks.ps1`, which returned the feature dir and tasks template). The template is a prerequisite/context helper and does not itself scaffold `tasks.md`, so this roadmap was authored directly into the active Speckit feature directory `specs/001-platform-baseline/` following Speckit-style task conventions and this command's required structure. A manual analyze-equivalent alignment check was performed (see below) because the native `/speckit-analyze` targets Speckit's per-task-ID format rather than this slice-based platform roadmap.

## Source Documents

- `knowledge/specs/platform-baseline-context.md` — Speckit branch/dir + artifact paths.
- `knowledge/specs/001-platform-baseline/spec.md` (`status: Approved`) — 120 EARS requirements, 33 Gherkin scenarios.
- `knowledge/specs/001-platform-baseline/plan.md` (`status: Approved`) — architecture, repo responsibilities, 15 slices, contract/testing strategy.
- `knowledge/.specify/memory/constitution.md` — approval gates, acceptance-test-first, contract-first, pinned stack, taxonomy, patcher limit.
- `knowledge/TECHNOLOGY.md` — pinned frontend stack.
- `knowledge/DATA-TAXONOMY.md` — Tier-1/Tier-2 discipline, entity catalog.

## Task Roadmap Summary

Delivery proceeds one slice at a time, dependency-ordered, contract-first and acceptance-test-first. SLICE-00 scaffolds both empty repos (pinned stacks + BDD/unit harnesses + `npm run check`) so every later slice can write failing acceptance tests before implementation. Each full-stack slice: define/approve contract → write failing acceptance + contract tests → implement backend → implement frontend → E2E vs real backend → validation report. Cross-repo slices open separate frontend and backend PRs; knowledge is finalized only after those PRs merge. No slice may be implemented until its `Status: Approved`.

## Slice Index

| Slice | Name | Status | Target repos | Depends on | Priority |
|---|---|---|---|---|---|
| SLICE-00 | Repository scaffolding & quality gates | Complete | frontend, backend | — | P1 |
| SLICE-01 | Platform shell & navigation | Complete | frontend | SLICE-00 | P1 |
| SLICE-02 | Focus Set management | Complete | frontend, backend | SLICE-01 | P1 |
| SLICE-03 | Product Grid | Complete | frontend, backend | SLICE-02 | P1 |
| SLICE-04 | Guardrails management | Complete | frontend, backend | SLICE-01 | P1 |
| SLICE-05 | Promotions calendar | Complete | frontend, backend | SLICE-02 | P2 |
| SLICE-06 | Discount Modeling | Complete | frontend, backend | SLICE-02 | P2 |
| SLICE-07 | Price Scenario optimization | Complete | frontend, backend | SLICE-02, SLICE-04 | P2 |
| SLICE-08 | Deep Dive | Complete | frontend, backend | SLICE-07 | P2 |
| SLICE-09 | Approvals queue | Complete | frontend, backend | SLICE-06, SLICE-07, SLICE-04 | P2 |
| SLICE-10 | Agent roster | Complete | frontend, backend | SLICE-01 | P3 |
| SLICE-11 | Pricing Autonomy | Complete | frontend, backend | SLICE-10 | P3 |
| SLICE-12 | Measurement | Approved | frontend, backend | SLICE-07 | P3 |
| SLICE-13 | Like-Item Mapping (Cold Start) | Approved | frontend, backend | SLICE-02 | P3 |
| SLICE-14 | Configuration | Approved | frontend, backend | SLICE-01 | P3 |

## Slice Sections

## Slice `SLICE-00` — `Repository scaffolding & quality gates`

Status: Complete

Shipped: frontend PR #1 (`3687301`) + backend PR #1 (`21acafa`) merged to `main` 2026-07-07. Validation: `validation/SLICE-00.md` (PASS).

Target repos:
- frontend, backend (knowledge for gate docs)

Depends on:
- none

Requirements covered:
- Foundation for all (no product EARS directly; enables acceptance-test-first for REQ-SHELL-* onward)

Acceptance scenarios covered:
- none directly (establishes the runnable BDD setup the acceptance-test-first rule depends on)

Contracts needed:
- no

Expected PRs:
- frontend PR (scaffold)
- backend PR (scaffold)

### Acceptance-Test-First Tasks
- [ ] Add a smoke Cucumber `.feature` (`app boots`) before app code
- [ ] Add Playwright-backed step definitions + `test:bdd` script
- [ ] Run BDD smoke test and confirm expected failure (no app yet)
- [ ] Record expected failure in validation notes

### Backend Tasks
- [ ] Scaffold NestJS + TypeScript app (`package.json`, `tsconfig`, entry module)
- [ ] Add unit-test runner + one placeholder test
- [ ] Add `npm run check` (lint + typecheck + build) and `lint:policy` stub
- [ ] Add in-memory/seed repository layer scaffold (ORM/DB deferred — plan Risk)

### Frontend Tasks
- [ ] Scaffold React 19 + TS + Vite app
- [ ] Install/pin stack: shadcn/ui, Nivo, Zustand, nuqs, React Router v7, RHF+Zod, Framer Motion, date-fns, AG Grid
- [ ] Add Vitest + React Testing Library; Cucumber + Playwright BDD harness (`features/**`, `tests/bdd/**`, `test:bdd`)
- [ ] Add `npm run check` (lint + `lint:policy` pinned-stack enforcement + typecheck + build)
- [ ] Add design tokens scaffold (Zinc+teal, Geist, `RN_TYPOGRAPHY`)

### Validation Tasks
- [ ] Run unit tests
- [ ] Run integration tests if available
- [ ] Run BDD/Cucumber smoke test
- [ ] Run E2E if available
- [ ] Run `npm run check` (both repos)
- [ ] Run build (both repos)
- [ ] Create validation report `validation/SLICE-00.md`

### Patcher Tasks
- [ ] If validation mismatches remain, run patcher for up to 3 attempts
- [ ] Record each attempt in validation report
- [ ] Stop with blocker report after 3 failed attempts

### Completion Criteria
- Both repos build; `npm run check` passes; BDD/unit runners execute; smoke BDD test runs (red→green after minimal shell).

## Slice `SLICE-01` — `Platform shell & navigation`

Status: Complete

Shipped: frontend PR #2 (`a9f4221`) merged to `main` 2026-07-08. Validation: `validation/SLICE-01.md` (PASS).

Target repos:
- frontend

Depends on:
- SLICE-00

Requirements covered:
- REQ-SHELL-001, REQ-SHELL-002 (REQ-SHELL-003 "Ask anything" deferred — spec Open Q7)

Acceptance scenarios covered:
- Navigation renders; routes resolve; global brand/channel filters present

Contracts needed:
- no (static shell; brand/channel are client state)

Expected PRs:
- frontend PR

### Acceptance-Test-First Tasks
- [ ] Add `app-shell-navigation.feature` (sidebar sections, route resolution, global filters) before UI
- [ ] Add Playwright step definitions
- [ ] Run BDD and confirm expected failure
- [ ] Record expected failure in validation notes

### Frontend Tasks
- [ ] React Router v7 routes for all screens in the spec navigation map
- [ ] Sidebar grouped by Pricing Team / Pricing Strategist / Admin (Lucide 1.5px icons)
- [ ] Global `brand`/`channel` filters in `useRNState` (Zustand) + nuqs URL sync
- [ ] Framer Motion page-transition standard; design tokens applied

### Validation Tasks
- [ ] Run unit tests
- [ ] Run BDD/Cucumber tests
- [ ] Run `npm run check`
- [ ] Run build
- [ ] Create validation report `validation/SLICE-01.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- All routes reachable; nav + global filters render; BDD green; `npm run check` passes.

## Slice `SLICE-02` — `Focus Set management`

Status: Complete

Shipped: backend PR #2 (`aab22b9`) + frontend PR #3 (`9a2fa8c`) merged to `main` 2026-07-09. Validation: `validation/SLICE-02.md` (PASS). Contract promoted to Stable: `contracts/slice-02-focus-sets/contract.md`.

Target repos:
- frontend, backend

Depends on:
- SLICE-01

Requirements covered:
- REQ-FOCUS-001…012

Acceptance scenarios covered:
- Create a Focus Set with a live preview; Warn when a filter matches nothing; Search and sort the Focus Set library; Export a Focus Set to CSV

Contracts needed:
- yes — `knowledge/contracts/slice-02-focus-sets/` (Focus Set CRUD, resolve-to-SKUs, attribute enumeration, export payload)

Merged PRs:
- Backend PR #2: https://github.com/saadnaveed-amaskit/demo-backend/pull/2 (SHA: `aab22b9`)
- Frontend PR #3: https://github.com/saadnaveed-amaskit/demo-frontend/pull/3 (SHA: `9a2fa8c`)

### Acceptance-Test-First Tasks
- [ ] Add `focus-set-management.feature` covering create/preview/no-match, search/sort, export before UI
- [ ] Add Playwright step definitions
- [ ] Run BDD and confirm expected failure
- [ ] Add backend contract test for CRUD + resolve-to-SKUs before API implementation
- [ ] Run backend test and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Define/approve contract in `contracts/slice-02-focus-sets/`
- [ ] `FocusSetEntity` CRUD endpoints
- [ ] Canonical **resolve Focus Set → SKUs** operation (record live vs snapshot decision — spec Open Q3)
- [ ] Catalog attribute enumeration endpoint
- [ ] Matched-SKU CSV export payload; ID pattern `focus-set-##`

### Frontend Tasks
- [ ] Focus Builder library (shadcn Table/list): name, ID, AND/OR chip criteria, created date, product count
- [ ] Sort (name/date/count) + case-insensitive search via nuqs
- [ ] Create/edit form (RHF+Zod): nestable AND/OR condition tree (≤3 levels), exact-equality rules
- [ ] Live preview (first 8 SKUs + count + no-match warning); duplicate; delete (confirm); export CSV; drill to Product Grid
- [ ] `FocusSetView`/`ProductView` Tier-2 constructs (no `*Entity` as UI state)

### Validation Tasks
- [ ] Run unit tests
- [ ] Run integration tests (resolve-to-SKUs)
- [ ] Run BDD/Cucumber tests
- [ ] Run E2E vs real backend
- [ ] Run `npm run check` (both repos)
- [ ] Run build (both repos)
- [ ] Create validation report `validation/SLICE-02.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- All REQ-FOCUS-* scenarios green E2E vs real backend; resolution source is single/canonical; `npm run check` passes.

## Slice `SLICE-03` — `Product Grid`

Status: Complete

Shipped: backend PR [#3](https://github.com/saadnaveed-amaskit/demo-backend/pull/3) (`67df4a3`) + frontend PR [#4](https://github.com/saadnaveed-amaskit/demo-frontend/pull/4) (`36d559c`) merged to `main` 2026-07-09. Validation: `validation/SLICE-03.md` (PASS). Contract promoted to Stable: `contracts/slice-03-product-grid/contract.md`.

Target repos:
- frontend, backend

Depends on:
- SLICE-02

Requirements covered:
- REQ-GRID-001…007

Acceptance scenarios covered:
- Soft-delete and restore a SKU; Toggle product and SKU views

Contracts needed:
- yes — `knowledge/contracts/slice-03-product-grid/` (SKU resolution; persisted exclusions if chosen)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `product-grid.feature` (rollup toggle, soft-delete/restore, export) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for SKU resolution/exclusions; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-03-product-grid/`; SKU resolution for a Focus Set
- [ ] Persisted soft-exclusion model (decide view-local vs persisted; record)

### Frontend Tasks
- [ ] AG Grid dense browser (`.ag-root-wrapper { height: 100% }`); product/SKU rollup toggle + live row counts
- [ ] Row rendering per REQ-GRID-003 (MSRP strike, price range, stock status)
- [ ] Soft-delete → Deleted Items pane; restore one/all; free-text search; CSV export; read-only criteria bar
- [ ] Reconcile canonical qty/inventory-health color scale with Focus Builder (REQ-GRID-007)

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-03.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-GRID-* scenarios green; qty color scale reconciled; `npm run check` passes.

## Slice `SLICE-04` — `Guardrails management`

Status: Complete

Shipped: backend PR [#4](https://github.com/saadnaveed-amaskit/demo-backend/pull/4) (`bda55a9`) + frontend PR [#5](https://github.com/saadnaveed-amaskit/demo-frontend/pull/5) (`59d77c2`) merged to `main` 2026-07-09. Validation: `validation/SLICE-04.md` (PASS). Contract promoted to Stable: `contracts/slice-04-guardrails/contract.md`.

Target repos:
- frontend, backend

Depends on:
- SLICE-01

Requirements covered:
- REQ-GUARD-001…007

Acceptance scenarios covered:
- Inline-edit a guardrail threshold; Non-overridable guardrails are enforced

Contracts needed:
- yes — `knowledge/contracts/slice-04-guardrails/` (guardrail CRUD; live compliance evaluation)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `guardrails-management.feature` (inline edit, toggles, add/delete, enforcement) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for CRUD + live evaluation; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-04-guardrails/`; `GuardrailEntity` full CRUD (add/delete included — spec §8 gap)
- [ ] Live compliance evaluation endpoint (record versioning/audit decision — REQ-GUARD-007)

### Frontend Tasks
- [ ] Guardrails table (brand/division/rule/condition/value); inline value edit (RHF+Zod numeric validation)
- [ ] Active + overridable toggles; add/delete
- [ ] Surface active+overridable into scenario creation; enforce non-overridable active as hard constraints

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-04.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-GUARD-* scenarios green; CRUD persists; live evaluation returns pass/fail; `npm run check` passes.

## Slice `SLICE-05` — `Promotions calendar`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-02

Requirements covered:
- REQ-PROMO-001…008

Acceptance scenarios covered:
- Prevent an end date before the start date; Filter promotions by status with counts; View per-SKU promo price

Contracts needed:
- yes — `knowledge/contracts/slice-05-promotions/` (promo CRUD; per-SKU promo price; canonical Promotion/Discount entity — spec Open Q2)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `promotions-calendar.feature` (date validation, status filter counts, view products) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for promo CRUD + per-SKU price; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Decide/record canonical Promotion/Discount entity (spec Open Q2) before finalizing contract
- [ ] Contract in `contracts/slice-05-promotions/`; promo CRUD; per-SKU promo price computation; Focus Set link resolution

### Frontend Tasks
- [ ] Calendar (month grid, lane-packed bars) + List views; status tabs with live counts; month nav (date-fns)
- [ ] Detail drawer (badges, dates/duration, discount summary, linked Focus Set, View Products up to 20 SKUs w/ promo price, notes, edit/delete)
- [ ] Create/edit form (RHF+Zod): required fields, no past start, end strictly after start (auto-advance), searchable Focus Set + create shortcut

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-05.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-PROMO-* scenarios green; canonical discount entity decided; `npm run check` passes.

## Slice `SLICE-06` — `Discount Modeling`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-02

Requirements covered:
- REQ-DISC-001…010

Acceptance scenarios covered:
- Run Model gated on required fields; Open an existing model's output directly; Discard a model regardless of status

Contracts needed:
- yes — `knowledge/contracts/slice-06-discount-modeling/` (run-model → `DiscountModelOutput`; persistence + status; **v1 placeholder computation** — spec Open Q1)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `discount-modeling.feature` (run gating, open output, discard) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test asserting `DiscountModelOutput` shape; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-06-discount-modeling/`; run-model endpoint producing full `DiscountModelOutput` (placeholder computation behind stable shape)
- [ ] `DiscountModelEntity` persistence + status lifecycle (new/draft/pending/approved/returned/denied)

### Frontend Tasks
- [ ] List grouped New/Pending(incl. returned)/Approved/Denied; row click opens output directly
- [ ] Create form (RHF+Zod): name, Focus Set (est-SKU preview), dates, format (%/$/BOGO/Fixed; depth hidden for BOGO), channel; live preview + marketing handle
- [ ] Run gating; Output view: narrative + copyable handle, 5-KPI summary, 3 Nivo forecast charts, tabbed rollup, 6 risk panels; sell-through color coding; hard-vs-advisory escalation; submit/discard; SKU-detail CSV export + confidence

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-06.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-DISC-* scenarios green; output matches contract shape; `npm run check` passes.

## Slice `SLICE-07` — `Price Scenario optimization`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-02, SLICE-04

Requirements covered:
- REQ-SCEN-001…011

Acceptance scenarios covered:
- Objective sliders always sum to 100; Move the scenario position along the frontier; Freeze edits once submitted; Resubmit a returned scenario

Contracts needed:
- yes — `knowledge/contracts/slice-07-price-scenario/` (scenario run → frontier/uplift/per-SKU recs; live guardrail evaluation; **v1 placeholder** — spec Open Q1)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `price-scenario.feature` (weight-sum-100, slider bands, freeze, resubmit) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for scenario run output + guardrail eval; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-07-price-scenario/`; 6-phase run → frontier + uplift + per-SKU recs (placeholder)
- [ ] Live guardrail evaluation using SLICE-04 service; `ScenarioEntity` persistence + status/change-requests

### Frontend Tasks
- [ ] Create form: name, Focus Set, dates; three linked objective sliders summing to 100% (proportional redistribute); live preview + pre-filled overridable guardrails
- [ ] Nivo frontier chart (Current/ML Rec/Scenario points) + 0–100% optimization slider with risk bands + help modal
- [ ] 3-column comparison table + narrative + tagged recommendations; submit/discard/resubmit; freeze when pending/approved/denied; uncertainty framing

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-07.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-SCEN-* scenarios green; guardrail eval live; freeze/resubmit correct; `npm run check` passes.

## Slice `SLICE-08` — `Deep Dive`

Status: Complete

Shipped: frontend PR [#10](https://github.com/saadnaveed-amaskit/demo-frontend/pull/10) (`bcedd79`) + backend PR [#8](https://github.com/saadnaveed-amaskit/demo-backend/pull/8) (`506d508`) merged to `main` 2026-07-09. Validation: `validation/SLICE-08.md` (PASS, 1 patch attempt).

Target repos:
- frontend, backend

Depends on:
- SLICE-07

Requirements covered:
- REQ-DEEP-001…005

Acceptance scenarios covered:
- Explain a recommended price; Progressive unlock by optimization level

Contracts needed:
- yes — `knowledge/contracts/slice-08-deep-dive/` (per-SKU recommendation detail + explainability trace; placeholder v1)

Expected PRs:
- frontend PR [#10](https://github.com/saadnaveed-amaskit/demo-frontend/pull/10) ✓ merged
- backend PR [#8](https://github.com/saadnaveed-amaskit/demo-backend/pull/8) ✓ merged
- knowledge finalization commit ✓

### Acceptance-Test-First Tasks
- [x] Add `deep-dive.feature` (Explain modal, progressive unlock) before UI
- [x] Add Playwright step definitions; run and confirm expected failure
- [x] Add backend contract test for per-SKU detail + trace; run and confirm expected failure
- [x] Record expected failures in validation notes

### Backend Tasks
- [x] Contract in `contracts/slice-08-deep-dive/`; per-SKU recommendation detail + decision-ladder trace (placeholder)

### Frontend Tasks
- [x] Four tabs (All/Price Adjustments/Marketing/Discounts); AG Grid grouped columns + Products/SKUs toggle + CSV export
- [x] Explain modal (rationale, objectives, decision ladder, contextual factors)
- [x] Marketing/Discount tiles with expandable per-product lists; progressive unlock by optimization-level thresholds

### Validation Tasks
- [x] Run unit / integration / BDD / E2E vs real backend
- [x] Run `npm run check` + build (both repos)
- [x] Create validation report `validation/SLICE-08.md`

### Patcher Tasks
- [x] 1 patch attempt: fixed promotions ESLint + guardrails BDD timing

### Completion Criteria
- [x] REQ-DEEP-* scenarios green; unlock thresholds correct; `npm run check` passes.

## Slice `SLICE-09` — `Approvals queue`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-06, SLICE-07, SLICE-04

Requirements covered:
- REQ-APPR-001…012

Acceptance scenarios covered:
- Deny requires a reason; Return a discount model for changes requires a comment; Decided items move to a separate table

Contracts needed:
- yes — `knowledge/contracts/slice-09-approvals/` (approval state machine; live guardrail-compliance record; action-tracker linkback; one reason-required policy — spec Open Q5)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `approvals-queue.feature` (deny-reason, return-comment, decided table, drawers) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for approval state machine; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Decide/record consistent reason-required policy (spec Open Q5)
- [ ] Contract in `contracts/slice-09-approvals/`; approval state machine; risk derivation; guardrail-compliance record; cross-domain sub-action linkback

### Frontend Tasks
- [ ] Two tabs (Scenarios/Discounts) with pending-count badges; pending queue rows + risk badges + actions
- [ ] Deny (mandatory reason); scenario request-changes (optional) vs discount return-for-changes (mandatory) per policy
- [ ] Scenario view = full scenario output in approval mode; cross-domain sub-action drawer; discount drawer; decided table with read-only view

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-09.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-APPR-* scenarios green; reason policy consistent; state transitions correct; `npm run check` passes.

## Slice `SLICE-10` — `Agent roster`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-01

Requirements covered:
- REQ-AGENT-001…007

Acceptance scenarios covered:
- Pause and resume a monitor

Contracts needed:
- yes — `knowledge/contracts/slice-10-agents/` (roster + lifecycle + signals; canonical-version decision vs V2 fork — spec Open Q8; new Tier-1 entities — spec Open Q12)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `agent-roster.feature` (pause/resume monitor, hire agent) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for roster/lifecycle; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Decide/record canonical Agents version (spec Open Q8); define Monitor/Task-agent Tier-1 entities (Operators derived)
- [ ] Contract in `contracts/slice-10-agents/`; roster CRUD/lifecycle; signals feed

### Frontend Tasks
- [ ] KPI strip; three card sections (Monitors/Operators/Task agents)
- [ ] Monitor pause/resume + signals-today + last-activity; operator cards link to Autonomy; task-agent spawn/retire + deep link; hire from catalog

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-10.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-AGENT-* scenarios green; canonical version chosen; entities defined; `npm run check` passes.

## Slice `SLICE-11` — `Pricing Autonomy`

Status: Complete

Target repos:
- frontend, backend

Depends on:
- SLICE-10

Requirements covered:
- REQ-AUTO-001…010

Acceptance scenarios covered:
- Block promotion below the reversibility ceiling; Kill switch disables controls

Contracts needed:
- yes — `knowledge/contracts/slice-11-autonomy/` (action-class state; promotion-gate eval; live-action state machine; kill-switch semantics; audit — spec Open Q8/Q12)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `pricing-autonomy.feature` (promotion gating, kill-switch disables + freezes countdowns) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for promotion gates + state machine; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Decide/record kill-switch freeze semantics (spec Open Q10); action-class + live-action + audit Tier-1 entities
- [ ] Contract in `contracts/slice-11-autonomy/`; promotion-gate evaluation (ordered blockers); live-action state machine; kill switch; audit trail

### Frontend Tasks
- [ ] KPI strip; action-class cards + detail rail (proof panel, live-supervision veto/undo, audit feed)
- [ ] Promote (gated) / demote (always); veto/undo; kill switch disables all promote/veto/undo + freezes countdowns

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-11.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-AUTO-* scenarios green; gates + kill-switch semantics correct; `npm run check` passes.

## Slice `SLICE-12` — `Measurement`

Status: Approved

Target repos:
- frontend, backend

Depends on:
- SLICE-07

Requirements covered:
- REQ-MEAS-001…014

Acceptance scenarios covered:
- Go Live gated on balance and cost acknowledgment; Moving a cluster to control drops its ML price; Verdict banner on crossing a boundary

Contracts needed:
- yes — `knowledge/contracts/slice-12-measurement/` (cluster/block/balance; sequential estimator; outcome feedback; **v1 placeholder trajectories** — spec Open Q1/Q9)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `measurement.feature` (go-live gate, control drops ML price, verdict) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for cluster/block/balance + readout; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-12-measurement/`; cluster/block generation + balance computation; sequential readout (placeholder trajectories); experiment/outcome Tier-1 entities
- [ ] Outcome feedback hook to Autonomy accuracy (decide automatic vs manual — spec Open Q9)

### Frontend Tasks
- [ ] Setup: AI arm assignment, per-block balance (Nivo), cluster override (move/exclude), cost-of-control ack, Go-Live gate + tooltip
- [ ] Live: probability-of-winning trend + play/pause/restart; KPI tiles w/ credible interval; per-cluster contribution; verdict banner (gathering/win/kill) + scale/revert; reopen setup preserving overrides

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-12.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-MEAS-* scenarios green; go-live gating + balance + verdict correct; `npm run check` passes.

## Slice `SLICE-13` — `Like-Item Mapping (Cold Start)`

Status: Approved

Target repos:
- frontend, backend

Depends on:
- SLICE-02

Requirements covered:
- REQ-COLD-001…012

Acceptance scenarios covered:
- Validate the system's top pick; Override with a different candidate; Flag a close call

Contracts needed:
- yes — `knowledge/contracts/slice-13-like-items/` (matching + confidence scoring + mapping persistence + audit; **v1 placeholder matching** — spec Open Q1)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `like-item-mapping.feature` (validate, override w/ audit, close-call flag) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for matching/confidence/persistence; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-13-like-items/`; `ColdStartMappingEntity`; matching + weighted confidence scoring (placeholder); candidate-pool fallback; mapping persistence + audit
- [ ] Decide/record whether override reason is required (spec §13 gap)

### Frontend Tasks
- [ ] Queue grouped by category (pending/low-confidence counts, avg confidence, bulk accept); filters (division/dept/status/search); priority ordering
- [ ] Detail: summary banner, ranked candidates w/ System pick, confidence breakdown, attribute grid, borrowed priors, compare panel, impact preview; catalog search for alternate; validate/override (optional reason); close-call flag; audit line

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-13.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-COLD-* scenarios green; confidence tiers + audit correct; `npm run check` passes.

## Slice `SLICE-14` — `Configuration`

Status: Approved

Target repos:
- frontend, backend

Depends on:
- SLICE-01

Requirements covered:
- REQ-CONFIG-001 (REQ-CONFIG-002 enforcement only if access control brought into scope — spec Open Q6)

Acceptance scenarios covered:
- View user access list

Contracts needed:
- yes — `knowledge/contracts/slice-14-config/` (user list read model; CRUD/IdP/authorization only if in scope)

Expected PRs:
- frontend PR
- backend PR
- knowledge finalization commit after merge

### Acceptance-Test-First Tasks
- [ ] Add `configuration.feature` (read-only user list) before UI
- [ ] Add Playwright step definitions; run and confirm expected failure
- [ ] Add backend contract test for user list read model; run and confirm expected failure
- [ ] Record expected failures in validation notes

### Backend Tasks
- [ ] Contract in `contracts/slice-14-config/`; `AppUserEntity` read model (list)
- [ ] Access-control enforcement only if REQ-CONFIG-002 brought into scope (spec Open Q6)

### Frontend Tasks
- [ ] Read-only user list (name, role, active/inactive, brand/division access) via shadcn Table

### Validation Tasks
- [ ] Run unit / integration / BDD / E2E vs real backend
- [ ] Run `npm run check` + build (both repos)
- [ ] Create validation report `validation/SLICE-14.md`

### Patcher Tasks
- [ ] Up to 3 scoped patch attempts; record each; blocker report after 3

### Completion Criteria
- REQ-CONFIG-001 scenario green; `npm run check` passes.

## Global Validation Requirements

Before any slice PR is considered ready:

- New/changed acceptance tests were written and confirmed failing **before** implementation.
- Unit, integration (where applicable), BDD/Cucumber, and E2E (vs real backend) tests pass.
- `npm run check` (lint + `lint:policy` + typecheck + build) passes in each affected repo.
- Pinned-stack, data-taxonomy (Tier-1 `*Entity` vs Tier-2 constructs), naming/structure, and design-system compliance verified.
- No implemented scenario remains tagged `@wip`; no failing tests were deleted to pass.
- A validation report exists at `knowledge/specs/001-platform-baseline/validation/<slice-id>.md` mapping REQ-* ↔ scenarios/tests ↔ results (commands that did not run marked `Not available`).
- Patcher used ≤3 attempts; blocker report written if unresolved.

## Knowledge Finalization Tasks

After the slice's frontend/backend PRs merge (post-merge only):

- [ ] Mark the slice `Complete` in the Slice Index and its section.
- [ ] Promote the slice's contract from draft to stable in `knowledge/contracts/<slice-id>/`.
- [ ] Add merged frontend/backend PR links and commit SHAs.
- [ ] Update `knowledge/log.md`, wiki pages, and shipped-feature history.
- [ ] Record final validation status in the slice's validation report.
- [ ] Merge/update the `001-platform-baseline` knowledge branch to `main` per review.

## Speckit Analysis Findings

A manual analyze-equivalent alignment check was performed across spec.md, plan.md, tasks.md, and the constitution. **No blocking issues found.** The roadmap and all 15 slices were subsequently approved by Saad on 07/07/2026 (`status: Approved`).

| Check | Result | Notes |
|---|---|---|
| Requirements missing from tasks | Pass | All REQ groups (FOCUS, GRID, PROMO, DISC, SCEN, DEEP, GUARD, APPR, AGENT, AUTO, MEAS, COLD, CONFIG, SHELL) map to a slice. REQ-SHELL-003 and REQ-CONFIG-002 are explicitly deferred/conditional per spec Open Questions, not omitted. |
| Tasks not traceable to requirements | Pass | Every slice lists its covered REQ-* IDs; no orphan tasks beyond enabling scaffolding (SLICE-00). |
| Gherkin scenarios without test tasks | Pass | Each slice's acceptance-test-first tasks add `.feature` + step defs for its scenarios. |
| Missing acceptance-test-first tasks | Pass | Every user-facing slice writes failing acceptance tests before implementation; backend slices write failing contract tests first. |
| Missing contract-first tasks | Pass | Every full-stack slice defines a `knowledge/contracts/<slice-id>/` contract before backend implementation. |
| Spec/plan/tasks/constitution consistency | Pass | Slice set, dependencies, and repo responsibilities match plan; approval gates and patcher limit reflected. |
| Technology stack violations | Pass | Tasks reference pinned stack (shadcn/Nivo/Zustand/nuqs/RR7/RHF+Zod/Framer/date-fns/AG Grid) + `npm run check`/`lint:policy`. |
| Data taxonomy violations | Pass | Tier-1 `*Entity` backend-owned; Tier-2 constructs frontend-derived; new entities defined in slice contracts; no `*Entity` as UI state. |
| Slices too broad / not independently testable | Pass | 15 dependency-ordered, PR-sized slices, each with its own acceptance scenarios and completion criteria. |
| Unsafe scope expansion | Pass | No tasks beyond approved spec/plan; real ML, IdP, Marketing/Executive, "Ask anything" remain out of scope. |

**Non-blocking notes (resolve within the owning slice before its contract is finalized):** the 12 spec `[NEEDS CLARIFICATION]` items are each attached to a specific slice task (e.g. Focus Set resolution → SLICE-02; canonical discount entity → SLICE-05; reason-required policy → SLICE-09; Agents/Autonomy canonical version + kill-switch → SLICE-10/11; backend ORM/DB → SLICE-00). None block roadmap approval; each blocks only its slice's contract finalization.

---

_These tasks are `Approved` at 07/07/2026._
