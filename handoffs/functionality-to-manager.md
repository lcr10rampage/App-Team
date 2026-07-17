# Handoff: Functionality Engineer → Manager

Produced by the Engineer when requesting IMPLEMENTATION_REVIEW. Provides everything the Manager needs to review against `criteria/functionality-review-criteria.md` and `DEFINITION_OF_DONE.md`.

## Handoff contents
- **Feature implemented:**
- **Requirement IDs completed:**
- **Files changed:** (list)
- **Database changes:** (schema, migrations run)
- **API changes:** (endpoints added/changed)
- **Authentication changes:**
- **Permission changes:** (and where enforced — server-side)
- **Integrations added:**
- **Validation added:** (which forms/actions)
- **Error handling:** (which async paths, how failures surface)
- **States implemented:** loading / empty / error / success / disabled — with how each was verified
- **Tests written:** (which, and how to run them)
- **Commands run:** (build, lint, typecheck, tests — with results)
- **Manual checks completed:** (with evidence)
- **Design differences:** (any accepted design-change requests; link `DECISIONS.md`)
- **Known limitations:** (do not hide any)
- **Risks:**
- **Areas requiring close review:** (where you're least sure)

## Rule
"Done" claims require evidence attached. Missing evidence = not reviewable = returned.
