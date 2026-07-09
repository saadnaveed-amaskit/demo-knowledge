# Wiki Log

Chronological record of `/update-code-wiki` runs. Newest first.

### 2026-07-09 — update-code-wiki-baseline-1

| Field | Value |
|---|---|
| mode | full baseline (`--all`) |
| knowledge commit | 0c45b62ab4a00253b6faf1129cde58d84145eb7d |
| frontend commit | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |
| backend commit | a6df30032fabb22b0f07263469a0c787b0efd12b |
| API contract changed | No — `backend/contracts/api-contract.md` was read, not modified, by this run |
| validation report | N/A (this run reads validation reports; it does not create or update any) |

Updated wiki files:

- knowledge/wiki/index.md (created)
- knowledge/wiki/log.md (created)
- knowledge/wiki/architecture/overview.md (created)
- knowledge/wiki/codebase/frontend-structure.md (created)
- knowledge/wiki/codebase/backend-structure.md (created)
- knowledge/wiki/codebase/dependency-map.md (created)
- knowledge/wiki/api/contract-summary.md (created)
- knowledge/wiki/features/SLICE-00.md (created)
- knowledge/wiki/features/SLICE-01.md (created)
- knowledge/wiki/features/SLICE-02.md (created)
- knowledge/wiki/features/SLICE-03.md (created)
- knowledge/wiki/features/SLICE-04.md (created)
- knowledge/wiki/features/SLICE-05.md (created)
- knowledge/wiki/features/SLICE-06.md (created)
- knowledge/wiki/features/SLICE-07.md (created)
- knowledge/wiki/features/SLICE-08.md (created)
- knowledge/wiki/features/SLICE-09.md (created — status `Merged; knowledge finalization pending`)
- knowledge/wiki/features/SLICE-10.md (created — `Approved`, not yet implemented)
- knowledge/wiki/features/SLICE-11.md (created — `Approved`, not yet implemented)
- knowledge/wiki/features/SLICE-12.md (created — `Approved`, not yet implemented)
- knowledge/wiki/features/SLICE-13.md (created — `Approved`, not yet implemented)
- knowledge/wiki/features/SLICE-14.md (created — `Approved`, not yet implemented)

Templates: all 8 required templates under `knowledge/wiki/_templates/` already existed (pre-dating this run) and were reused as-is; none were created or overwritten.

Known gaps:

- SLICE-09 is merged to `main` in both frontend and backend but not yet finalized in the knowledge repo (`tasks.md` status still `In Review`, validation report front matter still `Draft`) — `/finalize-knowledge-after-merge SLICE-09` is recommended as a follow-up.
- `backend/contracts/api-contract.yaml` was deleted from `main` (commit `bb1a951`) after being added for SLICE-09; only `api-contract.md` currently exists.
- `api-contract.md`'s own embedded Source table has not been regenerated since the SLICE-09 merge and still shows the pre-merge feature branch/commit (`agent/slice-09-approvals-queue` @ `f192964`) rather than current `main` (`a6df300`).
- SLICE-10 through SLICE-14 have approved requirements/task sections but no implementation, contract, or validation report.
- A pre-existing file `knowledge/wiki/shipped-features.md` (not part of the templated structure, covering only SLICE-00/01) was left untouched.
