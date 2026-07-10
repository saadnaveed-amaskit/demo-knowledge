---
status: Draft
artifact_type: api-contract
slice_id: SLICE-11
feature_id: 001-platform-baseline
owner: backend
consumers: [frontend]
open_q1_resolution: >
  REQ-AUTO-010 (canonical Agents/Autonomy implementation vs. the Pricing
  Platform V2 Agentic fork; kill-switch freeze semantics — spec Open Q8,
  §10/§11): resolved by definition, same as SLICE-10 (Agent roster). The V2
  fork (`/pricing-platform-v2/*`) is not part of this workspace at all (only
  `knowledge/`, `frontend/`, `backend/` exist here), so there is no competing
  implementation to reconcile against — this implementation is canonical.

  Kill-switch freeze semantics (REQ-AUTO-001): "freeze" means the frontend's
  client-side veto-countdown ticker stops decrementing while
  `killSwitchEngaged` is true, and resumes decrementing (continuing from
  wherever it was) once disengaged. No countdown state is reset on
  engage/disengage. This is a v1 UI-level freeze, not a backend timer — the
  backend has no concept of elapsed time for a live action beyond
  `vetoWindowSeconds`/`engagedAt`.

  Promotion-gate ordering (REQ-AUTO-009): reversibility ceiling -> sample
  count (>= 100) -> accuracy (>= 0.85) -> acceptance rate (>= 0.80), each
  checked in that order with the first failing gate's reason returned.
  Reversibility ceiling is a hard cap: `Low` reversibility caps at
  `Supervised`; `Medium`/`High` reversibility cap at `Autonomous` (can reach
  the top of the ladder). These threshold values are v1 placeholders (no
  historical proof-accuracy dataset exists to derive them from) — consistent
  with the deterministic-placeholder precedent (spec Open Q1).

  New Tier-1 entities (spec Open Q12): `ActionClassEntity`, `LiveActionEntity`,
  and `AuditEntry` are new canonical entities, defined fresh in this contract
  (no prior DATA-TAXONOMY.md entries existed for any of the three).
---

# Contract — SLICE-11: Pricing Autonomy

## Base path

`/autonomy`

## Entities

### ActionClassEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | auto-increment |
| name | string | |
| trustRung | `"Manual" \| "Supervised" \| "Autonomous"` | 3-level trust ladder |
| reversibilityClass | `"Low" \| "Medium" \| "High"` | how hard the action class's changes are to reverse |
| atReversibilityCeiling | boolean | computed at read time: `trustRung === REVERSIBILITY_CEILING[reversibilityClass]` |
| sampleCount | number | promotion gate input |
| accuracy | number | 0–1, promotion gate input |
| acceptanceRate | number | 0–1, promotion gate input |
| liveDollarValue | number | $ currently live under autonomy for this action class |
| createdAt | string | ISO timestamp |

```typescript
const REVERSIBILITY_CEILING: Record<ReversibilityClass, TrustRung> = {
  Low: "Supervised",
  Medium: "Autonomous",
  High: "Autonomous",
}
```

### LiveActionEntity (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | |
| actionClassId | number | FK -> ActionClassEntity.id |
| description | string | |
| status | `"pending" \| "vetoed" \| "applied" \| "undone"` | |
| vetoWindowSeconds | number | initial countdown length; frontend computes remaining time client-side |
| engagedAt | string | ISO timestamp |

### AuditEntry (Tier-1)

