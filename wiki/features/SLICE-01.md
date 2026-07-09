# SLICE-01 — Platform shell & navigation

## Status

Complete

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

- frontend: [#2](https://github.com/saadnaveed-amaskit/demo-frontend/pull/2) (`a9f4221`)
- backend: — (not affected; this slice is frontend-only)

## Requirements Covered

REQ-SHELL-001, REQ-SHELL-002 (REQ-SHELL-003 — the "Ask anything" conversational surface — deferred/stub per spec Open Question 7).

## Acceptance Scenarios Covered

Sidebar navigation renders grouped by persona and routes resolve; global brand/channel filters are present in the app shell header.

## Code Locations

- `frontend/src/app/AppShell.tsx`, `frontend/src/app/routes.tsx`, `frontend/src/app/nav.ts`, `frontend/src/app/GlobalFilters.tsx`
- `frontend/src/state.ts` (`useRNState` — global brand/channel state)
- `frontend/features/app-shell-navigation.feature`, `frontend/tests/bdd/steps/app-shell-navigation.steps.ts`
- `frontend/features/app-boots.feature`, `frontend/tests/bdd/steps/app-boots.steps.ts`

## Frontend Notes

Establishes the persona-grouped sidebar (Pricing Team, Pricing Strategist, Admin — see `NAV_SECTIONS` in `nav.ts`) and the `IMPLEMENTED` route map in `routes.tsx` that every later screen slice registers into.

## Backend Notes

Not applicable — this slice does not touch the backend.

## API Contract Notes

None — no contract needed for this slice per `tasks.md`.

## Tests

- `features/app-boots.feature` / `tests/bdd/steps/app-boots.steps.ts` — application boots
- `features/app-shell-navigation.feature` / `tests/bdd/steps/app-shell-navigation.steps.ts` — shell and navigation
- `src/App.test.tsx` — nav sections and heading render (Vitest)

## Validation Summary

Per `knowledge/specs/001-platform-baseline/validation/SLICE-01.md`: **PASS**, ready for PR review (frontend only).

## Known Limitations / Follow-ups

REQ-SHELL-003 ("Ask anything" surface) remains a non-functional stub per spec Open Question 7 ("Keep it as a stub for now").
