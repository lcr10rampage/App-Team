# Commit Rules

Consistent, meaningful commits. Explain **what changed and why**.

## Format (Conventional Commits)
```
<type>(<scope>): <summary>
```

## Types
| Type | Use for |
|---|---|
| `feat` | new functionality |
| `fix` | bug fix |
| `design` | Architect design/UI work |
| `test` | tests added/changed |
| `docs` | documentation / workflow docs |
| `refactor` | internal change, no behavior change |
| `chore` | tooling, deps, config |

## Examples
```
feat(projects): connect project creation form to the database
fix(auth): block unauthenticated access to the dashboard
design(settings): add mobile account layout
test(projects): add deletion regression tests
docs(workflow): update Manager approval process
```

## Rules
- One logical change per commit; smallest safe change (matches `agents/shared-agent-rules.md`).
- The summary says what; the body (when needed) says why.
- Reference requirement IDs or Bug IDs where relevant (`feat(auth): AUTH-001 email signup`).
- Never commit secrets. Never commit commented-out dead code as "just in case."
