# Testing Criteria

Applied by the Tester (plan) and the Manager (accepting the test effort). A feature cannot leave TESTING until these are met.

## Coverage — the test plan must include
- [ ] Every requirement's acceptance criteria has at least one test case.
- [ ] Main flows and secondary flows.
- [ ] Invalid inputs, boundary values, duplicate actions.
- [ ] Empty / loading / error / success / disabled states.
- [ ] Authentication, authorization, and permission checks (including attempts to access what should be blocked).
- [ ] Account isolation for multi-user apps (user A cannot see user B's data).
- [ ] Mobile and desktop layouts (`testing/device-browser-matrix.md`).
- [ ] Navigation, refresh, sign out/in, persistence.
- [ ] External integrations and their failure modes.
- [ ] Network failures and interrupted workflows.
- [ ] Regression suite (`testing/regression-suite.md`).
- [ ] Accessibility basics; basic performance.
- [ ] Targeted tests for the stack's known Common Mistakes (`app-build-set/EngineeringMemory.md`).

## Evidence
- [ ] Every test case has: steps, expected result, actual result, status, evidence.
- [ ] Failures are logged as reproducible bugs in `BUGS.md` with severity.
- [ ] Intermittent failures are recorded with frequency, not dropped.

## Exit (gate to FINAL_REVIEW)
- [ ] Critical test cases pass.
- [ ] No Blocker or Critical bugs remain open.
- [ ] Major bugs are fixed or explicitly accepted (with reason) by the Manager.
- [ ] Retesting completed after fixes; regression passed.

## Decision
Met → Manager takes it to FINAL_REVIEW. Failing tests → Returned to Functionality Engineer (FIXING). The Tester never fixes; the Tester never decides release.
