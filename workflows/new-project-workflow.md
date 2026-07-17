# Workflow: New Project

Use this when starting a brand-new application from scratch.

## 0. Prepare
- Clear or archive `projects/current/` (move the old project to `projects/<name>/` if keeping it).
- The Manager creates a fresh `STATUS.md` at stage `PROJECT_BRIEF`.

## 1. PROJECT_BRIEF  (User + Manager)
- Fill `PROJECT_BRIEF.md` (`templates/project-brief-template.md`): product, problem, users, outcome, features, non-goals, constraints, success definition.
- Write `REQUIREMENTS.md` (`templates/requirements-template.md`) — every requirement gets a **stable ID** (AUTH-001, PROJ-001, …).
- Manager confirms requirements are clear → advance to DESIGNING.

## 2. DESIGNING  (Architect)
- Architect reads the Foundation design law (`agents/architect.md`) and designs everything into `DESIGN.md`, builds the mock-data shell, and prepares the handoff.
- Advance request → DESIGN_REVIEW.

## 3. DESIGN_REVIEW  (Manager)
- Manager applies `criteria/design-review-criteria.md`, records a review in `MANAGER_REVIEWS.md`.
- Pass → FUNCTIONALITY_BUILD. Fail → Returned to Architect.

## 4. FUNCTIONALITY_BUILD  (Functionality Engineer)
- Engineer wires the approved design to real data/backend/auth per `agents/functionality-engineer.md` and `DEFINITION_OF_DONE.md`.
- Handoff to Manager → IMPLEMENTATION_REVIEW.

## 5. IMPLEMENTATION_REVIEW  (Manager)
- Apply `criteria/functionality-review-criteria.md`. Pass → TESTING. Fail → Returned to Engineer.

## 6. TESTING → FIXING → RETESTING
- Tester executes the plan; failures → BUGS.md → Engineer fixes → Tester retests + regression. Loop until the testing gate is met.

## 7. FINAL_REVIEW  (Manager)
- Apply `criteria/release-criteria.md`; write `RELEASE_REPORT.md`.

## 8. RELEASE_APPROVED  (Manager)
- Mark released. Run retrospective. Manager-approved lessons → `memory/`.

Throughout: update `TRACEABILITY.md` at every transition; only the Manager moves the stage.
