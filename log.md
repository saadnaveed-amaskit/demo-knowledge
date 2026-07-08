# Knowledge Log

Chronological record of shipped platform work. Newest first.

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
