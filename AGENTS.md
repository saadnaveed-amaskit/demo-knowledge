# AGENTS.md

## Purpose

This file defines the canonical filename policy for this repository and the technology policy.
All AI agents and copilots should follow this policy when creating, renaming, or editing files.

## Technology Stack — Hard Rules

These rules are non-negotiable. Never deviate without an explicit instruction to do so in the current message.

### UI Components — always use shadcn/ui

Never write a custom implementation of any component that shadcn/ui provides. Add missing components via `npx shadcn@latest add <component>` — they land in `src/components/ui/`.

| Never write a custom… | Use shadcn instead |
|---|---|
| Button, icon button, loading button | `Button` |
| Text input, textarea | `Input`, `Textarea` |
| Modal, dialog, confirmation prompt | `Dialog` |
| Dropdown menu, context menu | `DropdownMenu` |
| Select / combobox | `Select` |
| Tooltip | `Tooltip` |
| Badge, pill, tag | `Badge` |
| Card / panel | `Card` |
| Tabs | `Tabs` |
| Non-AG Grid table | `Table` |
| Toast / notification | `Sonner` |
| Checkbox, radio, switch | `Checkbox`, `RadioGroup`, `Switch` |
| Popover | `Popover` |
| Accordion | `Accordion` |
| Separator / divider | `Separator` |

### Charts — always use Nivo

Never add Recharts, Chart.js, Victory, Highcharts, or any other charting library. Use `@nivo/bar`, `@nivo/line`, `@nivo/pie`, etc.

### State — always use Zustand + nuqs

Never add Redux, Jotai, Recoil, React Context-based global state, or any other state library. Zustand for UI state; nuqs for URL-synced filters.

### Routing — always use React Router v7

Never add TanStack Router, Next.js routing, or any other router.

### Forms — always use React Hook Form + Zod

Never add Formik, Final Form, or standalone validation libraries.

### Animations — always use Framer Motion

Never add GSAP, React Spring, or other animation libraries.

### Dates — always use date-fns

Never add moment.js, dayjs, luxon, or other date libraries.

---

## Source of Truth

This document is the naming and technology source of truth.
If other instruction files mention naming or stack choices, they should align with this file.
`TECHNOLOGY.md` provides additional context on the stack but this file's rules take precedence for agents.

## Concrete Naming Policy

1. UI-rendering files must use PascalCase.
2. Non-UI module files must use kebab-case.
3. Directory names must use kebab-case.
4. Keep framework entrypoint exceptions as-is when appropriate.

### UI-rendering files (PascalCase)

Use PascalCase for files that export or primarily define UI elements.

Examples:
- Screens
- Views
- UI components
- Modals, drawers, charts, animations used as UI components

Valid examples:
- BusinessObjectives.tsx
- PriceScenario.tsx
- ScenarioCreateView.tsx
- MLAnimation.tsx

### Non-UI module files (kebab-case)

Use kebab-case for domain or utility modules.

Examples:
- constants
- types
- store
- hooks
- helpers
- mappers
- data modules
- validators

Prefer descriptive names over generic names.

Valid examples:
- scenario-types.ts
- scenario-constants.ts
- pricing-store.ts
- roas-helpers.ts

Avoid vague names when a module is shared or domain-specific.

Avoid:
- types.ts
- constants.ts

Prefer:
- <domain>-types.ts
- <domain>-constants.ts

## Exceptions

These exceptions are allowed:
- index.ts
- index.tsx

Use index files only when they serve as clear module entrypoints.
Do not rename existing stable entrypoints unless required by architecture changes.

## Required Behavior for AI Agents

When making edits:
1. Follow this naming policy for all new files.
2. If you touch a file that violates policy and rename is in scope, perform a safe rename.
3. Update all imports in the same change set.
4. Validate by searching for stale paths and running build checks when feasible.
5. Do not mix unrelated renames into feature changes.

## Cross-App Shared Code Policy (RetailNucleus + Content Enrichment)

Use this policy when creating or moving files intended for both apps.

### Goal

Prevent duplicate logic and divergent UI behavior while avoiding premature over-sharing.

### Placement Rules

1. Keep code app-local by default:
- `src/retail-nucleus/**` for RN-specific behavior, domain models, and UI flows.
- `src/content-enrichment/**` for CE-specific behavior, domain models, and UI flows.

2. Promote to `src/shared/**` only when all are true:
- Used by both apps now, or required by an approved near-term roadmap.
- API can remain domain-neutral (no CE-only or RN-only terminology in public types/props).
- No dependency on app-local store, routes, query params, or screen state.
- Extraction reduces duplication or drift (not just "might be useful later").

3. Never place app-specific mock data, business rules, or persona workflows in `src/shared/**`.

### Shared Naming and Structure

1. Apply the canonical naming policy from "Concrete Naming Policy" and "Exceptions" to all files under `src/shared/**`.

2. Prefer subfolders by concern, not by app:
- `src/shared/components/**`
- `src/shared/theme/**` or `src/shared/theme.ts` (existing entrypoint may stay)
- `src/shared/data/**` (domain-neutral only)
- `src/shared/mock-data/**` (only when truly cross-app)

3. Keep shared filenames descriptive and domain-specific.

### Extraction Workflow (Required)

When moving code from one app to shared:

1. Identify all current usage sites before extraction.
2. Define a minimal, domain-neutral public API.
3. Move code to `src/shared/**` using policy-compliant naming.
4. Update imports in both apps in the same change set.
5. Search for stale paths or duplicate legacy implementations.
6. Run build/typecheck validation.
7. Stop and report if extraction introduces unrelated breakage.

### Technical Debt Guardrails

1. If only one app uses the code, do not move it to shared yet.
2. If shared code needs app-specific branching flags, keep separate app-local wrappers.
3. Keep shared modules focused and small; avoid "god" shared utility files.
4. Prefer composition: shared base primitives + app-local adapters.

## Safe Rename Procedure

When renaming:
1. Identify all import sites before renaming.
2. Rename the file.
3. Update direct and indirect imports.
4. Run a stale-reference search.
5. Run build or typecheck validation.
6. Stop and report if unrelated breakage appears.

## Practical Guidance

- Keep naming stable once adopted.
- Prefer consistency over personal style.
- Use one canonical name per concept across the repo.
- If unsure, choose descriptive kebab-case for non-UI modules and PascalCase for UI files.

## Applicability

This policy applies repository-wide unless a subdirectory has an explicit documented override.
