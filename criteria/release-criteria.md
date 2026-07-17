# Release Criteria

Applied by the Manager at **FINAL_REVIEW**. This is the last gate. All must be true, with evidence, before `RELEASE_APPROVED`. Mirrors `automation/quality-gates.md` (Release gate).

## Completeness
- [ ] All required features are complete (deferred items explicitly marked and accepted).
- [ ] `TRACEABILITY.md` is current — every required requirement is at `Passed`.

## Approvals in place
- [ ] Design was approved (DESIGN_REVIEW passed).
- [ ] Implementation was approved (IMPLEMENTATION_REVIEW passed).
- [ ] Required tests pass; retesting complete.

## Quality & safety
- [ ] No Blocker or Critical bugs open.
- [ ] Major bugs fixed or explicitly accepted with recorded reason.
- [ ] Security concerns resolved (auth, permissions, data scoping).
- [ ] Known limitations are documented in `RELEASE_REPORT.md`.

## Operational
- [ ] Release smoke tests pass (`testing/release-smoke-tests.md`).
- [ ] Documentation updated.
- [ ] Automated checks / CI green (`automation/ci-requirements.md`).

## Decision & aftermath
- Pass → Manager marks `RELEASE_APPROVED` in `STATUS.md` and writes `RELEASE_REPORT.md`.
- Then → run the retrospective (`templates/retrospective-template.md`) and write **Manager-approved** lessons into `memory/`.
- Any unmet critical item → **Block release** with the specific blocking reason and responsible role.
