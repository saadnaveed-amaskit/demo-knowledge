# Frontend Structure

## Source Snapshot

| Repo | Branch | Commit SHA |
|---|---|---|
| frontend | main | d234cdbd42bc66353e1e0836a4c32152bbfaa828 |

## Package Scripts

| Script | Command | Purpose |
|---|---|---|
| build | `tsc -b && vite build` | Type-check then produce production Vite build |
| check | `npm run lint && npm run lint:policy && npm run typecheck && npm run build` | Aggregate CI-style gate: lint, policy lint, typecheck, build |
| dev | `vite` | Start Vite dev server |
| lint | `eslint .` | Run ESLint over the repo |
| lint:policy | `node scripts/lint-policy.mjs` | Run the project's pinned-stack/naming policy lint script |
| preview | `vite preview --port 4173 --strictPort` | Serve the built output locally on a fixed port |
| test | `vitest run` | Run unit tests once (Vitest) |
| test:bdd | `node --import tsx ./node_modules/@cucumber/cucumber/bin/cucumber.js --config cucumber.mjs` | Run Cucumber BDD tests via the tsx loader, using `cucumber.mjs` |
| test:watch | `vitest` | Run unit tests in watch mode |
| typecheck | `tsc -b --noEmit` | Type-check the project without emitting output |

## Dependencies

| Package | Version | Purpose |
|---|---|---|
| @cucumber/cucumber | ^11.2.0 | BDD test runner (Cucumber/Gherkin) (dev) |
| @eslint/js | ^9.17.0 | ESLint's official JS rule configs (dev) |
| @hookform/resolvers | ^3.9.1 | Validation resolver adapters for react-hook-form (e.g. Zod integration) |
| @nivo/bar | ^0.99.0 | Bar chart component library |
| @nivo/core | ^0.99.0 | Core/shared library for Nivo charts |
| @nivo/line | ^0.99.0 | Line chart component library |
| @radix-ui/react-label | ^2.1.11 | Accessible label primitive (Radix UI) |
| @radix-ui/react-select | ^2.3.3 | Accessible select/dropdown primitive (Radix UI) |
| @radix-ui/react-slot | ^1.3.0 | Slot composition primitive (Radix UI) |
| @radix-ui/react-switch | ^1.3.3 | Accessible toggle-switch primitive (Radix UI) |
| @testing-library/jest-dom | ^6.6.3 | Custom Jest/Vitest DOM matchers (dev) |
| @testing-library/react | ^16.1.0 | React component testing utilities (dev) |
| @types/node | ^22.10.5 | TypeScript types for Node.js (dev) |
| @types/react | ^19.0.2 | TypeScript types for React (dev) |
| @types/react-dom | ^19.0.2 | TypeScript types for react-dom (dev) |
| @vitejs/plugin-react | ^4.3.4 | Vite plugin enabling React (JSX/Fast Refresh) support (dev) |
| ag-grid-community | ^33.0.0 | Data grid engine (community edition) |
| ag-grid-react | ^33.0.0 | React wrapper for AG Grid |
| autoprefixer | ^10.4.20 | PostCSS plugin adding vendor prefixes (dev) |
| class-variance-authority | ^0.7.1 | Utility for defining Tailwind class variants |
| clsx | ^2.1.1 | Conditional className string builder |
| date-fns | ^4.1.0 | Date manipulation/formatting library |
| eslint | ^9.17.0 | JS/TS linter (dev) |
| eslint-plugin-react-hooks | ^5.1.0 | ESLint rules for React hooks correctness (dev) |
| eslint-plugin-react-refresh | ^0.4.16 | ESLint rules validating Fast Refresh compatibility (dev) |
| framer-motion | ^12.0.0 | Animation library for React |
| globals | ^15.14.0 | Predefined global-variable identifier sets for ESLint (dev) |
| jsdom | ^25.0.1 | Browser-DOM emulation for Node-based tests (dev) |
| lucide-react | ^0.469.0 | Icon component library |
| nuqs | ^2.4.0 | URL search-param state management for React |
| playwright | ^1.49.1 | Browser automation, used to drive BDD tests (dev) |
| postcss | ^8.4.49 | CSS transformation tool used by Tailwind (dev) |
| react | ^19.0.0 | UI library (core) |
| react-dom | ^19.0.0 | React renderer for the DOM |
| react-hook-form | ^7.54.2 | Form state management library |
| react-router-dom | ^7.1.1 | Client-side routing |
| tailwind-merge | ^2.6.0 | Merge/resolve conflicting Tailwind class strings |
| tailwindcss | ^3.4.17 | Utility-first CSS framework (dev) |
| tsx | ^4.19.2 | TypeScript execution loader for Node, used by `test:bdd` (dev) |
| typescript | ^5.7.2 | TypeScript compiler (dev) |
| typescript-eslint | ^8.19.0 | TypeScript support for ESLint (dev) |
| vite | ^6.0.7 | Build tool / dev server (dev) |
| vitest | ^2.1.8 | Unit test runner, Vite-native (dev) |
| zod | ^3.24.1 | Schema validation library |
| zustand | ^5.0.2 | Lightweight global state management |

