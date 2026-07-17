# TEAM WORKFLOW

The complete lifecycle every project and feature moves through. **Only the Manager advances the official stage** (recorded in `projects/current/STATUS.md`).

---

## The lifecycle

```
PROJECT_BRIEF
      ↓
DESIGNING              ← Architect
      ↓
DESIGN_REVIEW          ← Manager  (gate: design-review-criteria.md)
      ↓
FUNCTIONALITY_BUILD    ← Functionality Engineer
      ↓
IMPLEMENTATION_REVIEW  ← Manager  (gate: functionality-review-criteria.md)
      ↓
TESTING                ← Tester
      ↓
FIXING                 ← Functionality Engineer
      ↓
RETESTING              ← Tester
      ↓
FINAL_REVIEW           ← Manager  (gate: release-criteria.md)
      ↓
RELEASE_APPROVED       ← Manager
```

---

## Stage ownership

| Stage | Owner | Output | Advances when |
|---|---|---|---|
| PROJECT_BRIEF | User + Manager | `PROJECT_BRIEF.md`, `REQUIREMENTS.md` (each requirement gets a stable ID) | Requirements are clear and IDed |
| DESIGNING | Architect | `DESIGN.md` + mock-data shell | Design is complete per the Architect's Definition of Complete Design |
| DESIGN_REVIEW | Manager | A review in `MANAGER_REVIEWS.md` | Design passes `criteria/design-review-criteria.md` |
| FUNCTIONALITY_BUILD | Functionality Engineer | Working code + `FUNCTIONALITY.md` handoff | Implementation matches approved design, real data connected |
| IMPLEMENTATION_REVIEW | Manager | A review in `MANAGER_REVIEWS.md` | Passes `criteria/functionality-review-criteria.md` |
| TESTING | Tester | `TEST_PLAN.md`, `TEST_RESULTS.md`, `BUGS.md` | All planned tests executed with evidence |
| FIXING | Functionality Engineer | Fixes + updated `FUNCTIONALITY.md` | All assigned bugs addressed |
| RETESTING | Tester | Updated `TEST_RESULTS.md` | Fixes verified + regression checked |
| FINAL_REVIEW | Manager | Final review + `RELEASE_REPORT.md` | Passes `criteria/release-criteria.md` |
| RELEASE_APPROVED | Manager | Signed release, `RETROSPECTIVE.md`, memory updates | — |

---

## Approval requirements

- Every stage transition is a **Manager decision**, recorded with evidence in `MANAGER_REVIEWS.md`.
- No stage may be skipped. A feature that "obviously works" still goes through review and testing.
- The Manager's approval options: **Approved · Approved with minor corrections · Rejected · Returned to Architect · Returned to Functionality Engineer · Returned to Tester · Blocked pending user decision.**

---

## Return paths (when work fails)

| Problem found | Returns to | Stage goes back to |
|---|---|---|
| Design flaw / missing screen / missing state | Architect | DESIGNING |
| Missing or broken functionality | Functionality Engineer | FUNCTIONALITY_BUILD |
| Failed test | Functionality Engineer | FIXING |
| Unclear product requirement | User | PROJECT_BRIEF (paused) |
| Architecture/system conflict | Architect + Functionality Engineer | joint resolution |
| Release-blocking concern | — | Manager blocks release |

Work always moves **backward to the role that owns the problem**, never sideways into another role's lane.

---

## When user input is required

The team pauses and asks the User only when:
- A product requirement is ambiguous or missing.
- A design change materially alters product intent or scope.
- A tradeoff affects cost, data, privacy, or a promise made to the end user.
- A release-blocking risk needs an accept/reject decision.

Routine technical choices do **not** require the user — they are recorded as Decision Records (`projects/current/DECISIONS.md`).

---

## How design changes are proposed

The Functionality Engineer never silently edits an approved design. When the design is impractical/unsafe/incompatible, it files a **design-change request** (`templates/design-change-request-template.md`) → Manager decides → if accepted, Architect updates `DESIGN.md` and the change is logged in `DECISIONS.md`. See `workflows/design-change-workflow.md`.

## How bugs are reported

Tester files each bug with `templates/bug-report-template.md` into `BUGS.md`, assigns severity, and hands off to the Functionality Engineer. The Tester never fixes bugs. See `workflows/bug-fix-workflow.md`.

## How retesting works

After FIXING, the Tester re-runs the failed cases **plus** the regression suite (`testing/regression-suite.md`). A fix is not "done" until retest passes and no regression appeared.

## How final approval works

FINAL_REVIEW checks requirements coverage, design approval, implementation approval, testing evidence, known limitations, and the release gate. Only then does the Manager mark `RELEASE_APPROVED`. See `workflows/release-workflow.md`.

## How project lessons enter memory

After release, the Manager runs the retrospective (`templates/retrospective-template.md`), and **Manager-approved** lessons are written into `memory/`. Nothing enters permanent memory without Manager approval. This mirrors the Foundation's `/remember` loop (`app-build-set/session-skills.md`).
