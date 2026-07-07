---
status: Approved
artifact_type: platform-spec
feature_id: 000-platform
approved_by: Saad
approved_at: 07/07/2026
source: knowledge/PLATFORM-REQUIREMENTS.md
---

# Platform Specification

## Status

`Approved` — approved by Saad on 07/07/2026 (see front matter). `/create-platform-plan` may now proceed.

> **Speckit note:** This spec follows Speckit specification conventions and structure. It was authored directly rather than via the `/speckit-specify` script (`.specify/scripts/powershell/create-new-feature.ps1`), because that script creates a new git feature branch and an auto-numbered `specs/NNN-<slug>/` directory, which conflicts with (a) this harness's requirement to stay on `main` and stop on unexpected branch state (constitution Principle II; `multi-repo-workspace-safety`), and (b) the mandated fixed output path `knowledge/specs/000-platform/spec.md`. The `/create-platform-spec` command explicitly authorizes direct authoring in this case.

## Source Documents

- `knowledge/PLATFORM-REQUIREMENTS.md` — primary product input (15 sections: platform overview + 13 screens + cross-cutting issues).
- `knowledge/.specify/memory/constitution.md` — governance principles, approval gates, quality gates.
- `knowledge/TECHNOLOGY.md` — pinned production stack (governance constraint only).
- `knowledge/DATA-TAXONOMY.md` — Tier-1/Tier-2 data discipline, entity catalog, FK summary.
- `knowledge/reports/platform-requirements-analysis.md` — prior analysis (domains, personas, candidate features, risks).
- `knowledge/.specify/templates/spec-template.md` — Speckit spec conventions.
- `knowledge/specs/**` — none present (greenfield).
- `knowledge/contracts/**` — none present.

## Platform Overview

Retail Nucleus is a B2B retail **pricing intelligence platform** for The Children's Place (TCP) and Gymboree brands. It lets a **Pricing Team** define reusable product scopes ("Focus Sets") and run AI/ML-driven pricing tools against them — discount modeling, price-scenario optimization, promotions scheduling, cold-start (new-product) entry pricing, and matched-cluster A/B measurement. A **Pricing Strategist** governs the financial risk of those actions through Guardrails, an Approvals queue, an AI-agent roster, and a graduated **Autonomy** trust ladder (from human-recommend to fully autonomous execution). An **Admin** manages user access.

**Business value:** move pricing and markdown decisions from spreadsheet-driven guesswork to a governed, auditable workflow where every price change is scoped, simulated, risk-checked against explicit guardrails, human-approved, and (eventually) causally measured — with AI agents able to earn graduated autonomy under hard financial limits.

**Scope note:** this spec covers the RetailNucleus app only (route `/retail-nucleus/*`). Pricing Platform V2 Agentic (`/pricing-platform-v2/*`) is a separate, partially-diverged fork and is out of scope except where a canonical-version decision is required (Agents, Autonomy).

**Prototype reality that shapes this spec:** `PLATFORM-REQUIREMENTS.md` was reverse-engineered from a design-reference prototype. The core "intelligence" (Discount Modeling, Price Scenarios, Like-Item Mapping) is faked with deterministic formulas today, guardrails are advisory (never block), several CRUD operations are UI-only stubs, and the closed loops between Measurement → Autonomy → Approvals are narrated but unbuilt. The documented output *shapes* are a credible target contract; the modeling itself is unbuilt. These are called out as `[NEEDS CLARIFICATION]` and in Out of Scope.

## Personas

Only personas supported by `PLATFORM-REQUIREMENTS.md` are included.

- **Pricing Team** — builds Focus Sets, runs Price Scenarios and Discount Models, manages Promotions, reviews Like-Item Mappings, sets up Measurement experiments. Creates and submits work for approval; does not approve.
- **Pricing Strategist** — authors Guardrails, approves/denies scenarios and discount models, governs agent trust levels and autonomy, manages the agent roster. Holds the emergency kill switch.
- **Admin** — configuration and user access (currently a stub).

Personas referenced by the constitution but **not** supported by `PLATFORM-REQUIREMENTS.md` for this app's scope — **Executive** and **Marketing** — are treated as Out of Scope / Open Questions rather than served here. A **Merchant** role is implied by Like-Item Mapping (§13) and is flagged as an Open Question (may be the same as Pricing Team).

## Product Domains

1. **Scoping** — Focus Sets (the shared scoping primitive) and the Product Grid SKU browser.
2. **Promotions** — calendar/list scheduling of promotional events tied to Focus Sets.
3. **Discount Modeling** — AI/ML markdown simulation → margin/revenue/risk forecast → approval.
4. **Price Scenario Optimization** — ML what-if pricing with a profit/revenue frontier and SKU-level Deep Dive.
5. **Guardrails** — Strategist-authored financial constraints by brand × division.
6. **Approvals** — Strategist review queue gating scenarios, discount models, and cross-domain actions.
7. **Agents** — roster of AI monitors/operators/task agents.
8. **Pricing Autonomy** — trust-ladder governance, live-action supervision, kill switch.
9. **Measurement** — matched-cluster A/B experimentation with sequential readout.
10. **Cold Start (Like-Item Mapping)** — new-product entry pricing via prior-season comparables.
11. **Administration** — user access and (potential) brand/division scoping.
12. **Platform Shell (cross-cutting)** — navigation, global brand/channel filters, AI entry point.

## User Stories

### Scoping

- As a **Pricing Team** member, I want to create and save reusable named Focus Sets via an AND/OR condition tree, so that every downstream pricing tool can target a consistent product scope.
- As a **Pricing Team** member, I want to browse, search, sort, duplicate, edit, delete, and export Focus Sets, so that I can manage scopes efficiently.
- As a **Pricing Team** member, I want to inspect and soft-exclude SKUs in a Focus Set via the Product Grid, so that the scope is clean before it is used downstream.

### Promotions

- As a **Pricing Team** member, I want to schedule promotions on a calendar tied to a Focus Set and discount terms, so that I can visualize overlap and pacing.
- As a **Pricing Team** member, I want to see the per-SKU promo price for a promotion's Focus Set, so that I can confirm the discount lands correctly.

### Discount Modeling

