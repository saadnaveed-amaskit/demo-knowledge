---
status: Draft
artifact_type: api-contract
slice_id: SLICE-09
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q1_resolution: >
  Reason-required policy (spec Open Q5 / REQ-APPR-012), resolved by human decision
  2026-07-09: a mandatory reason/comment is required for ALL three action types —
  deny, request-changes (scenario), and return-for-changes (discount). This updates
  REQ-APPR-007, which previously said "optional" for scenario request-changes.
  submitter/team/brand/division fields are v1 deterministic placeholders rotated by
  item id — real identity capture is out of scope per spec ("Full identity-provider
  (SSO/IAM) integration... unless the access-control decision brings it into scope").
  Fallback scenario risk uses the existing "Inventory Risk" comparison-row value
  (Low/Medium/High) computed by SLICE-07, since no dedicated
  inventory-risk-reduction dollar metric exists in the v1 placeholder engine.
  REQ-APPR-005/008 (cross-domain sub-action drawer + action-tracker linkback) are
  DEFERRED for this slice: no BusinessIssueEntity/action-tracker exists anywhere in
  the codebase (spec Open Q9, unresolved), and building that subsystem now would be
  scope expansion beyond an approvals queue for the scenario/discount workflows
  already built in SLICE-06/07. Tracked as a known gap in the SLICE-09 validation
  report, not silently dropped.
---

# Contract — SLICE-09: Approvals Queue

## Base path

`/approvals`

## Entities

### ApprovalItemView (Tier-2 aggregated queue row)

Aggregates a `ScenarioEntity` (SLICE-07) or `DiscountModelEntity` (SLICE-06) into the shape the Approvals queue renders. Not persisted — computed on read from the underlying entity.

| Field | Type | Notes |
|---|---|---|
| domain | `"scenario" \| "discount"` | which underlying service owns this item |
| id | number | underlying entity id |
| name | string | underlying entity name |
| submitter | string | v1 placeholder, rotated by id (see resolution note) |
| team | string | v1 placeholder |
| brand | string | v1 placeholder |
| division | string | v1 placeholder |
| impact | string | scenario: Rev Uplift from comparison row; discount: formatted `kpis.revenueImpact` |
| risk | `"Low" \| "Medium" \| "High"` | REQ-APPR-011 derivation, see below |
| status | `"pending" \| "approved" \| "denied" \| "returned"` | mirrors underlying entity status |
| changeRequests | `ChangeRequest[]` | mirrors underlying entity's change-request history |

### ApprovalsQueueView

| Field | Type | Notes |
|---|---|---|
| scenarios | ApprovalItemView[] | status === "pending" only |
| discounts | ApprovalItemView[] | status === "pending" only |
| scenarioPendingCount | number | `scenarios.length` |
| discountPendingCount | number | `discounts.length` |

### DiscountReviewView (Tier-2, extends ApprovalItemView)

| Field | Type | Notes |
|---|---|---|
| riskBanner | `{ hardCount: number; advisoryCount: number }` | counts from `output.riskPanels` |
| constraintWarnings | string[] | `riskPanels` titles where `severity !== "low"` |
| competitiveFlags | string[] | v1: empty array — no competitive-price integration exists (spec Out of Scope) |

Per REQ-APPR-009, the discount drawer intentionally excludes an itemized guardrail-compliance list and SKU-level breakdown.

### ScenarioReviewView (Tier-2, extends ApprovalItemView)

| Field | Type | Notes |
|---|---|---|
| output | `ScenarioOutput` | the full SLICE-07 output (narrative, comparison, guardrailResults, frontier) reused as-is for approval-mode viewing |

### DecisionDto

```typescript
type ApprovalDecisionAction = "approve" | "deny" | "request_changes"

interface DecisionDto {
  action: ApprovalDecisionAction
  comment?: string  // REQUIRED (non-empty after trim) for "deny" and "request_changes"; ignored for "approve"
}
```

## Status Lifecycle

Approvals does not introduce a new status vocabulary — it drives the existing SLICE-06/07 state machines via their `updateStatus` endpoint:

```
action "approve"          → underlying status "approved"
action "deny"              → underlying status "denied"   (comment required; recorded in the approvals decision log)
action "request_changes"   → underlying status "returned"  (comment required; appended to underlying changeRequests[])
```

`decideScenario`/`decideDiscount` throw `400 Bad Request` if `action !== "approve"` and `comment` is missing or blank.

## Risk Bands (REQ-APPR-011)

```
discountRisk(riskPanels):
  High   if any panel.isHard === true
  Medium if riskPanels.length > 0 (any violation) and no hard panel
  Low    otherwise

scenarioRisk(output) — fallback (no dedicated $ inventory-risk-reduction metric in v1):
  = output.comparison["Inventory Risk"].scenario   // "Low" | "Medium" | "High", computed by SLICE-07
```

## Placeholder Computation (v1)

```
SUBMITTER_ROSTER = [
  { submitter: "J. Alvarez", team: "Pricing Team", brand: "TCP", division: "Girls" },
  { submitter: "M. Chen",    team: "Pricing Team", brand: "Gymboree", division: "Boys" },
  { submitter: "R. Patel",   team: "Pricing Team", brand: "Both", division: "Baby" },
]
placeholderSubmission(id) = SUBMITTER_ROSTER[id % SUBMITTER_ROSTER.length]

scenarioImpact(output)  = output.comparison.find(r => r.metric === "Rev Uplift").scenario
discountImpact(output)  = (kpis.revenueImpact >= 0 ? "+" : "") + "$" + round(kpis.revenueImpact).toLocaleString()
```

## Endpoints

### GET /approvals/queue

**Response 200:** `ApprovalsQueueView`

### GET /approvals/decided

**Response 200:** `{ scenarios: ApprovalItemView[], discounts: ApprovalItemView[] }` — status !== "pending" and !== "new" and !== "draft"

### GET /approvals/scenarios/:id/review

**Response 200:** `ScenarioReviewView`
**Response 404:** scenario not found or has no output

### GET /approvals/discounts/:id/review

**Response 200:** `DiscountReviewView`
**Response 404:** discount model not found or has no output

### POST /approvals/scenarios/:id/decision

**Body:** `DecisionDto`
**Response 200:** `ApprovalItemView` (updated)
**Response 400:** comment missing for "deny"/"request_changes"
**Response 404:** scenario not found

### POST /approvals/discounts/:id/decision

**Body:** `DecisionDto`
**Response 200:** `ApprovalItemView` (updated)
**Response 400:** comment missing for "deny"/"request_changes"
**Response 404:** discount model not found

## Known Gaps (deferred, not implemented this slice)

- **REQ-APPR-005 / REQ-APPR-008** (cross-domain sub-action drawer, action-tracker linkback): no `BusinessIssueEntity` or action-tracker subsystem exists anywhere in the codebase (spec Open Q9 unresolved). Deferred pending a future slice that defines those Tier-1 entities.
