# Knowledge Log

Chronological record of shipped platform work. Newest first.

## 2026-07-09 — SLICE-06: Discount Modeling (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#8](https://github.com/saadnaveed-amaskit/demo-frontend/pull/8) merged to `main` — merge SHA `8b6f4a452e40ef16b4dd58eb1101b0ade005cfb4`. DiscountModelingScreen at `/discount-modeling`: grouped model list (new/draft/pending/approved/denied/returned); CreateForm (RHF+Zod): name, focus group async select, date range, format, depth/BOGO, channel; Run Model gated by `isRunEnabled`; OutputView: narrative, copyable marketing handle, 5 KPIs, ResponsiveLine (revenue + margin forecasts), ResponsiveBar (units forecast), rollup table (By Division / By Channel tabs + CSV export), 6 collapsible risk panels with hard-violation banner; sell-through color coding ≥85% teal / ≥70% amber / <70% red.
- **backend:** PR [#6](https://github.com/saadnaveed-amaskit/demo-backend/pull/6) merged to `main` — merge SHA `0b6d58450fac7ba2970511d23eaf807505a5a097`. DiscountModelingController (7 endpoints: list, get, create, run, submit, updateStatus, delete 204). DiscountModelingService with in-memory store; deterministic placeholder computation (SKU count × avgPrice=29.99 × rate × unitLiftFactor=1.2); 5 division rollup rows; 6 risk panels; marketing handle format `TCP-PCT-20-DIG-SUMMER-202607`; status lifecycle: new → draft → pending → approved/denied/returned. `FocusSetsService` injected for `productCount` lookup.
- **Requirements:** REQ-DISC-001…010 covered. Open Q1 resolved: deterministic placeholder behind stable contract shape; real ML deferred.
- **Validation:** `specs/001-platform-baseline/validation/SLICE-06.md` — PASS (80 backend tests, 18/18 BDD scenarios — 3 new + 15 existing; `npm run check` + build green on both repos). 0 patch attempts.
- **Contracts promoted:** `contracts/slice-06-discount-modeling/contract.md` Draft → **Stable**.
- **Open items carried forward:** ORM/DB `[NEEDS CLARIFICATION]`; in-memory store resets on restart; real ML model deferred (locked contract shape); SLICE-09 (Approvals queue) now unblocked by SLICE-06 completing.
- **Next:** SLICE-07 (Price Scenario optimization) or SLICE-08 (Deep Dive) — both approved. SLICE-09 now also unblocked.

## 2026-07-09 — SLICE-05: Promotions calendar (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#7](https://github.com/saadnaveed-amaskit/demo-frontend/pull/7) merged to `main` — merge SHA `30ebbccc0ec41dda9f9d380f6a615f532f714635`. PromotionsScreen at `/promos`: list view + mini calendar toggle; status filter tabs (All/Active/Scheduled/Expired) with live counts; PromoForm (RHF+Zod, end-after-start validation); PromoDetail drawer with status badge, discount summary, "View Products" button; per-SKU promo price table (≤20 SKUs, `data-testid="promo-product-row"`, `data-testid="promo-price"`). date-fns calendar grid with colored promotion bars.
- **backend:** PR [#5](https://github.com/saadnaveed-amaskit/demo-backend/pull/5) merged to `main` — merge SHA `1ec578722d9fbd4eefd5d81d25f7b8258c069f0c`. PromotionsController (5 endpoints: list, create, update, delete 204, getProducts). PromotionsService with in-memory store (no seed — BDD-driven). Status (`active`/`scheduled`/`expired`) derived at query time from dates; `BadRequestException` when `endDate ≤ startDate`. `GET /:id/products` resolves Focus Set link, filters catalog SKUs (≤20), computes promo price via percentage or flat discount.
- **Requirements:** REQ-PROMO-001…008 covered. Open Q2 resolved: canonical entity is `PromotionEntity` (pricing domain, distinct from marketing `PromoEntity`).
- **Validation:** `specs/001-platform-baseline/validation/SLICE-05.md` — PASS (59 backend tests, 15/15 BDD scenarios — 3 new promotions + 12 existing; `npm run check` + build green on both repos). 0 patch attempts.
- **Contracts promoted:** `contracts/slice-05-promotions/contract.md` Draft → **Stable**.
- **Open items carried forward:** ORM/DB `[NEEDS CLARIFICATION]`; promotion store resets on backend restart (in-memory); promo price does not account for stacking/priority rules (out of scope for this slice).
- **Next:** SLICE-06 (Discount Modeling) or SLICE-07 (Price Scenario optimization) — both approved.

## 2026-07-09 — SLICE-04: Guardrails management (SHIPPED)

- **Baseline:** `001-platform-baseline`
- **Slice status:** Complete
- **frontend:** PR [#5](https://github.com/saadnaveed-amaskit/demo-frontend/pull/5) merged to `main` — merge SHA `59d77c2`. GuardrailsScreen: management table (brand/division/rule/op/value/unit columns), inline value edit (RHF+Zod), active/overridable Switch toggles, add/delete, hard-constraint badge (amber) for active+non-overridable guardrails, enforcement preview panel (Pricing Team read-only view). shadcn Switch component added. Route `/guardrails` wired.
- **backend:** PR [#4](https://github.com/saadnaveed-amaskit/demo-backend/pull/4) merged to `main` — merge SHA `bda55a9`. GuardrailsController (7 endpoints: list, create, update, toggleActive, toggleOverridable, remove, evaluate). GuardrailsService with 4-row in-memory seed. `POST /guardrails/evaluate` returns per-guardrail severity (hard/advisory) for live compliance checking.
- **Requirements:** REQ-GUARD-001…007 covered. REQ-GUARD-005 enforcement in scenario creation deferred to SLICE-07. REQ-GUARD-007 audit/versioning deferred (`[NEEDS CLARIFICATION]`).
- **Validation:** `specs/001-platform-baseline/validation/SLICE-04.md` — PASS (42 backend tests, 12 BDD scenarios — 2 new guardrails + 10 existing; `npm run check` green on both repos).
- **Contracts promoted:** `contracts/slice-04-guardrails/contract.md` Draft → **Stable**.
- **Open items carried forward:** ORM/DB `[NEEDS CLARIFICATION]`; guardrail store resets on backend restart; REQ-GUARD-007 versioning/audit; full REQ-GUARD-005 enforcement in SLICE-07.
- **Next:** SLICE-05 (Promotions calendar) or SLICE-07 (Price Scenario optimization, now unblocked by SLICE-04).

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
