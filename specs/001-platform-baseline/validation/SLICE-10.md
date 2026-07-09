---
status: Draft
artifact_type: validation-report
slice_id: SLICE-10
feature_id: 001-platform-baseline
---

# Validation Report — SLICE-10: Agent Roster

## Context

- **knowledge branch:** `001-platform-baseline`, base commit `009ba75849af0d7d0c8c8eb204112cc9acf8b63f` (uncommitted working-tree changes: this report, contract file, tasks.md status)
- **frontend branch:** `agent/slice-10-agent-roster`, base commit `d234cdbd42bc66353e1e0836a4c32152bbfaa828` (uncommitted: new AgentsScreen, api, types, feature, steps, routes wiring)
- **backend branch:** `agent/slice-10-agent-roster`, base commit `a6df30032fabb22b0f07263469a0c787b0efd12b` (uncommitted: new agents module, app.module.ts wiring, api-contract.md update)
- **Platform spec:** `knowledge/specs/001-platform-baseline/spec.md`
- **Platform plan:** `knowledge/specs/001-platform-baseline/plan.md`
- **Platform tasks:** `knowledge/specs/001-platform-baseline/tasks.md` (SLICE-10 section)
- **Contract:** `knowledge/contracts/slice-10-agents/contract.md`

## Commands Run

| Command | Repo | Result |
|---|---|---|
| `npx jest agents` | backend | Passed (8/8) |
| `npx jest` (full suite) | backend | Passed (121/121) |
| `npm run check` (lint + lint:policy + typecheck + build) | backend | Passed |
| `npx tsc -b --noEmit` | frontend | Passed |
| `npx eslint .` | frontend | Passed (1 pre-existing warning, unrelated to this slice) |
| `npx vitest run` | frontend | Passed (2/2) |
| `node --import tsx cucumber.js --config cucumber.mjs` (full suite) | frontend | Passed (30/30 scenarios, 104/104 steps) |
| `npm run check` (lint + lint:policy + typecheck + build) | frontend | Passed |

## Requirement Coverage

