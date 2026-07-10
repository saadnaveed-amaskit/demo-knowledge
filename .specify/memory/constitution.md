<!--
Sync Impact Report
- Version change: 1.0.1 → 1.1.0
- Modified principles:
  - Workspace Model → clarified backend-owned API contract location
  - I. Multi-Repo Workspace Source of Truth → knowledge references API contracts, backend owns canonical API contract
  - IV. EARS and Gherkin Requirements → explicitly requires backend API Gherkin for externally observable API behavior
  - VIII. Contract-First Multi-Repo Development → backend-owned canonical contract and backend API Gherkin/Cucumber required
  - IX. Acceptance-Test-First Implementation → separate preparation gate before implementation
  - X. Validation Report and Patcher Agent → backend API BDD/Cucumber and preparation artifact reporting
  - XI. Knowledge Updated After Code Merge → knowledge records contract references after merge
  - XII. Scope, Simplicity, and Repository Boundary Control → backend contract ownership clarified
  - Development Workflow and Quality Gates → added preparation approval gate
- Added principles:
  - None
- Templates requiring updates:
  - .claude/commands/prepare-next-slice.md
  - .claude/commands/prepare-approved-slice.md
  - .claude/commands/prepare-approved-feature.md
  - .claude/commands/implement-next-slice.md
  - .claude/commands/implement-approved-slice.md
  - .claude/commands/implement-approved-feature.md
  - .claude/rules/acceptance-test-first.md
  - .claude/rules/contract-first-development.md
  - .claude/rules/validation-reporting.md
  - .claude/rules/ears-gherkin-requirements.md
- Follow-up TODOs:
  - Ensure backend repo has runnable Cucumber/BDD setup for API acceptance tests.
  - Ensure backend/contracts/api-contract.md exists as the canonical API contract file.
  - Ensure slice preparation artifacts use `status: Approved` before implementation.
-->

# Retail Nucleus Multi-Repo Agent Workflow Constitution

This constitution governs how Retail Nucleus features are specified, planned, implemented,
validated, patched, and finalized across a local multi-repo workspace.

It combines the client-provided Retail Nucleus product/technology principles with the
multi-repo agent workflow used by this harness. The client-provided stack, data taxonomy,
design system, testing, naming, and simplicity constraints are non-negotiable unless an
explicit task-level exception is approved and recorded.

## Authoritative Reference Documents

The following client-provided reference documents should live at the root of the `knowledge`
repository and must be read when generating specs, plans, tasks, implementation guidance,
validation reports, or patch plans:

```text
knowledge/TECHNOLOGY.md
knowledge/DATA-TAXONOMY.md
```

If the client also provides runtime agent files, place them at the root of the `knowledge`
repository as well:

```text
knowledge/CLAUDE.md
knowledge/AGENTS.md
```

`TECHNOLOGY.md` is the authoritative reference for detailed technology mechanics, approved
libraries, stack constraints, and implementation policy checks.

`DATA-TAXONOMY.md` is the authoritative reference for detailed data tiers, naming suffixes,
entity/view/form/filter/state separation, and data-shape discipline.

This constitution states the non-negotiable workflow and governance principles. The reference
documents provide the detailed mechanics that agents must apply under those principles. If the
constitution and a reference document appear to conflict, the agent must stop and ask for human
clarification instead of guessing.

## Workspace Model

The workspace contains separate GitHub repositories under one local parent folder:

```text
harness-workspace/
├── knowledge/
│   ├── TECHNOLOGY.md
│   ├── DATA-TAXONOMY.md
│   ├── .specify/memory/constitution.md
│   ├── specs/
│   ├── reports/
│   └── wiki/
├── frontend/
└── backend/
    └── contracts/
        └── api-contract.md
```

The `knowledge` repository is the source of truth for product intent, specifications,
EARS requirements, Gherkin scenarios, implementation plans, task breakdowns,
validation reports, shipped-feature history, logs, and wiki-style documentation.

The backend-owned API contract is the canonical machine/human-readable API contract for
implementation work:

```text
backend/contracts/api-contract.md
```

