---
status: Draft
artifact_type: api-contract
slice_id: SLICE-06
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q1_resolution: >
  v1 uses deterministic placeholder computation behind the stable contract shape.
  Real ML is out of scope for this slice. The shape is locked so frontend and
  the governed approval workflow can be built and validated now.
---

# Contract — SLICE-06: Discount Modeling

## Entities

### `DiscountModelEntity` (Tier-1, backend-owned)

```typescript
interface DiscountModelEntity {
  id: number
  name: string
  focusGroupId: string        // FK → FocusSetEntity.id (empty string if none)
  focusGroupName: string      // denormalized display name
  skuCount: number            // resolved at create/run time
  startDate: string           // ISO date "YYYY-MM-DD"
  endDate: string             // ISO date "YYYY-MM-DD"
  discountFormat: "percentage" | "flat" | "bogo" | "fixed"
  discountDepth: number | null  // null when discountFormat === "bogo"
  channel: string
  status: "new" | "draft" | "pending" | "approved" | "returned" | "denied"
  createdAt: string           // ISO datetime
  marketingHandle: string     // generated on run; empty before run
  output: DiscountModelOutput | null  // null before run
}
```

### Status lifecycle

```
new → (POST /:id/run) → draft → (POST /:id/submit) → pending
pending → (Approvals SLICE-09) → approved | denied | returned
returned → (Pricing Team edits + re-submits) → pending
DELETE /:id → 204 at any status
```

---

## Tier-2 Output Shapes

### `DiscountModelOutput`

Produced by `POST /discount-models/:id/run`. Stored on the entity.

```typescript
interface DiscountModelOutput {
  narrative: string                     // AI-style narrative text (placeholder)
  marketingHandle: string               // copyable marketing slug
  kpis: DiscountModelKpis
  forecastRevenue: NivoLineDataset[]    // 2 series: baseline + promoted, 8 weeks
  forecastMargin: NivoLineDataset[]     // 2 series: baseline + promoted, 8 weeks
  forecastUnits: NivoBarDatapoint[]     // 8 bar datapoints: baseline + promoted keys
  rollupRows: RollupRow[]
  riskPanels: RiskPanel[]
}

interface DiscountModelKpis {
  revenueImpact: number      // $ delta
  marginImpact: number       // $ delta (negative = margin compression)
  unitLift: number           // percentage points
  sellThrough: number        // 0–100 %
  incrementalRevenue: number // $ net-new revenue
}

interface NivoLineDataset {
  id: string                         // "Baseline" | "Promoted"
  data: Array<{ x: string; y: number }>  // x = "Wk 1" … "Wk 8"
}

interface NivoBarDatapoint {
  week: string    // "Wk 1" … "Wk 8"
  Baseline: number
  Promoted: number
}

interface RollupRow {
  label: string         // division or channel name
  skuCount: number
  revenue: number
  margin: number
  sellThrough: number   // 0–100 — drives color coding
  stockOutRisk: boolean
  confidence: number    // 0–1 for display as %
}

interface RiskPanel {
  title: string
  severity: "high" | "medium" | "low"
  description: string
  isHard: boolean       // true = hard guardrail violation; false = advisory
}
```

### `CreateDiscountModelDto`

```typescript
interface CreateDiscountModelDto {
  name: string
  focusGroupId: string
  startDate: string
  endDate: string
  discountFormat: "percentage" | "flat" | "bogo" | "fixed"
  discountDepth?: number    // omit for BOGO
  channel: string
}
```

---

## Endpoints

| Method | Path                          | Request body                  | Response                  | Notes                        |
| ------ | ----------------------------- | ----------------------------- | ------------------------- | ---------------------------- |
| GET    | /discount-models              | —                             | `DiscountModelEntity[]`   | All models, newest first     |
| GET    | /discount-models/:id          | —                             | `DiscountModelEntity`     | 404 if not found             |
| POST   | /discount-models              | `CreateDiscountModelDto`      | `DiscountModelEntity`     | status = "new"               |
| POST   | /discount-models/:id/run      | —                             | `DiscountModelEntity`     | Computes output; status = "draft"; 404 if not found |
| POST   | /discount-models/:id/submit   | —                             | `DiscountModelEntity`     | "draft" → "pending"; 400 if not draft |
| DELETE | /discount-models/:id          | —                             | 204 No Content            | Any status; 404 if not found |

---

## Placeholder Computation (Open Q1)

The `POST /discount-models/:id/run` endpoint computes a deterministic `DiscountModelOutput`:

- **Revenue impact:** `skuCount × avgPrice × discountRate × unitLiftFactor`
- **Margin impact:** always negative (compression); roughly `revenueImpact × -0.3`
- **Unit lift:** `discountRate × 2.5` percentage points (capped at 40)
- **Sell-through:** base 70 + `discountRate × 0.5` (capped at 98)
- **Incremental revenue:** `revenueImpact × 0.6`
- **Forecast:** 8-week linear ramp starting at 80% of peak; weeks are "Wk 1"…"Wk 8"
- **Rollup:** one row per RN_DIVISION; sell-through distributed with variance
- **Risk panels:** 6 fixed titles; severity derived from constraint violations

`avgPrice` = 29.99 (v1 constant). `unitLiftFactor` = 1.2. `discountRate` = `discountDepth / 100` for percentage; `discountDepth / avgPrice` for flat; `0.5` for BOGO; `discountDepth / avgPrice` for fixed.

---

## Sell-Through Color Coding (REQ-DISC-007)

| Range        | Color  |
| ------------ | ------ |
| ≥ 85%        | green  |
| ≥ 70%, < 85% | amber  |
| < 70%        | red    |

## Hard vs Advisory Severity (REQ-DISC-008)

`isHard = true` panels show a banner; if any hard panel exists, the header escalates to "Hard Violation".

---

## Marketing Handle Format

`<BRAND>-<FORMAT>-<DEPTH>-<CHANNEL>-<SEASON>-<YYYYMM>`

Example: `TCP-PCT-20-DIG-SUMMER-202607`