## Main Directories

| Path | Purpose | Notes |
|---|---|---|
| src/app/ | App shell, router, nav model, global filters | `AppShell.tsx`, `routes.tsx`, `nav.ts`, `GlobalFilters.tsx` |
| src/components/ | Shared/reusable UI components | Contains `ui/` subfolder |
| src/components/ui/ | Low-level reusable UI primitives | `button.tsx`, `input.tsx`, `select.tsx`, `switch.tsx` — no `Sheet`/`Dialog` component exists yet |
| src/lib/ | Shared utility helpers | `utils.ts` (className merge helper) |
| src/screens/ | One folder per feature screen | `approvals`, `discount-modeling`, `focus-builder`, `guardrails`, `price-scenarios`, `product-grid`, `promotions`, plus `ScreenPlaceholder.tsx` |
| src/state.ts | Global Zustand store | App-wide UI state (brand/channel) |

## Routes

| Route | File | Notes |
|---|---|---|
| / | Redirect | `<Navigate to="/focus" replace />` |
| /agents | src/screens/ScreenPlaceholder.tsx | Placeholder — SLICE-10 (Agent roster) not yet implemented |
| /approvals | src/screens/approvals/ApprovalsScreen.tsx | SLICE-09 |
| /autonomy | src/screens/ScreenPlaceholder.tsx | Placeholder — SLICE-11 (Pricing Autonomy) not yet implemented |
| /config | src/screens/ScreenPlaceholder.tsx | Placeholder — SLICE-14 (Configuration) not yet implemented |
| /discount-modeling | src/screens/discount-modeling/DiscountModelingScreen.tsx | SLICE-06 |
| /focus | src/screens/focus-builder/FocusBuilder.tsx | SLICE-02 |
| /guardrails | src/screens/guardrails/GuardrailsScreen.tsx | SLICE-04 |
| /like-items | src/screens/ScreenPlaceholder.tsx | Placeholder — SLICE-13 (Like-Item Mapping / Cold Start) not yet implemented |
| /measurement | src/screens/ScreenPlaceholder.tsx | Placeholder — SLICE-12 (Measurement) not yet implemented |
| /product-grid | src/screens/product-grid/ProductGrid.tsx | SLICE-03; routed but not shown in the sidebar (`UNLISTED_SCREENS`) |
| /promos | src/screens/promotions/PromotionsScreen.tsx | SLICE-05 |
| /scenario | src/screens/price-scenarios/PriceScenariosScreen.tsx | SLICE-07 (Deep Dive / SLICE-08 renders inside this screen) |

All routes render as children of the layout route `path: "/"`, whose element is `AppShell` (`src/app/AppShell.tsx`).

## Components