The `knowledge` repository may reference this contract in specs, plans, tasks, validation
reports, logs, and wiki pages, but it must not duplicate or replace the canonical backend
contract file.

The `frontend` repository is the implementation target for user interface code,
frontend unit tests, Cucumber/Playwright acceptance tests, frontend build artifacts,
and frontend-facing contract consumption.

The `backend` repository is the implementation target for API code, backend API Gherkin/
Cucumber tests, backend unit tests, integration tests, the backend-owned API contract, and
business logic.

Agents may read all repositories when a workflow requires it, but they must respect the
repository boundaries defined in this constitution.

---

## Core Principles

### I. Multi-Repo Workspace Source of Truth

The `knowledge` repository is the canonical coordination layer for the multi-repo
workspace.

Specs, plans, tasks, validation reports, wiki pages, logs, and shipped-feature records
must live in `knowledge`.

The canonical API contract must live in the backend repo at:

```text
backend/contracts/api-contract.md
```

`knowledge` artifacts may reference the backend-owned API contract, but must not create a
separate canonical contract copy.

The `frontend` and `backend` repositories are implementation repositories. They must not
be treated as the source of truth for product requirements, acceptance criteria, or shipped
feature history.

Agents must read implementation repositories to understand existing code, tests, and
conventions, but implementation details discovered in code must not override approved
knowledge artifacts.

**Rationale**: Centralized knowledge keeps product intent auditable while allowing
frontend and backend to remain focused implementation repositories.

---

### II. Pull Latest Before Every Automated Run

At the start of every automated agent run, the agent must pull the latest changes from all
relevant GitHub repositories.

At minimum, the agent must update:

```text
knowledge/
frontend/
backend/
```

before reading requirements, creating artifacts, implementing code, validating behavior,
patching mismatches, or finalizing knowledge.

The agent must report the branch and latest commit SHA for each repository used in the run.

If any repository has uncommitted local changes, merge conflicts, failed pulls, detached
HEAD state, unexpected branch state, or unresolved rebase/merge state, the agent must stop
and report the blocker before continuing.

**Rationale**: Multi-repo automation is unsafe unless every run begins from the latest
shared state.

---

### III. Artifact Approval Gates

The required artifact sequence is:

```text
spec.md
→ plan.md
→ tasks.md
→ implementation
```

The agent must stop completely after creating or updating each governed artifact.

The human reviewer must review and approve the artifact before the next workflow command is
run.

Approval must be represented by a status field at the top of the relevant document.

Allowed artifact statuses:

```text
Draft
Needs Changes
Approved
Rejected
```

Each governed artifact must include a top-level status field. Front matter is preferred:

```md
---
status: Draft
feature_id: 042
artifact_type: spec
approved_by:
approved_at:
---
```

Plain Markdown status fields are acceptable when front matter is not used:

```md
Status: Draft
Feature ID: 042
Artifact Type: spec
```

The agent must not proceed from `spec.md` to `plan.md` unless `spec.md` status is
`Approved`.

The agent must not proceed from `plan.md` to `tasks.md` unless `plan.md` status is
`Approved`.

The agent must not proceed from `tasks.md` to implementation unless `tasks.md` status is
`Approved`.

**Rationale**: Human approval gates prevent the agent from silently moving from idea to
code without review.

---

### IV. EARS and Gherkin Requirements

Every feature specification must include clear, testable functional requirements.

Functional requirements must use EARS notation wherever practical.

Preferred EARS forms:

```text
The <system> shall <response>.
When <trigger>, the <system> shall <response>.
While <state>, the <system> shall <response>.
If <unwanted condition>, then the <system> shall <response>.
Where <feature is included>, the <system> shall <response>.
When <trigger> while <state>, the <system> shall <response>.
```

Every user-facing or externally observable behavior must include Gherkin acceptance
scenarios.

Externally observable backend/API behavior is included in this requirement.

For any feature or slice that creates or changes backend/API behavior, backend API Gherkin
scenarios are required unless the preparation or validation artifact explicitly proves that
no externally observable backend/API behavior changed.

Backend API Gherkin scenarios must describe observable request/response behavior, status
codes, validation errors, and contract-level outcomes.

