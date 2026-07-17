# Critical User Flows

The must-pass paths for the current project. If any of these fail, the release is blocked. The Tester keeps this current per project; the Manager confirms the list at DESIGN_REVIEW.

## Format
```
### FLOW-<id> — <name>
- Requirement IDs:
- Steps (as a user):
- Expected outcome:
- Priority: Critical
- Last result: Pass / Fail / Not run
```

## Example
### FLOW-001 — Sign up and reach the dashboard
- Requirement IDs: AUTH-001
- Steps: open app → sign up with valid email + password → land on dashboard signed in.
- Expected outcome: account created, session established, dashboard shows the new user's (empty) data.
- Priority: Critical
- Last result: Not run

## Flows for the current project
(Fill in from `projects/current/REQUIREMENTS.md` + approved `DESIGN.md`.)