- As a **Pricing Team** member, I want to run a markdown simulation against a Focus Set and review a full margin/revenue/risk forecast, so that I can de-risk a markdown before submitting it for approval.

### Price Scenario Optimization

- As a **Pricing Team** member, I want to run an ML pricing scenario and explore the revenue/profit tradeoff frontier via an optimization slider, so that I can choose a risk level and justify per-SKU price changes.
- As a **Pricing Team** member, I want a SKU-level Deep Dive with per-item explanations, so that I can understand and defend each recommended price.

### Guardrails

- As a **Pricing Strategist**, I want to author and manage margin/markdown/cadence guardrails by brand × division, so that all pricing actions respect explicit financial constraints.

### Approvals

- As a **Pricing Strategist**, I want a review queue for submitted scenarios and discount models with risk badges and guardrail compliance, so that I gate financial risk before a price change proceeds.

### Agents & Autonomy

- As a **Pricing Strategist**, I want to see and manage the AI pricing workforce, so that I know who is watching, who is acting, and who is ephemeral.
- As a **Pricing Strategist**, I want to govern how much unsupervised authority each pricing action class has, with live supervision and an emergency kill switch, so that AI pricing stays within safe, reversible limits.

### Measurement

- As a **Pricing Strategist**, I want to set up and monitor matched-cluster A/B experiments with a sequential readout, so that price-change impact can be proven causally without shopper-level randomization.

### Cold Start

- As a **Merchant / Pricing Team** member, I want to validate or override ML-proposed comparable mappings for new products, so that I can set defensible entry prices without sales history.

### Administration

- As an **Admin**, I want to see (and eventually manage) which users can access which brand/division scope, so that access is governed.

## Functional Requirements

EARS notation. Each requirement carries a stable ID and traces back to the `FR-*` IDs in `PLATFORM-REQUIREMENTS.md`.

### Focus Builder — `REQ-FOCUS-*`

- **REQ-FOCUS-001** (FR-FB-01): The system shall present a library of saved Focus Sets showing name, ID, criteria rendered as an AND/OR chip expression, created date, and matched product count.
- **REQ-FOCUS-002** (FR-FB-02): Where the Focus Set library is shown, the system shall allow sorting by name, created date, and product count (ascending/descending) and searching by name (case-insensitive substring).
- **REQ-FOCUS-003** (FR-FB-03): When a user creates a Focus Set, the system shall require a name and accept a condition tree of attribute-equals-value rules grouped by AND/OR, nestable up to 3 levels deep.
- **REQ-FOCUS-004** (FR-FB-04): While a user is building a Focus Set filter, the system shall show a live preview of the first 8 matching SKUs (SKU, name, division, category, qty, status), a total match count, and a "no matches" warning when the filter matches nothing.
- **REQ-FOCUS-005** (FR-FB-05): When a user edits an existing Focus Set, the system shall update its name and filter in place, preserving the same ID.
- **REQ-FOCUS-006** (FR-FB-06): When a user duplicates a Focus Set, the system shall create a copy under a new name inheriting the original filter.
- **REQ-FOCUS-007** (FR-FB-07): When a user deletes a Focus Set, the system shall remove it from the library and confirm the deletion.
- **REQ-FOCUS-008** (FR-FB-08): When a user exports a Focus Set, the system shall produce a CSV of matched SKUs (SKU, Name, Division, Category, Price, Qty, Status).
- **REQ-FOCUS-009** (FR-FB-09): When a user drills into a Focus Set, the system shall open its full product/SKU grid (Product Grid).
- **REQ-FOCUS-010** (FR-FB-10): The system shall match rules by exact per-attribute equality with no range/contains/wildcard operators, and shall treat an empty rule list as matching everything.
- **REQ-FOCUS-011** (business rule): The system shall assign Focus Set IDs following the pattern `focus-set-##`, auto-incremented from the current maximum.
- **REQ-FOCUS-012** (§15.2 gap): The system shall resolve a Focus Set to its SKUs through a single canonical operation such that displayed criteria and matched products always agree. `[NEEDS CLARIFICATION: live evaluation vs. persisted snapshot]`

### Product Grid — `REQ-GRID-*`

- **REQ-GRID-001** (FR-PG-01): Where a Focus Set is opened in the Product Grid, the system shall allow toggling between product-level (rolled-up) and SKU-level (expanded) views, each showing a live row count.
- **REQ-GRID-002** (FR-PG-02): The system shall allow free-text search/filter of the grid and export of the current view to CSV.
- **REQ-GRID-003** (FR-PG-03): The system shall display per row: category icon, IDs, name/description, color or SKU count, MSRP (struck through if discounted), current price (or range when collapsed with mixed pricing), discount name, quantity, on-order quantity, and stock status.
- **REQ-GRID-004** (FR-PG-04): When a user removes a SKU or a product's SKUs, the system shall soft-delete them from the active view into a "Deleted Items" pane.
- **REQ-GRID-005** (FR-PG-05): When a user restores deleted rows, the system shall restore individual rows or all of them at once from the Deleted Items pane.
- **REQ-GRID-006** (FR-PG-06): The system shall display the Focus Set criteria read-only above the grid.
- **REQ-GRID-007** (§4 gap): The system shall apply one canonical inventory-health / qty color scale consistent with Focus Builder. `[NEEDS CLARIFICATION: reconcile thresholds]`

### Promotions — `REQ-PROMO-*`

- **REQ-PROMO-001** (FR-PC-01): The system shall present promotions in a Calendar (month grid, lane-packed multi-day bars) or List (sortable table) view.
- **REQ-PROMO-002** (FR-PC-02): Where promotions are shown, the system shall allow filtering by status (All / Active / Scheduled / Expired) with live counts per tab.
- **REQ-PROMO-003** (FR-PC-03): When in calendar view, the system shall allow month navigation (previous / next / today).
- **REQ-PROMO-004** (FR-PC-04): When a user opens a promotion, the system shall show a detail drawer with status/channel badges, date range and duration, discount summary, linked Focus Set (name, product count, criteria, "View Products"), notes, and Edit/Delete actions.
- **REQ-PROMO-005** (FR-PC-05): When a user selects "View Products" on a promotion, the system shall show up to 20 SKUs in the linked Focus Set with a computed promo price per SKU.
- **REQ-PROMO-006** (FR-PC-06): When a user creates or edits a promotion, the system shall require name, start date (not in the past), end date (strictly after start, auto-advanced when needed), discount type, a discount value where the type needs a depth, a Focus Set (searchable, with a shortcut to create one), channel, calendar color, and optional notes.
- **REQ-PROMO-007** (FR-PC-07): When a user deletes a promotion, the system shall remove it.
- **REQ-PROMO-008** (business rule): The system shall derive promotion status at render time from dates versus today (start > today → scheduled; end < today → expired; else active) and default new promotions to scheduled.

