# Workflow: New Feature (existing project)

Adding a feature to an app that already exists. Same lifecycle as a new project, scoped to the feature.

## 1. Brief the feature
- Add the new requirement(s) to `REQUIREMENTS.md` with fresh IDs.
- Capture the feature intent (`templates/feature-spec-template.md`).
- Manager confirms scope → DESIGNING (for this feature).

## 2. Design (Architect)
- Design only the new/changed screens, states, and flows — but ensure they stay consistent with existing screens and the Foundation.
- Note any impact on existing components in `DESIGN.md`.
- → DESIGN_REVIEW.

## 3. Design review (Manager)
- `criteria/design-review-criteria.md`, plus: does the feature stay consistent with the existing app and not break established patterns? Pass → build.

## 4. Build (Functionality Engineer)
- Implement per `DEFINITION_OF_DONE.md`. **Preserve working systems** — smallest safe change. Update any dependent systems (see `memory/system-relationships.md`).
- → IMPLEMENTATION_REVIEW.

## 5. Review → Test → Fix → Retest → Final review → Release
- Same as the standard lifecycle. Regression testing matters more here: the Tester must confirm the new feature didn't break existing behavior (`testing/regression-suite.md`).

## Key difference from a new project
- **Regression is mandatory and central.** A feature that works but breaks something existing is a failed feature.
- Update `TRACEABILITY.md` for the new IDs and re-verify any rows the change touched.
