
# Data Taxonomy & Structure Guide

## Entity Relationship Diagram

```
shared/domain-data/taxonomy.ts  (CatalogBrand, CatalogDivision, CatalogChannel, CatalogSeason)
    │
    ├── shared/domain-data/product-catalog.ts (re-exports CatalogProductEntity, CatalogSkuEntity)
    │       └── CatalogSkuEntity  (productId FK → CatalogProductEntity.id)
    │
    ├── shared/domain-data/focus-sets.ts (FocusSetEntity)
    │       └── productIds[] ──────────────────────────────► CatalogProductEntity.id
    │
    ├── shared/domain-data/promos.ts (PromoEntity)
    │       └── brand + country (composite FK → taxonomy)
    │
    ├── [RN] focus-builder/  (FocusSetView / FocusPackageView)
    │       ├── FocusProductView  (mirrors CatalogProductEntity)
    │       ├── FocusSkuView      (productId FK → FocusProductView.id)
    │       │                     (discountId FK → FocusDiscountEntity.id)
    │       └── FocusDiscountEntity  (focus-set FK → FocusSetEntity.id)
    │
    ├── [RN] price-scenario/  (ScenarioEntity → DeepDiveView)
    │       ├── ScenarioEntity.focusId ────────────────────► FocusSetEntity.id
    │       └── DeepDiveView.scenarioId ────────────────────► ScenarioEntity.id
    │               ├── DeepDiveProductView.productId ──────► CatalogProductEntity.id
    │               └── DeepDiveMarketingTile.campaignId ───► CampaignEntity.id
    │
    ├── [RN] discount-modeling/  (DiscountModelEntity)
    │       └── DiscountModelEntity.focusGroupId ───────────► FocusSetEntity.id
    │
    ├── [RN] approvals/  (ApprovalEntity)
    │       ├── ApprovalEntity.scenarioId ──────────────────► ScenarioEntity.id
    │       └── ApprovalEntity.issueId? ────────────────────► BusinessIssueEntity.id
    │
    ├── [RN] guardrails/  (GuardrailEntity)
    │       └── brand + division (composite FK → taxonomy)
    │
    ├── [RN] config/  (AppUserEntity)
    │       └── (standalone — no FK references)
    │
    ├── [RN] roas/  (RoasPlatformEntity, RoasCampaignEntity)
    │       └── (standalone — RetailNucleus-scoped ROAS snapshots)
    │
    ├── [MKT] CampaignEntity
    │       ├── focusSetId? ───────────────────────────────► FocusSetEntity.id
    │       └── CampaignExpansionView.campaignId ───────────► CampaignEntity.id
    │
    ├── [MKT] SpendDailyEntity
    │       └── campaignId ────────────────────────────────► CampaignEntity.id
    │
    ├── [MKT] WeeklyPlanEntity
    │       └── brand + country (composite FK → taxonomy)
    │
    └── [MKT] RecommendationEntity  (→ ActionItem in action-center)
            ├── campaignId? ───────────────────────────────► CampaignEntity.id
            └── focusSetId? ───────────────────────────────► FocusSetEntity.id
```

---

## Naming Convention

All types fall into one of two tiers. The suffix makes the tier unambiguous at a glance.

### Tier 1 — Domain Data (`*Entity`)

Canonical records with an `id` field and FK relationships. These map directly to NestJS `@Entity()` classes in the production backend. Never use `Entity` types as local UI state — always derive a screen construct from them.

Every Entity interface carries a single-line JSDoc comment explaining what it represents in business terms — who creates it, what it tracks, and its key FK relationships.

**Rule**: if a type has an `id` and is referenced by FK from another type, it is an Entity.

```
FocusSetEntity         CatalogProductEntity    CatalogSkuEntity
FocusDiscountEntity    ScenarioEntity          DiscountModelEntity
GuardrailEntity        ApprovalEntity          AppUserEntity
CampaignEntity         SpendDailyEntity        WeeklyPlanEntity
RoasPlatformEntity     RoasCampaignEntity      PromoEntity
RecommendationEntity   DataSourceEntity        SourceMappingEntity
OutputFieldEntity      PullRowEntity           ScheduledRunEntity
CompletedRunEntity     ReviewSubmissionEntity  UploadRecordEntity
ContentHealthRecord    QueryClusterEntity      RefreshRecommendationEntity
RefreshOutcomeEntity   BusinessIssueEntity     SubActionEntity
```

### Tier 2 — Screen Constructs (purpose suffix)

Built at render time for display, filtering, form binding, or table rows. Not stored, not FK-referenced. Use the suffix that matches the purpose:

| Suffix | Purpose | Example |
|---|---|---|
| `*View` | Display projection — denormalized, screen-specific shape | `DeepDiveProductView`, `CampaignExpansionView`, `SpendHourlyView`, `FocusSetView` |
| `*Row` | Table row shape (AG Grid or shadcn Table) | `RollupRow`, `BudgetWeeklyRow`, `CannibalizationRow`, `InventoryRiskRow` |
| `*Form` | React Hook Form field shape (Zod schema input) | `ScenarioForm`, `DiscountForm` |
| `*Filters` | Filter bar / URL-synced filter state | `CampaignFilters`, `ScenarioFilters` |
| `*State` | Local screen or panel UI state | `ScenarioGraphState` |
| `*Option` | Dropdown / select item | `FocusGroupOption` |

### Type aliases and enumerations

Taxonomy type aliases (`CatalogBrand`, `CatalogDivision`, `CatalogChannel`, `CatalogSeason`, `MarketingBrand`, `MarketingCountry`) and view-state unions (`PSView`, `DMView`) carry no suffix — they are not objects, they are value sets.

---

## Shared Domain Data

### `src/shared/domain-data/taxonomy.ts`

Root constants and types shared across all apps.

| Const | Values |
|---|---|
| `RN_BRANDS` | `"TCP"`, `"Gymboree"` |
| `RN_DIVISIONS` | `"BIG GIRLS"`, `"BOYS"`, `"TODDLER BOYS"`, `"GIRLS"`, `"TODDLER GIRLS"`, `"BABY BOYS"`, `"BABY GIRLS"` |
| `RN_CHANNELS` | `"digital"`, `"store"`, `"canada"` |
| `RN_MASTER_SEASONS` | `"Basic"`, `"License"`, `"Holiday"`, `"Spring"`, `"Summer"`, `"Fall"`, `"BTS"` |
| `MKT_BRANDS` | `"TCP"`, `"GYM"` |
| `MKT_COUNTRIES` | `"US"`, `"CA"` |

Derived types: `CatalogBrand`, `CatalogDivision`, `CatalogChannel`, `CatalogSeason`, `MarketingBrand`, `MarketingCountry`.

---

### `src/shared/domain-data/focus-sets.ts` — `FocusSetEntity`

The canonical focus set record referenced by RN and Marketing modules.

```ts
/** A named product focus group used by the Pricing Team to scope scenarios and discount models. Stores the filter tree that resolves to a set of product IDs. Referenced by ScenarioEntity, DiscountModelEntity, CampaignEntity, and PromoEntity via focusSetId/focusId/focusGroupId FKs. */
interface FocusSetEntity {
  readonly id: string
  readonly name: string
  readonly createdAt: string
  readonly query: string                 // human-readable query label
  readonly filter: RawFocusFilter        // nested condition tree
  readonly productCount: number
  readonly productIds: readonly string[] // FK → CatalogProductEntity.id
}
```

Exported constants: `SHARED_FOCUS_SETS: FocusSetEntity[]`, `FOCUS_SET_BY_ID: Record<string, FocusSetEntity>`.

---

### `src/shared/domain-data/promos.ts` — `PromoEntity`

Recurring marketing promotional events shared across the Marketing Pacing screen and any future calendar surface.

```ts
/** A single occurrence of a recurring promotional event in a given calendar year. */
interface PromoRange {
  year: number
  from: string    // ISO 8601 inclusive
  to: string      // ISO 8601 inclusive
  yearLabel: string
}

/** A recurring marketing promotional event tracked across multiple years for pacing comparison and promo-aligned analysis. Anchors the Pacing Cockpit's "promo-aligned" comparison series so the current week's performance can be benchmarked against the same event in prior years. */
interface PromoEntity {
  id: string
  name: string
  brand: "TCP" | "GYM"
  country: "US" | "CA"
  ranges: PromoRange[]
}
```

Exported constants: `SHARED_PROMOS: PromoEntity[]`, `PROMO_BY_ID: Record<string, PromoEntity>`.

---

### `src/shared/domain-data/product-catalog.ts` — Product Catalog

Re-exports `CatalogProductEntity` / `CatalogSkuEntity` from the generated catalog. Types are defined in `src/retail-nucleus/screens/focus-builder/focus-data/generated-focus-catalog/index.ts`.

```ts
/** A product in the RetailNucleus product catalog. Represents a style or line (not a specific size/color), grouped by brand, season, division, and category. SKUs are resolved via skuIds. Referenced by FocusSetEntity (via productIds[]) and DeepDiveProductView (via productId). */
interface CatalogProductEntity {
  id: string
  name: string
  brand: string
  masterSeason: string
  seasonCode: string
  division: string
  department: string
  category: string
  subClass: string
  bigIdea: string
  skuIds: readonly string[]             // FK → CatalogSkuEntity.id
}

/** A specific sellable unit (color/size combination) of a catalog product. Tracks pricing (MSRP, cost, current price), inventory position (qty on hand, on order), clearance flag, and on-hand status. FK → CatalogProductEntity via productId. */
interface CatalogSkuEntity {
  id: string
  productId: string                     // FK → CatalogProductEntity.id
  color: string
  clearance: boolean
  msrp?: number
  cost?: number
  currentPrice?: number
  qty: number
  onHandStatus: string
  onOrderQty: number
}
```