### Discount Modeling — `REQ-DISC-*`

- **REQ-DISC-001** (FR-DM-01): The system shall group discount models into New / Pending Review (including returned) / Approved / Denied, showing name, focus group, format, channel, SKU count, horizon, created date, and status.
- **REQ-DISC-002** (FR-DM-01a): When a user clicks any existing model row, the system shall open that model's Output view directly, bypassing the create form and Run step, regardless of status.
- **REQ-DISC-003** (FR-DM-02): When a user creates a model, the system shall require name and Focus Set (with live estimated-SKU-count preview) and accept start/end date, discount format (%/$/BOGO/Fixed), depth (hidden for BOGO), and channel.
- **REQ-DISC-004** (FR-DM-03): While all required fields are set, the system shall show a live preview (discount description, estimated SKUs, duration, generated marketing handle).
- **REQ-DISC-005** (FR-DM-04): The system shall enable "Run Model" only once name, focus group, both dates, and a valid depth (or BOGO) are set, and on run shall produce the full `DiscountModelOutput`.
- **REQ-DISC-006** (FR-DM-05): The system shall present an Output view with AI narrative + copyable marketing handle, a 5-KPI Impact Summary, three forecast charts, a tab-switchable rollup table, and 6 collapsible risk panels.
- **REQ-DISC-007** (FR-DM-06): The system shall color-code rollup sell-through (≥85% green / ≥70% amber / <70% red) and flag stock-out risk per row.
- **REQ-DISC-008** (FR-DM-07): The system shall distinguish hard vs advisory constraint-warning severity and escalate visually if any hard violation exists.
- **REQ-DISC-009** (FR-DM-08): When a user submits a model, the system shall set its status to pending; when a user discards a model, the system shall permanently remove it after confirmation regardless of current status.
- **REQ-DISC-010** (FR-DM-09): The system shall allow CSV export of the SKU-level detail table and show a per-row confidence rating.

### Price Scenario — `REQ-SCEN-*`

- **REQ-SCEN-001** (FR-PS-01): The system shall list all scenarios (name, status, created, focus set, objective) and, when a non-draft scenario is clicked, reopen its output.
- **REQ-SCEN-002** (FR-PS-02): When a user creates a scenario, the system shall require name, Focus Set, and start/end dates, and provide three linked objective sliders (Revenue / Gross Margin / Sell-Through) that must sum to 100%, redistributing proportionally when one moves.
- **REQ-SCEN-003** (FR-PS-03): While creating a scenario, the system shall show a live preview (estimated SKUs, duration, weighted objective summary) and a read-only list of active + overridable guardrails pre-filled from Pricing Strategist defaults.
- **REQ-SCEN-004** (FR-PS-04): When a user runs a scenario with valid required fields, the system shall execute a multi-phase simulation (Product Look-alikes → Baseline Demand & Elasticity → Promotion Calendar → Inventory Velocity → Business Guardrails → Business Objectives & Goals).
- **REQ-SCEN-005** (FR-PS-05): The system shall present a profit-vs-revenue frontier chart with three marked points (Current, ML Recommendation, Scenario Position) and an interactive 0–100% Optimization Level slider that moves the Scenario Position along the frontier.
- **REQ-SCEN-006** (FR-PS-06): The system shall label the optimization slider by risk band (0–20% Conservative / 20–40% Moderate / 40–60% Balanced / 60–80% Aggressive / 80–100% Full Optimization), each with an expected uplift range and risk description, with a help modal available.
- **REQ-SCEN-007** (FR-PS-07): The system shall present a 3-column comparison table (Current / Scenario / ML Rec.) across Revenue, Profit, Rev Uplift, Margin Δ, Full-Price Mix, Sell-Through, Inventory Risk, plus an AI narrative and tagged recommendations (Pricing/Marketing/Merch/Inventory).
- **REQ-SCEN-008** (FR-PS-08): When a user submits a scenario, the system shall set status to pending; the system shall allow discard (with confirmation) and, once returned with change requests, show the latest reviewer comment and allow resubmit.
- **REQ-SCEN-009** (FR-PS-09): While a scenario status is pending, approved, or denied, the system shall freeze the slider and further edits.
- **REQ-SCEN-010** (FR-PS-10): Where a dashboard alert deep-links into scenario creation, the system shall pre-fill context (division, issue, recommendation) shown as a banner. `[NEEDS CLARIFICATION: no live caller today]`
- **REQ-SCEN-011** (business rule): The system shall present all projections with explicit uncertainty framing (e.g. "±8–15% depending on data recency").

### Deep Dive — `REQ-DEEP-*`

- **REQ-DEEP-001** (FR-DD-01): The system shall present four tabs — All / Price Adjustments / Marketing / Discounts.
- **REQ-DEEP-002** (FR-DD-02): The system shall present Price Adjustments as a data grid with grouped columns (Item Overview, Pricing, Financials, Inventory), a Products/SKUs toggle, and CSV export.
- **REQ-DEEP-003** (FR-DD-03): When a user selects "Explain" on a price-adjustment row, the system shall open a modal showing a plain-language rationale, the driving objective(s), a decision ladder of competing constraints, and contextual factors (competitor price range, weeks-of-supply, elasticity).
- **REQ-DEEP-004** (FR-DD-04): The system shall present Marketing and Discounts tabs as campaign/discount tiles with expandable per-product lists and CSV export.
- **REQ-DEEP-005** (FR-DD-05): While the optimization-level slider increases, the system shall progressively reveal more price adjustments, marketing tiles, and discount tiles per the defined unlock thresholds.

### Guardrails — `REQ-GUARD-*`

