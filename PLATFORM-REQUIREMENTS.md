# Retail Nucleus — Pricing Platform: Functional Requirements

**Source:** reverse-engineered from the design-reference prototype at `src/retail-nucleus/` (this repo), cross-checked against `src/retail-nucleus/docs/*.md` build specs. This is a draft for review — flag anything that reflects a prototype shortcut rather than an intended production behavior.

**Scope:** the RetailNucleus app only (route `/retail-nucleus/*`). Does not cover Pricing Platform V2 Agentic (`/pricing-platform-v2/*`), which is a separate, partially-diverged fork — differences are called out in [[#11. Pricing Autonomy|§11]] and [[#15. Cross-Cutting Issues|§15]] where they matter.

**How to read this doc:** each screen section has Purpose, Key Entities, Functional Requirements (numbered `FR-<Screen>-##`), Business Rules/Calculations, and Gaps. "Gaps" means: the prototype fakes or skips this, and production needs a real decision/implementation.

---

## 1. Platform Overview

**Personas:**
- **Pricing Team** — builds Focus Sets, runs Price Scenarios and Discount Models, manages Promotions, reviews Like-Item Mappings, sets up Measurement experiments.
- **Pricing Strategist** — authors Guardrails, approves/denies scenarios and discount models, governs agent trust levels and autonomy, manages the agent roster.
- **Admin** — configuration/user access (currently a stub, see [[#14. Configuration|§14]]).

**Navigation (sidebar sections → screens → routes):**

| Section | Screen | Route |
|---|---|---|
| Pricing Team | Focus Builder | `focus` |
| Pricing Team | Promotions | `promos` |
| Pricing Team | Price Scenarios | `scenario` |
| Pricing Team | Discount Modeling | `discount-modeling` |
| Pricing Team | Like-Item Mapping | `like-items` |
| Pricing Team | Measurement | `measurement` |
| Pricing Strategist | Agents | `agents` |
| Pricing Strategist | Guardrails | `guardrails` |
| Pricing Strategist | Approvals | `approvals` |
| Pricing Strategist | Autonomy | `autonomy` |
| (unlisted, routed) | Product Grid | `product-grid` |
| Admin | Config | `config` |

Global filters: `brand` (`both|tcp|gymboree`), `channel` (`all|digital|store|canada`) — held in `useRNState`, referenced by screen headers but not deeply wired into filtering logic in most screens today (Gap — see [[#15. Cross-Cutting Issues|§15]]).

**Cross-app note:** `Focus Set` is the shared scoping primitive referenced by Discount Modeling, Promotions, Price Scenarios, and Product Grid — it should be treated as a first-class, independently persisted resource in production (list/get/create/update/delete + "resolve to SKUs" as its own operation).

---

## 2. Focus Builder

**Purpose:** Define and save a reusable, named product/SKU scope ("Focus Set") via an AND/OR condition-tree filter. Every other pricing tool (Discount Modeling, Promotions, Price Scenarios) targets a Focus Set rather than raw SKU lists.

**Key entities:**
- `FocusSetView { id, name, filter: ConditionGroup, productCount, createdAt }`
- `ConditionRule { attr, val }`, `ConditionGroup { logic: "AND"|"OR", rules: (ConditionRule|ConditionGroup)[] }` — recursive, nestable.
- `ProductView { sku, name, brand, masterSeason, seasonCode, division, department, category, subClass, bigIdea, price, qty, onOrderQty, onHandStatus }` — one row = one SKU.
- Filterable attributes are derived dynamically from the catalog (Season, Season Code, Division, Department, Sub Class, Category, Big Idea, Brand, Status) — not a hardcoded enum.
- `FocusDiscountEntity { id, focusSetId, name, startDate, endDate, discountType: "percentage"|"amount", discountValue }` — a lightweight discount tied directly to a focus set (distinct from `DiscountModelEntity`, see [[#5. Discount Modeling|§5]] Gap).

**Functional Requirements:**
1. ==**FR-FB-01**: User can browse a library of saved Focus Sets showing name, ID, criteria (rendered as an AND/OR chip expression), created date, and matched product count.== 
2. ==**FR-FB-02**: Library is sortable by Name, Created date, and Product Count (ascending/descending), and searchable by name (substring, case-insensitive).==
3. ==**FR-FB-03**: User can create a new Focus Set: name (required) + a condition tree (attribute = value rules, grouped by AND/OR, nestable up to 3 levels deep).==
4. ==**FR-FB-04**: Creation form shows a live preview of the first 8 matching SKUs (SKU, name, division, category, qty, status) plus a total match count and a "no matches" warning when the filter matches nothing.==
5. ==**FR-FB-05**: User can edit an existing Focus Set's name and filter in place (same ID preserved).== 
6. ==**FR-FB-06**: User can duplicate ("Copy") an existing Focus Set under a new name, inheriting its filter.==
7. ==**FR-FB-07**: User can delete a Focus Set from the library (with confirmation feedback).==
8. ==**FR-FB-08**: User can export a Focus Set's matched SKUs to CSV (SKU, Name, Division, Category, Price, Qty, Status).==
9. ==**FR-FB-09**: User can drill from a Focus Set into its full product/SKU grid ([[#4. Product Grid|→ Product Grid, §4]]).==
10. ==**FR-FB-10**: Rule matching is exact-equality per attribute; no range, contains, or wildcard operators. Empty rule list matches everything.==

**Business Rules:**
- Focus Set IDs follow pattern `focus-set-##`, auto-incremented from the current max.
- Qty coloring in preview: 0 → red, < 500 → amber, else default.

**Gaps (flag for production decision):**
- **No minimum-match enforcement**: a Focus Set matching 0 SKUs can still be saved and used downstream. Decide whether to block save/use below some threshold.
- **Divergent count sources**: for the 7 seeded focus sets, displayed criteria and the actual matched-product list come from separately pre-baked data, not live filter evaluation — in production these must always agree (either compute live, or persist a snapshot and label it as such).
- **Nested-group filters don't fully render** in Product Grid's criteria bar (only flat rules shown) — a display parity gap to close.
- Three unreconciled "discount tied to a scope" concepts exist across the app (`FocusDiscountEntity` here, `DiscountModelEntity` in [[#5. Discount Modeling|§5]], ad-hoc `Promo` in [[#3. Promotions|§3]]) — production needs one canonical model.

---

## 3. Promotions (Promo Calendar)

**Purpose:** Calendar and list view of promotional events for visualizing overlap/pacing and managing promo scheduling, tied to a Focus Set and discount terms.

**Key entities:**
- `Promo { id, name, startDate, endDate, discountType, discountValue, channel, focusId, color, status, notes }`.
- `status` is **derived at render time** from dates vs. "today," not stored: `start > today → scheduled`, `end < today → expired`, else `active`.
- Discount type options: Percentage Off, Flat % Off, Fixed Amount Off, Flat $ Off, Buy 2 Get 1, BOGO 50%, Free Shipping.
- Channel options: All Channels, Digital Only, In-Store Only.

**Functional Requirements:**
1. ==**FR-PC-01**: User can view promotions in a Calendar (month grid, lane-packed multi-day bars) or List (sortable table) mode.==
2. ==**FR-PC-02**: User can filter by status: All / Active / Scheduled / Expired, with live counts per tab.==
3. ==**FR-PC-03**: User can navigate months (prev/next/today) in calendar view.==
4. ==**FR-PC-04**: Clicking a promo opens a detail drawer: status/channel badges, date range + duration, discount summary, linked Focus Set (name, product count, criteria, "View Products"), notes, Edit/Delete actions.==
5. ==**FR-PC-05**: "View Products" shows up to 20 SKUs in the linked focus set with a computed promo price per SKU.==
6. ==**FR-PC-06**: User can create/edit a promo: name (required), start date (required, cannot be in the past), end date (required, must be after start — auto-advanced when needed), discount type, discount value (required for types that need a depth), Focus Set (required, searchable, with a shortcut to create a new one), channel, calendar color, notes (optional)==.
7. ==**FR-PC-07**: User can delete a promo.==

**Business Rules:**
- End date must be strictly after start date.
- New promos default to `status: scheduled`.

---

## 4. Product Grid

**Purpose:** Dense SKU/Product browser scoped to one Focus Set, for inspecting and curating (soft-excluding) which SKUs remain in scope before the set is used downstream.

**Key entities:**
- `SkuRow { id, productId, name, description, color, msrp, currentPrice, discountName, qty, onOrderQty, status: "in-stock"|"low-stock"|"out-of-stock", brand, division, category, ... }`.
- Product-level rollup row aggregates SKUs sharing a `productId` (summed qty, price min/max range, deduped colors, merged discount names).

**Functional Requirements:**
1. ==**FR-PG-01**: User can toggle between Product-level (collapsed/rolled-up) and SKU-level (expanded) grid views, each showing a live row count.==
2. ==**FR-PG-02**: User can free-text search/filter the grid and export the current view to CSV.==
3. ==**FR-PG-03**: Grid displays, per row: category icon, IDs, name/description, color or SKU count, MSRP (struck through if discounted), current price (or range if collapsed with mixed pricing), discount name, quantity, on-order quantity, stock status.==
4. ==**FR-PG-04**: User can remove ("soft-delete") a SKU or an entire product's SKUs from the active view into a "Deleted Items" pane.==
5. ==**FR-PG-05**: User can restore individual deleted rows or all of them at once from the Deleted Items pane.==
6. ==**FR-PG-06**: The criteria used to build the Focus Set is shown read-only above the grid.==

**Business Rules:**
- Stock status buckets: `In_Stock`, `Low_Stock`, `Out_of_stock` (fallback bucket for anything unrecognized).
- Qty color thresholds here (< 50 amber, > 200 green) differ from Focus Builder's preview thresholds — reconcile into one canonical inventory-health scale for production.

---

## 5. Discount Modeling

**Purpose:** AI/ML-driven markdown/promotion simulation tool. User selects a Focus Set, defines a discount (format/depth/channel/dates), runs a simulated forecast, and reviews a full margin/revenue/risk report before submitting for approval.

**Key entities:**
- `DiscountModelEntity { id: "DM-###", name, status, focusGroupId (FK), discountFormat: "pct"|"dollar"|"bogo"|"fixed", discountDepth, channel, skuCount, output: DiscountModelOutput, submittedInputs }`
- `status: "new" | "draft" | "pending" | "approved" | "returned" | "denied"`.
- `DiscountModelOutput`:
  - `narrative` (AI summary text), `marketingHandle` (copy-pasteable promo headline)
  - `metrics`: revenue / grossMargin / netMargin, each `{baseline, scenario, delta, deltaPct, rangeMin, rangeMax}`, plus unitsSold `{baseline, scenario, liftPct}` and sellThrough `{baseline, scenario, pct}`
  - `forecastCharts`: cumulative revenue (with 80% CI band), gross margin walk (waterfall: Baseline → +Volume → −Price → −Returns → −Cannibalization → Scenario), inventory burndown (with stock-out marker)
  - `rollup`: breakdown by category / brand / season / color / lifecycle stage — each row has SKU count, baseline/scenario revenue, delta, sell-through %, stock-out-risk flag
  - `riskPanels`: **cannibalization** (lost units/revenue to substitute SKUs), **inventory risk** (SKUs projected to sell out before the promo ends), **returns cost** (modeled return-rate increase → margin erosion $), **constraint warnings** (guardrail violations, severity `advisory`|`hard`), **competitive flags** (scenario price vs. tracked competitor floor), **SKU-level detail** (per-SKU projection with `confidence: high|medium|low`)

**Functional Requirements:**
1. ==**FR-DM-01**: List view groups models into New / Pending Review (includes returned) / Approved / Denied, showing name, focus group, format, channel, SKU count, horizon, created date, status.==
2. ==**FR-DM-01a**: Clicking anywhere on a model's row (including the Focus Group cell) opens that model's Output view (FR-DM-05) directly — for any existing model, regardless of status, since its `DiscountModelOutput` was already produced when the model was run. This bypasses the create form and "Run Model" step (FR-DM-04), which only applies when building a brand-new model from scratch.==
3. ==**FR-DM-02**: User can create a model: name (required), Focus Set (required, with live estimated-SKU-count preview), start/end date, discount format (%/$/BOGO/Fixed Price — depth input hidden for BOGO), depth value, channel (Digital/Stores/Both).==
4. ==**FR-DM-03**: Creation form shows a live preview: discount description, estimated SKUs, duration, and a generated marketing-handle string once all fields are set.==
5. ==**FR-DM-04**: "Run Model" is enabled only once name, focus group, both dates, and a valid depth (or BOGO) are set; it triggers a simulated run (a multi-phase progress sequence) and produces the full `DiscountModelOutput`.==
6. ==**FR-DM-05**: Output view shows: AI narrative + marketing handle (copyable), a 5-KPI Impact Summary (Revenue/Margin/Net Margin/Units/Sell-Through, each with baseline→scenario and range where applicable), three forecast charts, a tab-switchable rollup table, and 6 collapsible risk panels.==
7. ==**FR-DM-06**: Rollup table color-codes sell-through (≥85% green / ≥70% amber / <70% red) and flags stock-out risk per row.==
8. ==**FR-DM-07**: Constraint Warnings panel distinguishes `hard` vs `advisory` severity; visually escalates if any hard violation exists.==
9. ==**FR-DM-08**: User can submit a model for approval (status → pending) or discard it (with confirmation). Discard is available regardless of the model's current status — including already-**approved** or **pending** models opened via FR-DM-01a — and permanently removes it from the list with no status check.==
10. ==**FR-DM-09**: SKU-level detail table supports CSV export and shows a per-row confidence rating.==

**Business Rules / Thresholds (seed data):**
- Example guardrails referenced in output: min gross margin 35%, max markdown depth 60%.
- Inventory-risk urgency coloring: ≤2 days before promo end → red, ≤4 → amber.
- Money formatting: `$X.XM` at ≥$1M, `$XK` at ≥$1K, else whole dollars.


---

## 6. Price Scenarios

**Purpose:** ML-driven "what-if" pricing optimization against a Focus Set: simulate a run, explore a revenue/profit tradeoff frontier via an interactive optimization-level slider, and drill into SKU-level recommendations before submitting for approval.

**Key entities:**
- `ScenarioEntity { id: "SCN-###", name, focusId (FK), status, created, horizon, obj, chart, uplift, synopsis, mlSummary[], graphState: {sliderPct}, submittedInputs, changeRequests[] }`
- `status: "new" | "approved" | "pending" | "returned" | "draft" | "denied" | "running"`.
- `chart { curRev, mlRev, xMin/xMax/yMin/yMax, revPeak, profitPeak, curvature }` — defines the profit-vs-revenue frontier curve.
- `uplift { maxRevM, maxRevPct, maxMarginPp, fpMixBase, fpMixRange, maxSellPp, maxInvRiskM }`.
- `Weights { revenue, margin, sellthrough }` (must sum to 100) and `Guardrails { maxMarkdown, minMargin }` captured per scenario submission.
- `changeRequests: {comment, date}[]` — audit trail of approver feedback.

**Functional Requirements — lifecycle:**
1. ==**FR-PS-01**: List view shows all scenarios (name, status, created, focus set, objective); clicking an existing (non-draft) scenario reopens its output.==
2. ==**FR-PS-02**: User can create a scenario: name (required), Focus Set (required), start/end date (required, both), and a Price Objective Prioritization control — three linked sliders (Revenue / Gross Margin / Sell-Through) that must sum to 100%, redistributing proportionally when one moves.==
3. ==**FR-PS-03**: Create form shows a live preview (est. SKUs, duration, weighted objective summary) and a read-only list of currently active + overridable guardrails pre-filled from Pricing Strategist defaults.==
4. ==**FR-PS-04**: "Run Scenario" (enabled once required fields are valid) triggers a simulated multi-phase ML run: Product Look-alikes → Baseline Demand & Elasticity → Promotion Calendar → Inventory Velocity → Business Guardrails → Business Objectives & Goals==.
5. ==**FR-PS-05**: Output view shows a profit-vs-revenue frontier chart with three marked points (Current, ML Recommendation, Scenario Position) and an interactive 0–100% "Optimization Level" slider that moves the Scenario Position along the frontier.==
6. ==**FR-PS-06**: Slider is labeled by risk band: 0–20% Conservative / 20–40% Moderate / 40–60% Balanced (ML sweet spot) / 60–80% Aggressive / 80–100% Full Optimization, each with an expected uplift range and risk description (help modal available).==
7. ==**FR-PS-07**: Output shows a 3-column comparison table (Current / Scenario / ML Rec.) across Revenue, Profit, Rev Uplift, Margin Δ, Full-Price Mix, Sell-Through, Inventory Risk, plus an AI-generated narrative and tagged recommendations (Pricing/Marketing/Merch/Inventory) linking to the cross-domain action tracker.==
8. ==**FR-PS-08**: User can submit a scenario for approval (status → pending), discard it (with confirmation), or — once returned with change requests — see the latest reviewer comment and resubmit.==
9. ==**FR-PS-09**: Scenario slider and further edits are frozen (disabled) once status is pending, approved, or denied.==
10. ==**FR-PS-10**: A dashboard alert can deep-link into scenario creation with pre-filled context (division, issue, recommendation) — shown as a banner in the create form.==

**Business Rules / Calculations:**
- All projections should be presented with explicit uncertainty (prototype states "±8–15% depending on data recency" as a standing caveat) — carry this framing into production copy.
- The frontier curve is a smooth profit-peaks-at-a-revenue-point relationship; the optimization slider explores points along it, with the ML Recommendation marking the model's suggested optimum.
- Narrative/recommendation generation should vary meaningfully based on the dominant weighted objective (margin-protective vs. liquidation vs. balanced/revenue framing all produce materially different guidance and tactics in the prototype).

**Gaps — highest priority for production:**
- **No real ML optimization exists.** The chart curve, uplift figures, and narrative are generated by deterministic arithmetic formulas, not a real elasticity/demand model or optimizer. The 6 named "ML phases" are a reasonable outline for what a real pipeline should compute.
- **Guardrail compliance shown at approval time is hand-authored per scenario**, not computed live from the actual [[#8. Guardrails|Guardrails]] table ([[#8. Guardrails|§8]]) or real projected impact — production needs live guardrail evaluation.
- The dashboard-alert deep-link (FR-PS-10) has no live caller today in this app (a working analogous pattern exists in the sibling Pricing Platform V2 app's Orchestrator/Command Panel and can serve as a reference).
- Weight-slider UI doesn't expose a direct "liquidate inventory" objective toggle even though the data model supports an `objective` field for it — decide if that should be a first-class selector.

---

## 7. Deep Dive (rendered inline from Price Scenarios / reused in Approvals)

**Purpose:** SKU-level drill-down beneath a scenario's output — what specific price changes, marketing campaigns, and discounts make up the recommendation, progressively "unlocked" as the optimization-level slider increases.

**Key entities:**
- `DeepDiveProductView { productId, name, cost, msrp, currentPrice, recommendedPrice, why, discountName, priceChangePct, elasticity, qty, returnRate, revenue, profit, profitMargin, salesBeforeReturns, projectedSales, campaign, discCampaign }`.
- `DeepDiveMarketingTile { id, campaignId, channels[], budget, duration, reach, projectedROAS, productIds[] }`.
- `DeepDiveDiscountTile { id, type, savingsAvg, duration, productIds[] }`.

**Functional Requirements:**
1. ==**FR-DD-01**: Four tabs — All / Price Adjustments / Marketing / Discounts.==
2. ==**FR-DD-02**: Price Adjustments is a data-grid with grouped columns (Item Overview, Pricing, Financials, Inventory), toggleable between Products and SKUs, with CSV export.==
3. ==**FR-DD-03**: Each price-adjustment row has an "Explain" action opening a modal that shows: a plain-language summary of why this price is recommended, the optimization objective(s) driving it, a "decision ladder" visualizing how competing constraints (margin floor, liquidation target, price options considered) narrowed to the final recommended price, and contextual factors (competitor price range, weeks-of-supply, elasticity).==
4. ==**FR-DD-04**: Marketing and Discounts tabs show campaign/discount "tiles" (channel, budget, duration, reach/savings, associated products) with expandable per-product lists and CSV export.==
5. ==**FR-DD-05**: The number of visible price adjustments, marketing tiles, and discount tiles increases progressively as the optimization-level slider (carried over from Price Scenarios output) increases — deeper markdowns and secondary campaigns require higher optimization confidence before being surfaced.==

**Business Rules:**
- Progressive-unlock thresholds by optimization level: price increases need ≥10%; price holds always shown; discounts ≥15% need ≥35%; moderate markdowns need ≥25%; markdowns ≥15% need ≥60%; primary marketing/discount tile needs ≥20–30%, secondary needs ≥50–55%.
- Grid coloring: profit margin ≥65% green else amber; return rate >4% flagged amber.

**Gaps:**
- The per-SKU "why" explanation (competitor price range, decision ladder) is generated from simple synthetic math, not real competitive intelligence or a real constrained-optimization trace — production needs a real elasticity/optimization engine that can produce an actual explainability trace, and a real competitive-price feed.
- No category/brand/date filters exist in Deep Dive today beyond the optimization-level gate — confirm whether additional filtering is needed.

---

## 8. Guardrails

**Purpose:** Pricing Strategist–authored business constraints (margin floors, liquidation targets, markdown ceilings, price-change cadence) segmented by brand × division, that scenarios and discount models must respect.

**Key entity:**
- `GuardrailEntity { id, brand, division, rule, op, value, unit, active, isOverridable }`.

Example seeded rules: Gross Margin ≥ 38–42% (varies by brand/division, not overridable), End-of-season liquidation ≥ 84–85% (not overridable), Max markdown depth ≤ 35–40% (overridable), Price change frequency ≤ 2/week across all TCP (overridable).

**Functional Requirements:**
1. ==**FR-GR-01**: Table view of all guardrails: brand, division, rule, condition, value, "allow overrides" toggle, active toggle, delete.==
2. ==**FR-GR-02**: User can inline-edit a guardrail's threshold value.==
3. ==**FR-GR-03**: User can toggle a guardrail active/inactive without deleting it.==
4. ==**FR-GR-04**: User can toggle whether a guardrail is overridable by the Pricing Team when building a scenario.==
5. ==**FR-GR-05**: Only guardrails that are both active and overridable are surfaced as editable options in the Price Scenario creation flow; non-overridable active guardrails are enforced as hard constraints.==
6. ==**FR-GR-06**: User can add a new guardrail and delete an existing one.==

**Gaps:**
- **Add and Delete are not implemented in the prototype** (buttons present, not wired) — production needs full CRUD.
- Guardrail edits aren't persisted beyond the session today.
- No versioning/audit trail on guardrail definition changes — decide if that's required (given these are hard financial constraints, likely yes).
- No numeric-range or type validation on inline edits.
- The [[#9. Approvals|Approvals]] screen's guardrail-compliance check ([[#9. Approvals|§9]]) does not read live from this table — production should compute compliance live against these definitions, not hand-author it per scenario.

---

## 9. Approvals

**Purpose:** Pricing Strategist's review queue for scenarios and discount models submitted by the Pricing Team (and cross-domain sub-actions surfaced from the Performance Platform), gating financial risk before a price change proceeds.

**Key entities:**
- `ApprovalEntity { id, scenarioId, submitter, team, brand, division, impact, risk: Low|Medium|High, status: pending|approved|denied|changes_requested, issueId?, actionId? }`.
- `GuardrailCheck { rule, cond, ctx, threshold, projected, pass }` — per-scenario compliance record shown in the review drawer.
- Discount models carry their own risk/approval flow via the same status values, sourced from `DiscountModelEntity.status`.

**Functional Requirements — Shared (both tabs):**

1. ==**FR-AP-01**: Two tabs — Price Scenarios and Discounts — each showing a pending-count badge.==

2. ==**FR-AP-02**: Pending queue shows submitter, team, brand, division, estimated impact, and a risk badge (Low/Medium/High), with View / Request Changes / Deny / Approve actions per row.==

3. ==**FR-AP-05**: **Deny requires a mandatory reason** before it can be confirmed.==

4. ==**FR-AP-08**: Decided items move to a separate table (name, submitter, brand, division, impact, risk, status) with a read-only "View" reopening the full output.==

  

**Functional Requirements — Price Scenarios tab:**

5. ==**FR-AP-03**: For cross-domain sub-action approvals (standalone approvals with no backing `ScenarioEntity`), "View" opens a detail drawer: header context, a risk explanation, a [[#8. Guardrails|Guardrail Compliance]] list (pass/fail per rule with projected value vs. threshold), an Impact Summary (revenue, margin delta, sell-through, units), and a SKU-level breakdown (current → proposed price, depth %, units).==

6. ==**FR-AP-09**: For scenarios submitted through the [[#6. Price Scenarios|Price Scenario]] workflow, "View" instead opens the full Price Scenario output view (profit/revenue frontier chart, optimization slider, KPI table, "[[#7. Deep Dive|Deep Dive]]" entry point) in approval mode, with inline Approve/Deny/Request Changes actions replacing the submit button.==

7. ==**FR-AP-06a**: "Request Changes" captures a reviewer comment (**optional**) and returns the item to the submitter; the comment is appended to the item's change-request history.==

8. ==**FR-AP-07**: Approving a scenario tied to a cross-domain sub-action also marks that sub-action approved and links back to the Performance Platform's action tracker.==

  

**Functional Requirements — Discounts tab:**

9. ==**FR-AP-04**: "View" opens a detail drawer: header context, a risk banner summarizing hard/advisory constraint-violation counts, an Impact Summary (revenue, gross margin, units lift, sell-through), Constraint Warnings (hard/advisory severity counts feeding the risk rating), and Competitive Flags (SKUs priced below tracked competitor floors) — this drawer does not include an itemized Guardrail Compliance list or a SKU-level current→proposed price breakdown.==

10. ==**FR-AP-06b**: "Return for Changes" captures a reviewer comment (**mandatory**) and returns the item to the submitter; the comment is appended to the item's change-request history.==

**Business Rules:**
- Discount risk is derived: High if any hard constraint violation exists, Medium if any violation (advisory or hard), else Low.
- Fallback scenario risk (when no explicit risk metadata exists): High if inventory-risk-reduction estimate > $10M, Medium if > $5M, else Low.

**Gaps:**
- **Reason-required inconsistency**: Deny is mandatory everywhere; "Request Changes" on a scenario is optional while "Return for Changes" on a discount model is mandatory. Decide on one consistent policy.
- **No execution linkage**: approving a scenario/model today does not create any executable price-change record or automatically kick off a [[#12. Measurement|Measurement]] experiment — confirm whether that should happen automatically or via a separate manual step in production (the Measurement build spec proposes extending scenario status with `applied → measuring → measured`, not yet implemented).
- Guardrail-compliance figures shown here are hand-authored per historical scenario rather than computed live (see [[#8. Guardrails|§8]] Gap).

---

## 10. Agents (Agent Roster)

**Purpose:** Inventory and management surface for the AI pricing workforce, distinguishing three agent archetypes so the Strategist can see who's watching, who's acting, and who's ephemeral.

**Key concepts:**
- `AgentArchetype`: **Monitor** (standing, watches signals — e.g. Competitive-Price Monitor, Plan-Deviation Monitor, Elasticity-Drift Monitor, Margin-Erosion Monitor), **Operator** (an autonomous action class, derived live from the Autonomy trust ladder — not separately stored), **Task agent** (ephemeral, spawned for a specific job — e.g. a Measurement agent tied to one experiment, a Cold-Start pricing agent tied to a newness event, an Investigation agent spawned by another agent).
- Task agents record what spawned them (an event, a monitor, or a manual hire) and what condition retires them.
- `EvidenceStatus`: whether an operator's autonomy is backed by tested causal (A/B) proof, folded into a broader model, currently untestable/capped, or not applicable.

**Functional Requirements:**
1. ==**FR-AR-01**: KPI strip: agents on the team, signals generated today, count acting autonomously, count evidence-backed (of total), task agents currently running.==
2. ==**FR-AR-02**: Three sections: Monitors, Operators, Task Agents, each rendered as cards.==
3. ==FR-AR-03: Monitor cards can be paused/resumed and show a signals-today counter, plus a last-activity line summarizing the most recent signal it raised (or "no signals today" when idle).==
4. ==**FR-AR-04**: Operator cards show trust level, evidence status, and a track-record summary; clicking navigates to [[#11. Pricing Autonomy|Autonomy]] ([[#11. Pricing Autonomy|§11]]) for governance.==
5. ==**FR-AR-05**: Task-agent cards show what spawned them and their retirement condition, with an "open" deep link to the relevant working screen (e.g. Measurement, Like-Item Mapping) and a "retire" action to remove them early.==
6. ==**FR-AR-06**: User can "hire" a new standing agent from a provisionable catalog of additional monitor/operator types.==

**Business Rules:**
- An operator's displayed status reflects the global kill switch only if its trust level is at or above the threshold where it auto-executes — lower-trust operators are unaffected by the kill switch since they aren't autonomously acting anyway.
- "Acting autonomously" and "evidence-backed" KPIs are computed from the same trust-level and evidence-status data used in Autonomy.

**Gaps:**
- Retiring a task agent is immediate and has no undo.
- Agent Roster is not the same code as its Pricing Platform V2 counterpart — the two have already diverged (see [[#15. Cross-Cutting Issues|§15]]); a rebuild should pick one canonical version rather than maintain both.

---

## 11. Pricing Autonomy

**Purpose:** Governs how much unsupervised authority each *pricing action class* (not each SKU) has earned — a trust ladder from full human control to full autonomy — plus live supervision of in-flight autonomous actions and an emergency kill switch.

**Key concepts:**
- **Trust ladder** (low → high): **Recommend** (surface the insight, a human takes every action) → **Stage** (pre-filled and queued, human approves) → **Auto with Veto** (executes automatically after a veto window; human can cancel before it lands) → **Autonomous** (executes immediately and logs; reversible actions only).
- **Reversibility class** (low/medium/high) sets a hard ceiling on how high an action class can ever be promoted, regardless of track record: low → Stage max, medium → Auto-with-Veto max, high → Autonomous max.
- **Promotion gates**: an action class is eligible to move up a rung only once it has enough observed samples, and meets minimum accuracy and acceptance-rate thresholds (seed data uses ~85% accuracy and ~80% acceptance as the bar, with a target sample count per class).
- **Live action states**: Staged (awaiting human approval) → Auto-Pending-Veto (counting down, human can cancel) → Applied → (Vetoed | Rolled Back).
- **Autonomy caps**: independent of trust level, each action class has a max $ per action, max $ per day, and max actions per day.
- **Audit trail**: every promotion, demotion, kill-switch engagement, veto, auto-applied action, rollback, and model recalibration is logged.

**Functional Requirements:**
1. ==**FR-PA-01**: Emergency kill switch (global): engaging it halts new promotions and is intended to freeze all in-flight veto countdowns platform-wide.==
2. ==**FR-PA-02**: KPI strip: total action classes, count eligible to promote, total $ currently live under autonomy, average proof accuracy across classes.==
3. ==**FR-PA-03**: Grid of action-class cards (e.g. clearance re-pricing, markdown cadence, new-product entry pricing, full-price repositioning), each showing its current trust rung; selecting one populates a detail rail.==
4. ==**FR-PA-04**: Detail rail shows: a closed-loop proof panel (predicted-vs-realized accuracy trend over time, with a plain-language verdict), a live-supervision list (in-flight actions for this class with Veto/Undo controls), and an audit-trail feed.==
5. ==**FR-PA-05**: User can promote an action class up one rung only if it passes all promotion gates and is below its reversibility ceiling; promotion is blocked (with a clear reason) otherwise.==
6. ==**FR-PA-06**: User can demote an action class at any time, with no gating, always logged.==
7. ==**FR-PA-07**: User can veto an auto-pending action before its window elapses, or undo (roll back) an already-applied action.==
8. ==**FR-PA-08**: All promote/veto/undo controls are disabled while the kill switch is engaged.==

**Business Rules:**
- Promotion blockers are evaluated in order and can co-occur: reversibility ceiling reached (hard stop, others not shown), insufficient sample count, accuracy below threshold, acceptance rate below threshold.

**Gaps:**
- The kill switch's "veto windows stop counting down" behavior isn't actually enforced in the prototype's timer logic — needs a real implementation decision (pause countdowns vs. some other freeze semantic).
- "Staged" actions have no distinct Approve control in this screen (only Veto/Undo are wired) — confirm how a human is expected to approve a staged action.
- Live-action accuracy figures are static and not fed by real [[#12. Measurement|Measurement]] outcomes — production should close this loop (a completed A/B experiment should update the relevant action class's accuracy record).
- **This screen has already diverged from its Pricing Platform V2 counterpart**, which has added: a renamed lowest rung ("Shadow" instead of "Recommend"), a configurable autonomy *ceiling* independent of the reversibility cap, and richer deep-link/orientation behavior when arriving from the [[#10. Agents|Agent Roster]]. Decide which version is canonical before building — Pricing Platform V2's is more feature-complete on this specific screen.

---

## 12. Measurement

**Purpose:** Because every shopper sees the same price (no shopper-level randomization is possible), price-change impact must be proven via a matched-cluster A/B test: comparable-but-distinct SKU groups are assigned to treatment/control, and results are read out with a sequential test that can stop early on a win or kill signal.

**Key concepts:**
- `ClusterView` — a group of cross-elastic (substitute) SKUs assigned as a whole to one arm (treatment/control), with pre-period revenue/margin, a velocity score, BAU vs. ML-recommended price, predicted lift %, and a confidence score for why it was clustered this way.
- `BlockView` — a comparability stratum (same category × price band × season) pairing one treatment cluster with one or more control clusters.
- **Balance check**: within a block, treatment and control must be within ~20% of each other on revenue, gross margin, and velocity for the block to be considered "balanced"; a block also needs both arms present (excluding user-excluded clusters).
- **Go-live gate**: the experiment can only go live once every block is balanced (or explicitly resolved) *and* the user has explicitly acknowledged the cost of running a control (foregone revenue from not applying the price change everywhere).
- **Sequential readout**: a live probability-of-winning metric is tracked daily; the experiment concludes automatically once it crosses a "clear win" or "clear kill" boundary, without waiting for a fixed end date.

**Functional Requirements — Setup:**
1. ==**FR-ME-01**: Setup screen shows an AI-proposed arm assignment (treatment/control clusters grouped into comparability blocks) for the strategist to review and adjust before launch.==
2. ==**FR-ME-02**: KPI strip: blocks (balanced/total), treatment/control cluster counts, SKUs under test, estimated cost of running a control, and expected time-to-signal.==
3. ==**FR-ME-03**: Each block shows a Balanced / Imbalanced / Missing-an-arm status; selecting a cluster shows why it was grouped this way (cross-elasticity rationale + confidence), its BAU/ML price and predicted lift, and override controls to move it to the other arm or exclude it from the test.==
4. ==**FR-ME-04**: Block balance is visualized per metric (revenue, gross margin, velocity) with a match percentage and pass/fail indicator.==
5. ==**FR-ME-05**: A validity & cost panel requires the user to explicitly acknowledge the cost-of-control tradeoff before the experiment can go live.==
6. ==**FR-ME-06**: "Go Live" is disabled with an explanatory tooltip until all blocks are balanced/resolved and the cost has been acknowledged.==
7. ==**FR-ME-07**: Moving a cluster to control removes its ML price recommendation (control always runs BAU pricing); moving it back to treatment restores the original recommendation.==

**Functional Requirements — Live:**
8. ==**FR-ME-08**: Live view streams a daily probability-of-winning trend against explicit win/kill decision boundaries, with play/pause/restart controls over the day-by-day readout.==
9. **FR-ME-09**: KPI tiles: probability of beating business-as-usual, incremental margin estimate with a 95% credible interval, and the current observation day — each annotated with whether the confidence interval is clearly positive, clearly negative, or still inconclusive.
10. **FR-ME-10**: A per-cluster contribution view shows how much each active treatment cluster is contributing to the overall lift signal.
11. **FR-ME-11**: A verdict banner shows "gathering" while inconclusive, or "win"/"kill" once a boundary is crossed, with an explicit action to scale the win to all matching SKUs or kill and revert to BAU pricing.
12. **FR-ME-12**: Concluding an experiment (scale or kill) should feed the causal result back into the relevant pricing action class's accuracy track record (see [[#11. Pricing Autonomy|§11]] Gap — not implemented today).
13. ==**FR-ME-13**: User can reopen setup from a live/complete experiment to adjust and relaunch without losing prior overrides.==

**Business Rules:**
- Balance tolerance: arms must be within ~80% of each other (min/max ratio ≥ 0.8) on each of the three balance metrics.
- Decision boundaries: experiment concludes as a win once win-probability crosses roughly the high-90s%, or as a kill once it falls to roughly low single digits.

**Gaps:**
- The current implementation is a simplified prototype of the methodology described in the Measurement build spec, which calls for a real difference-in-differences / CUPED estimator with bootstrapped confidence intervals, a richer stratification design (brand × division × category × price-tier × velocity-band), and a formal experiment/outcome data model — none of which exist as real computation today (the live readout uses pre-authored "winning"/"losing" demo trajectories).
- No real link exists yet between a concluded experiment and (a) the originating scenario's status, or (b) the relevant [[#11. Pricing Autonomy|Autonomy]] action class's accuracy record, despite both being narrated as happening.
- Per the build spec, an executive-level summary of measurement outcomes is intended to live in the Performance Platform, not here — not built anywhere yet.
- Cold-start (new) SKUs are expected to use a related but distinct measurement approach (matched-control comparison instead of pre/post, since there's no pre-period) — confirm this is in scope and how it should share methodology with [[#13. Like-Item Mapping|§13]].

---

## 13. Like-Item Mapping (Cold Start)

**Purpose:** A new-season product has no sales history. This screen surfaces an ML-proposed mapping from each new product to its closest prior-season comparable ("comp"), borrowing that comp's demand/elasticity assumptions to set an entry price — and lets a merchant validate or override each mapping.

**Key entities:**
- `ColdStartMappingEntity { id, newProductId (FK), systemSuggestedId, selectedCandidateId, merchantChoseId, candidates[], confidenceScore, confidenceTier: high|med|low, status: pending|validated|overridden, overrideReason, validatedAt, validatedBy }`.
- `CompCandidateView` — a ranked candidate comp: similarity score, confidence score + breakdown by driver, matched/mismatched attributes, borrowed elasticity/demand priors, comp MSRP and entry price.
- `ImpactPreviewView { inheritedElasticity, suggestedEntryPrice, msrp, cost, initialMarkdownPct, entryVsMsrpPct }`.

**Functional Requirements:**

1. ==**FR-CS-01**: Queue view lists new products grouped by category, each row showing the new product, its top suggested comp, similarity %, confidence tier, borrowed elasticity, suggested entry price, and validation status.==

2. ==**FR-CS-02**: User can filter by division, department (dependent on division), status, and free-text search.==

3. ==**FR-CS-03**: Each category group shows its pending count, a low-confidence count, average confidence, and a bulk "accept all pending in this group" action.==

4. ==**FR-CS-04**: Rows are prioritized so pending, lowest-confidence items surface first within a group.==

5. ==**FR-CS-05**: Detail view opens with a plain-language summary banner narrating the recommended mapping (or, if the merchant has changed the selection, the override): which comp is being mapped to, the confidence score and tier backing it, how many of the tracked attributes align, and the resulting entry price and % markdown off MSRP.==

6. ==**FR-CS-06**: Detail view shows the new product alongside all ranked comp candidates (photo, similarity %, confidence, top matched attributes, elasticity), with the system's default recommendation visibly marked ("System pick") among the candidates so it's distinguishable from the merchant's current selection.==

7. ==**FR-CS-07**: Detail view additionally shows a confidence breakdown by driver, a full attribute match/mismatch grid, borrowed-priors rationale, a side-by-side product-data compare panel, and a pricing-impact preview (inherited elasticity, baseline demand, return rate, seasonality, suggested entry price vs. MSRP).==

8. ==**FR-CS-08**: If the top two candidates' confidence scores are very close, the UI flags this as an ambiguous "close call" worth manual review.==

9. ==**FR-CS-09**: User can search the full catalog (not just the ranked shortlist) to pick an alternate comp for a mapping.==

10. ==**FR-CS-10**: User can validate the system's top pick, or override it with a different candidate; overriding accepts an optional reason (explicitly recorded in the audit trail but not required).==

11. ==**FR-CS-11**: Validated/overridden mappings display a stamped audit line (who, when, and — for overrides — what was suggested vs. what was chosen).==

**Business Rules / Confidence Scoring:**
- Confidence is a weighted blend of four drivers: attribute match (~40%), product-data/PIM similarity (~20%), price-band match (~15%), and comp data depth — i.e., how much sales history and SKU breadth the comp itself has (~25%).
- Confidence tiers: ≥80 high, ≥60 medium, else low.
- Candidate pool falls back progressively from the tightest scope (division + department + category) to broader scopes if fewer than 3 candidates are found.
- Suggested entry price applies a deeper opening discount off MSRP for more price-elastic comps (bounded between roughly a 5% and 25% opening markdown).
- Status transitions: `pending → validated` (accept, individually or in bulk by category) or `pending → overridden` (pick a different comp); no path back to pending is exposed in the UI.

**Gaps:**
- **The entire matching/confidence engine, PIM enrichment, and comp sell-through history are synthetic** in the prototype (deterministically generated from the catalog, not a real similarity/embedding model or real PIM/sales-history feed). Production needs a genuine look-alike model and real PIM/DAM + historical-sales integration.
- Override reason is optional here, unlike the mandatory-reason pattern used for denials in [[#9. Approvals|Approvals]] ([[#9. Approvals|§9]]) — decide if cold-start overrides should also require a reason given they set live pricing.
- No reversal path from validated/overridden back to pending is exposed to users — confirm whether that should be possible (e.g., if a mistake is caught later).
- Per the Measurement build spec, cold-start mappings are expected to share matching methodology with [[#12. Measurement|Measurement]]'s control-cluster design ([[#12. Measurement|§12]]) — not currently wired together.

---

## 14. Configuration

**Purpose:** Minimal admin view of user access — who can see/edit which brand/division scope.

**Key entity:**
- `AppUserEntity { id, name, role, access: string[], status: active|inactive }`.

**Functional Requirements:**
1. ==**FR-CF-01**: Read-only list of users showing name, role, active/inactive status, and category (brand/division) access.==

**Gaps — this screen is a stub:**
- No add/edit/deactivate/role-change actions exist at all.
- No other screen in the app actually enforces brand/division scoping based on a user's access list — if row-level security by brand/division is a real requirement, it needs to be designed and enforced platform-wide, not just displayed here.
- Production should source this from a real identity provider (SSO/IAM), not a static seed list.

---

## 15. Cross-Cutting Issues

1. **Fragmented discount/promo model.** At least three separate "discount tied to a product scope" shapes exist across [[#2. Focus Builder|Focus Builder]], [[#5. Discount Modeling|Discount Modeling]], and [[#3. Promotions|Promotions]], with no shared entity or FK discipline between them. Production should define one canonical Promotion/Discount entity with a clear lifecycle and real foreign keys to Focus Set.
2. **Focus Set as shared infrastructure.** [[#5. Discount Modeling|Discount Modeling]], [[#3. Promotions|Promotions]], [[#6. Price Scenarios|Price Scenarios]], and [[#4. Product Grid|Product Grid]] all depend on consistent resolution of "which SKUs belong to this Focus Set" — this should be a single backend-owned resource/service, not independently re-derived per screen.
3. **No real ML/forecasting anywhere yet.** [[#5. Discount Modeling|Discount Modeling]], [[#6. Price Scenarios|Price Scenarios]], and [[#13. Like-Item Mapping|Like-Item Mapping]] all currently fake their core "intelligence" with deterministic formulas or hashing. The data contracts (output shapes) defined in each screen's types are a reasonable target for what a real backend needs to produce — but the actual modeling work is entirely unbuilt.
4. **Guardrails are advisory today, not enforced.** Constraint violations are surfaced but never block a workflow transition (submission, approval) anywhere in the prototype. Decide, screen by screen, where a hard violation should actually block progress vs. just warn.
5. **Reason-required inconsistency.** Deny requires a reason; "Request Changes" is optional in one place and mandatory in another; Cold-Start overrides don't require one at all. Needs one consistent policy across the platform for any action that overrides a system recommendation or rejects a submission.
6. **No enforced brand/division access control**, despite [[#14. Configuration|Configuration]] implying it should exist.
7. **The "Ask anything" AI entry point present in the shell header is a non-functional stub across the app** — confirm whether a conversational/command surface is in scope for this rebuild or explicitly out of scope.

---

