# Code Wiki Index

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| knowledge | 001-platform-baseline | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend | main | a6df30032fabb22b0f07263469a0c787b0efd12b |

## Architecture

- [Overview](architecture/overview.md)

## Codebase

- [Frontend Structure](codebase/frontend-structure.md)
- [Backend Structure](codebase/backend-structure.md)
- [Dependency Map](codebase/dependency-map.md)

## API

- [Contract Summary](api/contract-summary.md)

## Features and Slices

| Slice | Title | Status |
|---|---|---|
| [SLICE-00](features/SLICE-00.md) | Repository scaffolding & quality gates | Complete |
| [SLICE-01](features/SLICE-01.md) | Platform shell & navigation | Complete |
| [SLICE-02](features/SLICE-02.md) | Focus Set management | Complete |
| [SLICE-03](features/SLICE-03.md) | Product Grid | Complete |
| [SLICE-04](features/SLICE-04.md) | Guardrails management | Complete |
| [SLICE-05](features/SLICE-05.md) | Promotions calendar | Complete |
| [SLICE-06](features/SLICE-06.md) | Discount Modeling | Complete |
| [SLICE-07](features/SLICE-07.md) | Price Scenario optimization | Complete |
| [SLICE-08](features/SLICE-08.md) | Deep Dive | Complete |
| [SLICE-09](features/SLICE-09.md) | Approvals queue | Complete |
| [SLICE-10](features/SLICE-10.md) | Agent roster | Approved — not yet implemented |
| [SLICE-11](features/SLICE-11.md) | Pricing Autonomy | Approved — not yet implemented |
| [SLICE-12](features/SLICE-12.md) | Measurement | Approved — not yet implemented |
| [SLICE-13](features/SLICE-13.md) | Like-Item Mapping (Cold Start) | Approved — not yet implemented |
| [SLICE-14](features/SLICE-14.md) | Configuration | Approved — not yet implemented |

## Wiki Log

- [Wiki Log](log.md)

## Known Gaps

- `backend/contracts/api-contract.yaml` was deleted from `main` (commit `bb1a951`) after being added; only `backend/contracts/api-contract.md` currently exists.
- `backend/contracts/api-contract.md`'s own embedded Source table has not been regenerated since the SLICE-09 merge and still shows the pre-merge feature branch/commit.
- SLICE-10 through SLICE-14 are `Approved` in `knowledge/specs/001-platform-baseline/tasks.md` but have no implementation, contract, or validation report yet.
- A pre-existing, unrelated file `knowledge/wiki/shipped-features.md` (predating this templated wiki, covering only SLICE-00/01) was left untouched — it is not part of the generated wiki structure and may be superseded by this index/log going forward.