- **REQ-GUARD-001** (FR-GR-01): The system shall present a table of guardrails (brand, division, rule, condition, value, allow-overrides toggle, active toggle, delete).
- **REQ-GUARD-002** (FR-GR-02): When a user edits a guardrail threshold inline, the system shall persist the new value.
- **REQ-GUARD-003** (FR-GR-03): When a user toggles a guardrail active/inactive, the system shall change its state without deleting it.
- **REQ-GUARD-004** (FR-GR-04): When a user toggles overridable, the system shall change whether the Pricing Team may override it during scenario creation.
- **REQ-GUARD-005** (FR-GR-05): The system shall surface only active-and-overridable guardrails as editable options in Price Scenario creation, and shall enforce non-overridable active guardrails as hard constraints.
- **REQ-GUARD-006** (FR-GR-06): When a user adds or deletes a guardrail, the system shall persist the change (full CRUD).
- **REQ-GUARD-007** (§8/§9 gap): The system shall compute guardrail compliance live against current definitions and projected impact rather than from hand-authored values. `[NEEDS CLARIFICATION: versioning/audit required?]`

### Approvals — `REQ-APPR-*`

- **REQ-APPR-001** (FR-AP-01): The system shall present two tabs (Price Scenarios, Discounts), each with a pending-count badge.
- **REQ-APPR-002** (FR-AP-02): The system shall show, per pending item, submitter, team, brand, division, estimated impact, and a risk badge (Low/Medium/High), with View / Request Changes / Deny / Approve actions.
- **REQ-APPR-003** (FR-AP-05): If a user denies an item, then the system shall require a mandatory reason before the denial can be confirmed.
- **REQ-APPR-004** (FR-AP-08): The system shall move decided items to a separate table (name, submitter, brand, division, impact, risk, status) with a read-only "View" reopening the full output.
- **REQ-APPR-005** (FR-AP-03): When a user views a cross-domain sub-action approval (no backing scenario), the system shall open a drawer with header context, risk explanation, guardrail-compliance list (pass/fail vs threshold), impact summary, and SKU-level breakdown.
- **REQ-APPR-006** (FR-AP-09): When a user views a scenario submitted through the Price Scenario workflow, the system shall open the full scenario output in approval mode with inline Approve/Deny/Request Changes actions.
- **REQ-APPR-007** (FR-AP-06a): When a user requests changes on a scenario, the system shall accept an optional comment, return the item to the submitter, and append the comment to change-request history.
- **REQ-APPR-008** (FR-AP-07): When a user approves a scenario tied to a cross-domain sub-action, the system shall also mark that sub-action approved and link back to the action tracker.
- **REQ-APPR-009** (FR-AP-04): When a user views a discount model, the system shall open a drawer with header context, a risk banner (hard/advisory counts), impact summary, constraint warnings, and competitive flags, without an itemized guardrail-compliance list or SKU-level price breakdown.
- **REQ-APPR-010** (FR-AP-06b): When a user returns a discount model for changes, the system shall require a mandatory comment, return the item to the submitter, and append the comment to change-request history.
- **REQ-APPR-011** (business rule): The system shall derive discount risk (High if any hard violation; Medium if any violation; else Low) and fallback scenario risk (High if inventory-risk-reduction > $10M; Medium if > $5M; else Low).
- **REQ-APPR-012** (§15.5 gap): The system shall apply one consistent reason-required policy across deny / request-changes / override actions. `[NEEDS CLARIFICATION: which policy]`

### Agents — `REQ-AGENT-*`

- **REQ-AGENT-001** (FR-AR-01): The system shall present a KPI strip: agents on the team, signals generated today, count acting autonomously, count evidence-backed (of total), and task agents currently running.
- **REQ-AGENT-002** (FR-AR-02): The system shall present three sections — Monitors, Operators, Task Agents — as cards.
- **REQ-AGENT-003** (FR-AR-03): When a user pauses or resumes a monitor, the system shall change its state and show a signals-today counter and a last-activity line.
- **REQ-AGENT-004** (FR-AR-04): The system shall show operator cards with trust level, evidence status, and track record, navigating to Autonomy when clicked.
- **REQ-AGENT-005** (FR-AR-05): The system shall show task-agent cards with what spawned them and their retirement condition, an "open" deep link, and a "retire" action.
- **REQ-AGENT-006** (FR-AR-06): When a user hires a standing agent, the system shall add it from a provisionable catalog of monitor/operator types.
- **REQ-AGENT-007** (§10 gap): The system shall use one canonical Agents implementation. `[NEEDS CLARIFICATION: reconcile with Pricing Platform V2 fork]`

### Pricing Autonomy — `REQ-AUTO-*`

- **REQ-AUTO-001** (FR-PA-01): When a user engages the emergency kill switch, the system shall halt new promotions and freeze all in-flight veto countdowns platform-wide.
- **REQ-AUTO-002** (FR-PA-02): The system shall present a KPI strip: total action classes, count eligible to promote, total $ currently live under autonomy, and average proof accuracy.
- **REQ-AUTO-003** (FR-PA-03): The system shall present action-class cards showing the current trust rung, and populate a detail rail when one is selected.
- **REQ-AUTO-004** (FR-PA-04): The system shall show, in the detail rail, a closed-loop proof panel (predicted-vs-realized accuracy trend + verdict), a live-supervision list with Veto/Undo, and an audit-trail feed.
- **REQ-AUTO-005** (FR-PA-05): When a user promotes an action class, the system shall allow the promotion only if it passes all promotion gates and is below its reversibility ceiling, blocking otherwise with a clear reason.
- **REQ-AUTO-006** (FR-PA-06): When a user demotes an action class, the system shall allow it at any time without gating and log it.
- **REQ-AUTO-007** (FR-PA-07): When a user vetoes an auto-pending action before its window elapses, or undoes an applied action, the system shall perform and log that action.
- **REQ-AUTO-008** (FR-PA-08): While the kill switch is engaged, the system shall disable all promote/veto/undo controls.
- **REQ-AUTO-009** (business rule): The system shall evaluate promotion blockers in order (reversibility ceiling → sample count → accuracy → acceptance rate), and shall enforce reversibility class as a hard ceiling on trust rung.
- **REQ-AUTO-010** (§11 gap): The system shall implement the kill-switch countdown-freeze semantics. `[NEEDS CLARIFICATION: exact freeze behavior; canonical version vs V2 fork]`