| Requirement | Scenario/Test | Result | Notes |
|---|---|---|---|
| REQ-AGENT-001 (KPI strip) | `AgentsScreen.tsx` KPI tiles; exercised implicitly by both BDD scenarios | Passed | 5 tiles: agents on team, signals today, acting autonomously, evidence-backed, task agents running |
| REQ-AGENT-002 (three card sections) | `AgentsScreen.tsx` Monitors/Operators/Task Agents sections | Passed | |
| REQ-AGENT-003 (pause/resume monitor) | "Pause and resume a monitor" scenario (reuses spec.md's Gherkin verbatim) | Passed | `monitor-toggle-btn`, `monitor-status`, `monitor-signals-today` |
| REQ-AGENT-004 (operator cards, navigate to Autonomy) | `OperatorCard` component (manual verification — no dedicated BDD scenario) | Passed (manual) | Navigates to `/autonomy`, which currently renders `ScreenPlaceholder` since SLICE-11 is not yet implemented — not a defect in this slice |
| REQ-AGENT-005 (task-agent cards, open/retire) | `TaskAgentCard` component (manual verification — no dedicated BDD scenario) | Passed (manual) | `task-agent-open-btn` navigates to `openLink`; `task-agent-retire-btn` hidden once already retired |
| REQ-AGENT-006 (hire from catalog) | "Hire a standing agent" scenario | Passed | `hire-agent-btn`, `hire-kind-select`, `hire-subtype-select`, `hire-confirm-btn` |
| REQ-AGENT-007 (one canonical Agents implementation) | N/A — resolved by definition | Passed (non-blocking) | No Pricing Platform V2 Agentic fork exists in this workspace to reconcile against; this implementation is canonical trivially. Documented in contract `open_q1_resolution`. |

## BDD/Cucumber Results

`features/agent-roster.feature` — 2/2 scenarios passed:
- Pause and resume a monitor
- Hire a standing agent

Full suite (30 scenarios / 104 steps across all features, including pre-existing ones) passed with no regressions.

## Pinned-Stack Compliance

- No new libraries introduced. UI built with existing `Button` (shadcn/ui) primitive, Tailwind utility classes, `framer-motion` page transition, and `react-router-dom`'s `useNavigate`, matching established conventions.
- No Zustand store added — followed the established local-`useState`-per-screen pattern (hire form open/closed, kind/subtype selection).
- Plain HTML `<select>` elements used for the hire kind/subtype pickers, consistent with the existing `focus-group-select` precedent in `DiscountModelingScreen.tsx` (no shadcn `Select` wrapper needed for this simple case, matching prior practice).

## Data-Taxonomy Compliance

- Two new Tier-1 `*Entity` types introduced: `MonitorEntity`, `TaskAgentEntity` — both new canonical entities per spec Open Q12, not promoted from a `*View`.
- `OperatorView` is explicitly Tier-2 (derived, not persisted as its own entity), per plan.md's guidance that "Operators are derived."
- `AgentKpis`, `AgentRosterView`, `AgentCatalogView` are Tier-2 screen constructs, correctly suffixed `*View`/no suffix-conflict for the plain-data `AgentKpis` aggregate.

## Naming/Structure Compliance

- Backend: `backend/src/agents/{agent-types.ts, agents.controller.ts, agents.service.ts, agents.controller.spec.ts}` — matches the flat-module convention of `approvals/`, `price-scenarios/`, `discount-modeling/`.
- Frontend: `frontend/src/screens/agents/{agent-types.ts, agents-api.ts, AgentsScreen.tsx}` — matches `screens/approvals/`, `screens/price-scenarios/` conventions (kebab-case modules, PascalCase component file).

## Design-System Compliance

- Zinc/teal palette; trust-level badges (Low=zinc, Medium=amber, High=teal) and status badges reuse the same color vocabulary as `ApprovalsScreen`/`PriceScenariosScreen`.
- KPI tile layout matches the general card/tile visual language used elsewhere in the app (border, rounded, zinc-200/800 border colors).

## Mismatches Found

None. REQ-AGENT-007 required a canonical-version decision but resolved trivially (no competing V2 fork exists in this workspace) — not a mismatch, documented in the contract.

## Patch Attempts

None required — no test failures after initial implementation (backend contract test failed as expected before the module existed; all tests passed on first implementation pass thereafter).

## Remaining Blockers / Risks

1. **Operator trust data is a deterministic placeholder.** `trustLevel`/`evidenceStatus`/`trackRecord` do not derive from any real trust-ladder computation, since SLICE-11 (Pricing Autonomy) — the system that would supply that data — does not exist yet. Flagged in the contract for revisit once SLICE-11 lands.
2. **Task agents are seeded, not spawned.** No live trigger exists yet to programmatically create a task agent from a real platform event (e.g. a scenario submission). v1 seeds 2 examples. A future slice wiring real spawning should not need to change this contract's read/retire shape.
3. **REQ-AGENT-004's "navigate to Autonomy" lands on a placeholder screen** until SLICE-11 is implemented — expected, not a defect.
4. Two dev servers were reused from a prior session (backend on port 3000 via `start:dev` watch mode, which auto-recompiled to pick up the new module; frontend on port 5176 after ports 5173-5175 were occupied by other leftover sessions). No action needed, flagged for transparency.

## Completion Criteria (from tasks.md)

> REQ-AGENT-* scenarios green; canonical version chosen; entities defined; `npm run check` passes.

- REQ-AGENT-* scenarios: green (REQ-AGENT-004/005 verified manually pending future dedicated BDD coverage if desired).
- Canonical version chosen: yes, by definition (see REQ-AGENT-007 above).
- Entities defined: yes — `MonitorEntity`, `TaskAgentEntity` (Tier-1), `OperatorView` (Tier-2), documented in `knowledge/contracts/slice-10-agents/contract.md`.
- `npm run check`: passes in both frontend and backend.