Backend API Gherkin scenarios must be executable through backend Cucumber tests that call
the backend API directly, not through the frontend UI.

Recommended backend locations:

```text
backend/tests/features/**/*.feature
backend/tests/steps/**/*.ts
backend/tests/support/**/*.ts
```

Contract tests, unit tests, and integration tests may supplement backend API Gherkin
coverage, but they do not replace backend API Gherkin/Cucumber coverage for externally
observable API behavior.

Preferred Gherkin form:

```gherkin
Scenario: <short behavior name>
  Given <initial context>
  When <event or action occurs>
  Then <expected outcome>
```

Use `Scenario Outline` when one behavior must be validated across multiple examples.

EARS and Gherkin must remain aligned:

```text
EARS = what the system shall do
Gherkin = how the behavior is validated
```

Gherkin scenarios must not invent behavior that is not supported by approved EARS
requirements.

Unknowns must be marked `[NEEDS CLARIFICATION]` rather than guessed.

**Rationale**: EARS keeps requirements precise; Gherkin turns requirements into testable
behavior.

---

### V. Retail Nucleus Library-Pinned Stack

Detailed technology mechanics are defined in `knowledge/TECHNOLOGY.md`. This principle
summarizes the non-negotiable stack constraints that plans and implementation must follow.

Every UI concern must use the one approved library for that concern. Alternatives and custom
reimplementations are forbidden unless an explicit task-level exception is approved.

Approved stack:

- UI components: shadcn/ui only. Do not hand-write Button, Dialog, Select, Table, or similar
  primitives when an approved shadcn/ui component exists. Add missing components via
  `npx shadcn@latest add <component>`.
- Charts: Nivo only (`@nivo/*`). Inline SVG is allowed only for decorative/custom visuals
  with no Nivo equivalent. Do not use Recharts, Chart.js, Victory, or Highcharts.
- State: Zustand for UI state and nuqs for URL-synced filters. Do not use Redux, Jotai,
  Recoil, or Context as global state.
- Routing: React Router v7 only.
- Forms: React Hook Form + Zod only.
- Animation: Framer Motion only.
- Dates: date-fns only.

Any approved deviation must include a policy exception annotation:

```ts
// tech-policy-exception: <id> ticket:<TICK> reason:<why>
```

`npm run lint:policy` and `npm run check` must enforce this policy where those scripts exist.

**Rationale**: A pinned stack prevents dependency drift and keeps the product internally
consistent.

---

### VI. Retail Nucleus Data Taxonomy Discipline

Detailed taxonomy mechanics are defined in `knowledge/DATA-TAXONOMY.md`. This principle
summarizes the non-negotiable data-shape discipline that specs, plans, APIs, and UI state
must follow.

Every type belongs to exactly one tier.

Tier 1 — Domain Data (`*Entity`): canonical records with an `id` and foreign-key
relationships. These map to backend entity classes. Do not use an `*Entity` directly as
local UI state.

Tier 2 — Screen Constructs: screen-specific shapes derived from entities using the correct
suffix:

```text
*View     display projection
*Row      grid/table row
*Form     React Hook Form shape
*Filters  filter state
*State    local UI state
*Option   dropdown item
```

Specs must name Key Entities at the Tier-1 level and describe screen constructs as derived
projections, never the reverse.

Plans and tasks must preserve this separation when designing frontend state, backend data,
and API contracts.

**Rationale**: Clear data tiers prevent domain leakage, brittle UI state, and inconsistent
API shapes.

---

### VII. Persona-Anchored, Technology-Agnostic Specs

Retail Nucleus is a B2B platform serving distinct personas, including Executive, Pricing
Team, Pricing Strategist, Marketing, and Admin.

Every user story and functional requirement must name the persona it serves and the value
delivered to that persona.

Approval and governance flows, including Guardrails, Approvals, and Autonomy, are
first-class requirements. Pricing actions must move through explicit human decision points
before execution.

Specifications must describe what the system does and why it matters. They must not describe
how to implement it, except where the constitution's pinned stack is itself a requirement.

Implementation choices, component selections, chart choices, state shapes, and technical
tradeoffs belong in `plan.md`, not `spec.md`.

