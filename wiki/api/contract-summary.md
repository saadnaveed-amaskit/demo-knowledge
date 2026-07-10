# API Contract Summary

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| backend | main | 5f46cc599365cfcd2b3180776eacaa93f5086fd2 |

## Canonical Contract

| Field | Value |
|---|---|
| Markdown contract | backend/contracts/api-contract.md |
| YAML source, if present | **Not present.** `backend/contracts/api-contract.yaml` existed on this branch but was deleted from `main` in commit `bb1a951` ("deleted redundant file") before the SLICE-09 merge. |

Note: `api-contract.md`'s own embedded "Source" table currently records `Backend branch: agent/slice-12-measurement`, commit "pending commit on this branch" — a placeholder left over from before that branch's PR merged. It has not been regenerated against the current `main` HEAD (`5f46cc5`); see Known Gaps.

## Endpoint Groups

| Method | Path | Summary | Source |
|---|---|---|---|
| GET | `/agents/catalog` | List provisionable monitor/operator types | agents |
| POST | `/agents/hire` | Hire a new monitor or operator from the catalog | agents |
| POST | `/agents/monitors/{id}/pause` | Pause a monitor | agents |
| POST | `/agents/monitors/{id}/resume` | Resume a monitor | agents |
| GET | `/agents/roster` | Get the full agent roster (KPIs, monitors, operators, task agents) | agents |
| POST | `/agents/task-agents/{id}/retire` | Retire a task agent | agents |
| GET | `/approvals/decided` | List price scenarios and discount models that have been decided | approvals |
| POST | `/approvals/discounts/{id}/decision` | Approve, deny, or request changes on a pending discount model | approvals |
| GET | `/approvals/discounts/{id}/review` | Get the discount model's risk banner and impact for approval-mode review | approvals |
| GET | `/approvals/queue` | List pending price scenarios and discount models awaiting decision | approvals |
| POST | `/approvals/scenarios/{id}/decision` | Approve, deny, or request changes on a pending price scenario | approvals |
| GET | `/approvals/scenarios/{id}/review` | Get the full scenario output for approval-mode review | approvals |
| GET | `/autonomy/action-classes/{id}/audit` | Get the audit trail for an action class | autonomy |
| POST | `/autonomy/action-classes/{id}/demote` | Demote an action class's trust rung (never gated) | autonomy |
| POST | `/autonomy/action-classes/{id}/promote` | Promote an action class's trust rung (gated) | autonomy |
| POST | `/autonomy/kill-switch/disengage` | Disengage the emergency kill switch | autonomy |
| POST | `/autonomy/kill-switch/engage` | Engage the emergency kill switch | autonomy |
| POST | `/autonomy/live-actions/{id}/undo` | Undo an applied live action | autonomy |
| POST | `/autonomy/live-actions/{id}/veto` | Veto a pending live action | autonomy |
| GET | `/autonomy/roster` | Get the full autonomy roster (KPIs, action classes, live actions, kill-switch state) | autonomy |
| GET | `/catalog/attributes` | List filterable catalog attributes and their observed values | catalog |
| GET | `/discount-models` | List all discount models, newest id first | discount-models |
| POST | `/discount-models` | Create a discount model | discount-models |
| DELETE | `/discount-models/{id}` | Delete a discount model | discount-models |
| GET | `/discount-models/{id}` | Get a discount model by id | discount-models |
| POST | `/discount-models/{id}/run` | Run the discount model and populate its output | discount-models |
| PATCH | `/discount-models/{id}/status` | Force-set a discount model's status | discount-models |
| POST | `/discount-models/{id}/submit` | Submit a discount model for approval | discount-models |
| GET | `/focus-sets` | List all focus sets | focus-sets |
| POST | `/focus-sets` | Create a focus set | focus-sets |
| POST | `/focus-sets/resolve` | Preview SKUs matched by an ad-hoc (unsaved) filter tree | focus-sets |
| DELETE | `/focus-sets/{id}` | Delete a focus set | focus-sets |
| GET | `/focus-sets/{id}` | Get a focus set by id | focus-sets |
| PUT | `/focus-sets/{id}` | Update a focus set (name and/or filter) | focus-sets |
| POST | `/focus-sets/{id}/duplicate` | Duplicate a focus set under a new name | focus-sets |
| GET | `/focus-sets/{id}/skus` | Resolve the SKUs currently matched by a saved focus set's filter | focus-sets |
| GET | `/guardrails` | List all guardrails | guardrails |
| POST | `/guardrails` | Create a guardrail | guardrails |
| POST | `/guardrails/evaluate` | Evaluate metrics against active guardrails for a brand/division | guardrails |
| DELETE | `/guardrails/{id}` | Delete a guardrail | guardrails |
| PATCH | `/guardrails/{id}` | Partially update a guardrail | guardrails |
| PATCH | `/guardrails/{id}/active` | Toggle a guardrail's active flag | guardrails |
| PATCH | `/guardrails/{id}/overridable` | Toggle a guardrail's isOverridable flag | guardrails |
| GET | `/health` | Liveness check | health |
| GET | `/measurement/experiments` | List all matched-cluster experiments | measurement |
| GET | `/measurement/experiments/{id}` | Get an experiment (setup state + live readout if launched) | measurement |
| POST | `/measurement/experiments/{id}/clusters/{clusterId}/move` | Move a cluster between treatment and control arms | measurement |
| POST | `/measurement/experiments/{id}/acknowledge-cost` | Acknowledge the cost-of-control tradeoff | measurement |
| POST | `/measurement/experiments/{id}/go-live` | Launch an experiment (gated on balance + cost acknowledgment) | measurement |
| POST | `/measurement/experiments/{id}/scale` | Scale a live experiment's win to all matching SKUs | measurement |
| POST | `/measurement/experiments/{id}/kill` | Kill a live experiment and revert its treatment clusters to BAU | measurement |
| GET | `/price-scenarios` | List all price scenarios, newest id first | price-scenarios |
| POST | `/price-scenarios` | Create a price scenario | price-scenarios |
| DELETE | `/price-scenarios/{id}` | Delete a price scenario | price-scenarios |
| GET | `/price-scenarios/{id}` | Get a price scenario by id | price-scenarios |
| GET | `/price-scenarios/{id}/deep-dive` | Get the deep-dive breakdown for a scenario that has been run | price-scenarios |
| POST | `/price-scenarios/{id}/run` | Run the price scenario and populate its output | price-scenarios |
| PATCH | `/price-scenarios/{id}/status` | Force-set a price scenario's status | price-scenarios |
| POST | `/price-scenarios/{id}/submit` | Submit a price scenario for approval | price-scenarios |
| GET | `/product-grid/{focusSetId}` | Get the product grid derived from a saved focus set's filter | product-grid |
| POST | `/product-grid/{focusSetId}/exclude` | Mark a SKU as excluded within a focus set's grid | product-grid |
| DELETE | `/product-grid/{focusSetId}/exclusions` | Restore (un-exclude) all SKUs for a focus set | product-grid |
| DELETE | `/product-grid/{focusSetId}/exclusions/{skuId}` | Restore (un-exclude) a single SKU | product-grid |
| GET | `/promotions` | List all promotions | promotions |
| POST | `/promotions` | Create a promotion | promotions |
| DELETE | `/promotions/{id}` | Delete a promotion | promotions |
| PATCH | `/promotions/{id}` | Partially update a promotion | promotions |
| GET | `/promotions/{id}/products` | Get the (discounted) product rows for a promotion | promotions |

