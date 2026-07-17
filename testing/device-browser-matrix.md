# Device / Browser Matrix

Where the app must be verified. The foundation stack targets modern browsers and a responsive mobile+desktop layout (`app-build-set/app-system-guide.md` responsive rules — one-column-by-default, fluid scaling).

## Default matrix
| Surface | Viewport | Priority |
|---|---|---|
| Desktop — Chrome | ~1440×900 | Critical |
| Desktop — Firefox | ~1440×900 | High |
| Desktop — Safari | ~1440×900 | Medium (if Mac users targeted) |
| Mobile — Chrome/Android | ~390×844 | Critical |
| Mobile — Safari/iOS | ~390×844 | Critical (if iOS targeted) |
| Tablet | ~820×1180 | Low unless required |

## Rules
- Every critical user flow is verified on at least one desktop and one mobile surface.
- Layout checks: no overflow (`document.body.scrollWidth <= window.innerWidth`), no overlap, tap targets usable on mobile.
- Adjust this matrix per project's actual target audience (from `PROJECT_BRIEF.md`).
