# Requirements — <Product Name>

> Copy to `projects/current/REQUIREMENTS.md`. Every requirement gets a **stable ID** (e.g. AUTH-001, USER-001, PROJ-001, BILL-001, NOTIF-001). IDs never change or get reused — they are the backbone of `TRACEABILITY.md`.

## Requirement format

### <ID> — <short title>
- **Description:** (what the system must do)
- **Priority:** Must / Should / Could / Won't (this release)
- **User value:** (why it matters to the user)
- **Acceptance criteria:** (testable conditions — the Tester's pass/fail bar)
  - [ ] …
  - [ ] …
- **Dependencies:** (other requirement IDs)
- **Status:** Not designed / Designed / Approved / Implemented / Testing / Passed / Released / Blocked

---

## Example

### AUTH-001 — Email + password signup
- **Description:** A new user can create an account with email and password.
- **Priority:** Must
- **User value:** Gets the user into the product.
- **Acceptance criteria:**
  - [ ] Valid email + password (≥8 chars) creates an account and signs the user in.
  - [ ] Duplicate email is rejected with a clear error.
  - [ ] Password is stored hashed (bcrypt) — never plaintext.
- **Dependencies:** none
- **Status:** Not designed
