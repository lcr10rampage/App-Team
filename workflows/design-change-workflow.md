# Workflow: Design Change

Used when the approved design turns out to be impractical, unsafe, or incompatible during build. **The Functionality Engineer never silently changes an approved design.**

## 1. Raise (Functionality Engineer)
- File a design-change request (`templates/design-change-request-template.md`) with: original approved design · the technical problem · why it matters · recommended change · alternatives considered · user impact · technical impact.
- Do **not** implement the change yet. Pause the affected part; keep building what's unaffected.

## 2. Decide (Manager)
- Manager evaluates against requirements + Foundation criteria.
- Options:
  - **Accept** → the change becomes the new approved design.
  - **Accept with modification** → Manager specifies the acceptable variant.
  - **Reject** → the original design stands; Engineer must find another implementation path.
  - **Escalate to User** → if it materially affects product intent/scope.

## 3. Update design (Architect, if accepted)
- The Architect updates `DESIGN.md` to reflect the accepted change so design remains the single source of truth (design and code never silently diverge).
- If the change is minor and purely technical, the Manager may record it directly without re-engaging the Architect — Manager's call.

## 4. Record (Manager)
- Log the outcome as a Decision Record in `projects/current/DECISIONS.md` (ID, context, options, decision, reason, consequences, approved-by, date).
- Update `TRACEABILITY.md` if the change alters how a requirement is satisfied.

## 5. Resume (Functionality Engineer)
- Implement per the now-approved design.

## Rule
Approved design and shipped code must never diverge without an accepted, recorded design-change request.
