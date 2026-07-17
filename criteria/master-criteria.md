# Master Criteria — Index

This is the index of every standard the Manager applies. **The criteria are not invented here** — they are derived from and point back to the Foundation. This file tells each agent *which* standard applies *when*.

---

## The two sources of truth
1. **`app-build-set/app-system-guide.md`** — how things must be built/designed (stack, palette, components, states, auth, backend, data).
2. **`app-build-set/EngineeringMemory.md`** — what has proven to work (★ patterns), what repeatedly breaks (Common Mistakes), and the Design Principles + Architecture Decisions.

Everything below is a *lens* onto those two files for a specific stage.

---

## Which criteria apply at which stage

| Stage | Criteria file | Judged against |
|---|---|---|
| DESIGN_REVIEW | `criteria/design-review-criteria.md` | app-system-guide §1–9,17–19 · EngineeringMemory Design Principles & UX Decisions |
| IMPLEMENTATION_REVIEW | `criteria/functionality-review-criteria.md` | app-system-guide §10–14 · EngineeringMemory Proven Patterns, Common Mistakes, Architecture Decisions · `DEFINITION_OF_DONE.md` |
| TESTING | `criteria/testing-criteria.md` | REQUIREMENTS acceptance criteria · EngineeringMemory Common Mistakes · `testing/` |
| FINAL_REVIEW | `criteria/release-criteria.md` | all of the above + `automation/quality-gates.md` |

---

## Precedence
1. Product requirements (`REQUIREMENTS.md`) — what the app must do.
2. Foundation criteria (`app-build-set/`) — how it must be done.
3. Accepted Decision Records (`projects/current/DECISIONS.md`) — documented, approved deviations.

A deviation from the Foundation is only acceptable if it exists as an **accepted Decision Record**. Otherwise it is a finding.

## Severity mapping
See `agents/manager.md` for the six-level severity system. As a rule of thumb: violating a ★★★★★ proven pattern or a security/permission requirement is at least **Critical**; leaving mock data or a missing state is **Major**; a repeated named Common Mistake escalates one level each time it recurs.
