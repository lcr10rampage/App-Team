# Build Progress — AI App-Building Team System

> **STATUS: COMPLETE** — all planned files built (78 total: 73 team-system + 5 foundation). Cross-references validated, no broken links, pushed to the `App-Team` repo. Kept as a historical record of the build.

**Goal:** Build the full AI App-Building Team structure (from `AI_APP_BUILDING_TEAM_SPEC.md`) inside `/home/lcr10/App-System-Guide/`, wired to the existing `app-build-set/` Foundation. Reference the Foundation — never duplicate or rewrite it.

**Working directory:** `/home/lcr10/App-System-Guide/`
**Git:** not yet initialized (do this in the final step). No GitHub remote yet.

---

## Core principles to preserve while finishing (do not drift)
- The Foundation is `app-build-set/` (5 files). It is law. New files **reference** it, never copy it.
- Role boundaries: Architect = design/UX; Functionality Engineer = make it work; Manager = judge + control stage + only approver; Tester = independent verification, never fixes.
- No agent approves its own work. Only the Manager moves the official stage. Only Manager-approved lessons enter `memory/`.
- Keep files practical and concise. No bureaucracy for its own sake. Reference existing files heavily.
- Foundation mapping already used throughout: `app-system-guide.md` §1–9,17–19 → Architect; §10–14 → Engineer; `EngineeringMemory.md` Proven Patterns/Common Mistakes/Design Principles/Architecture Decisions → Manager criteria + Tester; `engineering-knowledge-prompt.md` → Manager-approved memory; `session-skills.md` → the /app-mind + /remember loop.

---

