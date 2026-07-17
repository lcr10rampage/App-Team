# Engineering Memory

Reusable engineering knowledge extracted from building the Bible Tracker app. Not project documentation — these lessons apply to any future project in this stack. Maintained per the system defined in `engineering-knowledge-prompt.md`.

---

## Proven Patterns

### `row(delay)` — Framer Motion page entrance
```tsx
const row = (delay: number) => ({
  initial: { opacity: 0, y: 22 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.55, delay, ease: [0.2, 0, 0.3, 1] },
})
// Usage: <motion.div {...row(0.18)}>
```
**Why it works:** 0.55s at this easing feels snappy without being jarring. The `y: 22` lift is just enough to read as motion without being dramatic. The easing `[0.2, 0, 0.3, 1]` has a fast start and gentle finish — reads as natural.

**Stagger order:** bottom card gets `row(0)`, top card gets the highest delay (`0.18` per row, so 5 rows = 0, 0.18, 0.36, 0.54, 0.72). Bottom-to-top builds feel like the page is assembling itself upward. Top-to-bottom feels like it's falling apart.

**Where it applies:** every page load, every panel entrance in React + Framer Motion projects.

**Confidence:** ★★★★★

---

### `createPortal` to `document.body` for modals and overlays
```tsx
import { createPortal } from 'react-dom'
{isOpen && createPortal(<div className="fixed inset-0 z-[9999]">...</div>, document.body)}
```
**Why it works:** portals escape the DOM tree, so the modal's `fixed` positioning is relative to the viewport, not a parent with `overflow: hidden` or `transform`. No z-index fights with the page layout.

**Where it applies:** every modal, confirmation dialog, full-screen animation iframe, milestone banner — anything that needs to float above everything else.

**Confidence:** ★★★★★

---

### Full-screen animations as standalone HTML files in `public/`
- Live in `frontend/public/` — Vite serves them as static files at `/animation-name.html`
- Pure HTML + CSS + JS, no React, no bundler
- Embedded in React via `<iframe src="/animation-name.html?param=value" className="w-full h-full border-0" />`
- Communication back to React: `window.parent.postMessage({ type: 'eventName' }, '*')`
- React listens: `window.addEventListener('message', onMessage)`

**Why it works:** completely decoupled from the React build. Fast to iterate — edit the file, browser reloads. No import errors, no TS compile step. Canvas-based drawing is straightforward in vanilla JS.

**URL params pattern:** pass data into the animation via query string (`?book=Genesis&dark=1`). Read with `new URLSearchParams(location.search)`.

**Confidence:** ★★★★★

---

### `setTimeout(tick, 16)` instead of `requestAnimationFrame` for animations
```js
function animateRaw(dur, onFrame) {
  return new Promise(resolve => {
    const start = performance.now()
    function tick() {
      const t = Math.min((performance.now() - start) / dur, 1)
      onFrame(t)
      if (t < 1) setTimeout(tick, 16); else resolve()
    }
    setTimeout(tick, 16)
  })
}
```
**Why it works:** `requestAnimationFrame` is throttled or paused when the tab is not in focus (background tabs, preview panels). `setTimeout(tick, 16)` ≈ 60fps and keeps running regardless of tab focus.

**Where it applies:** all standalone HTML animation files. In React with Framer Motion, rAF is fine because Framer handles it internally.

**Confidence:** ★★★★★

---

### CSS transition reflow fix
```js
element.style.transition = 'opacity 0.6s ease'
void element.offsetHeight   // forces reflow — DO NOT REMOVE
element.style.opacity = '1'
```
**Why it works:** when you set `transition` and the new value in the same JS frame, the browser batches them and sees no "before" state, so it skips the animation entirely. Forcing a reflow (`offsetHeight`, `getBoundingClientRect()`) flushes the style update and gives the browser a real start state to animate from.

**Where it applies:** every CSS transition triggered from JS. Required for opacity, transform, clip-path — any property you're transitioning via `element.style`.

**Confidence:** ★★★★★

---

### `authFetch` — authenticated API helper
```ts
export function authFetch(input: string, init: RequestInit = {}): Promise<Response> {
  const token = sessionStorage.getItem('token')
  return fetch(input, {
    ...init,
    headers: {
      ...(init.headers ?? {}),
      ...(token ? { Authorization: `Bearer ${token}` } : {}),
      ...(!init.headers || !(init.headers as Record<string, string>)['Content-Type']
        ? { 'Content-Type': 'application/json' }
        : {}),
    },
  })
}
```
**Why it works:** single place to inject the auth token. All API calls use it — no risk of forgetting the header on one endpoint. Content-Type defaulting means callers don't have to set it.

**Where it applies:** any project with JWT auth. Replace every `fetch('/api/...')` with `authFetch('/api/...')`.

**Confidence:** ★★★★☆

---

### Shared `current_user` FastAPI dependency
```python
# deps.py
def current_user(authorization: Optional[str] = Header(None), db: Session = Depends(get_db)) -> User:
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Not authenticated")
    token = authorization.split(" ", 1)[1]
    payload = jwt.decode(token, settings.secret_key, algorithms=["HS256"])
    user = db.query(User).filter(User.id == payload.get("sub")).first()
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user
```
**Why it works:** one place to define auth logic. Every router imports `Depends(current_user)` — no duplicated token parsing. If auth logic changes, it changes in one file.

**Where it applies:** any FastAPI project with JWT auth.

**Confidence:** ★★★★★

---

### `sessionStorage` for auth tokens (per-tab sessions)
- Token in `sessionStorage` → cleared when tab closes → each new tab requires sign-in
- User email / non-sensitive preferences in `localStorage` → persists across tabs
- `RequireAuth` component checks `sessionStorage.getItem('token')` and redirects to `/login` if missing

**Why it works:** sessionStorage is automatically scoped to the tab. No manual cleanup needed on tab close. Gives the per-device/per-session security feel without building complex session management.

**Confidence:** ★★★★☆

---

### `postMessage` iframe → React dismissal pattern
```js
// In the HTML animation file:
window.parent.postMessage({ type: 'bookCompleteAnimationDone' }, '*')

// In React:
useEffect(() => {
  function onMessage(e: MessageEvent) {
    if (e.data?.type === 'bookCompleteAnimationDone') setAnimBook(null)
  }
  window.addEventListener('message', onMessage)
  return () => window.removeEventListener('message', onMessage)
}, [])
```
**Why it works:** clean decoupling. The animation doesn't need to know anything about React. React doesn't need to poll or time out. Works even if the animation is paused or delayed.

**Confidence:** ★★★★★

---

### `bcrypt` directly, never `passlib`
```python
import bcrypt

def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode(), hashed.encode())
```
**Why:** `passlib`'s `CryptContext` reads `bcrypt.__about__` to detect the version. Newer versions of `bcrypt` removed `__about__`, causing `AttributeError` at runtime. Direct `bcrypt` calls have no such dependency.

**Confidence:** ★★★★★

---

### Bottom-up fill animation via CSS `height` transition on a layered div

**The wrong approach:** `background: linear-gradient(to top, amber X%, gray X%)` with `transition: background`. CSS cannot interpolate between two different gradient values — `transition: background` silently does nothing on gradients.

**The correct approach:** layer an absolutely-positioned amber div inside the button, animate its `height` from 0 to target %:

```tsx
// Button: position:relative, background: emptyColor, overflow:hidden
// Fill div: position:absolute, bottom:0, left:0, right:0, height:`${fillPct}%`
//           transition: `height 900ms cubic-bezier(0,0,0.2,1) ${staggerMs}ms`
// Text span: position:relative, z-index:1
```

Control fill start/end with a `fillsReady: boolean` state: false = height 0, true = height targetPct. Stagger via CSS `transition-delay` (not JS timeouts): `animIdx * 40ms`. Only partial books get `animIdx` ≥ 0.

