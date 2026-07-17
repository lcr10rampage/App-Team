# Quality Gates

The four hard gates. Work cannot pass a gate until every item is true, with evidence. These summarize and enforce the `criteria/` files — the Manager owns every gate decision.

## Design gate  (leaving DESIGNING → DESIGN_REVIEW pass)
- [ ] Every required screen exists
- [ ] Every critical user flow is documented
- [ ] Navigation complete; components identified
- [ ] Mobile + desktop behavior defined
- [ ] Loading / empty / error / success / permission states included
- [ ] `criteria/design-review-criteria.md` passes

## Functionality gate  (leaving FUNCTIONALITY_BUILD → TESTING)
- [ ] Approved controls connected; real data replaces mock
- [ ] Validation, loading, error handling in place
- [ ] Auth works; permissions enforced server-side; data persists
- [ ] Required automated checks pass (`ci-requirements.md`)
- [ ] Implementation matches approved design
- [ ] `criteria/functionality-review-criteria.md` passes

## Testing gate  (leaving TESTING/RETESTING → FINAL_REVIEW)
- [ ] Critical test cases pass
- [ ] No Blocker or Critical bugs open
- [ ] Major bugs fixed or explicitly accepted
- [ ] Regression passes; retest complete; evidence recorded
- [ ] `criteria/testing-criteria.md` met

## Release gate  (FINAL_REVIEW → RELEASE_APPROVED)
- [ ] Required features complete; traceability current
- [ ] Design + implementation approved; required tests pass
- [ ] Known limitations documented; security concerns resolved
- [ ] Release smoke tests pass; docs updated
- [ ] `criteria/release-criteria.md` passes; Manager gives final approval
