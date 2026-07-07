---
artifact_type: platform-speckit-context
platform_baseline: true
speckit_feature_id: 001-platform-baseline
speckit_branch: 001-platform-baseline
speckit_feature_dir: knowledge/specs/001-platform-baseline
spec_path: knowledge/specs/001-platform-baseline/spec.md
plan_path: knowledge/specs/001-platform-baseline/plan.md
tasks_path: knowledge/specs/001-platform-baseline/tasks.md
created_by: /create-platform-spec
---

# Platform Baseline Context

The **canonical platform baseline** for Retail Nucleus lives in the native-Speckit feature directory:

- **Feature ID:** `001-platform-baseline`
- **Speckit branch:** `001-platform-baseline` (created by `.specify/scripts/powershell/create-new-feature.ps1`)
- **Feature directory:** `knowledge/specs/001-platform-baseline/`
- **Spec:** `knowledge/specs/001-platform-baseline/spec.md` (exists — `status: Approved`, Saad 07/07/2026)
- **Plan:** `knowledge/specs/001-platform-baseline/plan.md` (exists — `status: Draft`, created via `/create-platform-plan`)
- **Tasks:** `knowledge/specs/001-platform-baseline/tasks.md` (not yet created — via `/create-platform-tasks`)

Future platform commands (`/create-platform-plan`, `/create-platform-tasks`, slice implementation, validation, finalization) **must use this context file** to locate the platform spec, plan, tasks, and Speckit branch. Do not assume the old fixed-path convention.

## History / migration note

An earlier, fixed-path platform baseline existed at `knowledge/specs/000-platform/` (`spec.md` + `plan.md`), authored directly and approved by the human. It was intentionally removed by the human in commit `cd2395e "Remove old files before Speckit-native"` to migrate to native Speckit infrastructure. **`001-platform-baseline` supersedes `000-platform`.** The `000-platform` directory no longer exists on `main`; its content history remains in git.

The requirements analysis at `knowledge/reports/platform-requirements-analysis.md` remains valid and was a source input.

## Branch note

Because native Speckit created the `001-platform-baseline` branch in the `knowledge/` repo, subsequent platform artifact work for this baseline occurs on that branch until it is reviewed/merged. This branch creation is expected and authorized by `/create-platform-spec`; it is not an "unexpected branch state" blocker for this baseline workflow.
