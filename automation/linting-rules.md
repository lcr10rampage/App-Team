# Linting Rules

Keep the codebase consistent. Use the stack's standard tools — do not invent a competing config that fights the framework defaults.

## Frontend (Vite + React + TypeScript)
- **ESLint** with the TypeScript + React plugins; start from the Vite React-TS template defaults.
- **Prettier** for formatting; let Prettier own formatting, ESLint own correctness (avoid overlapping/ fighting rules).
- TypeScript `strict: true` (matches the foundation frontend setup).
- No unused vars, no implicit `any`, exhaustive-deps on hooks (the foundation documents real bugs from stale hook closures and StrictMode — the linter helps catch them).

## Backend (FastAPI + Python)
- **Ruff** (lint + format) or Black + Flake8. Pick one and keep it.
- Follow PEP 8; type-hint public functions.

## Rules
- Lint + format must be clean before IMPLEMENTATION_REVIEW (`automation/ci-requirements.md`).
- Formatting is not a code-review discussion — the formatter decides. Reviews focus on correctness and criteria.
- If the project already has a lint config, use it; don't replace it without a Decision Record.
