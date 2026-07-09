---
status: Stable
artifact_type: api-contract
slice_id: SLICE-03
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
decision_exclusions: in-memory per-focusSetId (persisted model deferred — ORM/DB TBD)
promoted_at: 2026-07-09
backend_pr: https://github.com/saadnaveed-amaskit/demo-backend/pull/3
frontend_pr: https://github.com/saadnaveed-amaskit/demo-frontend/pull/4
---

# Contract — SLICE-03 Product Grid

## Base URL

`/product-grid`

---

## Endpoints

### GET /product-grid/:focusSetId

Returns the product grid for a Focus Set: all matching products with their SKUs (grouped by `productId`), and per-SKU exclusion state.

**Path params:** `focusSetId: string`

**Response 200:**
```json
{
  "focusSetId": "focus-set-01",
  "focusSetName": "Gymboree Fall",
  "filter": { "type": "group", "logic": "AND", "rules": [...] },
  "products": [
    {
      "productId": "P-GYM-BG-001",
      "productName": "Gymboree Floral Legging",
      "brand": "Gymboree",
      "division": "BIG GIRLS",
      "category": "Leggings",
      "skuCount": 2,
      "activeSkuCount": 2,
      "priceRange": [14.50, 14.50],
      "totalQty": 520,
      "stockStatus": "In_Stock",
      "skus": [
        {
          "sku": "GYM-BG-100",
          "productId": "P-GYM-BG-001",
          "productName": "Gymboree Floral Legging",
          "name": "Gymboree Floral Legging S",
          "brand": "Gymboree",
          "division": "BIG GIRLS",
          "category": "Leggings",
          "subClass": "Knit",
          "msrp": null,
          "price": 14.50,
          "qty": 300,
          "onOrderQty": 120,
          "status": "In_Stock",
          "excluded": false
        }
      ]
    }
  ],
  "totalSkuCount": 6,
  "activeSkuCount": 6
}
```

**Response 404:** Focus Set not found.

---

### POST /product-grid/:focusSetId/exclude

Soft-excludes one SKU from the active grid (moves to Deleted Items).

**Path params:** `focusSetId: string`

**Request body:**
```json
{ "skuId": "GYM-BG-100" }
```

**Response 200:**
```json
{ "excludedSkuId": "GYM-BG-100" }
```

**Response 404:** Focus Set or SKU not found.

---

### DELETE /product-grid/:focusSetId/exclusions/:skuId

Restores a single excluded SKU to the active grid.

**Path params:** `focusSetId: string`, `skuId: string`

**Response 204:** No content.

**Response 404:** Focus Set or exclusion not found.

---

### DELETE /product-grid/:focusSetId/exclusions

Restores all excluded SKUs (clears the Deleted Items pane).

**Path params:** `focusSetId: string`

**Response 204:** No content.

---

## Types

### ProductGridView (Tier-2 response shape)

| Field | Type | Notes |
|---|---|---|
| `focusSetId` | `string` | |
| `focusSetName` | `string` | |
| `filter` | `ConditionNode` | Read-only criteria bar |
| `products` | `ProductRow[]` | Grouped by productId |
| `totalSkuCount` | `number` | All SKUs in focus set |
| `activeSkuCount` | `number` | Non-excluded SKUs |

### ProductRow (Tier-2)

| Field | Type | Notes |
|---|---|---|
| `productId` | `string` | |
| `productName` | `string` | |
| `brand` | `string` | |
| `division` | `string` | |
| `category` | `string` | |
| `skuCount` | `number` | Total SKUs in product |
| `activeSkuCount` | `number` | Non-excluded SKUs |
| `priceRange` | `[number, number]` | [min, max] current price |
| `totalQty` | `number` | Sum of all SKU qty |
| `stockStatus` | `string` | Worst status among SKUs |
| `skus` | `SkuRow[]` | |

### SkuRow (Tier-2)

| Field | Type | Notes |
|---|---|---|
| `sku` | `string` | |
| `productId` | `string` | |
| `productName` | `string` | |
| `name` | `string` | |
| `brand` | `string` | |
| `division` | `string` | |
| `category` | `string` | |
| `subClass` | `string` | |
| `msrp` | `number \| null` | Struck through if current price < msrp |
| `price` | `number` | Current price |
| `qty` | `number` | |
| `onOrderQty` | `number` | |
| `status` | `"In_Stock" \| "Low_Stock" \| "Out_of_stock"` | Inventory health |
| `excluded` | `boolean` | True when in Deleted Items |

---

## Open Questions Resolved

- **Exclusion persistence:** Session-local in-memory store per `focusSetId`. Persisted model deferred pending ORM/DB decision.
- **REQ-GRID-007 color scale:** Defined as `In_Stock` → green (`qty > 200`), `Low_Stock` → amber (`qty > 0`), `Out_of_stock` → red (`qty === 0`). Applied consistently across Focus Builder preview and Product Grid.
