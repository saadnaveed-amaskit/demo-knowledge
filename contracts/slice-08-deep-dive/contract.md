---
status: Stable
slice_id: SLICE-08
contract_type: api-contract
approved_by: Saad
approved_at: 2026-07-09
promoted_at: 2026-07-09
frontend_pr: "https://github.com/saadnaveed-amaskit/demo-frontend/pull/10"
backend_pr: "https://github.com/saadnaveed-amaskit/demo-backend/pull/8"
---

# Contract: Slice 08 — Deep Dive

## Endpoint

```
GET /price-scenarios/:id/deep-dive
```

Returns placeholder deep-dive data for a given price scenario. Progressive unlock via `unlockLevel` field on each row/tile.

### Path Parameters

| Param | Type    | Description       |
|-------|---------|-------------------|
| id    | integer | Scenario ID       |

### Response: 200 OK

```json
{
  "priceAdjustments": [SkuRecommendationRow],
  "marketingTiles":   [MarketingTile],
  "discountTiles":    [DiscountTile]
}
```

### Error Responses

| Status | Condition             |
|--------|-----------------------|
| 404    | Scenario not found    |
| 404    | Scenario has no output (not yet run) |

---

## Types

### DeepDiveOutput (Tier-2 screen construct)

| Field             | Type                    | Description                                  |
|-------------------|-------------------------|----------------------------------------------|
| priceAdjustments  | SkuRecommendationRow[]  | All SKU-level price adjustment rows          |
| marketingTiles    | MarketingTile[]         | Marketing campaign tiles                     |
| discountTiles     | DiscountTile[]          | Discount event tiles                         |

---

### SkuRecommendationRow (Tier-2)

| Field                  | Type    | Description                                         |
|------------------------|---------|-----------------------------------------------------|
| sku                    | string  | SKU identifier                                      |
| productName            | string  | Product display name                                |
| category               | string  | Product category                                    |
| brand                  | string  | Brand                                               |
| currentPrice           | number  | Current selling price ($)                           |
| recommendedPrice       | number  | Recommended price ($)                               |
| priceChange            | number  | Delta: recommendedPrice − currentPrice ($)          |
| priceChangePct         | number  | Delta as % of currentPrice                          |
| currentGrossMargin     | number  | Current gross margin (%)                            |
| projectedGrossMargin   | number  | Projected gross margin (%)                          |
| weeklyRevenue          | number  | Current weekly revenue ($)                          |
| projectedRevenue       | number  | Projected weekly revenue ($)                        |
| currentWeeksOfSupply   | number  | Current weeks of supply                             |
| projectedWeeksOfSupply | number  | Projected weeks of supply                           |
| unlockLevel            | number  | Min optimization level (0–100) to surface this row  |

---

### ExplainTrace (Tier-2) — returned by frontend mock; no separate endpoint

| Field              | Type                  | Description                              |
|--------------------|-----------------------|------------------------------------------|
| sku                | string                | SKU identifier                           |
| rationale          | string                | Plain-language rationale                 |
| drivingObjectives  | string[]              | Weighted objectives driving this price   |
| decisionLadder     | DecisionLadderItem[]  | Ordered list of competing constraints    |
| contextualFactors  | ContextualFactor[]    | Market/inventory context signals         |

### DecisionLadderItem

| Field      | Type   | Description                        |
|------------|--------|------------------------------------|
| rank       | number | Priority rank (1 = highest)        |
| constraint | string | Constraint name                    |
| decision   | string | How the constraint was resolved    |

### ContextualFactor

| Field  | Type                                | Description                 |
|--------|-------------------------------------|-----------------------------|
| factor | "Competitor price range" \| "Weeks of supply" \| "Elasticity" | Signal name |
| value  | string                              | Signal value                |
| impact | "positive" \| "neutral" \| "negative" | Direction of impact       |

---

### MarketingTile (Tier-2)

| Field         | Type                    | Description                                     |
|---------------|-------------------------|-------------------------------------------------|
| id            | string                  | Tile identifier                                 |
| campaign      | string                  | Campaign name                                   |
| type          | string                  | Campaign type (e.g. "Email Blast", "Display")   |
| discount      | string                  | Discount description (e.g. "15% off")           |
| skuCount      | number                  | Number of SKUs in campaign                      |
| projectedLift | string                  | Revenue lift estimate (e.g. "+8% revenue")      |
| unlockLevel   | number                  | Min optimization level to show this tile        |
| products      | MarketingTileProduct[]  | Per-product list (expandable)                   |

### MarketingTileProduct

| Field       | Type   | Description                |
|-------------|--------|----------------------------|
| sku         | string | SKU identifier             |
| productName | string | Product display name       |
| currentPrice| number | Current price ($)          |
| promoPrice  | number | Promotional price ($)      |

---

### DiscountTile (Tier-2)

| Field                | Type                   | Description                                   |
|----------------------|------------------------|-----------------------------------------------|
| id                   | string                 | Tile identifier                               |
| name                 | string                 | Discount event name                           |
| type                 | string                 | Event type (e.g. "Clearance", "Promo")        |
| depth                | string                 | Discount depth (e.g. "20–30% off")            |
| skuCount             | number                 | Number of SKUs in event                       |
| projectedSellThrough | string                 | Sell-through estimate (e.g. "82%")            |
| unlockLevel          | number                 | Min optimization level to show this tile      |
| products             | DiscountTileProduct[]  | Per-product list (expandable)                 |

### DiscountTileProduct

| Field         | Type   | Description                |
|---------------|--------|----------------------------|
| sku           | string | SKU identifier             |
| productName   | string | Product display name       |
| currentPrice  | number | Current price ($)          |
| discountedPrice| number| Discounted price ($)       |

---

## Progressive Unlock Thresholds

Rows and tiles are generated with `unlockLevel` values per these bands:

| Optimization Level | SKUs/Tiles unlocked   |
|--------------------|-----------------------|
| 0–20               | ~20% of total         |
| 21–40              | ~40% of total         |
| 41–60              | ~60% of total         |
| 61–80              | ~80% of total         |
| 81–100             | 100% of total         |

The frontend filters `priceAdjustments`, `marketingTiles`, and `discountTiles` by `item.unlockLevel <= currentOptimizationLevel`.

---

## Placeholder Data Rules

- `priceAdjustments` length = scenario.skuCount (capped at 20 for placeholder)
- Each row assigned `unlockLevel = Math.floor(index / total * 100)` rounded to nearest 10
- `marketingTiles` = 5 tiles; unlock levels 0, 20, 40, 60, 80
- `discountTiles` = 5 tiles; unlock levels 0, 20, 40, 60, 80

---

## Status

| Version | Date       | Note                          |
|---------|------------|-------------------------------|
| v0.1    | 2026-07-09 | Initial draft for SLICE-08    |
