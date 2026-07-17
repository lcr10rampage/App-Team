# CI Requirements

Automated checks that should pass before the functionality gate and again at release. **Inspect the actual project stack before wiring tools** — don't add anything that conflicts with it. The foundation default stack (`app-build-set/app-system-guide.md` §1) is Vite + React + TypeScript (frontend) and FastAPI + SQLite (backend).

## Checks (enable those that fit the stack)
| Check | Frontend (Vite/React/TS) | Backend (FastAPI/Python) |
|---|---|---|
| Format | Prettier | Black / Ruff format |
| Lint | ESLint | Ruff / Flake8 |
| Type check | `tsc --noEmit` | mypy (optional) |
| Unit tests | Vitest / Jest | pytest |
| Integration tests | Playwright / API tests | pytest + httpx |
| Build success | `vite build` | import check: `python -c "from main import app"` |
| Security scan | `npm audit` | `pip-audit` |
| Dependency check | outdated/audit | outdated/audit |
| DB migration validation | — | apply migrations to a scratch DB; confirm schema |

## Rules
- The **build must succeed** and **type check must be clean** before IMPLEMENTATION_REVIEW.
- A failing required check is a gate failure — not a warning to ignore.
- Foundation reminder: a backend import check runs `create_all` at import (creates the DB early) — account for this when validating migrations.
- If the project has no CI runner yet, these run manually and their results are the "commands run + results" evidence in the functionality→manager handoff.
