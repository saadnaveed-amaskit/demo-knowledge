---
status: Stable
artifact_type: api-contract
slice_id: SLICE-12
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q_resolution: >
  REQ-MEAS-012 (feed the causal result into the relevant pricing action
  class's accuracy record — spec Open Question 9, `[NEEDS CLARIFICATION:
  closed loop unbuilt today]`): remains unbuilt in this slice. No wiring
  exists between MeasurementService's readout/verdict and AutonomyService's
  ActionClassEntity accuracy fields. Deferred, consistent with the spec's
  own annotation and the SLICE-09 (REQ-APPR-005/008) deferred-scope
  precedent.

  Balance rule and verdict boundaries (REQ-MEAS-014): a block is "Balanced"
  when each of three metrics (revenue, grossMargin, velocity) has a
  min/max match ratio >= 0.8; otherwise "Imbalanced", or "Missing-an-arm"
  if the block lacks a treatment or control cluster. Win/kill boundary
  progression (day-over-day probability-of-winning trajectory) is a v1
  placeholder — only a static seeded "already live and past the win
  boundary" experiment exists; there is no scheduled/simulated tick
  advancing an experiment through its trajectory day by day.

  REQ-MEAS-008 (streaming daily trend with play/pause/restart) and
  REQ-MEAS-013 (reopen setup from live/complete, preserving overrides) are
  not implemented — neither has approved acceptance-test coverage from the
  preparation gate (only list/get/move/acknowledge-cost/go-live/scale/kill
  were scaffolded), so implementing them now would have exceeded the
  reviewed scope. See `knowledge/specs/001-platform-baseline/validation/SLICE-12.md`
  Remaining Blockers.
---

# Contract — SLICE-12: Measurement

## Base path

`/measurement`

## Entities / Views

Naming note: this contract uses `*View` suffixes throughout (`ExperimentView`, `ClusterView`, `BlockView`, `ReadoutView`) rather than a `*Entity` for the top-level experiment record. This naming was locked in by the approved SLICE-12 preparation-gate contract test (`measurement.controller.spec.ts` imports `ExperimentView`/`ClusterView` by name) and mirrors the existing `OperatorView`-without-a-separate-entity precedent from SLICE-10's Agents module. Internally, `MeasurementService` stores data in unexported `StoredExperiment`/`StoredBlock`/`StoredCluster` shapes and maps them to these `*View` read projections on every read (the same `Stored*` → `toView()` pattern used by `AutonomyService`).

### ExperimentView (Tier-1, read projection)

| Field | Type | Notes |
|---|---|---|
| id | number | |
| name | string | |
| status | `"setup" \| "live" \| "concluded"` | |
| costAcknowledged | boolean | REQ-MEAS-005 |
| createdAt | string | ISO timestamp |
| blocks | BlockView[] | |
| goLiveEligible | boolean | computed fresh on every read (REQ-MEAS-006) |
| goLiveBlockedReason | string \| null | non-empty when `goLiveEligible` is false |
| readout | ReadoutView \| null | null until the experiment has gone live |

### BlockView

| Field | Type | Notes |
|---|---|---|
| id | number | |
| label | string | |
| status | `"Balanced" \| "Imbalanced" \| "Missing-an-arm"` | computed from `metricMatch` ratios + arm presence (REQ-MEAS-003/014) |
| clusters | ClusterView[] | |
| metricMatch? | MetricMatch | optional solely so the pre-existing approved contract test's block-literal mocks (written before this field was designed) still type-check unmodified — the real endpoint always populates it |

### MetricMatch

| Field | Type | Notes |
|---|---|---|
| revenue | number | 0-1 match ratio |
| grossMargin | number | 0-1 match ratio |
| velocity | number | 0-1 match ratio |

### ClusterView

| Field | Type | Notes |
|---|---|---|
| id | number | |
| blockId | number | |
| name | string | |
| arm | `"treatment" \| "control"` | |
| bauPrice | number | |
| mlPrice | number \| null | null while `arm === "control"` (REQ-MEAS-007); the underlying ML recommendation is retained internally and restored if moved back to `"treatment"` |
| crossElasticity | number | |
| confidence | number | 0-1 |

### ReadoutView

| Field | Type | Notes |
|---|---|---|
| probabilityOfWinning | number | 0-1 |
| day | number | observation day |
| verdict | `"gathering" \| "win" \| "kill"` | REQ-MEAS-011 |
| incrementalMargin | CredibleInterval | |
| clusterContributions | ClusterContribution[] | REQ-MEAS-010 |

### CredibleInterval

| Field | Type | Notes |
|---|---|---|
| estimate | number | |
| lower | number | |
| upper | number | |

### ClusterContribution

| Field | Type | Notes |
|---|---|---|
| clusterId | number | |
| name | string | |
| contribution | number | |

### MoveClusterDto (request body)

```typescript
interface MoveClusterDto {
  arm: "treatment" | "control"
}
```

## Balance Rule (REQ-MEAS-014)

```typescript
const MIN_BALANCE_RATIO = 0.8
// Balanced: block has both a treatment and control cluster, AND
// metricMatch.revenue/grossMargin/velocity are each >= MIN_BALANCE_RATIO.
// Missing-an-arm: block lacks a treatment or control cluster (e.g. after
// moving a block's only treatment cluster to control).
// Imbalanced: has both arms but at least one metric ratio < 0.8.
```

## Go-Live Gate (REQ-MEAS-005/006)

Blocked (400) unless: experiment status is `"setup"`, every block is `"Balanced"`, and `costAcknowledged` is `true`. On success: `status` becomes `"live"`, and `readout` initializes to `{ probabilityOfWinning: 0.5, day: 1, verdict: "gathering", incrementalMargin: {0,0,0}, clusterContributions: [] }` (v1 placeholder trajectory).

## Endpoints

### GET /measurement/experiments

**Response 200:** `ExperimentView[]`

### GET /measurement/experiments/:id

**Response 200:** `ExperimentView`
**Response 404:** experiment not found

### POST /measurement/experiments/:id/clusters/:clusterId/move

**Body:** `MoveClusterDto`
**Response 200:** `ClusterView` (the moved cluster only, not the whole experiment)
**Response 400:** `arm` missing or not exactly `"treatment"`/`"control"`
**Response 404:** experiment or cluster not found

### POST /measurement/experiments/:id/acknowledge-cost

**Response 200:** `ExperimentView`
**Response 404:** experiment not found

### POST /measurement/experiments/:id/go-live

**Response 200:** `ExperimentView` (`status: "live"`, `readout` initialized)
**Response 400:** not all blocks balanced and/or cost not acknowledged, or experiment not in `"setup"` status
**Response 404:** experiment not found

### POST /measurement/experiments/:id/scale

**Response 200:** `ExperimentView` (`status: "concluded"`; cluster arms unchanged — scaling the win beyond tested clusters is not modeled in this v1 placeholder)
**Response 400:** experiment status is not `"live"`
**Response 404:** experiment not found

### POST /measurement/experiments/:id/kill

**Response 200:** `ExperimentView` (`status: "concluded"`; every `"treatment"` cluster across all blocks moved to `"control"`, reverting to BAU)
**Response 400:** experiment status is not `"live"`
**Response 404:** experiment not found

## Seed Data

3 experiments seeded in-memory at process start:
- id 1 — "setup", 2 blocks (one Balanced, one Imbalanced), cost not acknowledged
- id 2 — "setup", 1 block (Balanced) with a treatment cluster carrying an `mlPrice`, cost not acknowledged
- id 3 — "live", 1 block (Balanced), readout already past the win boundary (`verdict: "win"`, `probabilityOfWinning: 0.97`)

## Known Gaps

- REQ-MEAS-008 (streaming daily probability trend with play/pause/restart) is not implemented — the live readout is a static current-state snapshot, not an animated/interactive trend. No approved acceptance scenario requires it.
- REQ-MEAS-013 (reopen setup from live/complete, preserving overrides) has no dedicated endpoint — not part of the approved preparation-gate test scaffolding (Jest mock covers only list/get/move/acknowledge-cost/go-live/scale/kill).
- REQ-MEAS-012 (closed-loop feedback into Autonomy's `ActionClassEntity` accuracy record) is unbuilt, per the spec's own `[NEEDS CLARIFICATION]` annotation.
- Verdict boundary progression (day-over-day trajectory toward win/kill) is a v1 deterministic placeholder; only the pre-seeded "already past the win boundary" experiment exercises the win-verdict path.
- Estimated cost-of-control $ and time-to-signal (REQ-MEAS-002) are not separately computed fields — the KPI strip surfaces the boolean acknowledgment state instead.
- `BlockView.metricMatch` is typed optional solely so the pre-existing approved contract test's mocks (written before this field was designed) still type-check unmodified — the real endpoint always populates it, same pattern as SLICE-11's `AutonomyRosterView.liveActions`.