**Why it works:** CSS `height` transitions are GPU-accelerated and fully declarative. Setting `fillsReady = true` in React triggers all spines to animate simultaneously, each with its own CSS delay — no JS loop or setInterval needed.

**Replay:** since navigation uses `<a href>` (full reloads), `fillsReady` resets to `false` on every mount. Replay is automatic — no explicit reset or key prop needed.

**Minimum visible fill:** when `rawPct > 0 && rawPct < 12`, use 12% so even 1 chapter of a 150-chapter book produces a visible fill on an 88px spine.

**Confidence:** ★★★★★

---

### SQLAlchemy bulk insert with `on_conflict_do_nothing()` for deduplication
```python
from sqlalchemy.dialects.sqlite import insert as sqlite_insert

stmt = sqlite_insert(MyModel).values([
    {"user_id": user_id, "field": value}
    for value in values
]).on_conflict_do_nothing()
db.execute(stmt)
db.commit()
```
**Why it works:** one round-trip to the DB instead of N SELECT + N INSERT calls. The unique constraint does the deduplication — no application-level existence check needed. For 150-chapter books, this is ~150× faster than the per-row pattern.
**Where it applies:** any time you're inserting a batch of records where some may already exist (chapter reads, tags, many-to-many joins). Import: `from sqlalchemy.dialects.sqlite import insert as sqlite_insert`.
**Note:** this is SQLite-specific. PostgreSQL uses the same API but imports from `sqlalchemy.dialects.postgresql`.
**Confidence:** ★★★★★

---

### Hiding scrollbars while keeping scroll behavior
```css
/* index.css */
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { scrollbar-width: none; }
```
Apply `no-scrollbar` alongside `overflow-y-auto` on any container where the scrollbar track looks out of place. Both rules are required: `scrollbar-width: none` covers Firefox, `::-webkit-scrollbar { display: none }` covers Chrome/Safari. The scroll behavior is unchanged — only the visible track/thumb is hidden.
**Where it applies:** small scrollable lists inside cards (reading log, archive section). Any `overflow-y-auto` container that feels cramped or where the scrollbar aesthetic breaks the design.
**Confidence:** ★★★★★

---

### Save `lastSeen` baseline immediately in the load function — not on unmount
```tsx
const lastSeen = parseInt(localStorage.getItem('lastSeenX') ?? '-1')
localStorage.setItem('lastSeenX', String(currentValue))  // save NOW
if (lastSeen < 0) return
const delta = currentValue - lastSeen
// animate...
```
**Why it works:** unmount cleanup (`return () => { ... }`) does not reliably fire on browser navigation (full page loads, tab close). Saving immediately after computing the delta ensures the baseline is always up to date — even if the user navigates away before cleanup runs.
**Why NOT on fetch arrival:** if you save on fetch arrival, the next visit sees `lastSeen === currentValue` and delta = 0 — no animation ever plays. The correct model is: save the baseline at the moment you've decided whether to animate, so the NEXT visit starts from the right place.
**Confidence:** ★★★★★

---

### PyJWT over `python-jose` for HS256 tokens
```python
import jwt  # PyJWT
token = jwt.encode({"sub": uid, "exp": expire}, secret, algorithm="HS256")
uid = jwt.decode(token, secret, algorithms=["HS256"]).get("sub")
```
**Why it works:** PyJWT does HS256 with the stdlib `hmac`/`hashlib` — no `cryptography` wheel to build. `python-jose[cryptography]` drags in `cryptography`, which can fail to install on brand-new Python versions (hit on Python 3.14). The encode/decode API is nearly identical to jose, so it is a drop-in.
**Where it applies:** any FastAPI/Flask project using symmetric (HS256) JWTs on a bleeding-edge interpreter. Use a secret ≥ 32 bytes to silence PyJWT's `InsecureKeyLengthWarning`.
**Confidence:** ★★★★★

---

### Fluid "scale the whole bar as one unit" via one clamp font-size + `em` children
Drive a header/toolbar off a single fluid font-size on the container, then size every child (logo, tab padding, gaps, icon buttons) in `em`. The whole thing scales proportionally as the viewport narrows and stays on one row — same look, just smaller — capping at the desktop size via the clamp max.
```tsx
const barStyle     = { fontSize: 'clamp(0.53rem, 2.35vw, 0.875rem)' } // container
const tabStyle     = { paddingInline: '0.66em', paddingBlock: '0.4em' }
const logoTextStyle= { fontSize: '1.22em' }
const iconBtnStyle = { width: '2.35em', height: '2.35em' }
// children use em → they all shrink together with the container's clamp
```
Keep icon glyphs (lucide `size={17}`) fixed px — they are small and do not drive overflow; the text width does. Add `whitespace-nowrap` to labels, a small `gap` between logo and nav, and `shrink-0` on the logo so nothing can overlap it.
**Why it works:** one clamp controls the entire scale; `em` makes padding/gaps/sizes track it, so proportions stay identical at every width. Beats per-element breakpoints (stepped, and you tune N values) and beats N independent clamps (they drift out of proportion).
**Where it applies:** any single-row nav/toolbar that must stay one row on mobile without a hamburger. Verify the *worst case* (most tabs / longest labels) fits, not the common case.
**Confidence:** ★★★★★

---

### Whole-app scale-UP on large viewports via root `font-size` clamp
The mobile-nav pattern above scales one component down; the inverse — making the *entire app* feel bigger on a large monitor — is one line, because Tailwind's text/padding/gap/rounded/width/height utilities are rem-based:
```css
html {
  font-size: clamp(16px, 14px + 0.22vw, 20px); /* unchanged at laptop widths, ~20px on 1440p+ */
}
```
Solve for the two endpoints you want (e.g. 16px at 768px viewport, 20px at 2560px) with simple linear interpolation: `slope = Δfont / Δviewport`, then `font = base + slope*100 * vw_units`. Keep px (not rem) in the clamp itself, or you get a chicken-and-egg reference.
**Why it works:** one change touches every rem-sized utility class app-wide — no per-component edits. `em`-based components (like the navbar above) are unaffected since `em` is relative to *local* computed font-size, not root — so a locally-tuned component keeps its own scale instead of double-compounding with this. Lucide `size={N}` icons are literal px and won't grow; acceptable since perceived "bigger" comes mostly from text/spacing.
**Where it applies:** any app where "make it feel more filled-in on a big screen" is the ask, as opposed to just widening the container (which alone looks sparse — bigger box, same small text, more dead space).
**Confidence:** ★★★★☆

---

### Persist collapsible/accordion open-state in localStorage, keyed per instance
Component `useState(defaultOpen)` resets to `defaultOpen` every time the page remounts (e.g. navigating away and back). If sections need to remember what the user left open/closed:
```tsx
function readStored(key, fallback) {
  const saved = localStorage.getItem(`section-open:${key}`)
  return saved !== null ? saved === '1' : fallback
}
const [open, setOpen] = useState(() => readStored(storageKey, defaultOpen))
function toggle() {
  setOpen(o => { const next = !o; localStorage.setItem(`section-open:${storageKey}`, next ? '1' : '0'); return next })
}
```
Give every rendered instance a unique key (e.g. `"customer-requests:open"`, `"business-dashboard:pending"`) — colliding keys make unrelated sections share state.
**Why it works:** localStorage survives full remounts/navigation, unlike component state. Consistent with the existing "cross-page state → localStorage, not context" rule.
**Where it applies:** any accordion/collapsible/tab whose open state should feel "sticky" across navigation rather than resetting to a hardcoded default.
**Confidence:** ★★★★★

---

