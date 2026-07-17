# Common Mistakes

> **Seeded source:** the extensive *Common Mistakes* section of `app-build-set/EngineeringMemory.md` — the Tester and Manager treat that as the primary "where bugs hide in this stack" checklist (only-success-state, missing permission checks, mock data left in, StrictMode double-fire, `res.ok` not checked, stale-timestamp tie-breaks, `create_all` won't ALTER, wrong authFetch import, delta-badge positioning, gradient animation, DOM-class not reactive, etc.).
>
> This file holds **new** mistakes surfaced by the team process. Manager-approved. When a mistake recurs, escalate its severity.

## Recurring high-level mistakes to always watch for (from the spec)
- Only building the success state (forgetting empty/loading/error/disabled).
- Forgetting permission checks (or enforcing them only in the UI, not server-side).
- Using mock data after implementation should have replaced it.
- Changing an approved design during coding without a design-change request.
- Testing only the main user flow.
- Failing to update dependent systems (see `memory/system-relationships.md`).
- Duplicating an existing component instead of reusing it.
- Hiding known limitations or failed tests.

## Entry format
```
### <Mistake>
- Symptoms:
- Cause:
- Prevention:
- Detection method:
- Projects where it occurred:
- Confidence: ★★★★★
```

## Entries
(Start empty for team-surfaced mistakes; the foundation holds the seeded set.)