### Measurement — `REQ-MEAS-*`

- **REQ-MEAS-001** (FR-ME-01): The system shall present an AI-proposed arm assignment (treatment/control clusters grouped into comparability blocks) for review and adjustment before launch.
- **REQ-MEAS-002** (FR-ME-02): The system shall present a KPI strip: blocks (balanced/total), treatment/control cluster counts, SKUs under test, estimated cost of running a control, and expected time-to-signal.
- **REQ-MEAS-003** (FR-ME-03): The system shall show per-block Balanced/Imbalanced/Missing-an-arm status, and per-cluster rationale (cross-elasticity + confidence), BAU/ML price, predicted lift, and override controls (move arm / exclude).
- **REQ-MEAS-004** (FR-ME-04): The system shall visualize block balance per metric (revenue, gross margin, velocity) with match percentage and pass/fail indicator.
- **REQ-MEAS-005** (FR-ME-05): The system shall require explicit acknowledgment of the cost-of-control tradeoff before an experiment can go live.
- **REQ-MEAS-006** (FR-ME-06): If not all blocks are balanced/resolved and the cost is not acknowledged, then the system shall keep "Go Live" disabled with an explanatory tooltip.
- **REQ-MEAS-007** (FR-ME-07): When a user moves a cluster to control, the system shall remove its ML price recommendation (control runs BAU); moving it back shall restore the recommendation.
- **REQ-MEAS-008** (FR-ME-08): While an experiment is live, the system shall stream a daily probability-of-winning trend against win/kill boundaries with play/pause/restart controls.
- **REQ-MEAS-009** (FR-ME-09): The system shall present KPI tiles: probability of beating BAU, incremental margin estimate with a 95% credible interval, and current observation day, each annotated as clearly positive / clearly negative / inconclusive.
- **REQ-MEAS-010** (FR-ME-10): The system shall present a per-cluster contribution view showing each treatment cluster's contribution to the overall lift signal.
- **REQ-MEAS-011** (FR-ME-11): The system shall present a verdict banner ("gathering" / "win" / "kill") with an action to scale the win to all matching SKUs or kill and revert to BAU.
- **REQ-MEAS-012** (FR-ME-12): When an experiment concludes, the system shall feed the causal result into the relevant pricing action class's accuracy record. `[NEEDS CLARIFICATION: closed loop unbuilt today]`
- **REQ-MEAS-013** (FR-ME-13): When a user reopens setup from a live/complete experiment, the system shall allow adjustment and relaunch without losing prior overrides.
- **REQ-MEAS-014** (business rule): The system shall treat arms as balanced when min/max ratio ≥ 0.8 on each of revenue, gross margin, and velocity, and conclude the experiment on crossing the high-90s% (win) or low-single-digit% (kill) boundary.

### Cold Start / Like-Item Mapping — `REQ-COLD-*`

- **REQ-COLD-001** (FR-CS-01): The system shall present a queue of new products grouped by category, each row showing the new product, top suggested comp, similarity %, confidence tier, borrowed elasticity, suggested entry price, and validation status.
- **REQ-COLD-002** (FR-CS-02): Where the queue is shown, the system shall allow filtering by division, department (dependent on division), status, and free-text search.
- **REQ-COLD-003** (FR-CS-03): The system shall show per category group its pending count, low-confidence count, average confidence, and a bulk "accept all pending in this group" action.
- **REQ-COLD-004** (FR-CS-04): The system shall order rows so pending, lowest-confidence items surface first within a group.
- **REQ-COLD-005** (FR-CS-05): When a user opens a mapping, the system shall show a plain-language summary banner narrating the recommended (or overridden) mapping, its confidence, attribute alignment, and resulting entry price and % markdown.
- **REQ-COLD-006** (FR-CS-06): The system shall show the new product alongside all ranked comp candidates with the system's default recommendation visibly marked as "System pick."
- **REQ-COLD-007** (FR-CS-07): The system shall additionally show a confidence breakdown by driver, attribute match/mismatch grid, borrowed-priors rationale, side-by-side product-data compare, and pricing-impact preview.
- **REQ-COLD-008** (FR-CS-08): If the top two candidates' confidence scores are very close, then the system shall flag an ambiguous "close call" for manual review.
- **REQ-COLD-009** (FR-CS-09): When a user searches the full catalog, the system shall allow picking an alternate comp beyond the ranked shortlist.
- **REQ-COLD-010** (FR-CS-10): When a user validates the top pick or overrides with another candidate, the system shall record the decision, accepting an optional reason on override.
- **REQ-COLD-011** (FR-CS-11): The system shall display a stamped audit line on validated/overridden mappings (who, when, and for overrides what was suggested vs chosen).
- **REQ-COLD-012** (business rule): The system shall compute confidence as a weighted blend (attribute match ~40%, PIM similarity ~20%, price-band ~15%, comp data depth ~25%), tiered ≥80 high / ≥60 medium / else low, with progressive candidate-pool fallback.

### Configuration — `REQ-CONFIG-*`

- **REQ-CONFIG-001** (FR-CF-01): The system shall present a read-only list of users showing name, role, active/inactive status, and category (brand/division) access.
- **REQ-CONFIG-002** (§14/§15.6 gap): Where brand/division access control is in scope, the system shall enforce it platform-wide. `[NEEDS CLARIFICATION: is row-level security a real requirement?]`

### Platform Shell (cross-cutting) — `REQ-SHELL-*`

- **REQ-SHELL-001** (§1): The system shall present sidebar navigation grouping screens under Pricing Team, Pricing Strategist, and Admin sections, routing to the defined routes.
- **REQ-SHELL-002** (§1): The system shall provide global `brand` (both/tcp/gymboree) and `channel` (all/digital/store/canada) filters referenced by screen headers.
- **REQ-SHELL-003** (§15.7): Where a conversational "Ask anything" surface is in scope, the system shall provide it. `[NEEDS CLARIFICATION: currently a non-functional stub; in or out of scope?]`

## Acceptance Scenarios

Gherkin for user-facing behavior. Each scenario traces to EARS requirement(s). This is a representative (not exhaustive) set; scenarios will be expanded per slice during implementation.

