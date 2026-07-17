# Branch Rules

Git branching for team work. Adjust to the project's actual host/flow; inspect before enforcing.

## Structure
```
main       — always releasable; only the Manager approves merges here
develop    — integration branch (optional for small projects)
design/*   — Architect design work
feature/*  — Functionality Engineer feature work
fix/*      — bug fixes
test/*     — Tester scaffolding / test additions
```

## Who does what
| Branch | Created by | Merged by | Requires |
|---|---|---|---|
| `design/*` | Architect | Manager | DESIGN_REVIEW pass |
| `feature/*` | Functionality Engineer | Manager | IMPLEMENTATION_REVIEW pass + CI green |
| `fix/*` | Functionality Engineer | Manager | bug verified by Tester |
| `test/*` | Tester | Manager | — |

## Rules
- Nothing merges to `main` without Manager approval and passing checks.
- Branch names describe the work: `feature/project-creation`, `fix/auth-dashboard-guard`.
- Delete branches after merge.
- **Emergency fix:** `fix/*` off `main`, minimal change, still Tester-verified and Manager-approved before merge — the process shrinks, it doesn't disappear.
- For a solo/simple project, `main` + short-lived `feature/*` branches is enough; keep the review gates regardless of branch count.
