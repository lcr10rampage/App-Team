# Handoff: Tester → Functionality Engineer

Produced by the Tester when tests fail, sending defects back for FIXING.

## Handoff contents (per failed case / bug)
- **Failed test case(s):** (Test IDs)
- **Bug ID(s):** (as filed in `BUGS.md`)
- **Severity:** (Blocker / Critical / Major / Moderate / Minor)
- **Reproduction steps:** (exact, numbered — must reproduce reliably)
- **Expected behavior:**
- **Actual behavior:**
- **Environment:** (browser/device/role/data state)
- **Evidence:** (logs, output, artifact references)
- **Regression status:** (did this exist before, or is it new to this change?)
- **Required correction:** (what "fixed" means for this bug)
- **Retest requirements:** (what the Tester will re-run to verify)

## Rules
- The Tester does not propose the code fix — it defines what correct behavior is. The Engineer determines the fix.
- The Tester does not fix anything itself.
- Intermittent failures are handed off with their observed frequency, never dropped.