### "Recurring"/cadence features without cron: trigger the next cycle at the natural lifecycle event, guard with a parent-link idempotency check
No scheduler infrastructure needed for a "weekly/monthly repeat" feature — spawn the next occurrence at the moment the current one naturally closes out (e.g. marked complete), not on a timer:
```python
def spawn_next_if_needed(db, r):
    if not r.recurring:
        return
    already = db.query(Model).filter(Model.recurring_parent_id == r.id).first()
    if already:
        return  # idempotent — safe if this trigger fires more than once
    next_row = Model(..., recurring=r.recurring, recurring_parent_id=r.id)
    db.add(next_row); db.commit()
```
Extract this into one shared helper called from every place that can trigger it — the original completion path *and* any retroactive path (e.g. "make this already-finished job recurring after the fact") — so both go through identical spawn/guard logic instead of duplicating it.
**Why it works:** avoids building/maintaining scheduling infrastructure for an MVP; the idempotency check (search by parent-link, not a boolean flag) makes the trigger safe to call more than once, which matters once there's more than one call site.
**Where it applies:** any "repeat this on a cadence" feature where the cadence is really "do it again when the current cycle finishes," not calendar-exact timing.
**Confidence:** ★★★★★

---

### Global event bus for cross-cutting auth state changes
When a backend returns a special error (e.g. 403 "Account suspended", forced logout), propagate it to the app shell without prop-drilling or threading it through context — dispatch a window event from the API layer and listen in App.tsx:

```ts
// api.ts — inside parseError():
if (res.status === 403 && data.detail === 'Account suspended') {
  window.dispatchEvent(new Event('account-suspended'))
}

// App.tsx
useEffect(() => {
  const handler = () => setSuspended(true)
  window.addEventListener('account-suspended', handler)
  return () => window.removeEventListener('account-suspended', handler)
}, [])
```

**Why it works:** the API helper has no access to React state or context — dispatching a DOM event is the cleanest way to signal "something global happened" without coupling the layers. App.tsx catches it, sets state, and renders whatever the situation calls for (suspended wall, forced logout screen, etc.).
**Where it applies:** any forced-state situation detected at the API layer — session expiry, account ban, forced password reset, maintenance mode.
**Confidence:** ★★★★★

---

### Full-stack account suspension pattern
Complete pattern for suspending a user — blocks login, blocks API use, filters from public routes, and shows a full-screen wall to suspended users already logged in:

**1 — Model:** add `is_suspended = Column(Boolean, nullable=False, default=False)` to User.

**2 — Migrate existing DB:**
```python
conn.execute('ALTER TABLE users ADD COLUMN is_suspended BOOLEAN NOT NULL DEFAULT 0')
```

**3 — Block at login:**
```python
if user.is_suspended:
    raise HTTPException(status_code=403, detail="Account suspended")
```

**4 — Block at every API call (in `current_user` dep):**
```python
if user.is_suspended:
    raise HTTPException(status_code=403, detail="Account suspended")
```

**5 — Filter from public routes:**
```python
# Join User to filter suspended accounts out of public listings
db.query(BusinessProfile).join(User, User.id == BusinessProfile.user_id).filter(User.is_suspended == False)
```

**6 — Frontend wall (App.tsx):** listen for the `account-suspended` window event (fired from api.ts on 403) and render a full-screen `<SuspendedScreen>` that shows the user's real name + "Account Suspended" in red + Sign Out button. Always show their real name — never replace it with "Account Suspended" as the name itself.

**Key UX rule:** show the account name (company name for businesses, full name for customers) prominently at the top of the suspended screen so they know which account is suspended. The "Account Suspended" label is a badge/status — not a name replacement.

**Confidence:** ★★★★★

---

### Two-view admin console: list → detail drill-in
Admin pages with multiple object types work best as two React views in one component, controlled by a single `selectedId` state:

```tsx
export default function AdminPage() {
  const [selectedUserId, setSelectedUserId] = useState<string | null>(null)
  return (
    <PageShell>
      {selectedUserId
        ? <UserDetail userId={selectedUserId} onBack={() => setSelectedUserId(null)} />
        : <UserList onSelect={setSelectedUserId} />
      }
    </PageShell>
  )
}
```

- **List view:** shows all items with role/status badges, action buttons (suspend, delete) inline, click row to drill in
- **Detail view:** back button at top, subject header with account actions, then content in labeled sections (Jobs, Quotes, Messages) — each item has a delete button
- **Optimistic updates:** on delete/suspend, update local state immediately with `setRows(prev => prev.filter/map(...))` — no refetch needed

**Why it works:** avoids a separate route for admin detail (keeps admin fully within one page), and the two-view pattern is dead simple to extend with more sections or object types.
**Where it applies:** any admin/moderation console where you need a directory + drill-in detail.
**Confidence:** ★★★★★

---

## Lessons Learned

### Always run seed.py before testing login
**What happened:** app launched, credentials pasted from seed.py comments, all logins returned 401. Wasted time checking auth logic before realizing the DB was simply empty — seed.py had never been run.

**Root cause:** `Base.metadata.create_all` creates tables but writes no data. The seed.py script must be run manually the first time. If you test login on a fresh DB, every credential will fail.

**Fix:** `cd backend && .venv/bin/python seed.py`

**Checklist for first launch:**
1. Start backend (uvicorn)
2. `python seed.py` — populate demo accounts
3. Now login works

**Prevention:** add a startup check or a note in LOCAL_DEV.md: "Run seed.py once after first setup before testing logins."
**Confidence:** ★★★★★

---

### Account isolation (user_id scoping)

**Initial approach:** all data tables had no `user_id` column. All users shared all data.

**Problem:** signing in with account B showed account A's plans and streaks.

**Changes made:**
1. Added `user_id TEXT` column to every data table (`custom_plans`, `reading_log`, `completed_books`, `plan_day_completions`)
2. Created shared `current_user` dependency in `deps.py`
3. Updated every router endpoint to require auth and filter by `user_id`
4. Created `authFetch` on the frontend to send the token on every request
5. Migrated DB with `ALTER TABLE ... ADD COLUMN user_id TEXT` (SQLite can add columns but not drop constraints, so `completed_books` needed drop + recreate)

**Engineering lesson:** if a project will ever have multiple users, add `user_id` to every data table on the first day — before any data exists. Retrofitting requires a migration on every table, a change to every router, and a new auth layer on the frontend. Starting with it costs nothing. Adding it later costs a full session.

**Confidence:** ★★★★★

---

### Streak resetting to 0 when user hadn't read today yet

**What happened:** user read yesterday, opened the app today before reading anything, and saw streak = 0.

**Root cause:** `_compute_streaks` only counted backward from today if today was in the active dates set. If today wasn't marked yet, `current` stayed 0 — even if yesterday (and many days before) were active.

**Fix:** count from yesterday when today isn't active yet. Streak only resets to 0 when neither today nor yesterday has a mark — meaning a full calendar day was missed.
```python
if today in active:
    check = today
    while check in active:
        current += 1; check -= timedelta(days=1)
elif yesterday in active:
    check = yesterday
    while check in active:
        current += 1; check -= timedelta(days=1)
# else: current stays 0 — full day missed
```

**Engineering lesson:** a streak counter should reflect "is my streak still alive?" not "did I read today?" Those are different questions. The streak is alive as long as the user hasn't skipped a full calendar day. Always count from the most recent active date (today or yesterday), not strictly from today.

**Confidence:** ★★★★★

---

### Display state frozen after action (streak not updating)

**What happened:** after marking a plan day, the streak number on the Dashboard stayed frozen at the old value. The database was correct — the backend streak was right — but the display never updated.

**Root cause:** `setDisplayStreak(oldStreak)` froze the display intentionally (to hold it while a badge animation played). `displayStreak` was only cleared to `null` (showing the real value) inside `if (streakDelta > 0)`. When the streak had already been bumped today (`streakDelta === 0`), the `else` branch never ran, so `displayStreak` stayed frozen forever.

**Fix:** always clear `displayStreak` after the badge sequence, even when `streakDelta === 0`:
```js
if (streakDelta > 0) {
  setTimeout(() => { setDisplayStreak(null); ... }, delay)
} else {
  setTimeout(() => setDisplayStreak(null), delay)  // always unfreeze
}
```

**Engineering lesson:** whenever you freeze a display value to hold it during an animation, you must have an unconditional unfreeze path. A conditional unfreeze is a latent bug — it works when the condition is true and silently breaks when it's not.

