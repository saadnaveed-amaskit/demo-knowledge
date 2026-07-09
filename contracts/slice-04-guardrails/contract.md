---
status: Draft
artifact_type: api-contract
slice_id: SLICE-04
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
decision_versioning_audit: deferred — REQ-GUARD-007 [NEEDS CLARIFICATION]
---

# Contract — SLICE-04 Guardrails Management

## Base URL

`/guardrails`

---

## Endpoints

### GET /guardrails

Returns all guardrails (active and inactive).

**Response 200:**
```json
[
  {
    "id": 1,
    "brand": "Gymboree",
    "division": "BIG GIRLS",
    "rule": "Gross Margin",
    "op": ">=",
    "value": "38",
    "unit": "%",
    "active": true,
    "isOverridable": true
  }
]
```

---

### POST /guardrails

Creates a new guardrail.

**Request body:**
```json
{
  "brand": "TCP",
  "division": "BOYS",
  "rule": "Min Price",
  "op": ">=",
  "value": "5.00",
  "unit": "$"
}
```

`active` defaults to `true`. `isOverridable` defaults to `true`.

**Response 201:** `GuardrailEntity`

---

### PATCH /guardrails/:id

Updates a guardrail's rule fields (brand, division, rule, op, value, unit). Partial — only provided fields are changed.

**Path params:** `id: number`

**Request body (all optional):**
```json
{ "value": "42", "op": ">=" }
```

**Response 200:** `GuardrailEntity`

**Response 404:** Guardrail not found.

---

### PATCH /guardrails/:id/active

Toggles active state.

**Path params:** `id: number`

**Response 200:** `GuardrailEntity`

**Response 404:** Guardrail not found.

---

### PATCH /guardrails/:id/overridable

Toggles isOverridable state.

**Path params:** `id: number`

**Response 200:** `GuardrailEntity`

**Response 404:** Guardrail not found.

---

### DELETE /guardrails/:id

Deletes a guardrail.

**Path params:** `id: number`

**Response 204:** No content.

**Response 404:** Guardrail not found.

---

### POST /guardrails/evaluate

Evaluates a set of metric values against all active guardrails.
Returns per-guardrail pass/fail and overall compliance.

**Note:** Versioning and audit of compliance records deferred — REQ-GUARD-007 [NEEDS CLARIFICATION].

**Request body:**
```json
{
  "brand": "Gymboree",
  "division": "BIG GIRLS",
  "metrics": {
    "Gross Margin": 35.5,
    "Min Price": 6.00
  }
}
```

**Response 200:**
```json
{
  "compliant": false,
  "results": [
    {
      "id": 1,
      "rule": "Gross Margin",
      "op": ">=",
      "threshold": "38",
      "unit": "%",
      "actual": 35.5,
      "passed": false,
      "isOverridable": true,
      "severity": "advisory"
    }
  ]
}
```

`severity`: `"hard"` when `active && !isOverridable`; `"advisory"` when `active && isOverridable`.

---

## Types

### GuardrailEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| `id` | `number` | Auto-increment |
| `brand` | `string` | FK → CatalogBrand |
| `division` | `string` | FK → CatalogDivision |
| `rule` | `string` | Human label, e.g. "Gross Margin" |
| `op` | `string` | One of `>=`, `<=`, `=`, `>`, `<` |
| `value` | `string` | Threshold as string (e.g. "38") |
| `unit` | `string` | `"%"`, `"$"`, `""` |
| `active` | `boolean` | Inactive guardrails are not evaluated |
| `isOverridable` | `boolean` | If false, Pricing Team cannot override in scenarios |

### GuardrailEvaluationResult (Tier-2 response)

| Field | Type | Notes |
|---|---|---|
| `compliant` | `boolean` | True if all active guardrails pass |
| `results` | `GuardrailCheckResult[]` | Per-guardrail breakdown |

### GuardrailCheckResult (Tier-2)

| Field | Type | Notes |
|---|---|---|
| `id` | `number` | |
| `rule` | `string` | |
| `op` | `string` | |
| `threshold` | `string` | Raw value string |
| `unit` | `string` | |
| `actual` | `number` | Provided metric value |
| `passed` | `boolean` | |
| `isOverridable` | `boolean` | |
| `severity` | `"hard" \| "advisory"` | hard = active + non-overridable |

---

## Open Questions

- **REQ-GUARD-007 versioning/audit:** Compliance evaluation is stateless (no record persisted). Audit log deferred.
- **REQ-GUARD-005 enforcement in scenario creation:** Surfaced in SLICE-07 (Price Scenario optimization).