| Field | Type | Notes |
|---|---|---|
| id | number | |
| actionClassId | number | FK -> ActionClassEntity.id |
| action | string | human-readable description, e.g. "Promoted to Autonomous" |
| actor | string | v1 placeholder — always `"Pricing Strategist"`, no real identity capture (same precedent as SLICE-09's submitter placeholders) |
| timestamp | string | ISO timestamp |

### AutonomyKpis (Tier-2)

| Field | Type | Notes |
|---|---|---|
| totalActionClasses | number | |
| eligibleToPromote | number | count where `!atReversibilityCeiling` (does not additionally check sample/accuracy/acceptance gates) |
| totalLiveDollarValue | number | sum of `liveDollarValue` across all action classes |
| averageProofAccuracy | number | mean of `accuracy` across all action classes, rounded to 3 decimals |

### AutonomyRosterView (Tier-2)

| Field | Type | Notes |
|---|---|---|
| kpis | AutonomyKpis | |
| actionClasses | ActionClassEntity[] | |
| liveActions | LiveActionEntity[] | optional in the TS type (see Known Gaps) but always populated by the real endpoint |
| killSwitchEngaged | boolean | |

### KillSwitchState

```typescript
interface KillSwitchState {
  killSwitchEngaged: boolean
}
```

## Status Lifecycle

```
Trust ladder: Manual -> (promote) -> Supervised -> (promote) -> Autonomous
              Autonomous -> (demote) -> Supervised -> (demote) -> Manual
demote is always allowed, un-gated, even while the kill switch is engaged (REQ-AUTO-006).
promote is blocked while the kill switch is engaged, and by the ordered gates below.

Live action: pending -> (veto, only while pending) -> vetoed
             applied  -> (undo, only while applied)  -> undone
veto/undo are both blocked while the kill switch is engaged (REQ-AUTO-008).
```

## Promotion Gates (REQ-AUTO-009)

Evaluated in order; the first failing gate's `BadRequestException` message is returned:

1. Reversibility ceiling: `atReversibilityCeiling === true` -> blocked
2. Sample count: `sampleCount < 100` -> blocked
3. Accuracy: `accuracy < 0.85` -> blocked
4. Acceptance rate: `acceptanceRate < 0.80` -> blocked

If all four pass, `trustRung` advances one rung (capped at `Autonomous`).

## Endpoints

### GET /autonomy/roster

**Response 200:** `AutonomyRosterView`

### GET /autonomy/action-classes/:id/audit

**Response 200:** `AuditEntry[]`
**Response 404:** action class not found

### POST /autonomy/action-classes/:id/promote

**Response 200:** `ActionClassEntity` (trustRung advanced)
**Response 400:** blocked by kill switch, or by the first failing ordered gate
**Response 404:** action class not found

### POST /autonomy/action-classes/:id/demote

**Response 200:** `ActionClassEntity` (trustRung downgraded) — never blocked, even while kill switch is engaged
**Response 404:** action class not found

### POST /autonomy/live-actions/:id/veto

**Response 200:** `LiveActionEntity` (status: "vetoed")
**Response 400:** blocked by kill switch, or live action is not "pending"
**Response 404:** live action not found

### POST /autonomy/live-actions/:id/undo

**Response 200:** `LiveActionEntity` (status: "undone")
**Response 400:** blocked by kill switch, or live action is not "applied"
**Response 404:** live action not found

### POST /autonomy/kill-switch/engage

**Response 200:** `KillSwitchState` (`{ killSwitchEngaged: true }`)

### POST /autonomy/kill-switch/disengage

**Response 200:** `KillSwitchState` (`{ killSwitchEngaged: false }`)

## Known Gaps

- `AutonomyRosterView.liveActions` is typed as optional (`liveActions?:`) rather than required, solely so the SLICE-11 preparation-phase contract test's mock (written before this field was designed) still type-checks unmodified. The real endpoint always populates it.
- Promotion-gate thresholds (sample count, accuracy, acceptance rate) are v1 placeholders with no historical dataset behind them.
- `actor` on `AuditEntry` is a fixed placeholder string, not real user identity (no identity-provider integration exists, per spec's Out of Scope section).
- The BDD step for "Block promotion below the reversibility ceiling" selects the *first* rendered action-class card and assumes it is the one the `Given` step found at its reversibility ceiling — true for the current seed order (action class id 1) but not guaranteed in general if seed order changes.
