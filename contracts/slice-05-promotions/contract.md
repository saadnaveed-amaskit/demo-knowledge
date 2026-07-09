---
status: Stable
artifact_type: api-contract
slice_id: SLICE-05
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
promoted_at: 2026-07-09
backend_merge_sha: 1ec578722d9fbd4eefd5d81d25f7b8258c069f0c
frontend_merge_sha: 30ebbccc0ec41dda9f9d380f6a615f532f714635
open_q2_resolution: >
  Canonical entity is PromotionEntity (Tier-1, pricing domain).
  Distinct from PromoEntity (marketing pacing events, already in taxonomy).
  FocusDiscountEntity fragment is superseded by PromotionEntity for SLICE-05 scope.
  Status (active/scheduled/expired) is derived at render time from dates — not stored.
---

# Contract — SLICE-05 Promotions Calendar

## Open Q2 Resolution

Canonical Promotion/Discount entity (spec Open Q2): **`PromotionEntity`** — a time-boxed pricing promotion with a linked Focus Set, discount depth, channel, and calendar colour. Status is derived dynamically: `endDate < today → expired; startDate > today → scheduled; else → active`.

---

## Base URL

`/promotions`

---

## Endpoints

### GET /promotions

Returns all promotions. Status field is derived at response time.

**Response 200:**
```json
[
  {
    "id": 1,
    "name": "Back to School",
    "startDate": "2026-07-01",
    "endDate": "2026-07-31",
    "discountType": "percentage",
    "discountValue": 20,
    "focusSetId": "",
    "channel": "US",
    "color": "#0d9488",
    "notes": "Summer clearance",
    "status": "active"
  }
]
```

---

### POST /promotions

Creates a new promotion.

**Request body:**
```json
{
  "name": "Fall Launch",
  "startDate": "2026-08-01",
  "endDate": "2026-08-31",
  "discountType": "percentage",
  "discountValue": 15,
  "focusSetId": "fs-001",
  "channel": "US",
  "color": "#7c3aed",
  "notes": ""
}
```

`endDate` must be strictly after `startDate`. Backend validates and returns 400 on violation.

**Response 201:** `PromotionEntity` (with derived `status`)

**Response 400:** Validation error (e.g. end date not after start date).

---

### PATCH /promotions/:id

Updates promotion fields. Partial — only provided fields are changed.

**Path params:** `id: number`

**Response 200:** `PromotionEntity` (with derived `status`)

**Response 404:** Promotion not found.

**Response 400:** Validation error on date ordering.

---

### DELETE /promotions/:id

Deletes a promotion.

**Path params:** `id: number`

**Response 204:** No content.

**Response 404:** Promotion not found.

---

### GET /promotions/:id/products

Returns up to 20 SKUs from the linked Focus Set with computed promo price.

**Path params:** `id: number`

**Response 200:**
```json
{
  "promotionId": 1,
  "promotionName": "Back to School",
  "discountType": "percentage",
  "discountValue": 20,
  "focusSetId": "fs-001",
  "focusSetName": "Gymboree Fall",
  "skus": [
    {
      "sku": "GYM-BG-100",
      "name": "Gymboree Floral Legging S",
      "brand": "Gymboree",
      "price": 14.50,
      "promoPrice": 11.60,
      "savings": 2.90
    }
  ]
}
```

**Response 404:** Promotion not found, or Focus Set not found.

---

## Types

### PromotionEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| `id` | `number` | Auto-increment |
| `name` | `string` | Required |
| `startDate` | `string` | ISO 8601 date (`YYYY-MM-DD`) |
| `endDate` | `string` | ISO 8601; strictly after `startDate` |
| `discountType` | `"percentage" \| "flat"` | |
| `discountValue` | `number` | ≥ 0; depth for percentage, amount for flat |
| `focusSetId` | `string` | FK → FocusSetEntity.id |
| `channel` | `string` | e.g. `"US"`, `"CA"` |
| `color` | `string` | Hex colour for calendar bar |
| `notes` | `string` | Optional |
| `status` | `"active" \| "scheduled" \| "expired"` | Derived at response time; not persisted |

### PromoProductRow (Tier-2)

| Field | Type | Notes |
|---|---|---|
| `sku` | `string` | |
| `name` | `string` | |
| `brand` | `string` | |
| `price` | `number` | Current price |
| `promoPrice` | `number` | price × (1 - value/100) for percentage; max(0, price - value) for flat |
| `savings` | `number` | price − promoPrice |

### PromoProductsView (Tier-2 response)

| Field | Type | Notes |
|---|---|---|
| `promotionId` | `number` | |
| `promotionName` | `string` | |
| `discountType` | `string` | |
| `discountValue` | `number` | |
| `focusSetId` | `string` | |
| `focusSetName` | `string` | |
| `skus` | `PromoProductRow[]` | Up to 20 |

---

## Status Derivation

```
today = current date (YYYY-MM-DD string)
endDate < today   → "expired"
startDate > today → "scheduled"
else              → "active"
```

---

## Discount Computation

```
percentage: promoPrice = round(price × (1 - discountValue / 100), 2)
flat:        promoPrice = round(max(0, price − discountValue), 2)
savings = round(price − promoPrice, 2)
```
