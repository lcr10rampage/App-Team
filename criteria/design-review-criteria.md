# Design Review Criteria

Applied by the Manager at **DESIGN_REVIEW**. Judged against `app-build-set/app-system-guide.md` (§1–9, 17–19) and the Design Principles / UX Decisions in `app-build-set/EngineeringMemory.md`.

A design **passes** only when every item is satisfied. Any unmet item is a finding with a severity.

## Requirement coverage
- [ ] Every requirement ID in `REQUIREMENTS.md` maps to at least one screen or flow.
- [ ] No screen exists that serves no requirement (or it's justified).

## Completeness
- [ ] Every required screen is designed (screen inventory complete).
- [ ] App map shows how all screens connect.
- [ ] All primary user flows are documented end to end.
- [ ] Navigation is complete and understandable.
- [ ] Reusable components are identified and specced (not re-invented per screen).

## Every state is specified (not just the happy path)
- [ ] Loading state
- [ ] Empty state
- [ ] Error state
- [ ] Success state
- [ ] Disabled state (where applicable)
- [ ] Permission state (what each role/permission sees)

## Responsive
- [ ] Mobile behavior defined per screen.
- [ ] Desktop behavior defined per screen.

## Consistency with the Foundation
- [ ] Uses the color palette from app-system-guide §1 (or an accepted Decision Record justifies deviation).
- [ ] Uses established patterns (page-entrance `row(delay)`, `createPortal` modals, z-index stack, one-column-by-default layout).
- [ ] No design that contradicts a named EngineeringMemory Design Principle (e.g. show/hide toggles stack vertically; `—` not `0%` for no-data).

## Clarity for handoff
- [ ] Every element's intended behavior is documented — the Engineer won't have to guess.
- [ ] Required data per screen is listed.
- [ ] Design decisions are recorded.

## Decision
Pass → advance to FUNCTIONALITY_BUILD. Otherwise → Returned to Architect with findings + required corrections.