**Confidence:** ★★★★★

---

### White screen from partial state variable removal

**What happened:** removed a form field (`current` password input) — deleted the `useState`, the `<input>`, and the `onChange` — but left `!current` in the button's `disabled` prop. React evaluated `!current` where `current` is `undefined` (not false) — this didn't throw immediately but caused a runtime crash that white-screened the entire component.

**Fix:** when removing a state variable, do a full text search for the variable name and remove ALL references in a single edit: `useState`, the input, `onChange`, `value`, `disabled`, and the API request body.

**Engineering lesson:** partial removal is always wrong. A variable either exists everywhere it's used or nowhere. Search before deleting.

**Confidence:** ★★★★★

---

### CSS animation skipping (transition batching)

**What happened:** set `element.style.transition` and `element.style.opacity = '1'` in the same JS tick. The "Book Completed" text appeared instantly with no fade. `getComputedStyle(el).opacity` returned `'0'` even though `el.style.opacity` was `'1'` — the transition was skipped but the inline style was set.

**Fix:** `void element.offsetHeight` between the transition assignment and the value change.

**Engineering lesson:** CSS transitions require the browser to observe a "before" and "after" state. If both are set in the same frame, there is no before — only after. The reflow forces a style flush. This is required every time you trigger a CSS transition from JS.

**Confidence:** ★★★★★

---

### Spine clip vs spine fill mismatch

**What happened:** updated the spine width from hardcoded `9` to `BW * 0.35` in the `roundRect` clip call, but the `fillRect` inside the clip still had the old hardcoded value. The clip shape expanded but only 9px of paint filled it. Looked like the change did nothing.

**Fix:** update both the clip dimensions AND the fillRect that uses the same dimension.

**Engineering lesson:** any canvas draw that uses a clip followed by a fill has two places to update: the clip shape and the fill call. They must always be in sync. If a visual change appears to do nothing after you updated the code, check whether there is a second place that controls the same dimension.

**Confidence:** ★★★★★

---

### Animation playing under an opaque overlay

**What happened:** the blue burst filled the screen, then the bible slide-in animation played underneath it (same z-index layering). By the time the burst was hidden, the slide was finished — bibles just "appeared."

**Fix:** hide the burst before starting the slide animation. Since both the burst and the scene background are the same color (`#0d1b2a`), hiding the burst is visually seamless — the scene takes over with no flash.

**Engineering lesson:** when an animation plays under an opaque overlay, the user sees nothing until the overlay lifts — at which point the animation is already done. Always verify z-index layering before assuming an animation is broken.

**Confidence:** ★★★★☆

---

### `requestAnimationFrame` throttled in background tabs

**What happened:** animation froze silently after ~2 minutes in a background preview panel. `appearAlpha` stayed at 0 permanently. No error, no warning. Only caught by manual inspection.

**Fix:** replace `requestAnimationFrame(tick)` with `setTimeout(tick, 16)` in all animation loops.

**Engineering lesson:** rAF is a performance feature that pauses animations when the user isn't looking. For controlled animation sequences (not game loops), use setTimeout. rAF is appropriate for continuous render loops where skipping frames is acceptable.

**Confidence:** ★★★★★

---

## Common Mistakes

### Framer Motion `AnimatePresence` exit never completes in headless preview — don't verify collapse/close via DOM text
**Symptoms:** clicking a collapsible/accordion toggle to close it, then checking `document.body.innerText` for the section's content, still shows the content as "present" — looks like the toggle is broken, direction-dependent (open always works, close never does).
**Root cause:** two stacked issues. (1) This headless preview environment doesn't advance rAF/Framer-Motion-driven animations in wall-clock time (same class of limitation as `preview_screenshot` timing out) — the exit animation's height/opacity target never resolves, so the element never unmounts. (2) Even where it does resolve, `innerText` still includes text inside an `overflow:hidden` / animated-to-0-height ancestor — collapsed-via-height content is visually clipped, not `display:none`, so it still counts as "rendered" text.
**Correct pattern:** don't infer open/closed from `innerText` presence or from animated layout (`getBoundingClientRect().height` mid-animation). Verify the underlying **state** directly — temporarily add `console.log('open=', open)` inside the component, reload, click, read `preview_console_logs`, then remove the log. This confirms the toggle logic itself (which is what you actually control) independent of whether the animation can visually finish in this environment.
**Prevention:** trust that in a real browser the animation completes normally (same as the modal slide-down-exit case) — this is a test-environment artifact, not a product bug. Don't spend cycles trying to "fix" a toggle that a state-level check already proves correct.
**Confidence:** ★★★★★

---