Exported constants: `CATALOG_PRODUCTS: CatalogProductEntity[]`, `CATALOG_SKUS: CatalogSkuEntity[]`, `PRODUCT_BY_ID`, `SKU_BY_ID`.

---

## Retail Nucleus (RN)

### Store — `src/retail-nucleus/state.ts`

```ts
type RNBrand   = "both" | "tcp" | "gymboree"
type RNChannel = "all" | "digital" | "store" | "canada"

interface ScenarioCtx {
  division: string
  issue: string
  recommendation: string
  suggestedAction?: string
}
```

`useRNState` holds: `brand`, `channel`, `scenarioCtx`, `scenarioGuardrails: GuardrailEntity[] | null`, `scenarios: ScenarioEntity[]`, `discountModels: DiscountModelEntity[]`, `selectedDiscountModel: DiscountModelEntity | null`, `pendingDiscountModel: DiscountModelEntity | null`, `savedFocusSets: FocusSetView[]`.

---

### Focus Builder — `src/retail-nucleus/screens/focus-builder/`

#### `focus-types.ts`

```ts
interface ConditionRule {
  type: "rule"
  attr: string
  val: string
}

interface ConditionGroup {
  type: "group"
  logic: "AND" | "OR"
  rules: Array<ConditionRule | ConditionGroup>
}

interface FocusSetView {
  id: string
  name: string
  query?: string
  filter: ConditionGroup                // UI filter tree — use RawFocusFilter for persistence
  productCount: number
  createdAt: string
}

interface ProductView {
  sku: string
  name: string
  brand: string
  masterSeason: string
  seasonCode: string
  division: string
  department: string
  category: string
  subClass: string
  bigIdea: string
  price: number
  qty: number
  onOrderQty: number
  onHandStatus: string
}
```

Note: `FocusSetView` is the store's working representation of a focus set. It uses `ConditionGroup` (UI tree) for its `filter` field and omits `productIds`. The canonical persisted form is `FocusSetEntity` in `shared/domain-data/focus-sets.ts`.

#### `focus-data/focus-package-types.ts`

```ts
interface FocusPackageHeaderView {
  id: string
  name: string
  createdAt: string
  query: string
  filter: ConditionGroup
}

interface FocusProductView {
  id: string                            // mirrors CatalogProductEntity.id
  name: string
  brand: string
  masterSeason: string
  seasonCode: string
  division: string
  department: string
  category: string
  subClass: string
  bigIdea: string
  skuIds: string[]
}

interface FocusSkuView {
  id: string
  productId: string                     // FK → FocusProductView.id
  discountId?: string                   // FK → FocusDiscountEntity.id
  color: string
  clearance: boolean
  msrp?: number
  cost?: number
  currentPrice?: number
  qty: number
  onHandStatus: string
  onOrderQty: number
}

interface FocusPackageView {
  focus: FocusPackageHeaderView
  products: FocusProductView[]
  skus: FocusSkuView[]
}
```

#### `focus-data/focus-discounts.ts`

```ts
/** A time-boxed discount rule attached to a specific focus set. Defines the discount type (percentage or flat amount), depth, and date range during which it applies to SKUs in the linked focus group. FK → FocusSetEntity via "focus-set". */
type FocusDiscountType = "percentage" | "amount"

interface FocusDiscountEntity {
  id: string
  "focus-set": string                   // FK → FocusSetEntity.id
  name: string
  startDate: string
  endDate: string
  discountType: FocusDiscountType
  discountValue: number
}
```

Exported constants: `FOCUS_DISCOUNTS: FocusDiscountEntity[]`, `FOCUS_DISCOUNT_BY_ID`.

---

### Price Scenario — `src/retail-nucleus/screens/price-scenario/`

#### `scenario-types.ts`

