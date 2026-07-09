# SLICE-11 — Pricing Autonomy

## Status

Approved — not yet implemented

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Merged PRs

None — not yet implemented.

## Requirements Covered

REQ-AUTO-001…010.

## Acceptance Scenarios Covered

Block promotion below the reversibility ceiling; kill switch disables controls (per `tasks.md`'s SLICE-11 section).

## Code Locations

[UNKNOWN] — not yet implemented in either repo. Route `/autonomy` currently renders `ScreenPlaceholder` in `frontend/src/app/routes.tsx`.

## Frontend Notes

Not started.

## Backend Notes

Not started.

## API Contract Notes

No contract folder exists yet under `knowledge/contracts/`. REQ-AUTO-010 flags an open decision on exact kill-switch countdown-freeze semantics and canonical version vs. the V2 fork (spec Open Question 8).

## Tests

None exist yet.

## Validation Summary

No validation report exists (`knowledge/specs/001-platform-baseline/validation/SLICE-11.md` does not exist).

## Known Limitations / Follow-ups

Depends on SLICE-10 (Agent roster), which is also not yet implemented.