| Component/File | Path | Notes |
|---|---|---|
| ApprovalsScreen.tsx | src/screens/approvals/ApprovalsScreen.tsx | Approvals queue: tabs, pending rows, deny/request-changes/approve actions, decided table, scenario/discount review views |
| ConditionTree.tsx | src/screens/focus-builder/ConditionTree.tsx | Recursive filter-rule tree editor used inside Focus Builder |
| DeepDiveSection.tsx | src/screens/price-scenarios/DeepDiveSection.tsx | AG Grid-based price-adjustments explorer with row explain/export (SLICE-08) |
| DiscountModelingScreen.tsx | src/screens/discount-modeling/DiscountModelingScreen.tsx | Create/run discount models, Nivo charts, RHF/Zod form |
| FocusBuilder.tsx | src/screens/focus-builder/FocusBuilder.tsx | Build/browse/export Focus Sets |
| GuardrailsScreen.tsx | src/screens/guardrails/GuardrailsScreen.tsx | Manage pricing guardrails via RHF/Zod |
| PriceScenariosScreen.tsx | src/screens/price-scenarios/PriceScenariosScreen.tsx | Price scenario creation/output/approval-mode UI with objective sliders and Nivo line chart |
| ProductGrid.tsx | src/screens/product-grid/ProductGrid.tsx | Browse/curate SKUs within a Focus Set via AG Grid |
| PromotionsScreen.tsx | src/screens/promotions/PromotionsScreen.tsx | Manage time-boxed promotions (calendar + list), RHF/Zod, date-fns |
| ScreenPlaceholder.tsx | src/screens/ScreenPlaceholder.tsx | Generic placeholder page for unimplemented nav routes |

Each screen folder also contains non-component support files: types (e.g. `approval-types.ts`) and fetch-wrapper API modules (e.g. `approvals-api.ts`).

## State / Hooks

| Name | Path | Notes |
|---|---|---|
| useRNState | src/state.ts | Global Retail Nucleus UI state — `brand`/`channel`, mirrored from URL-synced nuqs filters |
| useFocusBuilderStore | src/screens/focus-builder/state.ts | Focus Builder's create/edit form state (open/close, editing id, form name, condition-tree being edited) |

No other `state.ts` files exist elsewhere under `src/`; other screens use local `useState` rather than a dedicated Zustand store.

## Tests

| Test file | Type | Notes |
|---|---|---|
| features/app-boots.feature + tests/bdd/steps/app-boots.steps.ts | BDD | Application boots |
| features/app-shell-navigation.feature + tests/bdd/steps/app-shell-navigation.steps.ts | BDD | Application shell and navigation |
| features/approvals-queue.feature + tests/bdd/steps/approvals-queue.steps.ts | BDD | Approvals Queue (SLICE-09) |
| features/deep-dive.feature + tests/bdd/steps/deep-dive.steps.ts | BDD | Deep Dive (SLICE-08) |
| features/discount-modeling.feature + tests/bdd/steps/discount-modeling.steps.ts | BDD | Discount Modeling (SLICE-06) |
| features/focus-set-management.feature + tests/bdd/steps/focus-set-management.steps.ts | BDD | Focus Set management (SLICE-02) |
| features/guardrails-management.feature + tests/bdd/steps/guardrails-management.steps.ts | BDD | Guardrails Management (SLICE-04) |
| features/price-scenario.feature + tests/bdd/steps/price-scenario.steps.ts | BDD | Price Scenario Optimization (SLICE-07) |
| features/product-grid.feature + tests/bdd/steps/product-grid.steps.ts | BDD | Product Grid (SLICE-03) |
| features/promotions-calendar.feature + tests/bdd/steps/promotions-calendar.steps.ts | BDD | Promotions Calendar (SLICE-05) |
| tests/bdd/steps/common.steps.ts | BDD support | Shared/common step definitions (no direct 1:1 feature file) |
| src/App.test.tsx | Unit (Vitest) | App shell rendering — "Retail Nucleus" heading and persona-grouped nav sections render |

## Known Gaps

- No `*.test.tsx`/`*.spec.tsx` unit test exists per screen — `src/App.test.tsx` is the only Vitest file; per-screen coverage relies on the BDD suite.
- `/agents`, `/autonomy`, `/config`, `/like-items`, and `/measurement` are `ScreenPlaceholder` routes pending SLICE-10 through SLICE-14.