### A cross-cutting filtered view must union every backend list source that can hold a match, not just the obvious one
**Symptoms:** a "show me all X" view (e.g. a dashboard's cross-cutting "Recurring" section) is missing rows that are clearly X by every definition — they just don't show up.
**Root cause:** the view was built by filtering a single existing list endpoint (e.g. `/requests/quoted` — "things I've bid on"), but a matching row can live in a *different* bucket depending on its lifecycle stage (e.g. a freshly created lead with no bid yet lives in `/requests/available`, not `/quoted`). Filtering only one source silently drops rows that satisfy the predicate but happen to be in another bucket.
**Correct pattern:** enumerate every endpoint/query that could contain a matching row for the *entire range* of states the predicate covers, fetch all of them, and merge (dedup by id) before filtering — or narrow the predicate itself so it only ever matches rows guaranteed to live in the one source you're already using (see the design principle below — often the better fix).
**Prevention:** before shipping a cross-cutting filtered view, ask "what are all the possible states/buckets a matching item could be in?" and check the view against a manually-verified item in each one.
**Confidence:** ★★★★★

---

### `ORDER BY created_at DESC` ties when timestamp resolution is coarser than real write frequency
**Symptoms:** "get the latest message/row" query intermittently returns the second-to-last row instead of the true latest, right after two writes happen close together (e.g. a reply sent right after the message it's replying to).
**Root cause:** the timestamp column (e.g. SQLAlchemy `server_default=func.now()`) has second-level resolution. Two inserts within the same second tie on `created_at`, and SQL doesn't guarantee tie-break order without an explicit secondary sort key.
**Correct pattern:** always tie-break with the autoincrement primary key, which reflects true insertion order even when timestamps collide: `.order_by(Message.created_at.desc(), Message.id.desc())`. When sorting a *list* of these "latest per group" results (e.g. thread previews), sort the outer list by that same id too, not by the (possibly-tied) timestamp string.
**Prevention:** any "most recent row per group" query needs an id (or other monotonic) tie-breaker, not just a timestamp column, unless the timestamp has sub-second precision guaranteed unique.
**Confidence:** ★★★★★

---

### DOM class changes are not reactive in React

**Symptoms:** toggling dark/light mode updates Tailwind `dark:` classes everywhere, but a component using inline `style={{ color: isDark ? ... : ... }}` stays wrong until something else causes a re-render.
**Root cause:** `document.documentElement.classList.contains('dark')` read during render is not reactive. React re-renders on state/props changes only — toggling a class on `<html>` doesn't trigger any re-render.
**Correct pattern:** use `MutationObserver` to convert the class change into a state update:
```tsx
const [isDark, setIsDark] = useState(document.documentElement.classList.contains('dark'))
useEffect(() => {
  const obs = new MutationObserver(() => setIsDark(document.documentElement.classList.contains('dark')))
  obs.observe(document.documentElement, { attributeFilter: ['class'] })
  return () => obs.disconnect()
}, [])
```
Then pass `isDark` as a prop to child components — never re-read the DOM inside children.
**Prevention:** any component that uses inline styles derived from dark mode must use this pattern, not a one-time DOM read.
**Confidence:** ★★★★★

---

### Framer Motion animations completing while component is hidden

**Symptoms:** user sees no entrance animation — content just appears instantly. The component mounts, animation fires, finishes, all while under an opaque overlay.
**Root cause:** Framer Motion `initial`/`animate` fires immediately on mount, regardless of whether the component is visually covered by another element (e.g. a z-50 intro overlay). By the time the overlay disappears, the 1s animation is already done.
**Correct pattern:** tie the component's `key` to the gate that makes it visible:
```tsx
<Route path="/login" element={<Login key={String(introComplete)} />} />
```
When `introComplete` flips, the key changes, React remounts the component, and the animation plays at the right moment.
**Prevention:** whenever a page mounts before it is visible, add `key` tied to the visibility gate. Don't trust that Framer animations will still be in-progress when the cover lifts.
**Confidence:** ★★★★★

---

### Trying to animate CSS gradients

**Symptoms:** spine fill percentage updates instantly with no animation, even though `transition: background 0.9s ease` is set on the element.
**Root cause:** CSS cannot interpolate between two `linear-gradient(...)` values. The browser treats them as non-animatable discrete values and snaps to the end state immediately.
**Correct pattern:** replace the gradient with a layered div whose `height` goes from 0 to the target %. CSS `height` transitions work perfectly. See the "Bottom-up fill animation" entry in Proven Patterns.
**Prevention:** never use `transition: background` when the background is a gradient. Always use a separate element whose size or opacity you animate instead.
**Confidence:** ★★★★★

---

### Forgetting to restart the backend after code changes
**Symptoms:** code changes appear correct but behavior doesn't change. API returns stale responses.
**Root cause:** uvicorn `--reload` sometimes misses changes, or the process died and needs a full restart.
**Correct pattern:** when behavior doesn't match code, kill all python processes and restart fresh.
**Prevention:** after any backend change that doesn't immediately work, restart before debugging further.
**Confidence:** ★★★★★

---

### Missing pip packages in new environment
**Symptoms:** `ModuleNotFoundError` at startup, server won't start, frontend gets "could not reach server."
**Root cause:** dependencies installed in one environment (e.g., a venv) not present in another.
**Correct pattern:** when a server fails to start after a restart, always run the import check first: `python -c "from app.main import app; print('OK')"`. The error will name the missing module.
**Prevention:** maintain a `requirements.txt` and install from it when the environment changes.
**Confidence:** ★★★★★

---

### Wrong authFetch import path
**Symptoms:** `Failed to resolve import "../utils/authFetch"` — vite import error, page fails to load.
**Root cause:** `authFetch` lives at `src/lib/api.ts`, not `src/utils/authFetch`.
**Correct pattern:** `import { authFetch } from '../lib/api'` — always.
**Prevention:** never guess the path. Check an existing page (Dashboard.tsx line 4) to confirm before writing the import.
**Confidence:** ★★★★★

---

### Delta badge positioned on card instead of number wrapper
**Symptoms:** `+N` badge doesn't appear, or appears in wrong position (corner of card, clipped by overflow).
**Root cause:** putting `relative` on the card div and `absolute` on the badge positions it relative to the whole card. With `-right-8` the badge lands outside the card's painted area and gets clipped.
**Correct pattern:** wrap ONLY the `<p>` number in `<div className="relative inline-block">`. Put the badge inside that wrapper. The `inline-block` sizes to the number text, so `-right-8` places the badge just outside the number — exactly where it should be.
**Prevention:** always use `relative inline-block` around the number, never `relative` on the card. See the stat card template in app-system-guide.md.
**Confidence:** ★★★★★

---

### Saving lastSeen on fetch instead of on unmount
**Symptoms:** baseline doesn't reflect the state when the user left the page — delta animation replays incorrectly or doesn't play at all.
**Root cause:** saving to localStorage inside the fetch `.then()` updates the baseline immediately on arrival. On the next visit the baseline already equals the current value, so delta = 0 and no animation plays.
**Correct pattern:** save `lastSeen` values in the `useEffect` cleanup (unmount). Use refs to access current values — state is stale in closures. Only save if `dataLoadedRef.current` is true so a fast reload before data arrives doesn't overwrite with 0.
**Prevention:** follow the stat card template in app-system-guide.md exactly.
**Confidence:** ★★★★★

---

### Using fade-in delay instead of delta pattern for stat cards
**Symptoms:** number appears to fade in or jump to new value with no badge, or animation fires before the card is visible.
**Root cause:** using `setTimeout` to delay a `number-fade-in` is not the same as the delta pattern. The delta pattern shows the OLD value, fires the badge, then fades in the NEW value. The delay approach just fades in the new value late — no badge, and timing is fragile.
**Correct pattern:** use the full delta pattern: `setDisplay(oldValue)` → `setTimeout(() => { showDelta('+N'); setDisplay(null); setFadeKey(k => k+1) }, 300)`. See the stat card template in app-system-guide.md.
**Prevention:** any time a stat card needs to animate on page arrival, use the delta pattern — not setTimeout + number-fade-in alone.
**Confidence:** ★★★★★

---

### Using Tailwind margin classes for pixel-precise spacing
**Symptoms:** spacing changes don't apply or aren't visible even after a correct-looking class change (e.g. `mb-4` → `mb-6` with no visual result).
**Root cause:** Tailwind's JIT compiler can miss both arbitrary values (`mb-[24px]`) and standard scale classes (`mb-6`) depending on context — confirmed across multiple sessions.
**Correct pattern:** switch to `style={{ marginBottom: '20px' }}` on the first failure. Don't retry with a different Tailwind class.
**Prevention:** for any spacing that needs to be exact, use inline styles from the start.
**Confidence:** ★★★★★

---

### Assuming grid/row layout when the user wants stacked columns
**Symptoms:** layout gets pushed back immediately. "I want them stacked, not side by side."
**Root cause:** default mental model for stat cards and feature sections is a grid. The user's default is one-column full-width.
**Correct pattern:** default to `flex flex-col` unless the existing page already uses a grid.
**Prevention:** before laying out any new section, check what layout the rest of the page uses.
**Confidence:** ★★★★★

---

### Filtered array index ≠ source array index (MTG duplicate drag bug)

**Symptoms:** dragging a duplicate card (e.g. a Forest land) from the hand moves one copy but spawns a ghost duplicate on the battlefield.
**Root cause:** `filteredHand` hides cards already on the battlefield, so its indices diverge from `handOrder` indices. Using `filteredHand.indexOf(card)` as the `handIndex` passed to the drag handler pointed to the wrong card in `handOrder`, causing the wrong entry to be removed — or not removed — leaving a duplicate.
**Correct pattern:** build `filteredHand` as `{ card, originalIndex }` pairs by mapping `handOrder` first:
```tsx
const filteredHand = handOrder
  .map((card, originalIndex) => ({ card, originalIndex }))
  .filter(({ card }) => { /* hide BF copies */ return true/false })
// Then render using originalIndex as key and for handIndex:
filteredHand.map(({ card, originalIndex }) => (
  <div key={originalIndex}>
    <CardDisplay onDragStart={(c, e) => handleDragStart(c, 'hand', undefined, e, originalIndex)} />
  </div>
))
```
**Prevention:** never use a filtered array's index to reference items in the source array. Always carry the original index through the filter.
**Confidence:** ★★★★★

---

### Opponent state is render-only — no extra sync needed

**Symptoms:** opponent tapped/flipped cards weren't showing on other players' screens, tempting a new sync mechanism.
**Root cause:** the data was already there. `battlefieldCards` is a JSON blob containing `{ tapped, flipped }` per card, already synced to the backend via the debounced PUT. The opponent render just wasn't reading those fields.
**Correct pattern:** the fix is purely in the opponent's render logic — no backend changes, no new columns, no new PUT calls:
```tsx
// Opponent card render:
style={{ transform: (item as any).tapped ? 'rotate(90deg)' : 'none' }}
card={(item as any).flipped ? { ...item.card, image_uri: item.card.back_image_uri || MTG_CARD_BACK } : item.card}
```
**Prevention:** before adding a new sync column/PUT, verify that the data isn't already in the JSON blob being synced. Full state in one blob means the data is always available; you just need to read it.
**Confidence:** ★★★★★

---

### FastAPI CORS must list every port explicitly

**Symptoms:** frontend on port 5174 gets CORS errors even though 5173 was in the allowed origins list. No error on startup.
**Root cause:** `CORSMiddleware` does exact-match on origin strings. `http://localhost:5173` does not match `http://localhost:5174`.
**Correct pattern:**
```python
allow_origins=["http://localhost:5173", "http://localhost:5174", "http://127.0.0.1:5173", "http://127.0.0.1:5174"]
```
Or use `allow_origins=["*"]` in development only.
**Prevention:** whenever Vite's dev server auto-selects a new port (e.g. 5174 because 5173 is taken), remember to add the new port to the backend's CORS list.
**Confidence:** ★★★★★

---

### Show/hide toggle placed beside content instead of below it
**Symptoms:** "Make the button centered below the hint text" (after placing it inline with `flex items-center gap-2`).
**Root cause:** default `flex-row` puts the toggle next to the content.
**Correct pattern:** for any "reveal" pattern (show password, show hint, expand detail), use `flex flex-col items-center gap-1`.
**Prevention:** if a value has a secondary action (show/hide, expand/collapse), always stack vertically.
**Confidence:** ★★★★★

---

### React StrictMode double-invocation
**Symptoms:** duplicate entries appear in lists. Completions get added twice.
**Root cause:** in development, StrictMode calls state updaters twice intentionally to surface side effects.
**Correct pattern:** use functional updates with deduplication: `prev => prev.some(p => p.id === id) ? prev : [...prev, item]`.
**Prevention:** never use non-functional state updates for list operations.
**Confidence:** ★★★★★

---

### React StrictMode double-fires `useEffect([])` — causes double animations
**Symptoms:** an animation (delta badge, number rise) plays twice in a row on page load. Only happens in development.
**Root cause:** StrictMode mounts → unmounts → remounts every component in development. Two `useEffect([], [])` calls fire, both async, both see the same `lastSeen` value before either saves the new one — so both play the animation.
**Correct pattern:** module-level cancellation token:
```tsx
let _loadGen = 0  // outside the component

// inside useEffect([], []):
const gen = ++_loadGen
async function loadAll() {
  // ... all async work ...
  if (gen !== _loadGen) return  // first invocation cancelled by second
  // animation logic here
}
```
The second invocation increments `_loadGen` to 2. When the first invocation checks `gen (1) !== _loadGen (2)`, it exits. The second invocation's `gen (2) === _loadGen (2)`, so it proceeds.
**Why it works:** module-level variables survive the StrictMode cleanup/remount cycle. The gen check is free.
**Prevention:** use this pattern on every `useEffect([], [])` that triggers a one-shot animation.
**Confidence:** ★★★★★

---

### `fetch` / `authFetch` does not throw on non-2xx — must check `res.ok`
**Symptoms:** a one-time flag gets set even after the API call failed (e.g. `chaptersBackfilled` set when backfill returned 500). Feature appears to have run but produced no effect.
**Root cause:** `fetch` only rejects (throws) on network errors. HTTP 4xx/5xx responses are resolved normally with `res.ok === false`. A try/catch around `authFetch(...)` catches nothing in this case.
**Correct pattern:**
```ts
const res = await authFetch('/api/endpoint', { method: 'POST' })
if (res.ok) {
  localStorage.setItem('flagName', '1')
}
```
**Prevention:** always check `res.ok` before treating a response as success. Never assume a resolved promise means the request succeeded.
**Confidence:** ★★★★★

---

### New SQLAlchemy model tables not created until backend restart
**Symptoms:** all endpoints touching the new table return 500 errors. Table doesn't exist in SQLite. Code looks correct.
**Root cause:** `Base.metadata.create_all(bind=engine)` only runs when the FastAPI app starts. Adding a model class while uvicorn is running does not create the table.
**Correct pattern:** after adding any new model to `models.py`, kill the uvicorn process and restart it. The new table will be created at startup.
**Prevention:** any time behavior is mysteriously 500 after adding a model, restart first before debugging further.
**Confidence:** ★★★★★

---

### localStorage is origin-specific — clearing in a preview/different port has no effect
**Symptoms:** cleared a localStorage key from a tool (preview server, dev tool at a different port) but the app still sees the old value.
**Root cause:** localStorage is scoped to the origin (protocol + host + port). `localhost:3456` and `localhost:5173` have completely separate localStorage stores. Clearing one has zero effect on the other.
**Correct pattern:** to clear localStorage for the app, open the app's actual URL in the browser → DevTools (F12 or right-click Inspect) → Application → Local Storage → select the correct origin → delete the key.
**Prevention:** never assume a localStorage write from one origin will be visible at another.
**Confidence:** ★★★★★

---

### `motion.div` conflict — appearance and zoom on same element
**Symptoms:** animations fight each other. One works, the other doesn't, or they produce unexpected combined transforms.
**Root cause:** Framer applies all variants to the element's single transform. Two independent animations compete on the same property.
**Correct pattern:** separate wrapper `motion.div` for each independent animation (one for fade-in, one for scale/zoom).
**Prevention:** each animation concern gets its own element.
**Confidence:** ★★★★★

---

### `preview_fill` / setting `.value` doesn't update React controlled inputs
**Symptoms:** filling an input via the preview tool (or `el.value = x`) then submitting sends empty/old state; the form doesn't POST, or POSTs stale data. The DOM shows the value but React's state is unchanged.
**Root cause:** React tracks controlled inputs via its own value tracker. Assigning `.value` directly (or a tool that does) doesn't fire React's synthetic `onChange`, so state stays empty and a re-render can even wipe the DOM value back.
**Correct pattern:** set through the native setter and dispatch a bubbling `input` event, then submit the form programmatically:
```js
const set = Object.getOwnPropertyDescriptor(window.HTMLInputElement.prototype,'value').set
set.call(el, val); el.dispatchEvent(new Event('input',{bubbles:true}))
// submit reliably — clicking the button often does NOT trigger onSubmit:
document.querySelector('form').requestSubmit()
```
**Prevention:** never trust `preview_fill`/`.value` for React forms; use the native-setter + `input` event + `form.requestSubmit()` recipe for every automated UI form test.

**Addendum — two dependent clicks in one `preview_eval` call race React's commit:** clicking a selection button (e.g. picking one of several options) and then immediately clicking a "Confirm" button *in the same synchronous script* can submit the *previous* selection — the state update from click 1 hasn't committed/re-rendered by the time click 2's handler reads it. Symptom: the confirmed value matches the default/prior selection, not the one just clicked, even though a DOM check right after click 1 shows the new selection applied. Fix: split dependent UI actions across separate `preview_eval` calls (or add a short `await new Promise(r => setTimeout(r, ...))` between them within one call) so React has a chance to commit between steps.
**Confidence:** ★★★★★

---

### `preview_screenshot` times out — verify layout with the a11y snapshot + `getBoundingClientRect`
**Symptoms:** `preview_screenshot` hangs/times out (~30s) even though the page renders fine (snapshots work). Wastes turns waiting.
**Root cause:** an environment/renderer quirk, not an app bug — confirmed by working `preview_snapshot`s taken moments before/after.
**Correct pattern:** verify structure/content with `preview_snapshot` (accessibility tree), and verify *pixel-exact layout* (overlap, overflow, fit, computed font sizes) with `preview_eval` reading `getBoundingClientRect()` / `getComputedStyle()`. This is more precise than eyeballing a screenshot anyway — e.g. checking `nav.left < logo.right` for overlap and `document.body.scrollWidth > window.innerWidth` for overflow.
**Prevention:** don't retry screenshots in this environment; reach for snapshot + eval measurements. Note the preview panel's own width can fluctuate between calls, so measure the worst case a few times.
**Confidence:** ★★★★☆

---

### `create_all` never ALTERs an existing table — new columns need a reset/ALTER, and an import check creates the DB early
**Symptoms:** added a column to a model, code looks right, but every query on it 500s with `no such column`. (Distinct from the "new *table* needs a restart" case.)
**Root cause:** `Base.metadata.create_all` only creates *missing tables*; it never adds columns to a table that already exists. Worse: running `python -c "from app.main import app"` as an import check executes `create_all` at import, so the DB file is born with the *then-current* schema — any model edit afterward is invisible until you migrate.
**Correct pattern (dev):** stop the server, delete the SQLite file, restart so all tables are recreated fresh; keep a `seed.py` so demo data/logins survive the reset. **(prod):** `ALTER TABLE ... ADD COLUMN`.
**Prevention:** in dev, decide the schema before the first import/run; when a `no such column` appears after a model edit, reset the DB first, then debug.
**Confidence:** ★★★★★

---

### `curl` to localhost returns 000 from the Git Bash shell (Windows)
**Symptoms:** `curl http://127.0.0.1:PORT/...` exits 7 / prints `000` from the bundled Git Bash, even though the server is up and the browser reaches it fine.
**Root cause:** environment quirk of that shell's curl reaching loopback on this Windows setup.
**Correct pattern:** hit the API from the browser origin via `preview_eval(fetch(...))` (also exercises the Vite proxy), or use PowerShell / the venv Python for HTTP checks. Backend-only checks: run curl from the backend venv context or use `requests`.
**Prevention:** don't conclude the server is down from a bash-curl `000`; confirm with an in-browser fetch or PowerShell.
**Confidence:** ★★★★☆

---

## Design Principles

### Every display value that gets frozen must have an unconditional unfreeze path
If you store an "old" value to freeze the display during an animation, clearing it must happen regardless of other conditions. A conditional clear is a latent bug. Every `setDisplayX(oldValue)` must have a guaranteed `setDisplayX(null)` that always fires.

---

### Scope data by user from day one
Any multi-user system needs `user_id` on every data table before any data is written. Adding it later requires migrating every table, updating every query, and building an auth layer that should have existed from the start. There is no cost to adding it early. The cost of adding it late is a full refactor session.

---

### Cross-entity rankings/awards are computed at the collection level, not per-entity
A "#1 in category" / "best value" award requires comparing every candidate, so a single entity's serializer can't produce it — it has no view of its peers. Compute per-entity badges (milestones, thresholds like ≥4.5★) in the entity serializer, but compute *rankings* once in the list/collection endpoint and attach the winner's spotlight there. (Marketplace: `base_badges(business)` per-entity vs `category_spotlights(db)` over all businesses.)
**Why:** ranking is a property of the set, not the element. Trying to compute it per-row means re-querying all peers on every serialize — slow and easy to get inconsistent.
**How to apply:** any "top N / best / #1" surface — compute at the endpoint that already has the full list, pass results down as an override map keyed by id.

---

### A "becomes true" outcome is a predicate over (flag, lifecycle status) — don't add a separate stage field for it
"Only counts as recurring once accepted on both ends" is not a new state to store — it's `recurring_flag AND status IN (accepted, completed)`, computed at read time from data you already have. Storing a redundant `is_active_recurring` boolean (set/cleared alongside status changes) would need updating in lockstep from every place status changes, and drift the moment one call site forgets.
**Why:** an intent flag (set once, at creation) and a lifecycle stage (changes many times, many places) are different axes. The moment "this is achieved" is their conjunction, not a fact that needs its own storage.
**How to apply:** before adding a new column/flag to represent "X has become true," check whether it's actually derivable as `existing_flag AND status_in_set` — if so, filter for it at the view/query layer instead of storing and maintaining a redundant field.

---

### Guard one-time side effects with a before/after transition check, not the new value alone
A notification (or any other one-shot side effect — email, webhook, audit log) triggered by "this field is now X" must check `was_X == False and is_now_X == True`, not just `is_now_X`. Otherwise every idempotent re-save (re-verifying an already-verified business, re-submitting an unchanged quote) re-fires the effect. `became_verified = body.verified and not existing.verified` computed *before* the field is overwritten, then act on that flag — not on the field's final value.
**How to apply:** any status/flag flip that fires a notification, badge award, or webhook — write the transition check first, mutate the field second.

---

### An inline-style default can't be overridden by a child's `className` — expose an explicit prop instead
If a shared component applies its default sizing via a class string (e.g. `max-w-5xl`) and a caller "overrides" it by appending another class via `className` prop, this only works by accident of Tailwind's generation order — and breaks completely if the default is ever changed to an inline `style` (inline style always wins over any class, full stop, regardless of source order). Give the component an explicit prop that fully replaces the default (e.g. `maxWidthClassName?: string`, used instead of the fluid default when present) rather than relying on class-string concatenation to "win."
**How to apply:** any shared layout component (page shell, card, modal) with a per-page size/width override — design the prop as a swap, not an append, especially once any part of the default is inline-styled.

---

### Validate end-to-end, not just locally
A backend change isn't done until the frontend sends the token and the backend validates it. A form change isn't done until you verify no orphaned references remain. Local correctness doesn't mean system correctness.

---

### One column by default, grid only when the page already uses it
New content sections are always stacked vertically until proven otherwise. Grids are a layout decision that should match the existing page, not a default choice.

---

### Deduplicate before delete — search all references before removing a variable
Removing a state variable without searching for all its uses creates dangling references that cause runtime crashes. The symptom (white screen) is worse than the mistake (orphaned reference) suggests.

---

### Standalone HTML animations over React for full-screen sequences
A full-screen cinematic animation in React requires wrestling with mount timing, Framer Motion lifecycle, and routing. The same animation in a standalone HTML file is just an async function with canvas draws. Reserve React for interactive UI; use vanilla HTML for controlled one-shot sequences.

---

### Use `setTimeout(tick, 16)` in standalone animations, never `rAF`
rAF is a display-sync optimization. Its throttling behavior makes it unreliable for animation sequences that play while the developer is looking at something else. This matters during development (preview panels) and in production (background tabs).

---

## Architecture Decisions

### sessionStorage for auth tokens, localStorage for preferences

**Decision:** JWT token in `sessionStorage`, email/theme/preferences in `localStorage`.

**Alternatives considered:**
- Token in `localStorage`: survives tab close, but means any open tab is permanently authenticated. No natural "each tab is its own session" boundary.
- Token in memory (React state): lost on page refresh. Bad UX — every refresh requires re-login.
- Cookies: requires backend `Set-Cookie` and CORS credential config. More complex.

**Why sessionStorage won:** automatic cleanup on tab close. No manual expiry logic. Gives the per-tab session behavior (each new tab requires login) without any implementation cost.

**Tradeoffs:** refreshing within a tab preserves the session (sessionStorage survives refresh). Opening a new tab does not (sessionStorage is not shared between tabs). This is the intended behavior.

**Reusability:** apply this pattern to any app where "one session per tab" is the desired model.

**Confidence:** ★★★★☆

---

### Shared `deps.py` for FastAPI auth dependency

**Decision:** extract the `current_user` dependency into a shared `deps.py` module, imported by all routers.

**Alternatives considered:**
- Define `current_user` in `auth.py` and import from there: creates a circular import risk when auth.py also imports from models.
- Inline the auth check in each router: extreme duplication, any change must be applied to every file.

**Why `deps.py` won:** neutral file with no outward imports, no circular risk. All routers depend on it, it depends on nothing router-specific.

**Reusability:** standard FastAPI pattern. Use for any shared dependency (current user, DB session, rate limiting, feature flags).

**Confidence:** ★★★★★

---

### `authFetch` wrapper instead of global fetch override

**Decision:** export an `authFetch` function that wraps `fetch` with the auth header. All API calls use `authFetch`, unauthenticated calls (login, register, health) use bare `fetch`.

**Alternatives considered:**
- Monkey-patch `window.fetch`: silently adds auth to every request including third-party. Dangerous.
- Axios instance with interceptors: adds a dependency. `authFetch` is 12 lines with no deps.
- React context with fetch functions: overengineered for this use case.

**Why `authFetch` won:** explicit call site — you can see which calls are authenticated by reading the code. No magic. No accidental auth on external requests.

**Reusability:** copy the 12-line function to any project. No modification needed except the storage key.

**Confidence:** ★★★★★

---

## Prompt Effectiveness

### Auth system + account isolation
- **Prompts required:** ~6 (bcrypt fix, passlib diagnosis, deps.py, user_id migration, authFetch, frontend wiring)
- **Requirements initially clear:** partial — "make it separate info for each account" was clear in intent, but the full scope (every table, every router, every frontend call) wasn't in the prompt
- **Biggest source of iteration:** `passlib`/`bcrypt` incompatibility — 2 prompts just on the dependency issue before any real auth work started
- **Final confidence:** high — full isolation working per account
- **Reusable:** Yes — `deps.py` + `authFetch` pattern applies to any FastAPI + React project

---

### Book completion animation (9-stage sequence)
- **Prompts required:** ~12 (initial build, book sizing ×3, spine issues ×3, rAF fix, text fade fix, blue burst intro, slide-in fix, genesis skip, click-to-dismiss)
- **Requirements initially clear:** the sequence was clear. Visual tuning (size, proportions) required multiple iterations.
- **Biggest source of iteration:** book proportions (height, width, spine) — went through 3 rounds each
- **Final confidence:** high
- **Reusable:** Yes — the animation template, easing functions, and `postMessage` pattern apply to any future animation

---

### Streak display freeze bug
- **Prompts required:** 1 (identified and fixed in a single turn)
- **Requirements initially clear:** "streak number didn't increase" was unambiguous
- **Biggest source of iteration:** none
- **Final confidence:** high
- **Reusable:** Yes — the pattern (always unfreeze unconditionally) applies to any display-freeze animation pattern

---

## UX Decisions

### Bottom-to-top stagger on page entrance
Cards animate in from the bottom card first, top card last. Feels like the page assembles itself upward. Top-to-bottom feels like cards are falling in, which reads as less intentional. Default delay per row: 0.18s.

---

### `—` (em dash) for empty stats instead of `0`
When no active plans exist, completion% and days read show `—` not `0`. Reason: `0%` implies "you've done nothing toward a goal." `—` implies "there is no goal to measure." These are different states and should look different.

---

### Delta badges for stat increases
When a number goes up, show a floating `+N` badge instead of just updating the number. This confirms to the user that their action was recorded and communicates the exact change. The badge appears, floats up, and fades out before the number updates — the sequence is: freeze → badge → update.

---

### Progressive form disclosure
On the plan creation form, additional fields (duration, start date, scripture note) only appear after the plan name is typed. Reduces visual noise for users who only want to set a name. Uses `AnimatePresence` with height animation so fields slide in rather than pop.

---

### Blur backdrop on all modals
All modals use `backdropFilter: blur(6px)` with `rgba(0,0,0,0.45)`. The blur tells the user "this content is behind the current interaction" without hiding it. Click backdrop to close — this is the standard pattern and should never be omitted.

---

### Per-tab auth (sessionStorage)
Each browser tab requires its own sign-in. This is intentional: it matches the expectation that opening a new tab is a fresh context, and prevents a shared computer from accidentally staying signed in across tabs.

---

## Reusable Components

### `row(delay)` — Framer Motion entrance helper
See **Proven Patterns**. Copy verbatim to every new page file. No modification needed.

---

### `authFetch` — authenticated fetch wrapper
See **Proven Patterns**. Lives in `src/lib/api.ts`. Import in any file that calls `/api/` endpoints.

---

### `RequireAuth` — route guard component
```tsx
function RequireAuth({ children }: { children: React.ReactNode }) {
  if (!sessionStorage.getItem('token')) {
    return <Navigate to="/login" replace />
  }
  return <>{children}</>
}
```
Wrap any route that requires authentication. If no token, redirects to `/login`. No flash of protected content.

---

### Full-screen animation iframe portal
```tsx
{animationKey && createPortal(
  <div className="fixed inset-0 z-[99999]">
    <iframe
      src={`/animation-name.html?param=${encodeURIComponent(animationKey)}`}
      className="w-full h-full border-0"
      title="Animation"
    />
  </div>,
  document.body
)}
```
Dismisses when the animation posts `{ type: 'animationDone' }` and the `onMessage` handler sets `animationKey` to null. Use `z-[99999]` for animation iframes so they sit above all other overlays.

---

### Blur modal
```tsx
createPortal(
  <div
    className="fixed inset-0 z-[9998] flex items-center justify-center"
    style={{ backdropFilter: 'blur(6px)', background: 'rgba(0,0,0,0.45)' }}
    onClick={onClose}
  >
    <div
      className="bg-white dark:bg-[#162032] rounded-xl p-6 w-full max-w-sm mx-4"
      style={{ animation: 'fade-up 0.25s ease both' }}
      onClick={e => e.stopPropagation()}
    >
      {/* content */}
    </div>
  </div>,
  document.body
)
```
Click backdrop to close, click inside to not close. Always use `createPortal`.

---

### Canvas bookshelf pattern
See `animation-retrospective.md` → "Canvas bookshelf pattern" for the full `drawBook` template with spine/clip/fill/label. Key rules:
- Book height: `window.innerHeight * 0.90`
- Book width: `BH * 0.52`
- Spine width: `BW * 0.35`
- Rounded left side only: `roundRect(ctx, bx, by, BW, BH, [SR, 3, 3, SR])`
- Spine clip AND fillRect must both use `SW` — never hardcode one

---

## Never Do Again

### Using `passlib.CryptContext` with bcrypt
`passlib` reads `bcrypt.__about__` to detect the installed version. Newer `bcrypt` versions removed `__about__`. This causes an `AttributeError` at startup, not at hash time — the entire server fails to start. Use `bcrypt.hashpw` / `bcrypt.checkpw` directly. No `passlib`.

---

### Leaving a display-freeze state without an unconditional unfreeze
If you freeze a display value (`setDisplayX(oldValue)`) to hold it during animation, you MUST have a code path that calls `setDisplayX(null)` regardless of conditions. A conditional unfreeze leaves the value frozen forever when the condition isn't met. The bug is silent — no error, just a number that never updates.

---

### Animating under an opaque overlay
If an element animates while something opaque covers it, the animation plays invisibly. By the time the overlay lifts, the animation is done — the user sees nothing. Always check z-index layering before assuming an animation is broken. Always hide or remove the overlay before starting the animation if you want the user to see it.

---

### Retrying Tailwind margin classes after a failure
If `mb-4` doesn't show, switching to `mb-6` will not fix it. If `mb-[24px]` doesn't show, `mb-[20px]` will not fix it. Switch to `style={{ marginBottom: '20px' }}` immediately — this always works. One failure = switch to inline style, no second attempt.

---

### Partial removal of a state variable
Removing a state variable without removing ALL its usages (`useState`, `value=`, `onChange=`, `disabled=`, API body) leaves dangling references that cause runtime crashes. Always search for the variable name before deleting.

---

## Future Improvements

- **DB migrations via Alembic** — `ALTER TABLE` via raw Python script works but is manual. Alembic would give versioned, reversible migrations.
- **`requirements.txt` + install step in setup** — missing packages are currently discovered at runtime. A requirements file and install check would surface them at setup time.
- **Refresh token / token expiry** — current tokens last 30 days with no refresh mechanism. A proper refresh flow would allow short-lived tokens with automatic renewal.
- **Error boundaries in React** — a white screen when a component crashes gives no feedback. A top-level error boundary would catch crashes and show a "something went wrong" state.
- **Animation timing constants shared across files** — currently each HTML animation file defines its own `T = {...}` constants independently. A shared config approach would let you tune the whole feel in one place.