```ts
type PSView = "list" | "create" | "running" | "output" | "deepdive"

interface ScenarioUplift {
  maxRevM: number; maxRevPct: number; maxMarginPp: number
  fpMixBase: number; fpMixRange: number
  maxSellPp: number; maxInvRiskM: number
}

interface ScenarioChart {
  curRev: number; mlRev: number
  xMin: number; xMax: number; yMin: number; yMax: number
  revPeak: number; profitPeak: number; curvature: number
}

interface ScenarioGraphState {
  sliderPct: number
}

/** A saved pricing optimization scenario created by the Pricing Team. Captures the target focus group, objective, time horizon, ML-generated output, approval change requests, and submitted form inputs. FK → FocusSetEntity via focusId. */
interface ScenarioEntity {
  id: string
  name: string
  focusId: string                       // FK → FocusSetEntity.id
  status: string
  created: string
  horizon: string
  obj: string
  chart: ScenarioChart
  uplift: ScenarioUplift
  synopsis?: string
  mlSummary?: MLSummaryItem[]
  graphState?: ScenarioGraphState
  submittedInputs?: ScenarioForm
  changeRequests?: ChangeRequest[]
}

interface Weights {
  revenue: number; margin: number; sellthrough: number
}

interface Guardrails {
  maxMarkdown: number; minMargin: number
}

interface ScenarioGuardrailSnapshot {
  id: number
  brand: string; division: string
  rule: string; op: string
  value: string; unit: string
  active: boolean; isOverridable: boolean
}

interface ScenarioForm {
  startDate: string; endDate: string
  objective: string
  weights: Weights
  guardrailSettings: Guardrails
  selectedGuardrails: ScenarioGuardrailSnapshot[]
}

interface ChangeRequest {
  comment: string
  date: string
}
```

#### `scenario-data/scenario-package-types.ts`

```ts
interface DeepDiveProductView {
  productId: string                     // FK → CatalogProductEntity.id
  name: string
  typeAbbr: string; typeColor: string
  cost: number; msrp: number
  currentPrice: number; recommendedPrice: number
  why: string
  discountName: string | null
  priceChangePct: number; elasticity: number
  qty: number; returnRate: number
  revenue: number; profit: number; profitMargin: number
  salesBeforeReturns: number; projectedSales: number
  theme: string
  campaign: string | null; discCampaign: string | null
}

interface DeepDiveMarketingTile {
  id: string
  campaignId?: string                   // FK → CampaignEntity.id
  color: string; handle: string
  description: string
  channels: string[]
  budget: string; duration: string; reach: string
  projectedROAS: string
  productIds: string[]
}

interface DeepDiveDiscountTile {
  id: string
  color: string; handle: string
  description: string; type: string
  savingsAvg: string; duration: string
  productIds: string[]
}

interface DeepDiveView {
  scenarioId: string                    // FK → ScenarioEntity.id
  scenarioName: string
  focus: string; horizon: string
  products: DeepDiveProductView[]
  marketingTiles: DeepDiveMarketingTile[]
  discountTiles: DeepDiveDiscountTile[]
}

interface ScenarioPackage {
  scenario: ScenarioEntity
  deepDiveData: DeepDiveView
}
```

---

### Discount Modeling — `src/retail-nucleus/screens/discount-modeling/`

#### `discount-types.ts`

```ts
type DMView          = "list" | "create" | "running" | "output"
type DiscountFormat  = "pct" | "dollar" | "bogo" | "fixed"
type DiscountChannel = "digital" | "stores" | "both"
type RollupDimension = "category" | "brand" | "season" | "color" | "lifecycle"

/** A saved discount modeling run created by the Pricing Team. Captures the focus group, discount format, depth, channel, approval status, and the full ML output (metrics, rollup tables, risk panels, forecast charts). FK → FocusSetEntity via focusGroupId. */
interface DiscountModelEntity {
  id: string
  name: string
  status: "new" | "draft" | "pending" | "approved" | "returned" | "denied"
  calendarColor?: string
  created: string; horizon: string
  focusGroup: string
  focusGroupId: string                  // FK → FocusSetEntity.id
  discountFormat: DiscountFormat
  discountDepth: number
  channel: DiscountChannel
  skuCount: number
  output: DiscountModelOutput
  submittedInputs?: DiscountForm
}

interface DiscountForm {
  modelName: string; focusGroup: string
  startDate: string; endDate: string
  discountFormat: DiscountFormat
  discountDepth: number
  channel: DiscountChannel
}

interface DiscountModelOutput {
  narrative: string; marketingHandle: string
  metrics: DiscountMetrics
  forecastCharts: DiscountForecastCharts
  rollup: Record<RollupDimension, RollupRow[]>
  riskPanels: DiscountRiskPanels
}

interface DiscountMetrics {
  revenue: MetricDelta; grossMargin: MetricDelta; netMargin: MetricDelta
  unitsSold: { baseline: number; scenario: number; liftPct: number }
  sellThrough: { baseline: number; scenario: number; pct: number }
}

interface RollupRow {
  group: string; skuCount: number
  baselineRevenue: number; scenarioRevenue: number
  deltaRevenue: number; deltaPct: number
  sellThrough: number; stockOutRisk: boolean
}

interface DiscountRiskPanels {
  cannibalization: CannibalizationRow[]
  inventoryRisk: InventoryRiskRow[]
  returnsCost: ReturnsCostSummary
  constraintWarnings: ConstraintWarning[]
  competitiveFlags: CompetitiveFlag[]
  skuDetail: SkuDetail[]
}
```