Unknowns must be marked `[NEEDS CLARIFICATION]` instead of guessed.

**Rationale**: Specs should capture product value and governance intent while leaving
implementation details to the plan.

---

### VIII. Contract-First Multi-Repo Development

For features involving both backend and frontend, the backend owns the canonical API
contract.

The canonical API contract is:

```text
backend/contracts/api-contract.md
```

Do not create API contract files under `knowledge/contracts/**`.

Do not create per-feature or per-slice API contract files unless the human explicitly
changes this rule.

`knowledge` may reference the canonical backend API contract in specs, plans, tasks,
validation reports, logs, and wiki pages, but it must not own or duplicate the contract
file.

The backend implementation must satisfy the approved backend-owned contract.

The frontend implementation must consume the approved backend-owned contract.

Frontend tests may mock the contract before the backend is running, but full validation must
run against the real frontend and real backend before merge.

For full-stack/API features or slices, the expected order is:

```text
1. Identify API behavior in the approved spec, plan, and tasks.
2. Update backend/contracts/api-contract.md before backend API implementation.
3. Prepare backend API Gherkin scenarios for externally observable API behavior.
4. Prepare backend Cucumber step definitions and API test support.
5. Prepare backend contract tests against the canonical API contract.
6. Run backend API Gherkin/Cucumber and contract tests and confirm expected failures.
7. Human reviews and approves the preparation artifact.
8. Implement backend until backend API Gherkin/Cucumber and contract tests pass.
9. Add or run frontend tests against the mocked or real contract before frontend implementation.
10. Implement frontend until tests pass.
11. Run end-to-end validation against real frontend and real backend behavior.
12. Record contract changes and validation results in the relevant knowledge validation report.
```

Backend API Gherkin/Cucumber scenarios validate externally observable API behavior.

Contract tests validate agreement with the canonical API contract.

Both are required for full-stack/backend API changes unless the preparation or validation
artifact explains why one is not applicable.

If API contract changes are needed during implementation, update
`backend/contracts/api-contract.md` only if the change is already covered by approved
requirements, plans, and tasks. Otherwise, stop and ask for human approval.

**Rationale**: Contract-first development prevents frontend/backend drift and gives each
repo a clear implementation boundary while keeping API ownership in the backend repository.

---

### IX. Acceptance-Test-First Implementation

Executable acceptance tests must be created before production implementation, never after.

The acceptance-test creation step must happen in a separate preparation command before
implementation.

For platform slices:

```text
/prepare-next-slice
or
/prepare-approved-slice <slice-id>
```

For standalone features:

```text
/prepare-approved-feature <feature-id>
```

The preparation command must:

```text
1. select or confirm the approved slice/feature;
2. create or reuse the actual implementation branches in affected implementation repos;
3. add or update executable acceptance tests only;
4. run the new or changed tests before implementation;
5. confirm they fail for the expected missing behavior;
6. create a preparation artifact in knowledge;
7. stop for human review.
```

The preparation command must not implement production code.

The human reviewer must approve the preparation artifact before implementation begins.

For platform slices, the preparation artifact is:

```text
knowledge/specs/<NNN-platform-baseline>/validation/<slice-id>-preparation.md
```

For standalone features, the preparation artifact is:

```text
knowledge/specs/<NNN-feature-slug>/preparation.md
```

Implementation commands must verify the preparation artifact has:

```text
status: Approved
```

before changing production code.

Implementation commands must reuse the implementation branches recorded in the approved
preparation artifact. They must not create a second implementation branch for the same
slice/feature unless the human explicitly requests it.

For frontend behavior, acceptance tests should use Cucumber/Gherkin with Playwright-backed
step definitions when the frontend repo has a runnable BDD setup.

A runnable frontend BDD setup is identified by:

```text
test:bdd package script
Cucumber config file
features/** files
tests/bdd/** files
```

For backend/API behavior, acceptance tests must use backend API Gherkin scenarios with
backend Cucumber step definitions when the slice or feature creates or changes externally
observable API behavior.

Recommended backend BDD locations:

```text
backend/tests/features/**/*.feature
backend/tests/steps/**/*.ts
backend/tests/support/**/*.ts
```

