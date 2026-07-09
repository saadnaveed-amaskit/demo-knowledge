---
status: Draft
artifact_type: api-contract
slice_id: SLICE-10
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q1_resolution: >
  REQ-AGENT-007 (canonical Agents implementation vs. the Pricing Platform V2
  Agentic fork, spec Open Q8): resolved by definition. The V2 fork
  (`/pricing-platform-v2/*`) is a separate app entirely outside this
  workspace (only `knowledge/`, `frontend/`, `backend/` exist here, and this
  RetailNucleus app is scoped to `/retail-nucleus/*` per spec.md line 44).
  There is no competing implementation in this workspace to reconcile
  against, so the Agents roster built in this slice is canonical by
  definition — no human decision was required.

  New Tier-1 entities (spec Open Q12): `MonitorEntity` and `TaskAgentEntity`
  are new canonical Tier-1 entities, defined fresh in this contract (no
  prior DATA-TAXONOMY.md entry existed for either). Per plan.md's guidance,
  Operators are explicitly "derived", not a stored Tier-1 entity — modeled
  here as `OperatorView` (Tier-2), computed from a fixed placeholder seed
  since Autonomy (SLICE-11, the system that would supply real trust-ladder
  data) does not exist yet. This mirrors the deterministic-placeholder
  precedent from spec Open Q1.

  Task agents are seeded, not spawned: no live "scenario submitted" or
  similar trigger exists yet to programmatically spawn a task agent, so v1
  seeds 2 example task agents deterministically rather than exposing a
  create endpoint. A future slice that wires real spawning (e.g. from
  SLICE-07/09 events) should add a creation path without changing this
  contract's read/retire shape.
---

# Contract — SLICE-10: Agent Roster

## Base path

`/agents`

## Entities

### MonitorEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | auto-increment |
| name | string | display name, equals `type` in v1 (no custom naming yet) |
| type | string | one of the provisionable monitor types, see Catalog below |
| status | `"active" \| "paused"` | toggled via pause/resume |
| signalsToday | number | placeholder counter, fixed at hire time, not live-incrementing |
| lastActivity | string | ISO timestamp, updated on pause/resume |
| createdAt | string | ISO timestamp |

### OperatorView (Tier-2, derived — not persisted as its own entity)

| Field | Type | Notes |
|---|---|---|
| id | number | |
| name | string | equals `type` |
| type | string | one of the provisionable operator types, see Catalog below |
| trustLevel | `"Low" \| "Medium" \| "High"` | v1 placeholder — real value would derive from SLICE-11 (Autonomy) trust-ladder data, which does not exist yet |
| evidenceStatus | `"evidence-backed" \| "unproven"` | v1 placeholder |
| trackRecord | string | human-readable placeholder summary, e.g. "18/20 accurate over 90 days" |

### TaskAgentEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | auto-increment |
| name | string | |
| spawnedBy | string | human-readable description of what created it (v1: seeded, not programmatically spawned) |
| retirementCondition | string | human-readable description of when it retires |
| status | `"running" \| "retired"` | |
| openLink | string | a route path in this app the "open" deep link navigates to (e.g. `/scenario`) |
| createdAt | string | ISO timestamp |

### AgentKpis (Tier-2)

| Field | Type | Notes |
|---|---|---|
| agentsOnTeam | number | `monitors.length + operators.length` |
| signalsToday | number | sum of `monitor.signalsToday` across all monitors |
| actingAutonomously | number | count of operators (v1: all seeded operators are treated as acting autonomously) |
| evidenceBackedCount | number | count of operators with `evidenceStatus === "evidence-backed"` |
| evidenceBackedTotal | number | `operators.length` |
| taskAgentsRunning | number | count of task agents with `status === "running"` |

### AgentRosterView (Tier-2)

| Field | Type | Notes |
|---|---|---|
| kpis | AgentKpis | |
| monitors | MonitorEntity[] | all monitors, seeded + hired |
| operators | OperatorView[] | all operators, seeded + hired |
| taskAgents | TaskAgentEntity[] | all task agents, seeded only in v1 |

### AgentCatalogView (Tier-2)

| Field | Type | Notes |
|---|---|---|
| monitorTypes | string[] | fixed list of provisionable monitor types |
| operatorTypes | string[] | fixed list of provisionable operator types |

```typescript
const MONITOR_TYPES = [
  "Price Drift Monitor",
  "Inventory Risk Monitor",
  "Competitor Price Monitor",
  "Guardrail Violation Monitor",
]
const OPERATOR_TYPES = [
  "Discount Approval Operator",
  "Scenario Optimization Operator",
]
```

### HireDto

```typescript
interface HireDto {
  kind: "monitor" | "operator"
  subtype: string  // must be one of MONITOR_TYPES (if kind="monitor") or OPERATOR_TYPES (if kind="operator")
}
```

## Endpoints

### GET /agents/roster

**Response 200:** `AgentRosterView`

### GET /agents/catalog

**Response 200:** `AgentCatalogView`

### POST /agents/monitors/:id/pause

**Response 200:** `MonitorEntity` (status: "paused", lastActivity updated)
**Response 404:** monitor not found

### POST /agents/monitors/:id/resume

**Response 200:** `MonitorEntity` (status: "active", lastActivity updated)
**Response 404:** monitor not found

### POST /agents/hire

**Body:** `HireDto`
**Response 201:** `MonitorEntity` (if `kind: "monitor"`) or `OperatorView` (if `kind: "operator"`)
**Response 400:** `subtype` is not in the catalog for the given `kind`

### POST /agents/task-agents/:id/retire

**Response 200:** `TaskAgentEntity` (status: "retired")
**Response 404:** task agent not found

## Known Gaps

- REQ-AGENT-004's "navigating to Autonomy when clicked" (operator card click) navigates to `/autonomy`, which currently renders `ScreenPlaceholder` since SLICE-11 is not yet implemented. Not a defect in this slice — tracked against SLICE-11.
- Operator `trustLevel`/`evidenceStatus`/`trackRecord` are deterministic placeholders. Once SLICE-11 (Pricing Autonomy) exists, this contract's `OperatorView` should be revisited to derive these fields from real trust-ladder/action-class data rather than the fixed seed.
- Task agents are seeded, not spawned by real platform events (see `open_q1_resolution` above).
