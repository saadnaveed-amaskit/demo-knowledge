# Knowledge Log

Chronological record of shipped platform work. Newest first.

## 2026-07-09 — SLICE-03: Product Grid (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#4](https://github.com/saadnaveed-amaskit/demo-frontend/pull/4) merged to `main` — merge SHA `36d559c`. ProductGrid screen (AG Grid community v33): product-level and SKU-level view toggle, soft-delete to Deleted Items pane, Restore / Restore All, `data-testid="active-sku-count"` live counter, Open-in-Grid drill button from Focus Builder cards. Routes `/product-grid?focus=<id>` wired.
- **backend:** PR [#3](https://github.com/saadnaveed-amaskit/demo-backend/pull/3) merged to `main` — merge SHA `67df4a3`. ProductGridController (4 endpoints: getGrid, excludeSku, restoreSku, restoreAll). ProductGridService with in-memory exclusions Map per focusSetId and product-rollup grouping by productId. CatalogSku extended with productId/productName/msrp; 3 new multi-SKU products added (15 total SKUs across 13 products).
- **Requirements:** REQ-PG-01…08 covered.
- **Validation:** `specs/001-platform-baseline/validation/SLICE-03.md` — PASS (25 backend tests, 10 BDD scenarios — 2 new product-grid + 8 existing; `npm run check` green on both repos).
- **Contracts promoted:** `contracts/slice-03-product-grid/contract.md` Draft → **Stable**.
- **Open items carried forward:** ORM/DB `[NEEDS CLARIFICATION]`; exclusions reset on backend restart (in-memory); AG Grid bundle contribution (advisory).
- **Next:** SLICE-04 (Guardrails management) or SLICE-05 (Promotions calendar) — both approved, SLICE-04 depends on SLICE-01 ✓.

## 2026-07-09 — SLICE-02: Focus Set management (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#3](https://github.com/saadnaveed-amaskit/demo-frontend/pull/3) merged to `main` — merge SHA `9a2fa8c`. Focus Builder screen: library panel (nuqs `?q=` search), form panel (create/edit), ConditionTree (AND/OR, ≤3 levels, shadcn Select), live preview (debounced resolve → count + 8 SKUs), CSV export, Zustand `useFocusBuilderStore`.
- **backend:** PR [#2](https://github.com/saadnaveed-amaskit/demo-backend/pull/2) merged to `main` — merge SHA `aab22b9`. FocusSetsController (8 endpoints), FocusSetsService (in-memory Map, sequential IDs), condition.ts (recursive AND/OR matching, MAX_DEPTH=3), CatalogController/Service (attribute enumeration from 12-SKU seed).
- **Requirements:** REQ-FOCUS-001…012 (all covered; REQ-FOCUS-010 duplicate-UI deferred to later slice).
- **Validation:** `specs/001-platform-baseline/validation/SLICE-02.md` — PASS (14 backend tests, 8 BDD scenarios; `npm run check` green on both repos).
- **Contracts promoted:** `contracts/slice-02-focus-sets/contract.md` Draft → **Stable**.
- **Open Q3 resolved:** resolution strategy = **live** (no snapshot/cache).
- **Open items carried forward:** ORM/DB `[NEEDS CLARIFICATION]`; duplicate UI not exposed; Vite chunk >500 kB (pre-existing advisory).
- **Next:** SLICE-03 (Product Grid) or SLICE-04 (Guardrails) — depends on SLICE-02 ✓.

## 2026-07-08 — SLICE-01: Platform shell & navigation (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#2](https://github.com/saadnaveed-amaskit/demo-frontend/pull/2) merged to `main` — merge `a9f4221`, feature `3ba29f0`. React Router v7 routes for all screens; persona-grouped sidebar; global brand/channel filters URL-synced via nuqs + mirrored to Zustand; shadcn Select/Button + zinc/teal theme; Framer Motion transitions.
- **backend:** not affected (frontend-only).
- **Requirements:** REQ-SHELL-001, REQ-SHELL-002 (REQ-SHELL-003 deferred).
- **Validation:** `specs/001-platform-baseline/validation/SLICE-01.md` — PASS (ATF red→green; `npm run check` green).
- **Contracts promoted:** none.
- **Open items:** bundle size >500 kB (advisory).
- **Next:** SLICE-02 (Focus Set management) — first full-stack, contract-first slice.

## 2026-07-07 — SLICE-00: Repository scaffolding & quality gates (SHIPPED)

- **Baseline:** `001-platform-baseline` (native Speckit)
- **Slice status:** Complete
- **frontend:** PR [#1](https://github.com/saadnaveed-amaskit/demo-frontend/pull/1) merged to `main` — merge `3687301`, feature `e4e62e1`. React 19 + TS + Vite + pinned stack; Vitest/RTL; Cucumber+Playwright BDD; `npm run check` (eslint + `lint:policy` + typecheck + build).
- **backend:** PR [#1](https://github.com/saadnaveed-amaskit/demo-backend/pull/1) merged to `main` — merge `21acafa`, feature `767cb7f`. NestJS 11 + TS; Jest; health endpoint; in-memory repository scaffold; `npm run check`.
- **Validation:** `specs/001-platform-baseline/validation/SLICE-00.md` — PASS (acceptance-test-first red→green in both repos).
- **Contracts promoted:** none (scaffolding slice).
- **Open items carried forward:** backend ORM/DB choice `[NEEDS CLARIFICATION]`; npm audit advisories in dev-deps.
- **Next:** SLICE-01 (Platform shell & navigation).
