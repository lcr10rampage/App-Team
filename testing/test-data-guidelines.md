# Test Data Guidelines

How to create and manage data and accounts for testing.

## Accounts
- Maintain distinct test accounts per role (e.g. regular user, admin, second user for isolation tests).
- Keep at least **two** normal-user accounts so account-isolation can be tested (user A must never see user B's data).

## Seeding
- **Run the seed script before testing login.** (Foundation lesson: `create_all` builds empty tables; a fresh DB has no accounts, so every login fails until seeded — `cd backend && python seed.py`.)
- Seed enough data to exercise empty, single, and many states — the empty state is a required test, so also have a way to test a truly empty account.

## Data hygiene
- Use obviously-fake data (e.g. `test+a@example.com`) so it's never confused with real records.
- Reset or recreate the DB between runs when schema changed (dev: delete the SQLite file and re-seed — foundation guidance).
- Never test against production data.

## Boundaries to prepare
- Empty, minimal, and large datasets (e.g. a 1-item list and a 150-item list) to catch pagination/limit bugs.
- Values at boundaries (0, 1, max length, over-max) for every validated field.