Forecast charts shape:

```ts
interface DiscountForecastCharts {
  cumulativeRevenue: CumulativeRevenueChartData  // baseline[], scenario[], upper[], lower[], xLabels
  grossMarginWalk: GrossMarginWalkChartData       // bars: GrossMarginWalkBar[]
  inventoryBurndown: InventoryBurndownChartData   // baseline[], scenario[], stockOutIdx, xLabels[]
}
```

---

### Approvals — `src/retail-nucleus/screens/approvals/approvals-data.ts`

```ts
/** An approval record for a submitted pricing scenario or discount model. Stores the submitter identity, business context (brand/division), projected impact, risk rating, and approval status. Created when a Pricing Team member submits a scenario for Pricing Strategist review. FK → ScenarioEntity via scenarioId. */
interface ApprovalEntity {
  id: string
  scenarioId: string                    // FK → ScenarioEntity.id
  submitter: string
  team: string
  brand: string
  division: string
  impact: string                        // human-readable projected revenue impact (e.g. "+$8.4M")
  risk: "Low" | "Medium" | "High"
  status: "pending" | "approved" | "denied" | "changes_requested"
  issueId?: string                      // FK → BusinessIssueEntity.id
  actionId?: string
}
```

Exported constants: `APPROVALS_SEED: ApprovalEntity[]`, `APPROVAL_BY_SCENARIO_ID: Record<string, ApprovalEntity>`.

---

### Guardrails — `src/retail-nucleus/screens/guardrails/`

```ts
/** A pricing constraint rule managed by the Pricing Strategist. Defines a threshold (e.g. gross margin ≥ 38%) for a specific brand/division combination. Guardrails are applied when running price scenarios and can be overridden by the Pricing Team if isOverridable is true. FK → CatalogBrand + CatalogDivision (composite). */
interface GuardrailEntity {
  id: number
  brand: string                         // FK → CatalogBrand (composite with division)
  division: string                      // FK → CatalogDivision
  rule: string
  op: string
  value: string
  unit: string
  active: boolean
  isOverridable: boolean
}
```

---

### Config — `src/retail-nucleus/screens/config/config-data.ts`

```ts
/** A RetailNucleus application user with a defined role and category-level access scope. Managed by admins in the Configuration screen. In production this maps to an identity provider record. */
interface AppUserEntity {
  id: string
  name: string
  role: string
  access: string[]
  status: "active" | "inactive"
}
```

Exported constants: `APP_USERS: AppUserEntity[]`.

---

### ROAS — `src/retail-nucleus/screens/roas/roas-data.ts`

```ts
/** A marketing platform (Meta, Google, TikTok, Pinterest) summarized at the RetailNucleus ROAS overview level. Captures aggregate spend, blended ROAS, week-over-week trend, impressions, clicks, and a health status used to triage which platforms need attention. */
interface RoasPlatformEntity {
  id: string
  name: string
  spend: number                         // in $M
  roas: number
  trend: string                         // human-readable delta (e.g. "+0.3x")
  impressions: string                   // human-readable (e.g. "8.4M")
  clicks: string                        // human-readable (e.g. "284K")
  status: "healthy" | "watch" | "star" | "underperforming"
}

/** A campaign-level ROAS record surfaced in the RetailNucleus Marketing ROAS screen. Tracks budget attainment, current ROAS, and whether the campaign is flagged for review. Distinct from the full CampaignEntity in the Marketing module — this is a RetailNucleus-scoped performance snapshot. */
interface RoasCampaignEntity {
  id: string
  name: string
  platform: string
  budget: number
  spent: number
  roas: number
  status: "active" | "completed" | "paused"
  flagged: boolean
}
```

Exported constants: `ROAS_PLATFORMS: RoasPlatformEntity[]`, `ROAS_CAMPAIGNS: RoasCampaignEntity[]`, `ROAS_PLATFORM_BY_ID`, `ROAS_CAMPAIGN_BY_ID`.

---

## Marketing Module

### Store — `src/marketing/state.ts`

```ts
interface MarketingState {
  selectedBrand: string    // default "TCP"
  selectedCountry: string  // default "US"
  setFilters: (brand: string, country: string) => void
}
```

`useMarketingState` holds: `selectedBrand: string`, `selectedCountry: string`.

### `src/marketing/data/marketing-types.ts`