67 endpoints total, across 12 tags.

## Schemas / Request-Response Objects

| Name | Purpose | Source |
|---|---|---|
| ActionClassEntity | Tier-1 canonical action class — the unit of trust-ladder governance | autonomy |
| AgentCatalogView | Fixed, hardcoded provisionable catalog (monitor/operator types) | agents |
| AgentKpis | Derived on every read from the current monitors/operators/taskAgents arrays — not stored | agents |
| AgentRosterView | [NOT SPECIFIED] | agents |
| ApprovalItemView | Tier-2, computed on read from a ScenarioEntity or DiscountModelEntity — not a persisted Tier-1 entity | approvals |
| ApprovalsDecidedView | [NOT SPECIFIED] | approvals |
| ApprovalsQueueView | [NOT SPECIFIED] | approvals |
| AttributeOption | [NOT SPECIFIED] | catalog |
| AuditEntry | Tier-1 canonical audit-trail entry for an action class | autonomy |
| AutonomyKpis | Derived on every read from the current action-class array — not stored | autonomy |
| AutonomyRosterView | [NOT SPECIFIED] | autonomy |
| BlockView | Comparability block grouping a treatment and control cluster pair; `status` computed from `metricMatch` ratios + arm presence | measurement |
| ChangeRequest | [NOT SPECIFIED] | price-scenarios / approvals |
| ClusterContribution | Per-cluster contribution to the overall lift signal (readout) | measurement |
| ClusterView | Read projection of a cluster; `mlPrice` is null while `arm === "control"` | measurement |
| ComparisonRow | [NOT SPECIFIED] | price-scenarios |
| ConditionGroupNode | [NOT SPECIFIED] | focus-sets |
| ConditionNode | Recursive AND/OR condition tree. Enforced max depth is 3 (root group = depth 1), but ONLY on focus-sets create/update — not on POST /focus-sets/resolve | focus-sets |
| ConditionRuleNode | [NOT SPECIFIED] | focus-sets |
| CreateDiscountModelDto | [NOT SPECIFIED] | discount-models |
| CredibleInterval | `{ estimate, lower, upper }` — used for the incremental margin readout figure | measurement |
| CreateGuardrailDto | [NOT SPECIFIED] | guardrails |
| CreatePromotionDto | [NOT SPECIFIED] | promotions |
| CreateScenarioDto | [NOT SPECIFIED] | price-scenarios |
| DecisionDto | Plain TS interface — zero runtime enum validation on `action`; unrecognized strings are treated as "request_changes" | approvals |
| DeepDiveOutput | [NOT SPECIFIED] | price-scenarios |
| DiscountModelEntity | [NOT SPECIFIED] | discount-models |
| DiscountModelKpis | [NOT SPECIFIED] | discount-models |
| DiscountModelOutput | [NOT SPECIFIED] | discount-models |
| DiscountModelUpdateStatusDto | status is not runtime-validated against the enum; any string is accepted | discount-models |
| DiscountReviewView | ApprovalItemView (discount domain) plus review-only fields | approvals |
| DiscountRiskBanner | [NOT SPECIFIED] | approvals |
| DiscountTile | [NOT SPECIFIED] | price-scenarios (deep-dive) |
| DiscountTileProduct | [NOT SPECIFIED] | price-scenarios (deep-dive) |
| ErrorResponse | Default Nest HttpException JSON shape (no custom exception filter observed) | shared |
| EvaluateDto | [NOT SPECIFIED] | guardrails |
| ExperimentView | Top-level matched-cluster experiment read projection; `goLiveEligible`/`goLiveBlockedReason` computed fresh on every read | measurement |
| FocusSetBody | [NOT SPECIFIED] | focus-sets |
| FocusSetView | [NOT SPECIFIED] | focus-sets |
| FrontierPoint | [NOT SPECIFIED] | price-scenarios |
| GuardrailCheckResult | [NOT SPECIFIED] | guardrails |
| GuardrailEntity | [NOT SPECIFIED] | guardrails |
| GuardrailEvaluationResult | [NOT SPECIFIED] | guardrails |
| HealthStatus | [NOT SPECIFIED] | health |
| HireDto | Plain TS interface — zero runtime enum validation on `kind`; only `subtype` is checked | agents |
| KillSwitchState | [NOT SPECIFIED] | autonomy |
| LiveActionEntity | Tier-1 canonical live (in-flight) autonomous action | autonomy |
| MarketingTile | [NOT SPECIFIED] | price-scenarios (deep-dive) |
| MarketingTileProduct | [NOT SPECIFIED] | price-scenarios (deep-dive) |
| MetricMatch | Per-metric (revenue/grossMargin/velocity) match ratio between a block's treatment and control clusters | measurement |
| MonitorEntity | Tier-1 canonical standing monitor; in-memory, seeded + hired | agents |
| MoveClusterDto | Request body for moving a cluster's arm — `{ arm: "treatment" \| "control" }` | measurement |
| NivoBarDatapoint | [NOT SPECIFIED] | discount-models |
| NivoLineDatapoint | [NOT SPECIFIED] | discount-models |
| NivoLineDataset | [NOT SPECIFIED] | discount-models |
| OperatorView | Tier-2 derived view — NOT a persisted Tier-1 entity; hired operators always get placeholder trust data | agents |
| ProductGridView | [NOT SPECIFIED] | product-grid |
| ProductRow | [NOT SPECIFIED] | product-grid |
| PromoProductRow | [NOT SPECIFIED] | promotions |
| PromoProductsView | [NOT SPECIFIED] | promotions |
| PromotionEntity | [NOT SPECIFIED] | promotions |
| ReadoutView | Live readout once an experiment has gone live — probability of winning, day, verdict, incremental margin CI, per-cluster contributions | measurement |
| Recommendation | [NOT SPECIFIED] | price-scenarios |
| ResolveResult | [NOT SPECIFIED] | focus-sets |
| RiskPanel | [NOT SPECIFIED] | discount-models |
| RollupRow | [NOT SPECIFIED] | discount-models |
| ScenarioEntity | [NOT SPECIFIED] | price-scenarios |
| ScenarioGuardrailCheck | [NOT SPECIFIED] | price-scenarios |
| ScenarioObjectives | [NOT SPECIFIED] | price-scenarios |
| ScenarioOutput | [NOT SPECIFIED] | price-scenarios |
| ScenarioReviewView | ApprovalItemView (scenario domain) plus the full SLICE-07 output, reused as-is | approvals |
| ScenarioUpdateStatusDto | status is not runtime-validated against the enum; any string is accepted | price-scenarios |
| SkuRecommendationRow | [NOT SPECIFIED] | price-scenarios (deep-dive) |
| SkuRow | [NOT SPECIFIED] | product-grid |
| SkuView | [NOT SPECIFIED] | focus-sets |
| TaskAgentEntity | Tier-1 canonical task agent; seeded only in v1, no creation endpoint | agents |
| UpdateGuardrailDto | All fields optional; server only assigns fields that are not undefined (partial update) | guardrails |
| UpdatePromotionDto | All fields optional; partial update | promotions |

