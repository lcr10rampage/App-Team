# System Relationships

> How major systems interact — so that a change in one place reminds the Engineer and Tester what else it touches. "Failing to update dependent systems" is a named Common Mistake. Consult this before building or fixing anything cross-cutting.

## Common relationships to check
| System A | System B | The relationship / what to remember |
|---|---|---|
| Authentication | Permissions/roles | A valid session is not authorization — permissions must be enforced server-side per request (foundation: shared `current_user` dep). |
| Database | UI state | Optimistic UI updates must reconcile with persisted data; a successful action isn't done until it survives refresh. |
| Notifications | User preferences | Don't notify against the user's stated frequency/quiet-hours; grouping and urgency come from prefs. |
| File uploads | Storage policy | Size/type limits, where files live, and cleanup on delete must all agree. |
| Billing | Account access | Plan/subscription state gates features; suspension must block login AND every API call AND filter public listings (foundation: full-stack suspension pattern). |
| Caching | Data freshness | Cached values must invalidate on write, or the UI shows stale data. |
| Background jobs | User-visible status | Long/async work needs a status the UI can reflect; don't leave the user guessing. |
| Multi-user data | Every data table | Scope by `user_id` from day one (foundation Design Principle) — account isolation is cross-cutting. |

## Project-specific relationships
(Add entries as the current project reveals its own cross-system dependencies.)