### Focus Builder

```gherkin
Scenario: Create a Focus Set with a live preview  # REQ-FOCUS-003, REQ-FOCUS-004
  Given I am a Pricing Team member on the Focus Builder
  When I enter a name and add a rule "Division = GIRLS"
  Then I see the first 8 matching SKUs and a total match count

Scenario: Warn when a filter matches nothing  # REQ-FOCUS-004
  Given I am building a Focus Set
  When my rules match no SKUs
  Then I see a "no matches" warning

Scenario: Search and sort the Focus Set library  # REQ-FOCUS-002
  Given the Focus Set library has multiple sets
  When I search "spring" and sort by Product Count descending
  Then only name-matching sets are shown, ordered by product count high to low

Scenario: Export a Focus Set to CSV  # REQ-FOCUS-008
  Given a saved Focus Set with matched SKUs
  When I export it
  Then I receive a CSV with SKU, Name, Division, Category, Price, Qty, Status columns
```

### Product Grid

```gherkin
Scenario: Soft-delete and restore a SKU  # REQ-GRID-004, REQ-GRID-005
  Given a Focus Set open in the Product Grid at SKU level
  When I remove a SKU and then restore it from the Deleted Items pane
  Then the SKU returns to the active grid and the row count updates

Scenario: Toggle product and SKU views  # REQ-GRID-001
  Given a Focus Set open in the Product Grid
  When I toggle from product-level to SKU-level
  Then the grid expands and shows a live SKU row count
```

### Promotions

```gherkin
Scenario: Prevent an end date before the start date  # REQ-PROMO-006
  Given I am creating a promotion
  When I set an end date earlier than the start date
  Then the system rejects it and requires an end date strictly after the start

Scenario: Filter promotions by status with counts  # REQ-PROMO-002, REQ-PROMO-008
  Given promotions in various date ranges
  When I select the "Active" tab
  Then only currently-active promotions are shown and each tab shows a live count

Scenario: View per-SKU promo price  # REQ-PROMO-005
  Given a promotion linked to a Focus Set
  When I select "View Products"
  Then I see up to 20 SKUs each with a computed promo price
```

### Discount Modeling

```gherkin
Scenario: Run Model gated on required fields  # REQ-DISC-005
  Given a new discount model with no dates set
  Then "Run Model" is disabled
  When I set name, focus group, both dates, and a valid depth
  Then "Run Model" becomes enabled

Scenario: Open an existing model's output directly  # REQ-DISC-002
  Given an approved discount model in the list
  When I click its row
  Then its Output view opens without the create form or Run step

Scenario: Discard a model regardless of status  # REQ-DISC-009
  Given an approved discount model opened via its row
  When I discard it and confirm
  Then it is permanently removed from the list
```

### Price Scenario

```gherkin
Scenario: Objective sliders always sum to 100  # REQ-SCEN-002
  Given I am creating a scenario
  When I increase the Revenue weight
  Then Gross Margin and Sell-Through redistribute proportionally so the total stays 100%

Scenario: Move the scenario position along the frontier  # REQ-SCEN-005, REQ-SCEN-006
  Given a scenario output with the optimization slider at 30%
  When I move the slider to 70%
  Then the Scenario Position moves along the frontier and the risk band label updates to "Aggressive"

Scenario: Freeze edits once submitted  # REQ-SCEN-009
  Given a scenario with status pending
  Then the optimization slider and edit controls are disabled

Scenario: Resubmit a returned scenario  # REQ-SCEN-008
  Given a scenario returned with a change request
  When I view it
  Then I see the latest reviewer comment and can resubmit
```

### Deep Dive

```gherkin
Scenario: Explain a recommended price  # REQ-DEEP-003
  Given the Price Adjustments grid in Deep Dive
  When I select "Explain" on a row
  Then a modal shows the rationale, driving objectives, decision ladder, and contextual factors

Scenario: Progressive unlock by optimization level  # REQ-DEEP-005
  Given the optimization slider at 20%
  When I raise it to 60%
  Then additional price adjustments and marketing/discount tiles become visible
```

### Guardrails

```gherkin
Scenario: Inline-edit a guardrail threshold  # REQ-GUARD-002
  Given the Guardrails table
  When I change a guardrail's value inline
  Then the new value is persisted

Scenario: Non-overridable guardrails are enforced  # REQ-GUARD-005
  Given an active non-overridable guardrail
  When a Pricing Team member creates a scenario
  Then that guardrail is enforced as a hard constraint and is not editable
```

### Approvals

```gherkin
Scenario: Deny requires a reason  # REQ-APPR-003
  Given a pending scenario in the Approvals queue
  When I choose Deny without entering a reason
  Then denial is blocked until I provide a mandatory reason

Scenario: Return a discount model for changes requires a comment  # REQ-APPR-010
  Given a pending discount model
  When I choose "Return for Changes" without a comment
  Then the return is blocked until I provide a mandatory comment

Scenario: Decided items move to a separate table  # REQ-APPR-004
  Given I approve a pending item
  Then it moves to the decided table with a read-only View
```

### Agents & Autonomy

```gherkin
Scenario: Pause and resume a monitor  # REQ-AGENT-003
  Given a running monitor card
  When I pause it and then resume it
  Then its state and signals-today counter update accordingly

Scenario: Block promotion below the reversibility ceiling  # REQ-AUTO-005, REQ-AUTO-009
  Given an action class at its reversibility ceiling
  When I attempt to promote it
  Then promotion is blocked with a clear reason

Scenario: Kill switch disables controls  # REQ-AUTO-001, REQ-AUTO-008
  Given in-flight autonomous actions
  When I engage the emergency kill switch
  Then all promote, veto, and undo controls are disabled and veto countdowns freeze
```

### Measurement

```gherkin
Scenario: Go Live gated on balance and cost acknowledgment  # REQ-MEAS-006, REQ-MEAS-005
  Given an experiment with at least one imbalanced block
  Then "Go Live" is disabled with an explanatory tooltip
  When all blocks are balanced and I acknowledge the cost of control
  Then "Go Live" becomes enabled

Scenario: Moving a cluster to control drops its ML price  # REQ-MEAS-007
  Given a treatment cluster with an ML price recommendation
  When I move it to control
  Then its ML recommendation is removed and it runs BAU pricing

Scenario: Verdict banner on crossing a boundary  # REQ-MEAS-011, REQ-MEAS-014
  Given a live experiment whose win-probability crosses the win boundary
  Then the verdict banner shows "win" with an action to scale to all matching SKUs
```

