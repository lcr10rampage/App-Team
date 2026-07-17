# Workflow: Release

## Pre-conditions
- Stage is at FINAL_REVIEW.
- Testing gate met (no open Blocker/Critical; Major fixed or accepted; retest + regression passed).

## 1. Final review (Manager)
- Apply `criteria/release-criteria.md` in full.
- Verify `TRACEABILITY.md`: every required requirement is `Passed`.
- Confirm design + implementation approvals are on record.
- Confirm security/permission/data-scoping concerns are resolved.
- Collect known limitations.

## 2. Release smoke tests (Tester, requested by Manager)
- Run `testing/release-smoke-tests.md` against a release-like state.
- Any smoke-test failure is release-blocking.

## 3. Release report (Manager)
- Write `RELEASE_REPORT.md` (`templates/release-report-template.md`): what shipped, requirement coverage, test evidence summary, known limitations, security notes, sign-off.

## 4. Approve (Manager)
- Set `STATUS.md` stage → `RELEASE_APPROVED`. Only the Manager does this.

## 5. Retrospective + memory (Manager)
- Run `templates/retrospective-template.md`.
- Judge each notable exchange per `app-build-set/engineering-knowledge-prompt.md` (first-try / multiple attempts / single correction / redesign).
- Write **Manager-approved** lessons into the correct `memory/` file. Nothing enters memory without this approval, and not every activity becomes a permanent lesson.

## Blocking
If any release-criteria item fails, the Manager **blocks release**, names the exact blocker + responsible role, and routes it back (Engineer for defects, Architect for design gaps, User for scope decisions).
