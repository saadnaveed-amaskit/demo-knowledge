# SLICE-13 — Like-Item Mapping (Cold Start)

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

REQ-COLD-001…012.

## Acceptance Scenarios Covered

Validate the system's top pick; override with a different candidate; flag a close call (per `tasks.md`'s SLICE-13 section).

## Code Locations

[UNKNOWN] — not yet implemented in either repo. Route `/like-items` currently renders `ScreenPlaceholder` in `frontend/src/app/routes.tsx`.

## Frontend Notes

Not started.

## Backend Notes

Not started.

## API Contract Notes

No contract folder exists yet under `knowledge/contracts/`.

## Tests

None exist yet.

## Validation Summary

No validation report exists (`knowledge/specs/001-platform-baseline/validation/SLICE-13.md` does not exist).

## Known Limitations / Follow-ups

Depends on SLICE-02 (Focus Set management), which is Complete. Spec Open Question 11 (persona resolution — Pricing Team vs. Merchant for Like-Item) is unresolved.
