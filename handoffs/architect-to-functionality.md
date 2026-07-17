# Handoff: Architect → Functionality Engineer

Produced by the Architect once the design passes DESIGN_REVIEW. The Engineer should be able to build entirely from this without guessing.

## Handoff contents (fill one per feature/screen)
- **Feature / screen:**
- **Requirement IDs:**
- **User purpose:** (why this exists for the user)
- **Approved design:** (link to the `DESIGN.md` section + mock shell location)
- **Components:** (which reusable components; link to component specs)
- **Required data:** (what data each screen needs; shape/fields)
- **User interactions:** (every action and its expected result)
- **Navigation behavior:** (entry points, exit points, transitions)
- **States — all specified:**
  - Loading:
  - Empty:
  - Error:
  - Success:
  - Disabled:
  - Permission:
- **Mobile behavior:**
- **Desktop behavior:**
- **Accessibility expectations:**
- **Analytics expectations (if any):**
- **Functionality still required:** (what the Engineer must wire — the whole point)
- **Important design decisions:** (and why, so they aren't undone)
- **Related files:**
- **Manager approval evidence:** (link to the DESIGN_REVIEW entry in `MANAGER_REVIEWS.md`)

## Rule
If any field is blank or vague, the design is not ready to hand off — the Engineer will have to guess, which violates the Architect's Definition of Complete Design.
