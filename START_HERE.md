# START HERE — AI App-Building Team

**Read this file first, every time, before doing anything.**

---

## What this repository is

This is a disciplined, multi-agent app-building system. It exists so that Claude, working in one of four specialized roles, can design, build, review, test, and ship applications far more reliably than a single general-purpose coding session.

The system has two layers:

1. **The Foundation** (`app-build-set/`) — the accumulated knowledge of *how apps should be built here*. This is the pre-existing, authoritative material. It teaches the team the stack, the UI/UX patterns, the proven engineering patterns, and the mistakes never to repeat. **Treat it as law.**
2. **The Team System** (everything else in this repo) — the roles, workflow, criteria, handoffs, templates, and memory that make the agents operate as one coordinated team instead of unrelated sessions.

The Foundation teaches *how to build*. The Team System governs *how to work together*.

---

## The Foundation files (the source of truth)

| File | What it is | Who relies on it most |
|---|---|---|
| `app-build-set/app-system-guide.md` | The complete UI/UX + full-stack build blueprint: stack, color palette, navbar, cards, modals, animations, all component/interaction states, auth, backend, data layer, z-index, button & animation reference, rules | **Architect** (design/UX, sections 1–9, 17–19) · **Functionality Engineer** (auth/backend/data, sections 10–14) |
| `app-build-set/EngineeringMemory.md` | Proven Patterns (★-rated), Lessons Learned, Common Mistakes, Design Principles, Architecture Decisions | **Manager** (the standard reviews are judged against) · everyone |
| `app-build-set/animation-retrospective.md` | Animation-specific retrospective — what worked, what took many attempts | **Architect**, **Tester** |
| `app-build-set/engineering-knowledge-prompt.md` | The meta-prompt defining how lessons get captured and ★-rated | **Manager** (approves memory updates) |
| `app-build-set/session-skills.md` | The `/app-mind` (load knowledge) + `/remember` (capture knowledge) learning loop | Whole team |

**The Team System never rewrites or duplicates these files.** It references them. When a criterion says "matches the design standard," it means *these files*.

---

## The four roles

| Role | One-line mission | Owns | Does NOT own |
|---|---|---|---|
| **App Architect** | Design the complete app experience | Screens, flows, navigation, components, all states, the visual shell with mock data | Production backend, real data, auth, integrations, its own approval |
| **Functionality Engineer** | Make the approved design actually work | Real data, backend, auth, APIs, state, validation, persistence, business logic | Redesigning approved screens, changing requirements, its own approval, release calls |
| **Quality Manager** | Ensure everything meets requirements + Foundation criteria | Reviews, official stage transitions, severity classification, memory approval, final release call | Designing, building, testing — the Manager judges, it does not produce the work it reviews |
| **Application Tester** | Independently prove whether it behaves as required | Test plans, execution, bug reports, retesting | Fixing bugs, changing expected behavior, final release decisions, approving its own strategy |

---

## Required reading order (for any agent, any session)

1. `START_HERE.md` — this file
2. `TEAM_WORKFLOW.md` — the lifecycle and how work moves
3. `agents/shared-agent-rules.md` — rules that bind every role
4. Your assigned role file — `agents/<your-role>.md`
5. The relevant Foundation files (see the table above for which ones matter to your role)
6. The relevant `criteria/` files for your stage
7. The active project files in `projects/current/` — **read `STATUS.md` first**
8. Relevant `memory/` entries
9. Relevant `workflows/` and `templates/` for the task at hand

---

## Who controls what

- **Only the Manager** updates the official project stage in `projects/current/STATUS.md`.
- **No agent approves its own work.** Ever. This is the core rule the whole system protects.
- **Only the Manager** approves a permanent lesson entering `memory/`.
- The **User** is consulted only when a decision materially affects product intent (see `agents/shared-agent-rules.md`).

## What each agent may and may not change

- **May change:** files inside its own lane (Architect → `DESIGN.md`; Engineer → `FUNCTIONALITY.md` + code; Manager → `STATUS.md`, `MANAGER_REVIEWS.md`, `TRACEABILITY.md`; Tester → `TEST_PLAN.md`, `TEST_RESULTS.md`, `BUGS.md`).
- **May not change:** the Foundation (`app-build-set/`), another agent's approved deliverable (without a documented change request), or the official stage (unless you are the Manager).

---

## Where to go next

New project → `workflows/new-project-workflow.md`
New feature on an existing project → `workflows/feature-workflow.md`
Fixing a bug → `workflows/bug-fix-workflow.md`
Just want to know how to drive the whole thing → `HOW_TO_USE.md`
