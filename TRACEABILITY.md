# TRACEABILITY

Every requirement must be traceable from design → implementation → tests → release. This is the Manager's master ledger for answering "is this actually ready?"

The live, per-project version lives at `projects/current/TRACEABILITY.md`. This root file defines the format and rules.

---

## The traceability table

| Requirement | Design | Implementation | Tests | Status |
|---|---|---|---|---|
| AUTH-001 | DESIGN-AUTH-01 | `src/auth/signup` | TEST-AUTH-001–005 | Passed |
| AUTH-002 | DESIGN-AUTH-02 | `src/auth/login` | TEST-AUTH-006–010 | Passed |
| PROJ-001 | DESIGN-PROJ-01 | `src/projects/create` | TEST-PROJ-001–008 | Testing |

## Status values

`Not designed` → `Designed` → `Design approved` → `Implemented` → `Implementation approved` → `Testing` → `Passed` → `Released`
(or `Blocked` / `Failed` with a link to the blocking bug or review)

---

## The seven questions the Manager answers from this table

For any requirement, the Manager must be able to answer:
1. Was the requirement **designed**?
2. Was the design **approved**?
3. Was it **implemented**?
4. Was the implementation **reviewed**?
5. Was it **tested**?
6. Did it **pass**?
7. Is it **ready for release**?

If any answer is "no" or "unknown," the requirement is not releasable.

---

## Rules

- Every requirement ID from `REQUIREMENTS.md` must appear in the table — no orphans.
- Every row must reference a real design section, a real code location, and real test IDs before it can reach `Passed`.
- The Manager updates this table at every stage transition.
- A release cannot be approved while any required (non-deferred) requirement is below `Passed`.
