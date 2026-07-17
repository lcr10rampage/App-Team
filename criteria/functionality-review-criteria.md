# Functionality Review Criteria

Applied by the Manager at **IMPLEMENTATION_REVIEW**. Judged against `app-build-set/app-system-guide.md` (§10–14), the Proven Patterns / Common Mistakes / Architecture Decisions in `app-build-set/EngineeringMemory.md`, and `DEFINITION_OF_DONE.md`.

## Design fidelity
- [ ] The approved design was implemented correctly — no undocumented redesign.
- [ ] No approved interaction was removed or altered.
- [ ] Any deviation exists as an accepted design-change request in `DECISIONS.md`.

## Functionality
- [ ] Every required interaction works.
- [ ] Real data replaced **all** placeholder/mock data.
- [ ] Saving, editing, deleting, searching, filtering work where specified.
- [ ] Data persists correctly and survives refresh.
- [ ] Business logic matches the requirements' acceptance criteria.

## Every state
- [ ] Loading, empty, error, success, disabled states all work — verified, not assumed.

## Security & access
- [ ] Forms and actions have validation.
- [ ] Permissions are enforced **server-side**, not just hidden in the UI.
- [ ] Authentication is secure (follows app-system-guide §10 patterns).
- [ ] Multi-user data is scoped by `user_id` (EngineeringMemory: "Scope data by user from day one").

## Foundation pattern compliance
- [ ] Uses `authFetch` / shared `current_user` dependency where applicable.
- [ ] Checks `res.ok` (fetch doesn't throw on 4xx/5xx).
- [ ] No named Common Mistake present (mock data left in, only-success-state, StrictMode double-add, un-restarted backend after model change, etc.).
- [ ] Systems that depend on each other were all updated together (no half-migrations).

## Coverage & evidence
- [ ] No requirement was skipped (cross-check `TRACEABILITY.md`).
- [ ] Automated checks pass (`automation/ci-requirements.md`).
- [ ] Evidence is provided for each completed item.
- [ ] Known limitations are recorded, not hidden.

## Decision
Pass → advance to TESTING. Otherwise → Returned to Functionality Engineer with findings + severity + required corrections.
