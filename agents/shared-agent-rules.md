# Shared Agent Rules

These rules bind **every** role. Read them before your role file. If your role file ever seems to conflict with these, these win — except that the Manager alone controls stage and approval.

---

## Before you act
1. **Read required files first.** Follow the reading order in `START_HERE.md`. Always read `projects/current/STATUS.md` before touching a project.
2. **Use existing standards.** The Foundation (`app-build-set/`) is law. Do not invent a competing convention when one already exists there.
3. **Do not duplicate existing rules or content.** Reference the Foundation; don't copy it.

## While you work
4. **Do not silently change requirements.** A requirement changes only through the User (product intent) or a documented Decision Record.
5. **Do not approve your own work.** No exceptions. Approval is always another role's job — final approval is always the Manager's.
6. **Do not redesign or rewrite another agent's approved work** without a documented change request that the Manager accepts.
7. **Preserve working systems.** Don't change something that works without a clear, recorded reason.
8. **Make the smallest safe change** that satisfies the requirement. Avoid opportunistic refactors mid-task.
9. **Record assumptions** the moment you make one. If you had to guess, write down what you guessed and why.
10. **Report blockers instead of guessing.** If you're blocked, say so and stop — don't paper over it.

## When you finish
11. **Provide evidence for completed work.** "It works" is not evidence. Show the check, the output, the test, the screenshot-equivalent.
12. **Do not hide test failures, unfinished work, or known limitations.** Surface them explicitly. Hiding them is the most serious process violation in this system.
13. **Update project documentation** in your lane (`DESIGN.md`, `FUNCTIONALITY.md`, `TEST_RESULTS.md`, etc.).
14. **Do not add permanent lessons to `memory/` without Manager approval.**

## When to involve the User
15. **Ask the User only when a decision materially affects product intent** — ambiguous/missing requirements, scope-changing design shifts, tradeoffs touching cost/data/privacy/user promises, or release-blocking accept/reject calls. Routine technical choices are recorded as Decision Records, not escalated.

---

## The one-line test for every action
> "Am I staying inside my lane, changing the smallest necessary thing, leaving evidence, and hiding nothing?"

If not, stop and reconsider.
