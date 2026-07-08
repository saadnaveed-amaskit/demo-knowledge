---
status: Draft
artifact_type: api-contract
slice_id: SLICE-02
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
---

# Contract — SLICE-02 Focus Set Management

Backend-owned API for the Focus Set scoping primitive (spec REQ-FOCUS-001…012).
Base path: `/focus-sets` and `/catalog`. JSON over HTTP. Promoted draft→stable at
post-merge finalization.

**Decisions recorded for this slice:**
- **Resolution is live** (Open Q3): matched SKUs are computed on demand from the
  stored condition tree; `productCount` is always derived from live resolution so
  displayed criteria and matched products cannot diverge.
- **No minimum-match enforcement** (FR-FB gap): a Focus Set matching 0 SKUs may be
  saved; the UI warns but does not block.

## Types

```ts
// Tier-2 request shape (condition tree). Recursive, nestable up to 3 levels.
type ConditionNode =
  | { type: "rule"; attr: string; val: string }
  | { type: "group"; logic: "AND" | "OR"; rules: ConditionNode[] }

// Tier-1 canonical record (backend @Entity).
interface FocusSetEntity {
  id: string            // pattern "focus-set-##", auto-incremented
  name: string
  filter: ConditionNode // root is a group
  createdAt: string     // ISO 8601
}

// Tier-2 projection returned to the client (adds derived productCount).
interface FocusSetView {
  id: string
  name: string
  filter: ConditionNode
  productCount: number  // derived live from resolution
  createdAt: string
}

// Tier-2 resolved SKU projection.
interface SkuView {
  sku: string
  name: string
  brand: string
  division: string
  category: string
  price: number
  qty: number
  status: string        // "In_Stock" | "Low_Stock" | "Out_of_stock"
}

interface AttributeOption { attr: string; values: string[] }
```

## Endpoints

| Method | Path | Body | Response | Notes / Requirement |
|---|---|---|---|---|
| GET | `/focus-sets` | — | `FocusSetView[]` | Library list (REQ-FOCUS-001). Client sorts/searches (REQ-FOCUS-002). |
| GET | `/focus-sets/:id` | — | `FocusSetView` \| 404 | Single set |
| POST | `/focus-sets` | `{ name, filter }` | `201 FocusSetView` | Create; requires name; id `focus-set-##` (REQ-FOCUS-003, 011) |
| PUT | `/focus-sets/:id` | `{ name, filter }` | `FocusSetView` \| 404 | Edit in place, id preserved (REQ-FOCUS-005) |
| POST | `/focus-sets/:id/duplicate` | `{ name }` | `201 FocusSetView` | Copy filter under new name (REQ-FOCUS-006) |
| DELETE | `/focus-sets/:id` | — | `204` \| 404 | Delete (REQ-FOCUS-007) |
| POST | `/focus-sets/resolve` | `{ filter }` | `{ count, skus: SkuView[] }` | Live resolve of an unsaved filter for preview (REQ-FOCUS-004, 012). Empty rules → all SKUs (REQ-FOCUS-010). |
| GET | `/focus-sets/:id/skus` | — | `{ count, skus: SkuView[] }` | Resolve a saved set (drives export REQ-FOCUS-008 and drill REQ-FOCUS-009) |
| GET | `/catalog/attributes` | — | `AttributeOption[]` | Distinct filterable attribute values from the catalog (REQ-FOCUS-003 dynamic attrs) |

## Rules

- Matching is **exact per-attribute equality**; no range/contains/wildcard (REQ-FOCUS-010).
- An empty rule list (root group with no rules) matches **all** SKUs (REQ-FOCUS-010).
- Condition tree nesting is limited to **3 levels**; deeper nesting is rejected `400` (REQ-FOCUS-003).
- `name` is required on create/edit/duplicate; missing → `400`.
- `productCount` in any `FocusSetView` equals the live resolved SKU count.

## Validation status

Backend contract tests under `backend/src/focus-sets/*.spec.ts` assert every endpoint
above. Frontend consumes via a typed client and mocks are not used for final E2E
validation (real backend). Contract remains **Draft** until SLICE-02 PRs merge.
