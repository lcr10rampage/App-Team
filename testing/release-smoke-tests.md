# Release Smoke Tests

The minimal must-pass set the Manager requests right before release (`workflows/release-workflow.md`). Any failure blocks release. Fast to run; covers the arteries, not every case.

## Standing smoke checks
- [ ] App loads without console errors.
- [ ] Sign up / sign in / sign out all work; session persists on refresh; new tab requires sign-in.
- [ ] The core create → read → update → delete path for the primary object works and persists.
- [ ] Permissions: a signed-out user cannot reach protected pages/data; user A cannot see user B's data.
- [ ] Each primary screen renders on desktop and mobile without overflow/overlap.
- [ ] The primary async action shows loading, then success or a real error (not a silent failure).
- [ ] No mock/placeholder data visible anywhere.

## Per-project smoke checks
(Add the 3–7 flows that would most embarrass the product if broken at launch — pulled from `critical-user-flows.md`.)

## Rule
Smoke tests run against a release-like state (real build, seeded like production). Green here is necessary but not sufficient — full testing + the release gate still apply.