```ts
/** An ad campaign sourced from Google, Meta, or Bing. Tracks the daily budget, target ROAS, brand/country scope, performance state, and optional focus set linkage. Serves as the core entity for the Marketing module — spend, recommendations, and expansions all FK back to this. FK → FocusSetEntity via focusSetId (optional). */
interface CampaignEntity {
  id: string
  platform: "GOOGLE" | "META" | "BING"
  rawName: string; displayName: string
  brand: "TCP" | "GYM"
  country: "US" | "CA"
  channel: "SEARCH" | "SOCIAL" | "RETAIL"
  status: "ACTIVE" | "PAUSED" | "ENDED"
  dailyBudget: number; targetRoas: number
  sourceTargetRoas?: number | null
  derivedTargetRoas?: number | null
  parseConfidence?: "high" | "medium" | "low"
  countrySource?: "parsed" | "brand-fallback" | "default-us"
  performanceState?: "OFF_PACE" | "BEHIND" | "ON_PACE" | "PAUSED"
  focusSetId?: string                   // FK → FocusSetEntity.id
}

/** A daily spend and revenue snapshot for a single campaign. Used to compute ROAS trends, pacing curves, and budget attainment in the Marketing Cockpit and Pacing screens. FK → CampaignEntity via campaignId. */
interface SpendDailyEntity {
  date: string
  campaignId: string                    // FK → CampaignEntity.id
  spend: number; revenue: number; roas: number
}

interface SpendHourlyView {
  hour: number
  spend: number; revenue: number
  planSpend: number; planRevenue: number
  lyRevenue: number; avg7dRevenue: number
}

interface SpendHourlyByDateView extends SpendHourlyView {
  date: string
}

interface BudgetWeeklyRow {
  brand: "TCP" | "GYM"
  country: "US" | "CA"
  weekStartDate: string
  plannedSpend: number; forecastRevenue: number
  targetRoas: number; forecastRoas: number; marginRoas: number
}
```

### `src/marketing/data/planning-data.ts`

```ts
/** A weekly budget plan record for a single brand × country scope. Captures the 5-week spending distribution (w1–w5), the aggregate planned spend, forecast revenue, and ROAS targets. Created by the Marketing team in the Plan & Forecast screen and used to track pacing against plan. */
interface WeeklyPlanEntity extends BudgetWeeklyRow {
  id: string
  label: string
  w1: number; w2: number; w3: number; w4: number; w5: number
  total: number
}
```

Exported constants: `WEEKLY_PLANS: WeeklyPlanEntity[]`.

### `src/marketing/data/campaign-expansions/campaign-expansion-types.ts`

```ts
interface CampaignExpansionView {
  campaignId: string                    // FK → CampaignEntity.id
  header: CampaignHeaderMetrics
  trend: CampaignTrendPoint[]
  sensitivity: CampaignSensitivityConfig
  recommendation: CampaignRecommendationContent
}

interface CampaignHeaderMetrics {
  spend: number; budget: number
  spendUsedPct: number; dayElapsedPct: number
  currentRoas: number; targetRoas: number; roasGapPct: number
  recommendationChip: string
}

interface CampaignTrendPoint {
  dateLabel: string; roas: number; targetRoas: number
}

interface CampaignSensitivityConfig {
  initialTroas: number
  minDeltaPct: number; maxDeltaPct: number; stepPct: number
  downRevDeltaPerPctK: number; upRevDeltaPerPctK: number
}

interface CampaignRecommendationContent {
  headlineTemplate: string; rationale: string; ctaLabel: string
}
```

---

## Content Enrichment (CE)

### `src/content-enrichment/shared/enrichment-types.ts`