Backend API Gherkin scenarios must exercise the backend API directly through a backend test
server or API client. They must not drive the frontend UI.

For approved externally observable behavior, the preparation workflow must:

1. add or update Gherkin scenarios in the relevant `knowledge/specs/**/spec.md` if needed;
2. add or update executable `.feature` files in the affected implementation repo;
3. add or update step definitions and support utilities;
4. run the new or changed tests before implementation;
5. confirm the tests fail for the expected missing behavior;
6. record the expected failure in the preparation artifact;
7. stop for human approval.

The implementation workflow must then:

1. verify the approved preparation artifact;
2. check out or reuse the branches recorded in the preparation artifact;
3. rerun the prepared tests and confirm the expected failure still occurs;
4. implement the approved task;
5. rerun the prepared tests and make them pass;
6. run validation and write a validation report.

Unit tests, contract tests, and integration tests may supplement acceptance tests.

They must not replace executable acceptance coverage for approved externally observable
behavior.

Implemented scenarios must not remain tagged as `@wip`.

Do not delete failing tests to make validation pass.

**Rationale**: Acceptance-test-first development proves the requested behavior before and
after implementation while giving humans a review gate before production code is written.

---

### X. Validation Report and Patcher Agent

At the end of testing, the agent must create a validation report.

The report must state what was expected, what was tested, what passed, what failed, and what
remains unmatched.

The report must include, where applicable:

```text
feature ID
Linear issue ID
knowledge repo commit SHA
frontend repo commit SHA
backend repo commit SHA
spec.md path
plan.md path
tasks.md path
contract path
test commands run
unit test results
integration test results
frontend BDD/Cucumber results
backend API BDD/Cucumber results
E2E test results
build results
coverage of EARS requirements
coverage of frontend Gherkin scenarios
coverage of backend API Gherkin scenarios
preparation artifact path and approval status
mismatches found
patch attempts made
remaining blockers
```

The report must explicitly map approved requirements to validation results.

Example:

```md
| Requirement | Scenario/Test | Result | Notes |
|---|---|---|---|
| EARS-SALES-1 | weekly-sales-aggregates.feature | Passed | UI button visible |
| EARS-SALES-2 | backend API Gherkin scenario + contract test | Failed | Missing week grouping |
```

The report must be stored in the knowledge repo.

Recommended locations:

```text
knowledge/reports/validation/<feature-id>/latest.md
knowledge/specs/<feature-id>/validation-report.md
```

If validation finds mismatches, failed tests, contract drift, or incomplete implementation,
a patcher agent may attempt fixes.

The patcher agent is limited to 3 attempts.

Each attempt must:

```text
read the validation report
identify the failing requirement or scenario
make the smallest scoped fix
avoid changing approved requirements unless explicitly allowed
rerun relevant validation
record the result
```

The patcher agent must not:

```text
expand feature scope
change approved EARS requirements
change approved Gherkin scenarios
modify unrelated files
hide failing tests
delete failing tests to make validation pass
change secrets or production configuration
continue after 3 failed attempts
```

After 3 failed attempts, the patcher agent must stop and produce a blocker report.

**Rationale**: Validation must be auditable, and patching must remain scoped and limited.

---

### XI. Knowledge Updated After Code Merge

The knowledge repo must not be finalized as shipped until the relevant frontend and/or
backend PRs are merged.

Before code PRs merge, knowledge artifacts may remain in draft, approved, or in-progress
states.

After code PRs merge, the agent may update the knowledge repo to reflect shipped reality.

Post-merge knowledge updates may include:

```text
recording backend API contract references and changes
updating wiki pages
updating feature lists
updating shipped-feature history
adding PR links
adding commit SHAs
writing log.md entries
marking specs complete
recording validation report results
```

Knowledge updates after code merge must reference the merged frontend and/or backend PRs.

**Rationale**: The knowledge base should represent the product that actually shipped, not
only what was planned.

---

### XII. Scope, Simplicity, and Repository Boundary Control

Build only what is needed now.

Every line of code is a liability. Scope decisions must be justified by a concrete, present
requirement.

The agent must not perform opportunistic refactors unless required by the approved task.

