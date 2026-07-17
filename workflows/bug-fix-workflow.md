# Workflow: Bug Fix

## 1. Report (Tester)
- File the bug with `templates/bug-report-template.md` into `projects/current/BUGS.md`.
- Assign **severity** (Blocker/Critical/Major/Moderate/Minor) and provide reproduction steps + evidence.
- Hand off to the Functionality Engineer (`handoffs/tester-to-functionality.md`).

## 2. Triage (Manager)
- Manager confirms severity and priority. Blocker/Critical jump the queue.
- Manager assigns to the Functionality Engineer and records it.

## 3. Fix (Functionality Engineer)
- Reproduce first. Find the **root cause**, not just the symptom (check `memory/common-mistakes.md` and `app-build-set/EngineeringMemory.md` — this bug may be a known pattern).
- Make the **smallest safe change**. Don't opportunistically refactor.
- Record the fix (files changed, root cause, why the fix is correct) in `FUNCTIONALITY.md` and reference the Bug ID.
- Hand back to the Tester.

## 4. Retest (Tester)
- Re-run the failed case **from scratch** — never assume the fix works.
- Run the regression suite around the fix area — a fix that breaks something else is not a fix.
- Update `BUGS.md` (status → Verified/Reopened) and `TEST_RESULTS.md`.

## 5. Close (Manager)
- Manager confirms the bug is resolved with evidence and closes it.
- If the bug revealed a reusable lesson, the Manager approves an entry into `memory/common-mistakes.md` or `memory/prompt-lessons.md`.

## Rules
- The Tester never fixes. The Engineer never closes its own bug as verified. The Manager confirms closure.
- Every bug fix carries evidence of the retest, not just the code change.
