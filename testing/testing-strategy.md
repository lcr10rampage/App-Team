# Testing Strategy

The Tester's overall approach. Detailed pass/fail rules live in `criteria/testing-criteria.md`; this file is the philosophy and structure.

## Principles
- **Test like a real user, not by reading code.** Exercise the running app.
- **Independence.** The Tester never fixes bugs and never assumes a fix works — always re-verify.
- **Beyond the happy path.** A feature that only works on the main path is a failure. Every state and edge case is in scope.
- **Evidence over claims.** Every result carries steps, expected, actual, and an artifact.

## What to test (layers)
1. **Functional** — every requirement's acceptance criteria; main + secondary flows; CRUD/search/filter.
2. **States** — loading, empty, error, success, disabled, permission.
3. **Security** — auth, authorization, permission enforcement (try to reach what should be blocked), account isolation.
4. **Resilience** — invalid inputs, boundary values, duplicate actions, network failures, interrupted workflows.
5. **Persistence** — save/load, refresh, sign out/in.
6. **Responsive** — mobile + desktop (`device-browser-matrix.md`).
7. **Regression** — `regression-suite.md` after every change.
8. **Known-mistake sweep** — target the stack's documented failure modes in `app-build-set/EngineeringMemory.md`.

## Flow
Build the plan (`templates/test-plan-template.md`) → write cases (`templates/test-case-template.md`) → execute with evidence → file bugs (`templates/bug-report-template.md`) → hand failures back → retest + regression after fixes → report results. The Manager decides release, not the Tester.
