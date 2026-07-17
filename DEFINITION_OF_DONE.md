# DEFINITION OF DONE

A feature is **not done** because its main action works. Done means every item below is true and has evidence. The Manager verifies this list before any feature is marked complete.

---

## Requirements & design
- [ ] Requirement IDs are identified and listed
- [ ] The Architect's design for these requirements is approved
- [ ] The implementation matches the approved design (no undocumented redesign)

## Data & logic
- [ ] Real data is connected (no mock/placeholder data remaining)
- [ ] Validation is implemented on all forms and actions
- [ ] Data persists correctly (saves, loads, survives refresh)
- [ ] Business logic behaves per the requirements' acceptance criteria

## Every interaction state (not just the happy path)
- [ ] Loading state works
- [ ] Empty state works
- [ ] Error state works
- [ ] Success state works
- [ ] Disabled state works where required

## Security & access
- [ ] Permissions are enforced (server-side, not just hidden in UI)
- [ ] Authentication is handled correctly
- [ ] Data is scoped by user where the app is multi-user (see EngineeringMemory: "Scope data by user from day one")

## Responsive
- [ ] Mobile behavior works
- [ ] Desktop behavior works

## Quality
- [ ] Accessibility basics are met (keyboard, labels, contrast)
- [ ] Automated checks pass (`automation/ci-requirements.md`)
- [ ] Manual testing is completed with evidence
- [ ] Regression checks pass (`testing/regression-suite.md`)

## Record-keeping
- [ ] Documentation is updated (`FUNCTIONALITY.md`, `TRACEABILITY.md`)
- [ ] Known limitations are recorded (not hidden)
- [ ] The Manager approves the feature

---

**Foundation cross-check:** the state-completeness items above exist because of repeated real lessons in `app-build-set/EngineeringMemory.md` — e.g. "Every display value that gets frozen must have an unconditional unfreeze path," "Only building the success state," and "Validate end-to-end, not just locally." A feature that only implements the success state is a known, named mistake — not done.