The agent must not modify unrelated repositories or unrelated files.

The agent must not modify secrets, credentials, production configuration, or deployment
settings unless explicitly approved.

Repository boundary expectations:

```text
knowledge/
- specs
- contract references
- wiki
- logs
- reports
- feature history
- preparation artifacts
- validation reports

frontend/
- UI code
- frontend unit tests
- Cucumber/Playwright feature tests
- frontend build config only when necessary

backend/
- API code
- backend API Gherkin/Cucumber tests
- backend unit tests
- integration tests
- backend/contracts/api-contract.md
- backend-owned contract implementation
```

If a feature requires changes in more than one repo, the agent must report that explicitly
before implementation.

For cross-repo changes, the agent must open separate PRs per implementation repo unless
instructed otherwise.

**Rationale**: Multi-repo workflows require strict boundaries to keep changes reviewable and
safe.

---

## Retail Nucleus Design System Constraints

The frontend must follow these design constraints:

- Color: Zinc grays + teal accent (`#14b8a6`). Semantic colors: green = success,
  amber = warning, red = error, blue = info.
- Typography: Geist Sans / Geist Mono. Use `.rn-text-*`, `.rn-heading-*`,
  `.rn-display-stat`, `.rn-eyebrow`, and `RN_TYPOGRAPHY` tokens when available. Do not use
  raw Tailwind `text-*` sizes or arbitrary `text-[Npx]` sizes.
- Icons: Lucide with 1.5px stroke.
- Page transitions: Framer Motion with
  `initial={{ opacity: 0, y: 6 }}`, `animate={{ opacity: 1, y: 0 }}`,
  and `transition={{ duration: 0.18, ease: "easeOut" }}`.
- Large data tables: AG Grid. `.ag-root-wrapper { height: 100% }` is required.

---

## Naming and Structure Conventions

UI-rendering files, including screens, views, components, charts, and modals, must use
PascalCase.

Non-UI modules, including types, constants, stores, hooks, helpers, data, and config, must
use kebab-case.

Directories must use kebab-case.

Avoid generic names. Prefer names such as `scenario-types.ts` over `types.ts`.

Stores are always `state.ts`, never `store.ts`.

Hooks follow `useXxxState` naming.

Cross-app shared code lives in `src/shared/` only when used by both apps and domain-neutral.
Everything else is app-local.

---

## Development Workflow

The mandated workflow is:

1. **Pull Latest**
   Pull latest changes from `knowledge`, `frontend`, and `backend`.

2. **Context Load**
   Read current knowledge, existing specs, contracts, frontend conventions, backend
   conventions, and client platform rules.

3. **Specify**
   Create or update `spec.md` in the knowledge repo using persona-anchored user stories,
   EARS requirements, and Gherkin scenarios. Set status to `Draft`.

4. **Human Gate 1**
   Stop. Human reviews `spec.md`. Continue only after `status: Approved`.

5. **Plan**
   Create or update `plan.md` in the knowledge repo. Capture implementation approach,
   affected repos, contracts, data taxonomy, stack choices, tests, and risks. Set status to
   `Draft`.

6. **Human Gate 2**
   Stop. Human reviews `plan.md`. Continue only after `status: Approved`.

7. **Tasks**
   Create or update `tasks.md` in the knowledge repo. Include explicit acceptance-test-first,
   backend, frontend, validation-report, and post-merge knowledge tasks. Set status to
   `Draft`.

8. **Human Gate 3**
   Stop. Human reviews `tasks.md`. Continue only after `status: Approved`.

9. **Prepare Slice or Feature**
   Run the relevant preparation command:
   `/prepare-next-slice`, `/prepare-approved-slice <slice-id>`, or
   `/prepare-approved-feature <feature-id>`.

10. **Acceptance Tests First**
    The preparation command creates or reuses the actual implementation branches and adds or
    updates executable frontend/backend acceptance tests before app/API implementation.
    For backend/API behavior, this includes backend API Gherkin/Cucumber tests.

11. **Expected Failure**
    Run the new or changed tests and confirm they fail for the expected missing behavior.