```ts
type Brand          = "TCP" | "Gymboree" | "Sugar & Jade"
type DataSourceType = "PIM" | "BOM/Tech Pack" | "Info Boards" | "Vendor Cloud" | "DAM"
type RowStatus      = "pending" | "approved" | "edited" | "held" | "feedback"
type UserRole       = "Contributor" | "Reviewer"

/** An external data feed connected to the Content Enrichment pipeline. Tracks the source type, connection status, last sync timestamp, and record count. Configured by admins in the Data Sources configuration tab. */
interface DataSourceEntity {
  id: string; type: DataSourceType; name: string
  description: string; format: string
  status: "connected" | "pending" | "error"
  lastSync: string; recordCount?: number
}

/** A mapping rule that instructs the AI pipeline how to extract a specific copy field from a data source for a given brand/division/department/category combination. */
interface SourceMappingEntity {
  id: string; source: DataSourceType
  brand: string; division: string; department: string; category: string
  targetField: string; instructions: string
}

/** A configurable output field that maps a Claude-generated copy field to its corresponding PIM field. Controls which AI-generated fields are enabled, which channel they apply to, and whether they are custom or standard. */
interface OutputFieldEntity {
  id: string; claudeField: string; pimField: string
  channel: string; enabled: boolean; isCustom: boolean
}

/** A product SKU pulled into the Content Enrichment pipeline for AI-assisted copy generation. Tracks raw PIM/BOM source data alongside generated fields (null until enriched) and the current review status. */
interface PullRowEntity {
  id: string; sku: string; productName: string
  brand: Brand; floorset: string
  division: string; department: string; subclass: string
  pimStatus: string; pimData: string
  bomExtract: string; infoBoardNote: string
  vendorNote: string; damImageUrl: string
  // Generated fields (null until enriched):
  productTitle: string | null
  shortDescription: string | null; longDescription: string | null
  sizeAndFit: string | null; fabricAndCare: string | null
  seoMetaTitle: string | null; amazonCopy: string | null
  rowStatus: RowStatus; confidence: number | null
}

/** A configured recurring pipeline run that pulls and enriches a set of SKUs on a schedule. */
interface ScheduledRunEntity {
  id: string; name: string; floorset: string
  brand: string; division: string; department: string
  frequency: string; nextRun: string
  status: "active" | "paused"; createdBy: string
}

/** A historical record of a completed pipeline run. Tracks how many SKUs were processed, approved, held, or flagged for feedback, and whether the run succeeded. */
interface CompletedRunEntity {
  id: string; configName: string; ranAt: string
  skusProcessed: number; approved: number; held: number; feedback: number
  duration: string; status: "success" | "partial" | "failed"
}

/** A batch of enriched SKU rows submitted to the Review Queue by a Contributor. Groups PullRowEntity rows with submission metadata and tracks whether the batch has been uploaded to PIM. */
interface ReviewSubmissionEntity {
  id: string; submittedBy: string; submittedAt: string
  skuCount: number; floorset: string; brand: string
  status: "pending" | "uploaded"
  rows: PullRowEntity[]
}

/** A finalized upload record representing a completed enrichment batch that was exported to PIM. Captures the full audit trail — who uploaded, source, how many SKUs were approved/held/flagged, and scope filters used. */
interface UploadRecordEntity {
  id: string; configName: string; ranAt: string
  uploadedBy: string; source: "scheduled" | "ad-hoc"
  skusProcessed: number; approved: number; held: number; feedback: number
  duration: string; status: "success" | "partial" | "failed"
  rows: PullRowEntity[]
  filters: { brand: string; floorset: string; division?: string; department?: string }
}

interface GuidelineVersionView { version: string; author: string; savedAt: string; content: string; }
```

### Store — `src/content-enrichment/state.ts`

`useEnrichmentState` holds: `openRunRecord: UploadRecordEntity | null`, `openRunMode: "snapshot" | null`, `uploadHistory: UploadRecordEntity[]`, `pendingCount: number`, `pipelineActive: boolean`, `role: UserRole`, `submissions: ReviewSubmissionEntity[]`, `openSubmission: ReviewSubmissionEntity | null`.

### `src/content-enrichment/features/content-intelligence/content-intelligence-types.ts`

```ts
/** A health snapshot for a single product page. Tracks SEO content scores over time and the severity of any detected content debt. */
interface ContentHealthRecord {
  id: string; pageUrl: string; productName: string
  productId?: string; brand: Brand; category: string
  currentScore: number; score30dAgo: number; score90dAgo: number
  lastRefreshed: string; contentVersion: number
  debtSeverity: "critical" | "moderate" | "low" | null
}

/** A cluster of semantically related search queries tracked for content coverage analysis. Groups member queries around a canonical phrase and lists the product pages affected by coverage gaps. */
interface QueryClusterEntity {
  id: string; canonicalPhrase: string
  memberQueries: { query: string; dailyVolume: number }[]
  locale: "en-US" | "en-CA" | "fr-CA"
  dailyVolume: number; weekOverWeekGrowth: number
  coverageStatus: "covered" | "partial" | "gap"
  coverageScore: number
  affectedPages: { pageUrl: string; productName: string; coverageScore: number }[]
  significance: "high" | "medium" | "low"
  trendPoints: QueryTrendPoint[]
}

/** An AI-generated recommendation to refresh the copy on a specific product page. Describes the issue, proposed field-level changes, projected impact, and confidence band. */
interface RefreshRecommendationEntity {
  id: string; rank: number
  type: "content-debt" | "search-opportunity"
  brand: Brand; productId?: string; productName: string
  pageUrl: string; issueTitle: string; issueSummary: string
  proposedChanges: ProposedChange[]
  projectedRevenueImpact: number; projectedTrafficImpact: number
  confidence: "high" | "medium" | "low"
  status: "pending" | "in-review" | "approved" | "rejected" | "published"
  createdAt: string; queryClusterId?: string
  debtScoreDrop?: number; issueId?: string
}

/** A post-publish measurement of a content refresh recommendation's actual impact. Compares projected vs. actual traffic and revenue uplift. FK → RefreshRecommendationEntity via recommendationId. */
interface RefreshOutcomeEntity {
  id: string; recommendationId: string
  productName: string; brand: Brand; publishedAt: string
  measurementWindowDays: number
  projectedTrafficImpact: number; actualTrafficImpact: number
  projectedRevenueImpact: number; actualRevenueImpact: number
  outcomeStatus: "improved" | "flat" | "declined"
  queryClusterCoverage?: "closed" | "partial" | "unchanged"
}
```