### Cold Start

```gherkin
Scenario: Validate the system's top pick  # REQ-COLD-010, REQ-COLD-011
  Given a pending mapping with a system-recommended comp
  When I validate the top pick
  Then the mapping status becomes validated and shows a stamped audit line

Scenario: Override with a different candidate  # REQ-COLD-010, REQ-COLD-011
  Given a pending mapping
  When I select a different candidate and optionally add a reason
  Then the mapping status becomes overridden and the audit line records suggested vs chosen

Scenario: Flag a close call  # REQ-COLD-008
  Given the top two candidates have very close confidence scores
  Then the mapping is flagged as an ambiguous close call
```

### Configuration

```gherkin
Scenario: View user access list  # REQ-CONFIG-001
  Given I am an Admin on the Configuration screen
  Then I see a read-only list of users with name, role, status, and brand/division access
```

## Data Taxonomy

Per `DATA-TAXONOMY.md` and constitution Principle VI. Specs name Tier-1 entities as Key Entities; screen constructs are derived projections.

### Tier-1 Entities

Already catalogued (canonical, backend-aligned):

- `FocusSetEntity`, `CatalogProductEntity`, `CatalogSkuEntity`, `FocusDiscountEntity`
- `PromoEntity` (reconcile with Promotions §3 `Promo` shape)
- `ScenarioEntity`, `DiscountModelEntity`
- `GuardrailEntity`, `ApprovalEntity`, `AppUserEntity`
- `BusinessIssueEntity`, `SubActionEntity` (cross-domain linkage)

Likely **new** Tier-1 entities to be defined during their slice specs (not yet in the taxonomy):

- A canonical **Promotion/Discount** entity reconciling the three fragmented shapes (§15.1). `[NEEDS CLARIFICATION]`
- `ColdStartMappingEntity` (§13).
- Autonomy: **action-class**, **live-action**, **audit-entry** entities (§11).
- Agents: **Monitor**, **Task-agent** records (Operators are derived) (§10).
- Measurement: **experiment**, **cluster**, **block**, **outcome** entities (§12 build spec).
- `GuardrailCheck` compliance record (§9) — Tier-1 vs Tier-2 undecided. `[NEEDS CLARIFICATION]`

### Tier-2 Screen Constructs

- **View:** `FocusSetView`, `ProductView`, `FocusProductView`, `FocusSkuView`, `DeepDiveView`, `DeepDiveProductView`, `ClusterView`, `BlockView`, `CompCandidateView`, `ImpactPreviewView`, `DeepDiveMarketingTile`, `DeepDiveDiscountTile`.
- **Row:** `RollupRow`, `CannibalizationRow`, `InventoryRiskRow`, `SkuRow`, guardrail table row.
- **Form:** `ScenarioForm`, `DiscountForm`, focus-set form, promo form, guardrail edit form.
- **Filters:** `ScenarioFilters`, focus-set library filters, product-grid filters, like-item queue filters.
- **State:** `ScenarioGraphState`, autonomy kill-switch/selection state, measurement playback state, deleted-items pane state.
- **Option:** `FocusGroupOption`, discount-type/channel/status options.

`*Entity` types must never be used directly as local UI state.

## API and Contract Needs

Backend-owned contracts (detailed API design deferred to plan/contract artifacts):

- **Focus Set CRUD** and **resolve Focus Set → SKUs** (single canonical operation shared by Focus Builder, Product Grid, Promotions, Discount Modeling, Price Scenario) — §15.2.
- Catalog attribute enumeration for the condition tree.
- Promo CRUD + per-SKU promo price computation.
- Discount model run → `DiscountModelOutput`; model persistence + status lifecycle.
- Scenario run pipeline (6 phases) → frontier + uplift + per-SKU recommendations + explainability trace; live guardrail evaluation.
- Guardrail CRUD + live compliance evaluation service.
- Approval state machine + cross-domain action-tracker linkback.
- Agent roster + lifecycle; Autonomy action-class state + promotion gates + live-action state machine + kill switch + audit.
- Measurement cluster/block/balance + sequential estimator + outcome feedback.
- Like-item matching + confidence scoring + PIM/DAM/sales-history feeds.
- User CRUD + IdP integration + brand/division authorization (if in scope).

## Cross-Cutting Requirements

- **Approvals / human decision gates:** Pricing actions must pass explicit human decision points before execution (constitution Principle VII). Deny requires a mandatory reason; a single consistent reason-required policy is needed across deny/request-changes/override (§15.5). `[NEEDS CLARIFICATION]`
- **Guardrails:** guardrail compliance should be computed live and, per screen, decide where a hard violation blocks a transition versus only warns (§15.4). Today guardrails are advisory everywhere. `[NEEDS CLARIFICATION]`
- **Auditability:** promotions, demotions, kill-switch engagements, vetoes, auto-applied actions, rollbacks, model recalibrations, approval decisions, and cold-start validations/overrides must be logged with actor and timestamp.
- **Permissions:** brand/division access is displayed but not enforced anywhere; if row-level security is real it must be enforced platform-wide (§15.6). `[NEEDS CLARIFICATION]`
- **Loading states:** multi-phase run sequences (Discount Modeling, Price Scenario) and live streaming readouts (Measurement) require explicit progress/loading affordances.
- **Empty states:** empty Focus Set library, no-match filters, empty approval queues, empty deleted-items pane, and empty measurement queues must show clear empty states.
- **Error states:** run failures, save/CRUD failures, and validation failures must be surfaced without data loss.
- **Validation behavior:** forms use required-field, date-order, weight-sum-100, and numeric-range validation (RHF + Zod per stack).
- **Accessibility:** accessible-role queries preferred in tests; approved typography/icon/color tokens; keyboard-operable controls (sliders, toggles, drawers).
- **Reporting / export:** CSV export is required in Focus Builder, Product Grid, Discount Modeling SKU detail, and Deep Dive; all projections carry explicit uncertainty framing.

