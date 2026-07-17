# Handoff: Manager → Tester

Produced by the Manager once IMPLEMENTATION_REVIEW passes and the feature enters TESTING.

## Handoff contents
- **Feature under test:**
- **Approved requirements:** (IDs + acceptance criteria)
- **Approved design references:** (links to `DESIGN.md`)
- **Critical user flows:** (must-pass paths — see `testing/critical-user-flows.md`)
- **High-risk areas:** (from the Engineer's "areas requiring close review" + Manager judgment)
- **Required environments:** (browsers/devices — `testing/device-browser-matrix.md`)
- **Test accounts / test data:** (`testing/test-data-guidelines.md`)
- **Known accepted limitations:** (do not file these as bugs)
- **Regression areas:** (what existing behavior could this have touched)
- **Release-blocking conditions:** (what would make this un-shippable)

## Rule
The Tester stays independent — this handoff informs the test plan but does not constrain the Tester from testing beyond it. If the Tester sees risk the Manager didn't flag, test it anyway.
