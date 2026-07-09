---
status: Stable
artifact_type: api-contract
slice_id: SLICE-07
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q1_resolution: >
  v1 uses deterministic placeholder computation behind the stable contract shape.
  Real ML is out of scope for this slice. The frontier, comparison metrics, and
  recommendations are computed from SKU count, objectives, and optimizationLevel
  using fixed coefficients. The contract shape is locked so frontend, the governed
  approval workflow, and SLICE-09 (Approvals) can be built and validated now.
---

# Contract — SLICE-07: Price Scenario Optimization

## Base path

`/price-scenarios`

## Entities

### ScenarioEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | auto-increment |
| name | string | user-provided |
| focusGroupId | string | FK → focus-sets |
| focusGroupName | string | denormalized at creation |
| skuCount | number | from focus set productCount |
| startDate | string | ISO date |
| endDate | string | ISO date |
| objectives | ScenarioObjectives | revenue+grossMargin+sellThrough must sum to 100 |
| optimizationLevel | number | 0–100 |
| status | ScenarioStatus | lifecycle below |
| createdAt | string | ISO timestamp |
| changeRequests | ChangeRequest[] | appended on return; empty by default |
| output | ScenarioOutput \| null | null until /run succeeds |

```typescript
type ScenarioStatus = "new" | "draft" | "pending" | "approved" | "denied" | "returned"

interface ScenarioObjectives {
  revenue: number      // 0–100
  grossMargin: number  // 0–100
  sellThrough: number  // 0–100
  // sum must equal 100
}

interface ChangeRequest {
  requestedAt: string  // ISO timestamp
  comment: string
}
```

### ScenarioOutput (Tier-2)

| Field | Type | Notes |
|---|---|---|
| narrative | string | AI narrative paragraph |
| uncertainty | string | e.g. "±8–15% depending on data recency" |
| guardrailResults | GuardrailCheckResult[] | from active guardrails at run time |
| comparison | ComparisonRow[] | 7 rows (spec REQ-SCEN-007) |
| frontier | FrontierPoint[] | 11 points at levels 0,10,…,100 |
| currentPoint | FrontierPoint | level=0 baseline |
| mlRecPoint | FrontierPoint | level=55 ML recommendation |
| scenarioPoint | FrontierPoint | at current optimizationLevel |
| recommendations | Recommendation[] | ≥4 tagged suggestions |

```typescript
interface ComparisonRow {
  metric: string
  current: string
  scenario: string
  mlRec: string
}

interface FrontierPoint {
  level: number    // 0–100
  revenue: number  // $k (placeholder)
  profit: number   // $k (placeholder)
}

interface Recommendation {
  tag: "Pricing" | "Marketing" | "Merch" | "Inventory"
  text: string
}
```

`GuardrailCheckResult` shape is the same as `knowledge/contracts/slice-04-guardrails/contract.md`.

## Status Lifecycle

```
new → (POST /:id/run) → draft → (POST /:id/submit) → pending
pending → (PATCH /:id/status { status: "approved" }) → approved
pending → (PATCH /:id/status { status: "denied" })   → denied
pending → (PATCH /:id/status { status: "returned", comment: "..." }) → returned
returned → (POST /:id/submit) → pending (resubmit allowed)
```

`submit` throws `400 Bad Request` if status is not `draft` or `returned`.
`run` throws `404 Not Found` if scenario does not exist.
`updateStatus` accepts any target status to support BDD and approval workflow.

## Risk Bands (optimization slider)

| Range | Label | Uplift range |
|---|---|---|
| 0–20% | Conservative | +2–5% revenue |
| 20–40% | Moderate | +5–10% revenue |
| 40–60% | Balanced | +10–15% revenue |
| 60–80% | Aggressive | +15–20% revenue |
| 80–100% | Full Optimization | +20–25% revenue |

## Placeholder Computation (v1)

```
BASE_REVENUE = skuCount × $29.99 × 26 weeks × 0.8 (sell-through assumption)
BASE_PROFIT  = BASE_REVENUE × 0.35 (margin assumption)

frontier[level]:
  revenue = BASE_REVENUE × (1 + level/100 × 0.22)
  profit  = BASE_PROFIT  × (1 + level/100 × 0.28 × (1 − level/180))

currentPoint   = frontier[0]
mlRecPoint     = frontier[55]
scenarioPoint  = frontier[optimizationLevel]

comparison metrics (scenario at current optimizationLevel vs current vs mlRec):
  Revenue        = revenue in $k
  Profit         = profit  in $k
  Rev Uplift     = ((scenario.revenue − current.revenue) / current.revenue × 100).toFixed(1) + "%"
  Margin Δ       = (scenario.profit/scenario.revenue − current.profit/current.revenue × 100).toFixed(1) + "pp"
  Full-Price Mix = (70 + optimizationLevel × 0.12).toFixed(1) + "%"
  Sell-Through   = (objectives.sellThrough + optimizationLevel × 0.05).toFixed(1) + "%"
  Inventory Risk = level ≤ 40 ? "Low" : level ≤ 70 ? "Medium" : "High"
```

## Endpoints

### GET /price-scenarios

Returns all scenarios, newest first.

**Response 200:**
```json
[ScenarioEntity, ...]
```

### GET /price-scenarios/:id

**Response 200:** `ScenarioEntity`
**Response 404:** scenario not found

### POST /price-scenarios

**Body:**
```json
{
  "name": "string",
  "focusGroupId": "string",
  "startDate": "2026-08-01",
  "endDate": "2026-08-14",
  "objectives": { "revenue": 50, "grossMargin": 30, "sellThrough": 20 },
  "optimizationLevel": 50
}
```

**Response 201:** `ScenarioEntity` (status: "new", output: null)

### POST /price-scenarios/:id/run

**Response 200:** `ScenarioEntity` (status: "draft", output populated)
**Response 404:** scenario not found

### POST /price-scenarios/:id/submit

**Response 200:** `ScenarioEntity` (status: "pending")
**Response 400:** status is not "draft" or "returned"
**Response 404:** scenario not found

### PATCH /price-scenarios/:id/status

**Body:**
```json
{ "status": "approved" | "denied" | "returned" | "pending", "comment": "optional, for returned" }
```

**Response 200:** `ScenarioEntity`
**Response 404:** scenario not found

### DELETE /price-scenarios/:id

**Response 204:** no content
**Response 404:** scenario not found