## Out of Scope

- Pricing Platform V2 Agentic app (`/pricing-platform-v2/*`) beyond deciding a canonical version for Agents/Autonomy.
- The Marketing module (present in DATA-TAXONOMY but not in `PLATFORM-REQUIREMENTS.md` for this app). `[NEEDS CLARIFICATION]`
- Executive-level measurement summary (intended for the Performance Platform, not built anywhere). `[NEEDS CLARIFICATION]`
- Real ML/optimization/forecasting engines and real PIM/DAM/competitive-price/sales-history integrations — the documented output shapes are the contract target; the modeling itself is deferred and its v1 scope is an open question.
- Full identity-provider (SSO/IAM) integration and enforced row-level security, unless the access-control decision brings it into scope.
- The "Ask anything" conversational surface, unless brought into scope. `[NEEDS CLARIFICATION]`
- Automatic execution linkage (approval → executable price-change record → auto-launch Measurement), unless decided in scope.

## Open Questions

1. `[NEEDS CLARIFICATION]` Real ML vs. deterministic-placeholder-behind-contract for Discount Modeling, Price Scenarios, and Like-Item Mapping v1 (§15.3). Deterministic-placeholder-behind-contract.
2. `[NEEDS CLARIFICATION]` Canonical Promotion/Discount entity reconciling the three fragmented shapes (§15.1). Yes
3. `[NEEDS CLARIFICATION]` Focus Set resolution: live evaluation vs. persisted+labeled snapshot; minimum-match enforcement (§2/§15.2). Live evaluation.
4. `[NEEDS CLARIFICATION]` Where hard guardrail violations must block a transition versus warn, screen by screen (§15.4).
5. `[NEEDS CLARIFICATION]` One consistent reason-required policy across deny/request-changes/override (§15.5).
6. `[NEEDS CLARIFICATION]` Is brand/division access control a real, platform-wide requirement (§14/§15.6)? Yes it is a requirement.
7. `[NEEDS CLARIFICATION]` Is the "Ask anything" surface in or out of scope (§15.7)? Keep it is a stub for show.
8. `[NEEDS CLARIFICATION]` Canonical version for Agents and Autonomy vs. the more-complete V2 fork; kill-switch freeze semantics (§10/§11).
9. `[NEEDS CLARIFICATION]` Closed loops: Measurement → Autonomy accuracy; approval → executable price-change / auto-launch Measurement — automatic vs. manual and in scope now?
10. `[NEEDS CLARIFICATION]` Is Marketing (and Executive) in scope for this rebuild?
11. `[NEEDS CLARIFICATION]` Persona resolution: Pricing Team vs. Merchant (Like-Item); who owns Measurement setup?
12. `[NEEDS CLARIFICATION]` New Tier-1 entities for Autonomy, Agents, Measurement, Cold Start, and the GuardrailCheck tier.

## Suggested Implementation Slices

Ordered roughly by dependency. Each slice is independently testable, PR-sized, and cross-repo (frontend + backend) unless noted. Requirements columns list primary coverage.

| Slice | Name | Scope | Target repos | Depends on | Requirements | Expected acceptance tests |
|---|---|---|---|---|---|---|
| SLICE-01 | Platform shell & navigation | Sidebar sections, routing, global brand/channel filters | frontend | — | REQ-SHELL-001, REQ-SHELL-002 | Navigation renders; routes resolve; global filters present |
| SLICE-02 | Focus Set management | Focus Builder CRUD, condition tree, live preview, export; resolve-to-SKUs backend | both | SLICE-01 | REQ-FOCUS-001…012 | Create/preview/no-match, search/sort, export |
| SLICE-03 | Product Grid | SKU curation, rollup toggle, soft-delete/restore, export | both | SLICE-02 | REQ-GRID-001…007 | Soft-delete/restore, product/SKU toggle |
| SLICE-04 | Guardrails management | Guardrail CRUD, toggles, live evaluation | both | SLICE-01 | REQ-GUARD-001…007 | Inline edit, non-overridable enforced |
| SLICE-05 | Promotions calendar | Promo CRUD, calendar/list, detail drawer, per-SKU price | both | SLICE-02 | REQ-PROMO-001…008 | Date validation, status filter counts, view products |
| SLICE-06 | Discount Modeling | Model create/run/output/lifecycle, risk panels, export | both | SLICE-02 | REQ-DISC-001…010 | Run gating, open output, discard |
| SLICE-07 | Price Scenario optimization | Scenario create/run, frontier + slider, comparison, narrative | both | SLICE-02, SLICE-04 | REQ-SCEN-001…011 | Weight-sum-100, slider bands, freeze, resubmit |
| SLICE-08 | Deep Dive | SKU-level drill, Explain modal, progressive unlock, tiles | both | SLICE-07 | REQ-DEEP-001…005 | Explain modal, progressive unlock |
| SLICE-09 | Approvals queue | Two-tab queue, drawers, decisions, linkage, reason rules | both | SLICE-06, SLICE-07, SLICE-04 | REQ-APPR-001…012 | Deny-reason, return-comment, decided table |
| SLICE-10 | Agent roster | Monitors/operators/task agents, hire/pause/retire | both | SLICE-01 | REQ-AGENT-001…007 | Pause/resume monitor, hire agent |
| SLICE-11 | Pricing Autonomy | Trust ladder, live supervision, kill switch, audit | both | SLICE-10 | REQ-AUTO-001…010 | Promotion gating, kill-switch disables controls |
| SLICE-12 | Measurement | Setup (arms/balance/go-live gate) + live readout | both | SLICE-07 | REQ-MEAS-001…014 | Go-live gate, control drops ML price, verdict |
| SLICE-13 | Like-Item Mapping | Cold-start queue, candidate ranking, validate/override | both | SLICE-02 | REQ-COLD-001…012 | Validate, override w/ audit, close-call flag |
| SLICE-14 | Configuration | User access list (+ enforcement if in scope) | both | SLICE-01 | REQ-CONFIG-001…002 | User list renders read-only |

---

_This spec describes WHAT and WHY, not implementation HOW. It contains no code, tasks, or final API/implementation design. It is `Approved` (by Saad on 07/07/2026); `/create-platform-plan` may now proceed._
