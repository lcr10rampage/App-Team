# How To Use The App-Building Team

A concise operator's guide. You drive; the team follows its own discipline. You can run this from the **App Builder** team in your dashboard, or by pointing any Claude session at this repo.

---

## The one rule to remember
You move work forward by asking the team to act **as a specific role at a specific stage**. Only the **Manager** advances the official stage (`projects/current/STATUS.md`) and only the Manager approves. No role approves its own work.

---

## Start a new project
1. Say: *"Start a new project."* The team reads `START_HERE.md` + `TEAM_WORKFLOW.md`.
2. Answer the **Project Brief** questions → fills `PROJECT_BRIEF.md`.
3. The team writes `REQUIREMENTS.md` with stable IDs. Confirm them.
→ Follow `workflows/new-project-workflow.md`.

## Design it (Architect)
- Say: *"Act as the Architect and design this."*
- Produces `DESIGN.md` + a mock-data shell, following `app-build-set/app-system-guide.md`.

## Review the design (Manager)
- Say: *"Act as the Manager and review the design."*
- Applies `criteria/design-review-criteria.md`; records a review; approves or returns to the Architect.

## Build it (Functionality Engineer)
- Say: *"Act as the Functionality Engineer and build the approved design."*
- Wires real data/backend/auth per `DEFINITION_OF_DONE.md`. Files a design-change request instead of silently changing the design.

## Review the build (Manager)
- Say: *"Act as the Manager and review the implementation."*
- Applies `criteria/functionality-review-criteria.md` + Definition of Done.

## Test it (Tester)
- Say: *"Act as the Tester and test this."*
- Builds a plan, tests like a real user, files bugs in `BUGS.md`. Never fixes.

## Fix bugs (Functionality Engineer) → Retest (Tester)
- *"Fix the open bugs"* → *"Retest and run regression."* Loop until the testing gate is met.
- Follow `workflows/bug-fix-workflow.md`.

## Final review + release (Manager)
- Say: *"Act as the Manager and do the final review."*
- Applies `criteria/release-criteria.md`, runs release smoke tests, writes `RELEASE_REPORT.md`, marks `RELEASE_APPROVED`.

## Capture lessons (Manager)
- Say: *"Run the retrospective."* Manager-approved lessons flow into `memory/`. Nothing enters memory without Manager approval.

---

## Handy pointers
- Lost? → `START_HERE.md`
- The lifecycle → `TEAM_WORKFLOW.md`
- What "done" means → `DEFINITION_OF_DONE.md`
- The standards → `criteria/` (indexed by `master-criteria.md`)
- The build/design law → `app-build-set/app-system-guide.md` + `app-build-set/EngineeringMemory.md`
- Adding a feature → `workflows/feature-workflow.md`

## From the dashboard
Open the **App Builder** team and just talk to it. It reads the role, criteria, and foundation files on demand and tells you which role and stage it's operating in at each step.
