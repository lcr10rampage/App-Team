# Regression Suite

Checks that must pass after **any** change, to prove nothing previously working broke. Grows over the life of a project — every fixed bug and shipped feature adds a regression check here.

## Rule
A feature or fix is not done until the regression suite passes. "It works" for the new thing is not enough — the old things must still work.

## Format
```
### REG-<id> — <what it protects>
- Origin: (feature shipped / bug fixed — link)
- Steps:
- Expected:
- Last result: Pass / Fail / Not run
```

## Standing regressions (apply to most apps)
### REG-AUTH — auth still works
- Steps: sign in, refresh (session persists), sign out (session cleared), new tab requires sign-in.
- Expected: matches the foundation sessionStorage-token model.
### REG-ISOLATION — account isolation holds
- Steps: as user A create data; sign in as user B.
- Expected: B never sees A's data.

## Project regressions
(Add one per shipped feature / fixed bug.)
