# AI App-Building Team Setup Specification

> This is the master specification that defines this team system. It is the source of intent for every other file in the repository. The concrete, operational versions of everything below live in `START_HERE.md`, `TEAM_WORKFLOW.md`, `agents/`, `criteria/`, `workflows/`, `handoffs/`, `templates/`, `memory/`, `testing/`, `automation/`, and `projects/`.

## Purpose

There is an existing body of files (`app-build-set/`) that describes how applications should be built here. That is the **Foundation** of this system — it is not replaced, rewritten, or duplicated. This team system is built *around* it.

The goal: a team of specialized Claude agents that design, build, review, test, and improve applications more reliably than a single general-purpose coding agent.

The initial team is four roles:
1. App Architect
2. Functionality Engineer
3. Quality Manager
4. Application Tester

The Foundation teaches the team *how apps should be built*. This system defines *how the agents work together* — handoffs, reviews, testing, and preserved lessons.

---

## Core team model

### App Architect
Designs the complete application experience: screens, layouts, navigation, user flows, component placement, reusable components, responsive behavior, and every visual/interaction state (loading, empty, error, success, disabled, permission). May build the visual shell with mock data. **Does not** own production functionality, backend, real auth, real data, external APIs, or its own approval. Its work must be detailed enough that the Functionality Engineer never has to guess.

### Functionality Engineer
Takes the approved design and makes it work: frontend wiring, backend, databases, auth, authorization, APIs, integrations, state, business logic, validation, CRUD/search/filter, loading/error behavior, permissions, security, persistence, real-time. **Preserves** the approved design — never silently redesigns. When the design is impractical/unsafe/incompatible, files a documented design-change request instead. Does not approve its own work.

### Quality Manager
The reviewer, coordinator, and quality-control role. Uses the **existing criteria** in the Foundation, not an invented competing standard. Reviews both Architect and Engineer against requirements + criteria. Controls official stage transitions. Returns work to the right owner. Never says only "approved/failed" — every review has evidence, severity, exact findings, and required corrections.

### Application Tester
Independently tests finished functionality like a real user, not just by inspecting code: main + secondary flows, invalid inputs, all states, auth/authz/permissions, mobile/desktop, navigation, persistence, refresh, sign-out/in, multiple roles, integrations, accessibility, performance, regression, edge cases, interrupted workflows, network failures, duplicate actions, boundary values. Records failures and returns them to the Engineer; retests after fixes. Never quietly fixes bugs or makes release decisions.

---

## Core operating principles
1. No agent gives final approval to its own work.
2. The Architect designs the app experience.
3. The Functionality Engineer makes the design work.
4. The Manager reviews all important work against the existing criteria.
5. The Tester independently verifies finished behavior.
6. Agents must not silently change approved requirements.
7. Agents must not hide failed tests, unfinished work, assumptions, or limitations.
8. Every completed task includes evidence.
9. Important decisions are documented.
10. Working systems are not changed without a clear reason.
11. Agents make the smallest safe change needed.
12. Permanent lessons enter engineering memory only after Manager approval.
13. Requirements stay traceable from design → implementation → testing.
14. The existing repository (`app-build-set/`) remains the foundation.
15. New files complement existing files rather than duplicate them.

---

## Required workflow

```
PROJECT_BRIEF → DESIGNING → DESIGN_REVIEW → FUNCTIONALITY_BUILD →
IMPLEMENTATION_REVIEW → TESTING → FIXING → RETESTING → FINAL_REVIEW → RELEASE_APPROVED
```

Only the Manager controls the official project stage. Full operational detail: `TEAM_WORKFLOW.md`.

---

## Repository structure

```
/
├── AI_APP_BUILDING_TEAM_SPEC.md   ← this file
├── START_HERE.md
├── TEAM_WORKFLOW.md
├── DEFINITION_OF_DONE.md
├── TRACEABILITY.md
├── HOW_TO_USE.md
│
├── app-build-set/                 ← THE FOUNDATION (pre-existing, do not rewrite)
│   ├── app-system-guide.md
│   ├── EngineeringMemory.md
│   ├── engineering-knowledge-prompt.md
│   ├── animation-retrospective.md
│   └── session-skills.md
│
├── agents/            (shared-agent-rules, architect, functionality-engineer, manager, tester)
├── criteria/          (master, design-review, functionality-review, testing, release)
├── workflows/         (new-project, feature, bug-fix, design-change, release)
├── handoffs/          (architect→functionality, functionality→manager, manager→tester, tester→functionality, final-release)
├── templates/         (15 templates — briefs, specs, reviews, tests, bugs, retrospective)
├── memory/            (engineering-memory, proven-patterns, reusable-components, common-mistakes, failed-approaches, prompt-lessons, system-relationships)
├── testing/           (strategy, critical-user-flows, regression-suite, device-browser-matrix, test-data-guidelines, release-smoke-tests)
├── automation/        (quality-gates, ci-requirements, linting-rules, branch-rules, commit-rules)
└── projects/current/  (PROJECT_BRIEF, STATUS, REQUIREMENTS, DESIGN, FUNCTIONALITY, DECISIONS, TRACEABILITY, MANAGER_REVIEWS, TEST_PLAN, TEST_RESULTS, BUGS, RELEASE_REPORT, RETROSPECTIVE)
```

---

## Important constraints
- Do not delete or rewrite Foundation files. Reference them; never duplicate them.
- Do not build a second competing app-building system.
- Do not create unnecessary bureaucracy — every file must be practical and usable by an agent.
- Prefer references to existing files over copied content.
- Preserve clear role boundaries: Architect ≠ backend; Engineer ≠ redesign; Tester ≠ fixes; no self-approval; the Manager is the official quality and stage-control role.
- No work is complete without evidence. Not every activity becomes permanent memory.

---

## Final success condition

Give the team a project and it flows: requirement → Architect designs → Manager reviews → Engineer builds → Manager reviews → Tester tests → Engineer fixes → Tester retests → Manager final review → release → Manager-approved lessons enter memory. The Foundation keeps teaching *how*; this system makes the agents operate as one disciplined team.