78 schemas total. `[NOT SPECIFIED]` above means the YAML's `description:` field for that schema was empty at generation time (for pre-SLICE-10 schemas) or intentionally omitted for hand-authored SLICE-10/11/12 entries with no natural one-line purpose beyond their field table — not an omission introduced by this summary. "Source" column groups schemas by the tag/module that owns them, inferred from usage, not a literal YAML field.

## Related Features / Slices

| Feature/Slice | Contract Areas |
|---|---|
| SLICE-02 (Focus Set management) | focus-sets |
| SLICE-03 (Product Grid) | product-grid |
| SLICE-04 (Guardrails management) | guardrails |
| SLICE-05 (Promotions calendar) | promotions |
| SLICE-06 (Discount Modeling) | discount-models |
| SLICE-07 (Price Scenario optimization) | price-scenarios |
| SLICE-08 (Deep Dive) | price-scenarios (`/price-scenarios/{id}/deep-dive` + deep-dive schemas) |
| SLICE-09 (Approvals queue) | approvals |
| SLICE-10 (Agent roster) | agents |
| SLICE-11 (Pricing Autonomy) | autonomy |
| SLICE-12 (Measurement) | measurement |
| catalog, health | [UNKNOWN] — no dedicated slice found in `tasks.md`; catalog/health appear to be foundational scaffolding rather than a numbered slice |

## Known Gaps

- `api-contract.md`'s embedded Source table still records the pre-merge feature branch/commit (`agent/slice-12-measurement`, commit placeholder "pending commit on this branch"), not the current `main` HEAD (`5f46cc5`) — it has not been regenerated since the SLICE-12 merge, and cannot be corrected by `/finalize-knowledge-after-merge` per its "do not edit api-contract.md" constraint.
- `backend/contracts/api-contract.yaml` no longer exists on `main` (deleted in commit `bb1a951`); only the Markdown contract is currently present. SLICE-10/11/12's Agents/Autonomy/Measurement endpoints and schemas were hand-authored directly into the Markdown for this reason.
- Most pre-SLICE-10 schemas have no `description:` in the original source YAML (`[NOT SPECIFIED]`) — this reflects the YAML's own content, not a gap introduced here.
- SLICE-12's `BlockView.metricMatch` is typed optional solely so the pre-existing approved contract test's mocks (predating this field) still type-check unmodified — the real endpoint always populates it.