12. **Human Gate 4**
    Stop. Human reviews test diffs and the preparation artifact. Continue only after the
    preparation artifact has `status: Approved`.

13. **Implement**
    Run the relevant implementation command. The implementation command must reuse the
    implementation branches recorded in the approved preparation artifact and implement only
    approved tasks in the affected repo or repos.

14. **Validate**
    Run unit, integration, frontend BDD, backend API BDD/Cucumber, E2E, `npm run check`, and
    build checks where applicable.

15. **Validation Report**
    Write a report mapping requirements and scenarios to pass/fail results.

16. **Patch If Needed**
    If mismatches exist, run the patcher agent for up to 3 scoped attempts.

17. **PR Review**
    Open PRs in affected implementation repos and stop for human review.

18. **Post-Merge Knowledge Update**
    After frontend/backend PRs merge, update the knowledge repo with contract references,
    docs, logs, validation results, and PR links.

---

## Quality Gates

| Gate | Requirement | Blocking? |
|---|---|---|
| Pull latest | All relevant repos are up to date before the run starts | Yes |
| Spec approval | `spec.md` status is `Approved` | Yes — blocks plan |
| Persona anchoring | User stories and functional requirements name the persona and value delivered | Yes — blocks plan/tasks/code |
| Requirement clarity | EARS requirements are clear and testable | Yes — blocks plan/tasks/code |
| Gherkin coverage | User-facing or externally observable behavior, including backend/API behavior, has Gherkin scenarios | Yes — blocks plan/tasks/preparation/code |
| Backend API BDD | Backend/API behavior has backend API Gherkin/Cucumber coverage, or explicit not-applicable evidence | Yes — blocks implementation |
| Plan approval | `plan.md` status is `Approved` | Yes — blocks tasks |
| Tasks approval | `tasks.md` status is `Approved` | Yes — blocks preparation |
| Preparation approval | Slice/feature preparation artifact is `Approved` | Yes — blocks implementation |
| Technology policy | Implementation follows pinned Retail Nucleus libraries and design constraints | Yes — blocks PR |
| Data taxonomy | Entity and screen construct tiers are correctly separated | Yes — blocks PR |
| Contract approval | `backend/contracts/api-contract.md` exists and is approved/referenced for full-stack/API work | Yes — blocks implementation |
| Acceptance-test-first | Executable tests are created in preparation before implementation and fail for expected missing behavior | Yes — blocks implementation |
| Scope control | Work matches approved tasks and repo boundaries | Yes — blocks PR |
| Validation report | Pass/fail report exists and maps requirements to tests | Yes — blocks PR |
| Patcher limit | Patcher stops after 3 failed attempts | Yes |
| Test passage | Relevant unit, integration, BDD, E2E, and build checks pass | Yes — blocks ready-for-review |
| `npm run check` | Lint, policy lint, and build pass where available | Yes — blocks done/PR readiness |
| Knowledge finalization | Knowledge repo is updated only after code PRs merge | Yes |

---

## Governance

This constitution supersedes informal team practices and agent defaults for this workspace.

When a spec, plan, task list, implementation, validation report, or patch conflicts with a
principle here, the constitution wins unless the task explicitly authorizes an exception and
records the required exception annotation.

Amendments require:

1. documented rationale;
2. semantic version bump;
3. updates to dependent templates, commands, and rules;
4. update to `Last Amended`.

## Versioning Policy

- **MAJOR**: Backward-incompatible workflow or principle changes
- **MINOR**: New principles or materially expanded guidance
- **PATCH**: Clarifications, wording fixes, or non-semantic refinements

## Compliance

All agent commands, artifact generation, implementation, validation, patching, and PR
handoffs must verify compliance with this constitution.

The authoritative copy of this file lives in:

```text
knowledge/.specify/memory/constitution.md
```

Runtime mechanics may be further specified in workspace files such as `CLAUDE.md`,
`AGENTS.md`, `TECHNOLOGY.md`, and `DATA-TAXONOMY.md` where those files exist. Those files may
provide implementation details, but they must not weaken the principles in this constitution.

**Version**: 1.1.0 | **Ratified**: 2026-07-07 | **Last Amended**: 2026-07-10