## DONE (already written, ~34 files)
- **Root:** `AI_APP_BUILDING_TEAM_SPEC.md`, `START_HERE.md`, `TEAM_WORKFLOW.md`, `DEFINITION_OF_DONE.md`, `TRACEABILITY.md`
- **agents/** (5): `shared-agent-rules.md`, `architect.md`, `functionality-engineer.md`, `manager.md`, `tester.md`
- **criteria/** (5): `master-criteria.md`, `design-review-criteria.md`, `functionality-review-criteria.md`, `testing-criteria.md`, `release-criteria.md`
- **workflows/** (5): `new-project-workflow.md`, `feature-workflow.md`, `bug-fix-workflow.md`, `design-change-workflow.md`, `release-workflow.md`
- **handoffs/** (5): `architect-to-functionality.md`, `functionality-to-manager.md`, `manager-to-tester.md`, `tester-to-functionality.md`, `final-release-handoff.md`
- **templates/** (15): project-brief, requirements, app-design, screen-spec, component-spec, feature-spec, design-change-request, implementation-plan, implementation-log, manager-review, test-plan, test-case, bug-report, release-report, retrospective

Directories already created: `agents criteria workflows handoffs templates memory testing automation projects/current` (all exist).

---

## TODO (remaining files)

### 1. memory/ (7 files) — curated, Manager-approved knowledge stores
These must **reference `app-build-set/EngineeringMemory.md` as the seeded, authoritative source** and hold NEW, ongoing, Manager-approved lessons — not duplicate the Foundation. Each file: 1-line purpose + "seeded by EngineeringMemory.md" note + the entry format from the spec + an empty "Entries" section.
- `engineering-memory.md` — principles, reliable workflows, major lessons, reasons behind rules. Points to Foundation as the primary store; this is the team-system-level index/extension.
- `proven-patterns.md` — entry format: Pattern name · When to use · How it works · Projects where it succeeded · Risks · When NOT to use.
- `reusable-components.md` — Component · Purpose · Where it lives · Approved use cases · Variants · Dependencies · Accessibility notes · Known limitations · Projects using it.
- `common-mistakes.md` — Mistake · Symptoms · Cause · Prevention · Detection method · Projects where it occurred. (Note: seeded by EngineeringMemory Common Mistakes.)
- `failed-approaches.md` — Approach · Why chosen · What failed · Impact · Replacement · When it might still be valid.
- `prompt-lessons.md` — Original instruction · Result · Problem · Improved instruction · Final outcome · Lesson for future agents.
- `system-relationships.md` — how major systems interact (auth↔permissions, DB↔UI state, notifications↔prefs, uploads↔storage, billing↔access, caching↔freshness, background jobs↔visible status).

### 2. testing/ (6 files)
- `testing-strategy.md` — overall approach; references `criteria/testing-criteria.md` + EngineeringMemory Common Mistakes as the "where bugs hide" list.
- `critical-user-flows.md` — template/list of must-pass flows per project (starts empty w/ format).
- `regression-suite.md` — running list of checks that must pass after any change (starts empty w/ format).
- `device-browser-matrix.md` — which devices/browsers/viewports to test (default: mobile + desktop; note the app-system-guide responsive rules).
- `test-data-guidelines.md` — how to make/seed test accounts + data; note "run seed.py before testing login" lesson from EngineeringMemory.
- `release-smoke-tests.md` — minimal must-pass checks before release (auth, core CRUD, permissions, refresh/persist).

### 3. automation/ (5 files)
- `quality-gates.md` — the 4 gates (Design/Functionality/Testing/Release) as hard checklists mirroring the `criteria/` files. This is the enforcement summary.
- `ci-requirements.md` — formatting, lint, typecheck, unit/integration tests, build success, security scan, dep check, migration validation. Note: inspect actual stack before inventing tools; default stack = Vite/React/TS + FastAPI/SQLite from app-system-guide §1.
- `linting-rules.md` — expected lint/format posture for the stack (don't invent conflicting tools).
- `branch-rules.md` — branch structure (main, develop, design/*, feature/*, fix/*, test/*); who creates/merges, required reviews/checks, naming, cleanup, emergency fixes.
- `commit-rules.md` — conventional commit format with the spec's examples: `feat(projects): ...`, `fix(auth): ...`, `design(settings): ...`, `test(projects): ...`, `docs(workflow): ...`.

### 4. projects/current/ (13 files) — scaffold, empty-but-structured
Each is a starting stub that points to its template and starts at the right empty state. `STATUS.md` starts at stage `PROJECT_BRIEF`, owner "unassigned", "No active project yet."
- `PROJECT_BRIEF.md` (→ project-brief-template), `STATUS.md` (single source of truth, format from spec), `REQUIREMENTS.md` (→ requirements-template), `DESIGN.md` (→ app-design-template), `FUNCTIONALITY.md` (→ implementation-log-template), `DECISIONS.md` (decision-record format: ID·Title·Status·Context·Options·Decision·Reason·Consequences·Approved by·Date), `TRACEABILITY.md` (table from root `TRACEABILITY.md`), `MANAGER_REVIEWS.md` (→ manager-review-template), `TEST_PLAN.md` (→ test-plan-template), `TEST_RESULTS.md` (test-case results log), `BUGS.md` (→ bug-report-template), `RELEASE_REPORT.md` (→ release-report-template), `RETROSPECTIVE.md` (→ retrospective-template).

### 5. Final steps
- `HOW_TO_USE.md` at root (referenced by START_HERE) — concise operator guide: how to start a project, assign Claude the Architect role, move to Engineer, run a Manager review, run testing, handle bugs, do final review, update memory. Map each to the Skill/agent invocation the user actually uses.
- **Validate** (spec Stage 5): roles separated? responsibilities/restrictions clear? handoffs clear? requirements traceable? failed work moves backward? Manager can block? Tester independent? memory can't be polluted? reuses Foundation? contradictions? duplicate files? Fix anything that fails.
- **git init**: `cd /home/lcr10/App-System-Guide && git init && git branch -m main`. Add a `.gitignore` if needed (none really required here — all markdown). Ask the user whether to create a GitHub repo + push, or leave local. Do NOT push without asking.
- Report: file count, structure tree, how it satisfies the spec.

---

## Task IDs (harness task list)
#6 = memory/ + testing/ + automation/ (in progress). #7 = projects/current scaffold. #8 = git init + validate + usage guide.

## Style notes for consistency with what's already written
- Markdown, concise, practical. Tables where it helps. Cross-link sibling files with backticks + relative paths.
- Every file that embodies a standard should name the exact Foundation file/section it derives from.
- Do not restate the whole spec in each file — point to it.