---

## Central App

### `src/central/features/central-types.ts`

```ts
/** A single coordinated action within a cross-domain business issue. Belongs to one domain, carries a deep link to the source app screen, and tracks approval/execution status. Ordered by sequence within its parent issue. */
interface SubActionEntity {
  id: string
  domain: "pricing" | "marketing" | "content" | "inventory" | "merch"
  title: string; team: string
  status: "pending" | "approved" | "in-progress" | "applied" | "blocked"
  sourceApp: "retail-nucleus" | "content-enrichment" | "marketing"
  sourceRoute: string
  sequence?: number
  delayAfter?: string
  dependencyNote?: string
  projectedRevenueImpact?: number
  projectedMarginCost?: number
  assignee?: string
}

/** A cross-domain business issue detected at the GM/director level. Groups a set of coordinated sub-actions across pricing, marketing, content, and inventory that together resolve the issue. Severity and revenue-at-risk drive prioritization in the Central Command Center. */
interface BusinessIssueEntity {
  id: string
  businessUnit: "tcp-us" | "tcp-ca" | "gym-us" | "gym-ca"
  title: string; driver: string
  revenueAtRisk: number; isOpportunity: boolean
  severity: "critical" | "elevated" | "watch"
  detectedAt: string
  subActions: SubActionEntity[]
}
```

---

## Design Tokens — `src/shared/theme.ts`

```ts
interface CTokens {
  navy: string; navyLight: string
  paper: string; cream: string; rule: string; borderStrong: string
  warmMid: string; slate: string; slateLight: string
  indigo: string; indigoLight: string; primaryHover: string; primaryMid: string
  green: string; greenLight: string; greenBorder: string
  amber: string; amberLight: string; amberBorder: string
  canary: string; canaryLight: string; canaryBg: string; canaryBorder: string; canaryDark: string
  red: string; redLight: string; redBorder: string
  purple: string; purpleLight: string
  info: string; infoBg: string; infoBorder: string
  overlay: string
}
```

Exported singletons: `C_LIGHT`, `C_DARK`, `SIDEBAR_C`, `NIVO_PALETTE`, `RN_TYPOGRAPHY`.

---

## FK Reference Summary

| From | Field | To |
|---|---|---|
| `CatalogProductEntity` | `skuIds[]` | `CatalogSkuEntity.id` |
| `CatalogSkuEntity` | `productId` | `CatalogProductEntity.id` |
| `FocusSetEntity` | `productIds[]` | `CatalogProductEntity.id` |
| `FocusSkuView` | `productId` | `FocusProductView.id` |
| `FocusSkuView` | `discountId?` | `FocusDiscountEntity.id` |
| `FocusDiscountEntity` | `"focus-set"` | `FocusSetEntity.id` |
| `ScenarioEntity` | `focusId` | `FocusSetEntity.id` |
| `ApprovalEntity` | `scenarioId` | `ScenarioEntity.id` |
| `ApprovalEntity` | `issueId?` | `BusinessIssueEntity.id` |
| `DeepDiveView` | `scenarioId` | `ScenarioEntity.id` |
| `DeepDiveProductView` | `productId` | `CatalogProductEntity.id` |
| `DeepDiveMarketingTile` | `campaignId?` | `CampaignEntity.id` |
| `DiscountModelEntity` | `focusGroupId` | `FocusSetEntity.id` |
| `GuardrailEntity` | `brand` + `division` | `CatalogBrand` + `CatalogDivision` |
| `CampaignEntity` | `focusSetId?` | `FocusSetEntity.id` |
| `CampaignExpansionView` | `campaignId` | `CampaignEntity.id` |
| `SpendDailyEntity` | `campaignId` | `CampaignEntity.id` |
| `WeeklyPlanEntity` | `brand` + `country` | `MarketingBrand` + `MarketingCountry` |
| `PromoEntity` | `brand` + `country` | `MarketingBrand` + `MarketingCountry` |
| `RefreshRecommendationEntity` | `issueId?` | `BusinessIssueEntity.id` |
| `RefreshOutcomeEntity` | `recommendationId` | `RefreshRecommendationEntity.id` |
