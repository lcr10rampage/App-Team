# Bible Tracker App — Complete System Guide & Build Reference

This file is the authoritative blueprint for every system built in this app. If you want to build a navbar, a streak counter, a progress tracker, an auth system, or a full-screen animation — start here. Every section is written so you can reproduce the system from scratch in a new project.

---

## Table of Contents

1. [Stack & Project Setup](#1-stack--project-setup)
2. [Navbar / Tab Bar](#2-navbar--tab-bar)
3. [Page Entrance Animations](#3-page-entrance-animations)
4. [Stat Cards with Delta Badges](#4-stat-cards-with-delta-badges)
5. [Progress Tracker / Plan Cards](#5-progress-tracker--plan-cards)
6. [Reading Log](#6-reading-log)
7. [Bookshelf Grid](#7-bookshelf-grid)
8. [Modals](#8-modals)
9. [Full-Screen Animations](#9-full-screen-animations)
10. [Authentication System](#10-authentication-system)
11. [Backend: Models & Routers](#11-backend-models--routers)
12. [Dark / Light Mode](#12-dark--light-mode)
13. [localStorage Communication Layer](#13-localstorage-communication-layer)
14. [How All the Numbers Work](#14-how-all-the-numbers-work)
15. [Button Behaviors Reference](#15-button-behaviors-reference)
16. [Animation Conditions Reference](#16-animation-conditions-reference)
17. [Page & Route Structure](#17-page--route-structure)
18. [z-index Stack](#18-z-index-stack)
19. [Rules & Lessons](#19-rules--lessons)

---

## 1. Stack & Project Setup

| Layer | Tool | Notes |
|---|---|---|
| Frontend framework | React + Vite | `npm create vite@latest` → React + TypeScript |
| Styling | Tailwind CSS v4 | `@tailwindcss/vite` plugin — NOT PostCSS |
| In-app animations | Framer Motion | `npm install framer-motion` |
| Full-screen animations | Vanilla HTML/CSS/JS + Canvas | No dependencies |
| Font | EB Garamond | Google Fonts CDN |
| Backend | FastAPI + SQLite | Python, `uvicorn`, `sqlalchemy` |
| Auth | `python-jose` + `bcrypt` (direct) | Never use `passlib` |
| Dev server | Vite | Port 5173 |

### `vite.config.ts`
```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  server: {
    port: 5173,
    proxy: {
      '/api': { target: 'http://localhost:8000', changeOrigin: true },
    },
  },
})
```

### `index.css` — required globals
```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@keyframes fade-up {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes fade-from-black {
  0%   { opacity: 1; }
  45%  { opacity: 1; }
  100% { opacity: 0; }
}
@keyframes delta-rise {
  from { transform: translateY(8px) scale(0.78); }
  to   { transform: translateY(-20px) scale(1.14); }
}
@keyframes delta-fade {
  0%   { opacity: 0; }
  18%  { opacity: 1; }
  62%  { opacity: 1; }
  100% { opacity: 0; }
}
@keyframes number-fade-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}
@keyframes milestone-pop {
  0%   { transform: scale(0.7); opacity: 0; }
  18%  { transform: scale(1.06); opacity: 1; }
  75%  { transform: scale(1); opacity: 1; }
  100% { transform: scale(0.9); opacity: 0; }
}
@keyframes book-flip-in {
  from { transform: perspective(800px) rotateY(-90deg) scale(0.6); opacity: 0; }
  to   { transform: perspective(800px) rotateY(0deg) scale(1); opacity: 1; }
}
@keyframes book-flip-out {
  from { transform: perspective(800px) rotateY(0deg) scale(1); opacity: 1; }
  to   { transform: perspective(800px) rotateY(90deg) scale(0.6); opacity: 0; }
}
@keyframes page-enter {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

### Color palette — always use these exact values
| Purpose | Light | Dark |
|---|---|---|
| Page background | `bg-amber-50` | `bg-[#0d1b2a]` |
| Card surface | `bg-white` | `bg-[#162032]` |
| Card border | `border-amber-100` | `border-[#2a3a52]` |
| Primary text | `text-slate-800` | `text-[#f0e6d0]` |
| Muted text | `text-slate-400` | `text-[#8a9ab5]` |
| Very muted | `text-slate-300` | `text-[#4a5a72]` |
| Gold accent | `text-amber-700` | `text-[#d4a847]` |
| Button bg | `bg-amber-800` | `bg-[#d4a847]` |
| Button text | `text-white` | `text-[#0d1b2a]` |
| Input bg | `bg-amber-50` | `bg-[#0d1b2a]` |
| Input border | `border-amber-200` | `border-[#2a3a52]` |
| Animation bg | `#0d1b2a` | (same) |

---

## 2. Navbar / Tab Bar

### What it does
Fixed bottom (or top) bar with tab links, dark/light toggle, and sign-out icon. Each tab is a plain `<a href>`, not React Router Link. Active tab is detected by `location.pathname`.

### How to build it
```tsx
// Navbar.tsx
import { useLocation } from 'react-router-dom'
import { Sun, Moon, LogOut } from 'lucide-react'

interface NavbarProps {
  dark: boolean
  onToggleDark: () => void
}

export default function Navbar({ dark, onToggleDark }: NavbarProps) {
  const { pathname } = useLocation()

  const tabs = [
    { href: '/',          label: 'Dashboard' },
    { href: '/progress',  label: 'Progress' },
    { href: '/shelf',     label: 'Shelf' },
    { href: '/profile',   label: 'Profile' },
  ]

  function handleSignOut() {
    sessionStorage.removeItem('token')
    localStorage.removeItem('userEmail')
    window.location.href = '/login'
  }

  return (
    <nav className="fixed bottom-0 inset-x-0 z-50 bg-white dark:bg-[#162032] border-t border-amber-100 dark:border-[#2a3a52] flex items-center justify-center gap-1 px-3 py-2">
      {tabs.map(tab => (
        <a
          key={tab.href}
          href={tab.href}
          className={`text-[10px] px-2 py-1 rounded font-medium transition-colors
            ${pathname === tab.href
              ? 'bg-amber-100 dark:bg-[#2a3a52] text-amber-800 dark:text-[#d4a847]'
              : 'text-slate-500 dark:text-[#8a9ab5] hover:text-amber-700 dark:hover:text-[#d4a847]'
            }`}
        >
          {tab.label}
        </a>
      ))}

      {/* Thin divider between tabs and icon buttons */}
      <div className="w-px h-5 bg-amber-200 dark:bg-[#2a3a52] mx-1" />

      {/* Dark/light toggle */}
      <button
        onClick={onToggleDark}
        className="w-8 h-8 flex items-center justify-center rounded text-slate-500 dark:text-[#8a9ab5] hover:text-amber-700 dark:hover:text-[#d4a847]"
      >
        {dark ? <Sun size={15} /> : <Moon size={15} />}
      </button>

      {/* Sign out */}
      <button
        onClick={handleSignOut}
        className="w-8 h-8 flex items-center justify-center rounded text-slate-500 dark:text-[#8a9ab5] hover:text-red-500"
      >
        <LogOut size={15} />
      </button>
    </nav>
  )
}
```

### Wiring in App.tsx
```tsx
const [dark, setDark] = useState(() => localStorage.getItem('theme') === 'dark')
useEffect(() => {
  document.documentElement.classList.toggle('dark', dark)
  localStorage.setItem('theme', dark ? 'dark' : 'light')
}, [dark])

// In JSX — Navbar sits outside the route content:
<Navbar dark={dark} onToggleDark={() => setDark(d => !d)} />
```

### Rules
- Tab text: `text-[10px] px-2 py-1` — smaller than default to avoid crowding
- Active tab uses bg highlight, not underline
- Icon buttons: `w-8 h-8` fixed size
- Divider between text tabs and icon buttons: `w-px h-5`
- Use plain `<a href>` not React Router `<Link>` — avoids client-side navigation bugs with full-page animation resets

---

## 3. Page Entrance Animations

### The `row(delay)` pattern
Every page uses this. Copy it verbatim to the top of every page file.

```tsx
const row = (delay: number) => ({
  initial: { opacity: 0, y: 22 },
  animate: { opacity: 1, y: 0 },
  transition: { duration: 0.55, delay, ease: [0.2, 0, 0.3, 1] as const },
})
```

Usage:
```tsx
<motion.div {...row(0)}>   {/* bottom card — animates first */}
<motion.div {...row(0.18)}>
<motion.div {...row(0.36)}>
<motion.div {...row(0.54)}>
<motion.div {...row(0.72)}> {/* top card — animates last */}
```

### Stagger order rule
Bottom card = delay 0. Top card = highest delay. Builds the page from the bottom up. This feels like the page assembles itself. Top-to-bottom stagger feels like cards falling in — avoid it.

### Page wrapper (route change animation)
```tsx
// PageWrapper.tsx
export default function PageWrapper({ children }: { children: React.ReactNode }) {
  return (
    <div style={{ animation: 'page-enter 0.35s ease both' }}>
      {children}
    </div>
  )
}
```

Wrap every route's content in `<PageWrapper>`.

### Padding for fixed navbar
Every page needs bottom padding so content isn't hidden behind the fixed navbar:
```tsx
<div className="min-h-screen bg-amber-50 dark:bg-[#0d1b2a] pb-20 px-4 pt-6">
```

---

## 4. Stat Cards with Delta Badges

### What it does
Displays a number (streak, completion%, days read). When the number increases, freezes the display at the old value, floats a `+N` badge upward, then fades in the new number. Multiple stats animate in sequence (1.6s apart).

### State you need
```tsx
// The real data
const [streak, setStreak] = useState({ current_streak: 0, longest_streak: 0, total_days_read: 0 })

// Frozen display values — hold old value during animation
const [displayStreak, setDisplayStreak] = useState<number | null>(null)
const [displayCompletion, setDisplayCompletion] = useState<number | null>(null)
const [displayDaysRead, setDisplayDaysRead] = useState<number | null>(null)

// Keys to retrigger fade-in animation
const [streakFadeKey, setStreakFadeKey] = useState(0)
const [completionFadeKey, setCompletionFadeKey] = useState(0)
const [daysReadFadeKey, setDaysReadFadeKey] = useState(0)

// Delta badge
interface DeltaState { field: 'streak'|'completion'|'daysRead'; value: string; key: number }
const [delta, setDelta] = useState<DeltaState | null>(null)
const deltaKeyRef = useRef(0)

function showDelta(field: DeltaState['field'], value: string) {
  deltaKeyRef.current += 1
  setDelta({ field, value, key: deltaKeyRef.current })
  setTimeout(() => setDelta(null), 1500)
}
```

### Animating a stat increase
```tsx
// 1. Freeze display at old value
setDisplayStreak(oldStreak)

// 2. After a delay, show delta badge and unfreeze
setTimeout(() => {
  showDelta('streak', '+1')
  setDisplayStreak(null)           // show real value
  setStreakFadeKey(k => k + 1)     // trigger fade-in animation
}, 300)
```

**Critical rule:** `setDisplayStreak(null)` MUST always be called, even if the delta is 0. A conditional clear is a bug — the display stays frozen forever.

### Stat card JSX
```tsx
<div className="bg-white dark:bg-[#162032] rounded-2xl shadow p-5 border border-amber-100 dark:border-[#2a3a52] text-center">
  {/*
    CRITICAL: wrap ONLY the number in `relative inline-block`.
    The delta badge uses absolute positioning relative to this wrapper.
    If you put `relative` on the card div instead, the badge positions
    relative to the whole card and either clips or appears in the wrong place.
  */}
  <div className="relative inline-block">
    {delta?.field === 'streak' && (
      <span
        key={delta.key}
        className="absolute -top-1 -right-8 text-amber-500 dark:text-[#d4a847] font-bold text-sm pointer-events-none z-10"
        style={{ animation: 'delta-rise 1.5s cubic-bezier(0.16, 1, 0.3, 1) forwards, delta-fade 1.5s ease forwards', willChange: 'transform, opacity' }}
      >
        {delta.value}
      </span>
    )}
    <p
      key={streakFadeKey}
      className="text-3xl font-bold text-amber-800 dark:text-[#d4a847]"
      style={streakFadeKey > 0 ? { animation: 'number-fade-in 0.35s ease both' } : undefined}
    >
      {displayStreak !== null ? displayStreak : streak}
    </p>
  </div>
  <p className="text-xs text-slate-400 dark:text-[#8a9ab5] mt-1 font-medium uppercase tracking-wide">Day Streak</p>
</div>
```

### Wiring a new stat card end-to-end (complete pattern)

Use this checklist any time you add a new animated stat card to any page. Every step is required.

**1 — State:**
```tsx
const [myValue, setMyValue]         = useState(0)
const [displayMyValue, setDisplay]  = useState<number | null>(null)
const [myFadeKey, setMyFadeKey]     = useState(0)
const [delta, setDelta]             = useState<{ field: string; value: string; key: number } | null>(null)
const deltaKeyRef  = useRef(0)
const myValueRef   = useRef(0)      // ref for safe unmount access
const dataLoadedRef = useRef(false) // guard: only save lastSeen when data actually loaded
```

**2 — Keep ref in sync:**
```tsx
useEffect(() => { myValueRef.current = myValue }, [myValue])
```

**3 — showDelta helper:**
```tsx
function showDelta(field: string, value: string) {
  deltaKeyRef.current += 1
  setDelta({ field, value, key: deltaKeyRef.current })
  setTimeout(() => setDelta(null), 1500)
}
```

**4 — Fetch, compare, animate:**
```tsx
useEffect(() => {
  const lastSeen = parseInt(localStorage.getItem('lastSeenMyValue') ?? '-1')

  authFetch('/api/my-endpoint')
    .then(r => r.json())
    .then(data => {
      const current = data.my_value ?? 0
      setMyValue(current)
      myValueRef.current = current
      dataLoadedRef.current = true

      if (lastSeen >= 0 && current > lastSeen) {
        setDisplay(lastSeen)               // freeze at old value
        setTimeout(() => {
          showDelta('myField', `+${current - lastSeen}`)
          setDisplay(null)                 // unfreeze
          setMyFadeKey(k => k + 1)        // fade new value in
        }, 300)
      }
    })
    .catch(() => {})

  // Save baseline on unmount (use ref — closure captures stale state)
  return () => {
    if (dataLoadedRef.current) {
      localStorage.setItem('lastSeenMyValue', String(myValueRef.current))
    }
  }
}, [])
```

**5 — JSX (relative inline-block around the number only):**
```tsx
<div className="bg-white dark:bg-[#162032] rounded-2xl shadow p-5 border border-amber-100 dark:border-[#2a3a52] text-center">
  <div className="relative inline-block">
    {delta?.field === 'myField' && (
      <span
        key={delta.key}
        className="absolute -top-1 -right-8 text-amber-500 dark:text-[#d4a847] font-bold text-sm pointer-events-none z-10"
        style={{ animation: 'delta-rise 1.5s cubic-bezier(0.16, 1, 0.3, 1) forwards, delta-fade 1.5s ease forwards', willChange: 'transform, opacity' }}
      >
        {delta.value}
      </span>
    )}
    <p
      key={myFadeKey}
      className="text-3xl font-bold text-amber-800 dark:text-[#d4a847]"
      style={myFadeKey > 0 ? { animation: 'number-fade-in 0.35s ease both' } : undefined}
    >
      {displayMyValue !== null ? displayMyValue : myValue}
    </p>
  </div>
  <p className="text-xs text-slate-400 mt-1 font-medium uppercase tracking-wide">Label</p>
</div>
```

**Import:** always `import { authFetch } from '../lib/api'` — not `../utils/authFetch`, not `../api`.

### Show `—` when no data
```tsx
// Never show 0% when there's no active plan — show — instead
{activePlans.length === 0 ? '—' : `${displayCompletion ?? completion}%`}
```

### Milestone celebration banner
```tsx
// Trigger
const milestones = [7, 30, 100]
for (const m of milestones) {
  if (prev < m && current_streak >= m) {
    setMilestone(m)
    setTimeout(() => setMilestone(null), 3500)
    break
  }
}

// Banner JSX (via createPortal)
{milestone && createPortal(
  <div
    className="fixed inset-x-0 top-8 flex justify-center z-[9997] pointer-events-none"
    style={{ animation: 'milestone-pop 3.5s ease forwards' }}
  >
    <div className="bg-amber-700 text-white px-6 py-3 rounded-2xl shadow-xl text-lg font-bold">
      🔥 {milestone} Day Streak!
    </div>
  </div>,
  document.body
)}
```

### Fetching streak from backend
```tsx
function fetchStreak() {
  authFetch('/api/streak')
    .then(r => r.json())
    .then((data: StreakData) => {
      const prev = prevStreakRef.current
      prevStreakRef.current = data.current_streak
      setStreak(data)
      // milestone check here
    })
}
```

---

## 5. Progress Tracker / Plan Cards

### What it does
Each plan card shows the plan name, daily goal, progress bar, and a "Track" button that expands a day tracker panel. Inside the panel: day navigator (◀ ▶), current day status, mark/unmark button, mini progress bar.

### Plan card structure
```tsx
interface CustomPlan {
  id: number
  name: string
  daily_goal: string | null
  total_days: number | null
  start_date: string | null
  scripture_note: string | null
  completed_at: string | null
}
```

### Current day number
```tsx
function currentDayNumber(plan: CustomPlan): number {
  if (!plan.start_date) return 1
  const [y, m, d] = plan.start_date.split('-').map(Number)
  const start = new Date(y, m - 1, d)
  const today = new Date()
  today.setHours(0, 0, 0, 0)
  start.setHours(0, 0, 0, 0)
  const diff = Math.floor((today.getTime() - start.getTime()) / 86400000)
  return Math.min(Math.max(diff + 1, 1), plan.total_days ?? 999)
}
```

### Per-day label formatting
```tsx
function perDayLabel(plan: CustomPlan): string {
  const match = plan.daily_goal?.match(/^(\d+)\s*(chapters?|verses?)/i)
  if (!match) return plan.daily_goal ?? ''
  const total = parseInt(match[1])
  const perDay = total / (plan.total_days ?? 1)
  if (Number.isInteger(perDay)) return `${perDay} chapter${perDay !== 1 ? 's' : ''}/day · ${plan.total_days} days`
  return `${total} chapters · ${plan.total_days} days`
}
```

### Tracker panel (AnimatePresence expand/collapse)
```tsx
const [tracking, setTracking] = useState(false)
const [completedDays, setCompletedDays] = useState<Set<number>>(new Set())

// Load days on first open
async function handleTrack() {
  if (!tracking) {
    const res = await authFetch(`/api/custom-plans/${plan.id}/days`)
    const days: { day_number: number }[] = await res.json()
    setCompletedDays(new Set(days.map(d => d.day_number)))
  }
  setTracking(t => !t)
}

// JSX
<AnimatePresence>
  {tracking && (
    <motion.div
      initial={{ height: 0, opacity: 0 }}
      animate={{ height: 'auto', opacity: 1 }}
      exit={{ height: 0, opacity: 0 }}
      transition={{ duration: 0.3, ease: [0.2, 0, 0.3, 1] }}
      className="overflow-hidden"
    >
      <div className="pt-3 border-t border-amber-100 dark:border-[#2a3a52]">
        {/* Day navigator */}
        <div className="flex items-center justify-between">
          <button onClick={() => setViewDay(d => Math.max(1, d - 1))}>◀</button>
          <span>Day {viewDay} of {plan.total_days}</span>
          <button onClick={() => setViewDay(d => Math.min(plan.total_days!, d + 1))}>▶</button>
        </div>

        {/* Mark/Unmark */}
        {completedDays.has(viewDay) ? (
          <button onClick={() => handleUnmark(viewDay)}>Unmark</button>
        ) : (
          <button onClick={() => handleMark(viewDay)}>Mark as Read</button>
        )}

        {/* Mini progress bar */}
        <div className="h-1.5 bg-amber-100 dark:bg-[#2a3a52] rounded-full mt-3">
          <div
            className="h-full bg-amber-700 dark:bg-[#d4a847] rounded-full transition-all"
            style={{ width: `${(completedDays.size / (plan.total_days ?? 1)) * 100}%` }}
          />
        </div>
      </div>
    </motion.div>
  )}
</AnimatePresence>
```

### Mark / Unmark API calls
```tsx
async function handleMark(day: number) {
  await authFetch(`/api/custom-plans/${plan.id}/days/${day}`, { method: 'POST' })
  setCompletedDays(prev => new Set(prev).add(day))
}

async function handleUnmark(day: number) {
  await authFetch(`/api/custom-plans/${plan.id}/days/${day}`, { method: 'DELETE' })
  setCompletedDays(prev => { const s = new Set(prev); s.delete(day); return s })
}
```

### Plan creation form (progressive disclosure)
```tsx
const [planName, setPlanName] = useState('')
const [totalDays, setTotalDays] = useState('')
const [dailyGoal, setDailyGoal] = useState('')

// Extra fields only appear after name is typed
{planName.trim() && (
  <motion.div
    initial={{ opacity: 0, height: 0 }}
    animate={{ opacity: 1, height: 'auto' }}
    exit={{ opacity: 0, height: 0 }}
    transition={{ duration: 0.25 }}
  >
    <input placeholder="Total days" value={totalDays} onChange={e => setTotalDays(e.target.value)} />
    <input placeholder="Daily goal (e.g. 2 chapters)" value={dailyGoal} onChange={e => setDailyGoal(e.target.value)} />
  </motion.div>
)}

async function handleSavePlan() {
  const res = await authFetch('/api/custom-plans', {
    method: 'POST',
    body: JSON.stringify({
      name: planName.trim(),
      daily_goal: dailyGoal.trim() || null,
      total_days: totalDays ? parseInt(totalDays) : null,
      start_date: new Date().toISOString().split('T')[0],
    }),
  })
  const created = await res.json()
  setPlans(prev => [created, ...prev])
  setPlanName('')
}
```

### Archive (completed plans)
Plans with `completed_at != null` are separated at render time:
```tsx
const activePlans = plans.filter(p => !p.completed_at)
const archivedPlans = plans.filter(p => p.completed_at)
```
No separate API call — same `/api/custom-plans` returns all, filter on frontend.

### Plan card fade-out when completing
```tsx
const [fadingOutPlanId, setFadingOutPlanId] = useState<number | null>(null)

// When plan is completed:
setFadingOutPlanId(planId)
setTimeout(() => {
  setPlans(prev => prev.filter(p => p.id !== planId))
  setFadingOutPlanId(null)
  setNoPlanFadeIn(true)
  setTimeout(() => setNoPlanFadeIn(false), 800)
}, 550)

// On the card:
className={`transition-opacity duration-500 ${fadingOutPlanId === plan.id ? 'opacity-0' : 'opacity-100'}`}
```

---

## 6. Reading Log

### What it does
Structured book + chapter picker. User selects a book via `BookAutocomplete` typeahead, then sets chapter start/end. POSTs to `/api/log`. Used in both "Log a Reading" (Dashboard) and the "Read Today" modal.

**The old free-text passage input no longer exists. Do not revert to it.**

### Request body (frontend → backend)
```tsx
await authFetch('/api/log', {
  method: 'POST',
  body: JSON.stringify({ book_name: logBook, chapter_start: chStart, chapter_end: chEnd }),
})
```

### Backend (`reading_log.py`)
Accepts `{book_name, chapter_start, chapter_end}`. Auto-generates `passage = f"{book_name} {chapter_start}–{chapter_end}"`. Calls `log_chapters_bulk` and returns `{book_complete, book_name}` so the frontend can trigger the book-complete animation.

### Log form (Dashboard) — shared UI for Log a Reading and Read Today modal
```tsx
<BookAutocomplete
  value={logBook}
  onChange={name => {
    setLogBook(name)
    if (name) { setLogChapterStart('1'); setLogChapterEnd(String(BOOK_CHAPTERS[name] ?? 1)) }
  }}
/>
{logBook && (
  <div className="flex gap-2">
    <div className="flex-1">
      <label className="text-xs text-slate-400 mb-1 block">Start chapter</label>
      <input type="number" value={logChapterStart} onChange={e => setLogChapterStart(e.target.value)}
        min={1} max={BOOK_CHAPTERS[logBook] ?? 1} className="..." />
    </div>
    <div className="flex-1">
      <label className="text-xs text-slate-400 mb-1 block">End chapter</label>
      <input type="number" value={logChapterEnd} onChange={e => setLogChapterEnd(e.target.value)}
        min={parseInt(logChapterStart)||1} max={BOOK_CHAPTERS[logBook] ?? 1} className="..." />
    </div>
  </div>
)}
```
Auto-fill: selecting a book via `BookAutocomplete` sets start=1, end=book max chapters.

### BookAutocomplete component (`components/BookAutocomplete.tsx`)
- Typeahead input filtering all 66 book names as you type
- Dropdown below input, max-h-52 scrollable, styled to match app
- Shows `{book.name}` + `{book.chapters} ch` per option
- Keyboard nav: arrow keys move highlight, Enter selects, Escape closes
- Click outside reverts query to last confirmed value
- Uses `onPointerDown` (not `onClick`) on list items to prevent blur-before-click race
- `style={{ marginTop: '4px' }}` on container (4px below the input)

---

## 7. Bookshelf Grid

### What it does
Two sections (Old Testament, New Testament), each showing all books as vertical spine tiles. Incomplete books are gray; completed books are solid amber; partially-read books show an animated amber fill that rises from the bottom. Tapping a spine opens a detail modal with a 3D flip animation.

### Data sources
- `GET /api/books` → completed books map (book fully read)
- `POST /api/chapters/backfill` → one-time per mount, backfills chapter_readings from completed_books + reading_log
- `GET /api/chapters/by-book` → `[{book_name, read_count}]` grouped aggregate — source of truth for partial fills

Both fetches run on every mount (no localStorage gate). Backfill runs first, then chapters/by-book.

### Spine component — layered div design

**NEVER use `background: linear-gradient(...)` to represent a fill percentage.** CSS cannot transition between two different gradient values — the fill will appear instantly, never animate. Use a layered div instead:

```tsx
const FILL_DELAY_MS = 150    // ms after data loads before animation starts
const FILL_DURATION_MS = 900 // ms for each spine's fill animation
const FILL_STAGGER_MS = 40   // ms between each book's animation

function Spine({ book, isComplete, readCount, fillsReady, animIdx, onClick }) {
  const totalChapters = BOOK_CHAPTERS[book.name] ?? 1
  const rawPct = isComplete ? 100 : Math.min(100, Math.round((readCount / totalChapters) * 100))
  // Minimum 12% visual fill so even 1 chapter of a long book is visible
  const targetPct = rawPct > 0 && rawPct < 12 ? 12 : rawPct

  const amberColor = isDark ? '#d4a847' : '#b45309'
  const emptyColor = isDark ? '#1e2d40' : '#e5e7eb'

  // Complete: always 100%. Partial: 0 until fillsReady, then targetPct. Empty: 0.
  const fillHeight = isComplete ? 100 : (fillsReady ? targetPct : 0)
  const shouldAnimate = animIdx >= 0  // only partial books

  return (
    <button
      style={{
        position: 'relative', width: 30, height: 88,
        background: emptyColor, overflow: 'hidden',
        writingMode: 'vertical-rl', display: 'flex',
        alignItems: 'center', justifyContent: 'center',
      }}
    >
      {/* Amber fill — rises from the bottom via height transition */}
      <div style={{
        position: 'absolute', bottom: 0, left: 0, right: 0,
        height: `${fillHeight}%`,
        background: amberColor,
        transition: shouldAnimate
          ? `height ${FILL_DURATION_MS}ms cubic-bezier(0,0,0.2,1) ${animIdx * FILL_STAGGER_MS}ms`
          : 'none',
      }} />
      {/* Text above the fill */}
      <span style={{ position: 'relative', zIndex: 1, fontSize: 9, fontWeight: 700, color: textColor }}>
        {book.abbr}
      </span>
    </button>
  )
}
```

### Animation state pattern

```tsx
const [fillsReady, setFillsReady] = useState(false)

// animOrder: only partial books get a stagger index, in canonical order
const animOrder = useMemo(() => {
  const order = new Map<string, number>()
  let idx = 0
  for (const book of [...OT_BOOKS, ...NT_BOOKS]) {
    const rc = chaptersRead.get(book.name) ?? 0
    if (rc > 0 && !completedBooks.has(book.name)) order.set(book.name, idx++)
  }
  return order
}, [chaptersRead, completedBooks])

// After data loads, wait FILL_DELAY_MS then trigger fills
useEffect(() => {
  if (animOrder.size === 0) return
  const t = setTimeout(() => setFillsReady(true), FILL_DELAY_MS)
  return () => clearTimeout(t)
}, [animOrder])
```

Since navigation uses plain `<a href>` tags (full reloads), `fillsReady` resets to `false` on every tab visit — animation replays automatically. No explicit reset needed.

### ShelfRows component
Pass `fillsReady` and `animOrder` down; each Spine gets `animIdx={animOrder.get(book.name) ?? -1}`.

### Detail modal (flip animation, via createPortal)
```tsx
const [closing, setClosing] = useState(false)

function handleClose() {
  setClosing(true)
  setTimeout(() => {
    setClosing(false)
    setSelectedBook(null)
  }, 280)
}

{selectedBook && createPortal(
  <div
    className="fixed inset-0 z-[9998] flex items-center justify-center"
    style={{ backdropFilter: 'blur(6px)', background: 'rgba(0,0,0,0.45)' }}
    onClick={handleClose}
  >
    <div
      className="bg-white dark:bg-[#162032] rounded-xl p-6 w-full max-w-sm mx-4"
      style={{ animation: closing ? 'book-flip-out 0.28s ease both' : 'book-flip-in 0.35s ease both' }}
      onClick={e => e.stopPropagation()}
    >
      {/* chapter progress bar, mark complete button, log entries */}
    </div>
  </div>,
  document.body
)}
```

### Chapter progress bar (inside detail modal)
```tsx
const selectedTotal = BOOK_CHAPTERS[selectedBook] ?? 1
const selectedRead = chaptersRead.get(selectedBook) ?? 0
const selectedPct = Math.min(100, Math.round((selectedRead / selectedTotal) * 100))

<div className="h-2 bg-gray-100 dark:bg-[#1e2d40] rounded-full overflow-hidden">
  <div
    className="h-full bg-amber-700 dark:bg-[#d4a847] rounded-full transition-all duration-500"
    style={{ width: `${selectedPct}%`, minWidth: selectedRead > 0 ? '6px' : '0' }}
  />
</div>
```

### `formatPassage` helper
```tsx
function formatPassage(passage: string): string {
  // "Psalms 43–43" (same start & end) → "Psalms 43"
  const m = passage.match(/^(.+?)\s+(\d+)[–\-](\d+)$/)
  if (m && m[2] === m[3]) return `${m[1]} ${m[2]}`
  return passage
}
```
Used in log entry display inside the detail modal.

### Detail modal (flip animation, via createPortal)
```tsx
const [closing, setClosing] = useState(false)

function handleClose() {
  setClosing(true)
  setTimeout(() => {
    setClosing(false)
    setSelectedBook(null)
  }, 280)
}

{selectedBook && createPortal(
  <div
    className="fixed inset-0 z-[9998] flex items-center justify-center"
    style={{ backdropFilter: 'blur(6px)', background: 'rgba(0,0,0,0.45)' }}
    onClick={handleClose}
  >
    <div
      className="bg-white dark:bg-[#162032] rounded-xl p-6 w-full max-w-sm mx-4"
      style={{ animation: closing ? 'book-flip-out 0.28s ease both' : 'book-flip-in 0.35s ease both' }}
      onClick={e => e.stopPropagation()}
    >
      <h2 className="text-xl font-bold">{selectedBook}</h2>
      <button onClick={handleToggle}>
        {completedBooks.has(selectedBook) ? 'Mark as Unread' : 'Mark as Completed'}
      </button>
      {/* log entries for this book */}
    </div>
  </div>,
  document.body
)}
```

### Toggle complete / incomplete
```tsx
async function handleToggle() {
  if (completedBooks.has(selectedBook)) {
    await authFetch(`/api/books/${encodeURIComponent(selectedBook)}`, { method: 'DELETE' })
    setCompletedBooks(prev => { const m = new Map(prev); m.delete(selectedBook); return m })
  } else {
    const res = await authFetch(`/api/books/${encodeURIComponent(selectedBook)}`, { method: 'POST' })
    const data = await res.json()
    setCompletedBooks(prev => new Map(prev).set(selectedBook, data.completed_at))
    // Also update chaptersRead to 100% so spine immediately shows full
    setChaptersRead(prev => new Map(prev).set(selectedBook, BOOK_CHAPTERS[selectedBook] ?? 0))
    const bookName = selectedBook
    handleClose()
    setCompletionAnimBook(bookName)  // triggers animation
  }
}
```

---

## 8. Modals

### Standard blur modal (any confirmation or input)
```tsx
{isOpen && createPortal(
  <div
    className="fixed inset-0 z-[9998] flex items-center justify-center"
    style={{ backdropFilter: 'blur(6px)', background: 'rgba(0,0,0,0.45)' }}
    onClick={onClose}  // click backdrop to close
  >
    <div
      className="bg-white dark:bg-[#162032] border border-amber-100 dark:border-[#2a3a52]
                 rounded-xl p-6 w-full max-w-sm mx-4 shadow-xl"
      style={{ animation: 'fade-up 0.25s ease both' }}
      onClick={e => e.stopPropagation()}  // click inside = don't close
    >
      <h3 className="text-lg font-semibold text-slate-800 dark:text-[#f0e6d0] mb-4">
        Title
      </h3>
      {/* content */}
      <div className="flex gap-2 mt-4">
        <button onClick={onClose} className="flex-1 py-2 rounded-lg border ...">Cancel</button>
        <button onClick={onConfirm} className="flex-1 py-2 rounded-lg bg-amber-800 text-white ...">Confirm</button>
      </div>
    </div>
  </div>,
  document.body
)}
```

### Input modal (e.g., "Read Today" chapter count)
Same pattern — add an `<input>` inside, handle `onKeyDown` for Enter key.

```tsx
<input
  ref={inputRef}  // autoFocus via useEffect
  type="number"
  value={value}
  onChange={e => setValue(e.target.value)}
  onKeyDown={e => e.key === 'Enter' && handleConfirm()}
  className="w-full px-3 py-2 rounded-lg border border-amber-200 dark:border-[#2a3a52]
             bg-amber-50 dark:bg-[#0d1b2a] text-slate-800 dark:text-[#f0e6d0]"
/>
```

### Show/hide toggle (password hint, reveal patterns)
```tsx
// ALWAYS stack vertically — toggle below the content, never beside it
<div className="flex flex-col items-center gap-1">
  <span>{showHint ? hintText : '••••••••'}</span>
  <button
    onClick={() => setShowHint(h => !h)}
    className="text-xs text-amber-700 dark:text-[#d4a847] underline"
  >
    {showHint ? 'Hide' : 'Show'}
  </button>
</div>
```

**Rule:** show/hide toggles ALWAYS go below the content with `flex-col items-center`. Never beside with `flex items-center gap-2`.

---

## 9. Full-Screen Animations

### Where they live
```
frontend/public/
  reading-check.html     ← triggered by marking a non-last plan day
  plan-complete.html     ← triggered by marking the last day of a plan
  book-complete.html     ← triggered by completing a book in Bookshelf
```

Vite serves these as static files. Accessible at `/animation-name.html` in dev and prod.

### HTML animation file template
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Animation</title>
  <link href="https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400;0,600;0,700&display=swap" rel="stylesheet"/>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body { width: 100vw; height: 100vh; overflow: hidden; background: #0d1b2a; font-family: 'EB Garamond', Georgia, serif; }
    #scene { position: fixed; inset: 0; background: #0d1b2a; overflow: hidden; }
  </style>
</head>
<body>
  <div id="scene"></div>

  <script>
    // ── TIMING CONSTANTS ──────────────────────────────────────
    const T = {
      fadeIn: 600,
      hold:   400,
      // add all durations here
    }

    // ── EASING FUNCTIONS ──────────────────────────────────────
    const ease = {
      outCubic:   t => 1 - Math.pow(1 - t, 3),
      inOutCubic: t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2,3)/2,
      outQuart:   t => 1 - Math.pow(1 - t, 4),
      // Camera slide: ease-in 35%, ease-out 65%
      slide: t => t < 0.35
        ? Math.pow(t / 0.35, 3) * 0.45
        : 0.45 + (1 - Math.pow(1 - (t - 0.35) / 0.65, 3)) * 0.55,
    }

    // ── ANIMATION PRIMITIVES ──────────────────────────────────
    // ALWAYS use setTimeout — rAF is throttled in background tabs
    function wait(ms) { return new Promise(r => setTimeout(r, ms)) }

    function animateValue(from, to, dur, easeFn, onUpdate) {
      return new Promise(resolve => {
        const start = performance.now()
        function tick() {
          const t = Math.min((performance.now() - start) / dur, 1)
          onUpdate(from + (to - from) * easeFn(t))
          if (t < 1) setTimeout(tick, 16); else resolve()
        }
        setTimeout(tick, 16)
      })
    }

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

    // ── CSS TRANSITION REFLOW FIX ─────────────────────────────
    // REQUIRED whenever you set transition + new value from JS:
    //   el.style.transition = 'opacity 0.6s ease'
    //   void el.offsetHeight   ← forces reflow — DO NOT REMOVE
    //   el.style.opacity = '1'

    // ── SEQUENCE ──────────────────────────────────────────────
    async function run() {
      // Stage 1 ...
      // Stage 2 ...
      // Signal React to dismiss:
      window.parent.postMessage({ type: 'animationDone' }, '*')
    }

    run()
  </script>
</body>
</html>
```

### Wiring an animation into React (4 steps)

**Step 1 — State:**
```tsx
const [animBook, setAnimBook] = useState<string | null>(null)
```

**Step 2 — Listen for done message:**
```tsx
useEffect(() => {
  function onMessage(e: MessageEvent) {
    if (e.data?.type === 'bookCompleteAnimationDone') setAnimBook(null)
  }
  window.addEventListener('message', onMessage)
  return () => window.removeEventListener('message', onMessage)
}, [])
```

**Step 3 — Trigger:**
```tsx
handleClose()           // close any open modal first
setAnimBook(bookName)   // this mounts the iframe
```

**Step 4 — Render iframe as portal:**
```tsx
{animBook && createPortal(
  <div className="fixed inset-0 z-[99999]">
    <iframe
      src={`/book-complete.html?book=${encodeURIComponent(animBook)}`}
      className="w-full h-full border-0"
      title="Book Complete"
    />
  </div>,
  document.body
)}
```

### Fade-from-black handoff after animation (book-complete pattern)
The iframe fades to black itself, then sends postMessage. React renders a black overlay instantly at opacity:1 (no transition), then on the next two rAF frames switches to opacity:0 with a CSS transition. The two black screens (iframe + React overlay) are indistinguishable — the user sees one seamless black screen that fades to the bookshelf.

```tsx
const [bookshelfFadeIn, setBookshelfFadeIn] = useState(false)

// When postMessage fires:
setBookshelfFadeIn(true)   // snap to black (transition: none)
setCompletionAnimBook(null) // remove iframe — bookshelf now visible underneath
requestAnimationFrame(() => {
  requestAnimationFrame(() => {
    setBookshelfFadeIn(false)  // triggers CSS fade to transparent
  })
})

// JSX (always rendered, not conditional — so it can transition out):
{createPortal(
  <div
    className="fixed inset-0 z-[99998] bg-black pointer-events-none"
    style={{
      opacity: bookshelfFadeIn ? 1 : 0,
      transition: bookshelfFadeIn ? 'none' : 'opacity 700ms ease',
    }}
  />,
  document.body
)}
```

Key: use `z-[99998]` (one below the iframe at z-[99999]) so it covers the bookshelf but not the iframe.

### Fireworks particle system pattern
Add a separate canvas layer above the main scene, below text overlays. Use `requestAnimationFrame` for the draw loop (fine here — user is actively watching). Control with `fwActive` flag so `stopFireworks()` is a clean single call.

```js
// HTML: <canvas id="fireworks-canvas" style="position:fixed;inset:0;z-index:45;pointer-events:none;opacity:0"></canvas>

const fwCanvas = document.getElementById('fireworks-canvas')
const fwCtx    = fwCanvas.getContext('2d')
fwCanvas.width = window.innerWidth; fwCanvas.height = window.innerHeight

const FW_COLORS = ['#ffd700','#ff6b6b','#4ecdc4','#ffe66d','#a8edea','#fff','#e056fd']
let fwParticles = [], fwActive = false, fwRafId = null, fwLaunchTimer = null

function spawnShell() {
  const x = window.innerWidth  * (0.15 + Math.random() * 0.70)
  const y = window.innerHeight * (0.10 + Math.random() * 0.50)
  const color = FW_COLORS[Math.floor(Math.random() * FW_COLORS.length)]
  for (let i = 0; i < 70; i++) {
    const angle = (Math.PI * 2 * i) / 70 + (Math.random() - 0.5) * 0.3
    const speed = 2.5 + Math.random() * 4.5
    fwParticles.push({ x, y, vx: Math.cos(angle)*speed, vy: Math.sin(angle)*speed,
      alpha: 1, color, radius: 2.2 + Math.random() * 1.8, decay: 0.013 + Math.random() * 0.008 })
  }
}

function fwTick() {
  fwCtx.clearRect(0, 0, fwCanvas.width, fwCanvas.height)
  fwParticles = fwParticles.filter(p => p.alpha > 0.02)
  for (const p of fwParticles) {
    p.x += p.vx; p.y += p.vy; p.vy += 0.07; p.vx *= 0.98; p.alpha -= p.decay
    fwCtx.globalAlpha = Math.max(0, p.alpha)
    fwCtx.fillStyle = p.color
    fwCtx.beginPath(); fwCtx.arc(p.x, p.y, p.radius, 0, Math.PI*2); fwCtx.fill()
  }
  fwCtx.globalAlpha = 1
  if (fwActive) fwRafId = requestAnimationFrame(fwTick)
}

function startFireworks() {
  fwActive = true; fwCanvas.style.opacity = '1'
  spawnShell(); fwRafId = requestAnimationFrame(fwTick)
  function schedule() { if (!fwActive) return; spawnShell(); fwLaunchTimer = setTimeout(schedule, 600 + Math.random() * 400) }
  fwLaunchTimer = setTimeout(schedule, 700)
}

function stopFireworks() {
  fwActive = false; clearTimeout(fwLaunchTimer); cancelAnimationFrame(fwRafId)
  fwCtx.clearRect(0, 0, fwCanvas.width, fwCanvas.height); fwCanvas.style.opacity = '0'
}

// In sequence: startFireworks() after green burst, stopFireworks() on click before black fade
```

Z-index layering: green burst (z-40) → fireworks canvas (z-45) → success text (z-50) → black fade (z-200).

### Canvas bookshelf pattern
For any animation drawing many objects on canvas:
```js
const canvas = document.getElementById('shelf-canvas')
const ctx = canvas.getContext('2d')

function resize() { canvas.width = window.innerWidth; canvas.height = window.innerHeight }
window.addEventListener('resize', resize); resize()

// Viewport-relative dimensions — compute once, reuse everywhere
const BH   = window.innerHeight * 0.90  // book height
const BW   = BH * 0.52                  // book width
const GAP  = BW * 0.38                  // gap between books
const SLOT = BW + GAP                   // stride per book
const SW   = Math.round(BW * 0.35)      // spine width
const SR   = Math.round(BW * 0.09)      // spine corner radius

// roundRect helper (browser fallback)
function roundRect(ctx, x, y, w, h, radii) {
  if (ctx.roundRect) { ctx.beginPath(); ctx.roundRect(x, y, w, h, radii); return }
  // ... manual fallback with quadraticCurveTo
}

// CRITICAL: when you change SW, update BOTH the roundRect clip AND the fillRect inside it
// The clip defines the shape. The fillRect paints the pixels. They must match.
```

### Color burst (clip-path circle expand)
```js
// Expand a color burst from a center point to fill the screen
const originX = canvas.width / 2
const originY = canvas.height / 2
const maxR = Math.hypot(originX, originY) + 20

burst.style.clipPath = `circle(0px at ${originX}px ${originY}px)`
await animateRaw(420, t => {
  const r = ease.outQuart(t) * maxR
  burst.style.clipPath = `circle(${r}px at ${originX}px ${originY}px)`
})
// Use outQuart — starts very fast, decelerates sharply. Feels like an explosion.
```

---

## 10. Authentication System

### Token flow
```
User registers/logs in
  → Backend returns { token, email, password_hint }
  → sessionStorage.setItem('token', token)       ← per-tab, clears on tab close
  → localStorage.setItem('userEmail', email)      ← persists
  → localStorage.setItem('passwordHint', hint)    ← persists

New tab opened → sessionStorage empty → RequireAuth redirects to /login
```

### `authFetch` helper — `frontend/src/lib/api.ts`
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
Use `authFetch` for every `/api/` call. Use bare `fetch` only for `/api/auth/login`, `/api/auth/register`, and `/api/health`.

### `RequireAuth` route guard — `frontend/src/App.tsx`
```tsx
function RequireAuth({ children }: { children: React.ReactNode }) {
  if (!sessionStorage.getItem('token')) return <Navigate to="/login" replace />
  return <>{children}</>
}

// Route structure:
<Routes>
  <Route path="/login" element={<Login />} />
  <Route path="/register" element={<Register />} />
  <Route path="/*" element={
    <RequireAuth>
      <Navbar dark={dark} onToggleDark={() => setDark(d => !d)} />
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/progress" element={<Progress />} />
        <Route path="/shelf" element={<Bookshelf />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </RequireAuth>
  } />
</Routes>
```

### Backend: `deps.py` — shared auth dependency
```python
from fastapi import Depends, Header, HTTPException, status
from sqlalchemy.orm import Session
from jose import jwt
from typing import Optional
from ..database import get_db
from ..models.models import User
from ..config import settings

ALGORITHM = "HS256"

def current_user(authorization: Optional[str] = Header(None), db: Session = Depends(get_db)) -> User:
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="Not authenticated")
    token = authorization.split(" ", 1)[1]
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
    except Exception:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user
```

### Backend: `auth.py` endpoints
```python
import uuid, bcrypt
from datetime import datetime, timedelta, timezone
from jose import jwt

ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_DAYS = 30

def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode(), hashed.encode())

def make_token(user_id: str) -> str:
    expire = datetime.now(timezone.utc) + timedelta(days=ACCESS_TOKEN_EXPIRE_DAYS)
    return jwt.encode({"sub": user_id, "exp": expire}, settings.secret_key, algorithm=ALGORITHM)

@router.post("/auth/register")
def register(body: RegisterRequest, db: Session = Depends(get_db)):
    email = body.email.strip().lower()
    if db.query(User).filter(User.email == email).first():
        raise HTTPException(status_code=409, detail="Email already registered")
    user = User(id=str(uuid.uuid4()), email=email, hashed_password=hash_password(body.password))
    db.add(user); db.commit(); db.refresh(user)
    return { "token": make_token(user.id), "email": user.email }

@router.post("/auth/login")
def login(body: LoginRequest, db: Session = Depends(get_db)):
    email = body.email.strip().lower()
    user = db.query(User).filter(User.email == email).first()
    if not user or not verify_password(body.password, user.hashed_password):
        raise HTTPException(status_code=401, detail="Invalid email or password")
    return { "token": make_token(user.id), "email": user.email }

@router.put("/auth/password")
def change_password(body: ChangePasswordRequest, user: User = Depends(current_user), db: Session = Depends(get_db)):
    if len(body.new_password) < 6:
        raise HTTPException(status_code=422, detail="Password must be at least 6 characters")
    user.hashed_password = hash_password(body.new_password)
    db.commit()
    return {"ok": True}
```

**Never use `passlib`** — it reads `bcrypt.__about__` which was removed in newer bcrypt versions. Use `bcrypt` directly.

### Profile page — settings pattern
```tsx
// Show email
<p>{localStorage.getItem('userEmail')}</p>

// Password hint with show/hide BELOW the text (never beside)
<div className="flex flex-col items-center gap-1">
  <span>{showHint ? hint : '••••••••'}</span>
  <button onClick={() => setShowHint(h => !h)}>
    {showHint ? 'Hide hint' : 'Show hint'}
  </button>
</div>

// Sign out
function handleSignOut() {
  sessionStorage.removeItem('token')
  localStorage.removeItem('userEmail')
  localStorage.removeItem('passwordHint')
  window.location.href = '/login'
}
```

### Per-account data isolation
Every data table needs `user_id TEXT`. Every router endpoint filters by it:
```python
# In every GET endpoint:
return db.query(CustomPlan).filter(CustomPlan.user_id == user.id).all()

# In every POST endpoint:
plan = CustomPlan(..., user_id=user.id)

# Validate ownership before operating on a child resource:
def _get_plan_for_user(plan_id, user_id, db):
    plan = db.query(CustomPlan).filter(CustomPlan.id == plan_id, CustomPlan.user_id == user_id).first()
    if not plan: raise HTTPException(status_code=404, detail="Plan not found")
    return plan
```

---

## 11. Backend: Models & Routers

### Project structure
```
backend/
  app/
    main.py           ← FastAPI app, includes all routers
    config.py         ← settings (secret_key, etc.)
    database.py       ← SQLAlchemy engine + get_db
    models/
      models.py       ← all SQLAlchemy models
    routers/
      deps.py         ← shared current_user dependency
      auth.py         ← register, login, change password
      custom_plans.py ← plan CRUD
      plan_days.py    ← day mark/unmark
      reading_log.py  ← log entries
      streak.py       ← streak computation
      books.py        ← bookshelf completion
      chapters.py     ← chapter-level tracking (source of truth for chapters read)
      health.py       ← health check
```

### CRITICAL: new models require a backend restart
`Base.metadata.create_all(bind=engine)` only runs at server startup. If you add a new SQLAlchemy model while the server is running, the table will NOT be created. Every endpoint that touches that table will return 500 errors. Kill the uvicorn process and restart it after adding any new model.

### Chapter tracking system (`chapters.py`)

**`ChapterReading` model (in `models.py`):**
```python
from sqlalchemy import UniqueConstraint

class ChapterReading(Base):
    __tablename__ = "chapter_readings"
    id             = Column(Integer, primary_key=True, autoincrement=True)
    user_id        = Column(String, ForeignKey("users.id"), nullable=False)
    book_name      = Column(String, nullable=False)
    chapter_number = Column(Integer, nullable=False)
    read_at        = Column(DateTime(timezone=True), server_default=func.now())
    __table_args__ = (
        UniqueConstraint('user_id', 'book_name', 'chapter_number', name='uq_user_chapter'),
    )
```

**Shared bulk logging helper — use `on_conflict_do_nothing()` for deduplication:**
```python
from sqlalchemy.dialects.sqlite import insert as sqlite_insert

def log_chapters_bulk(db: Session, user_id: str, book_name: str, chapters: List[int]) -> dict:
    if not chapters:
        return {"book_complete": False, "new_chapters": 0}
    before = db.query(ChapterReading).filter_by(user_id=user_id, book_name=book_name).count()
    stmt = sqlite_insert(ChapterReading).values([
        {"user_id": user_id, "book_name": book_name, "chapter_number": ch}
        for ch in chapters
    ]).on_conflict_do_nothing()
    db.execute(stmt)
    db.commit()
    after = db.query(ChapterReading).filter_by(user_id=user_id, book_name=book_name).count()
    total = BOOK_CHAPTERS.get(book_name, 0)
    return {"book_complete": total > 0 and after >= total, "new_chapters": after - before}
```
**Never use per-chapter SELECT loops** — one bulk INSERT is ~100× faster and handles deduplication via the unique constraint.

**Endpoints:**
- `POST /api/chapters/backfill` — walks all `CompletedBook` rows AND all `ReadingLogEntry` rows for user, calls `log_chapters_bulk` for each. Runs on every Bookshelf mount (no localStorage gate). Regex: `r'^(.+?)\s+(\d+)(?:[–\-](\d+))?$'` — matches both "Psalms 43" and "Psalms 43–45".
- `GET /api/chapters/by-book` → `[{book_name, read_count}]` — used by Bookshelf spine fill. Grouped COUNT query by book_name.
- `GET /api/chapters/count` → `{"count": N}` — source of truth for chapters read counter (Progress tab).
- `GET /api/chapters` → all chapter readings for user.
- `POST /api/chapters/bulk` → log a chapter range, returns `{book_complete, new_chapters}`.

**`log_chapters_bulk` is called from:**
- `books.py` `POST /api/books/{book_name}` — logs all chapters when a book is marked complete
- `plan_days.py` `POST /api/custom-plans/{id}/days/{day}` — logs chapters for that day's reading window
- `reading_log.py` `POST /api/log` — logs chapters from a structured log entry
- `chapters.py` backfill endpoint

**Backfill on every Bookshelf mount (no localStorage gate):**
```tsx
// In Bookshelf's loadChapters():
try { await authFetch('/api/chapters/backfill', { method: 'POST' }) } catch {}
// then fetch /api/chapters/by-book
```
Do NOT gate on `localStorage.chaptersBackfilled`. The gate causes the Bookshelf to skip backfill when Progress page has already set the flag — but the two pages use different flag names and the backfill endpoint is idempotent (`on_conflict_do_nothing`), so running it every mount is safe and fast.

### Model template
```python
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.sql import func
from ..database import Base

class MyModel(Base):
    __tablename__ = "my_table"
    id          = Column(Integer, primary_key=True, autoincrement=True)
    user_id     = Column(String, nullable=True)   # always add this
    name        = Column(String, nullable=False)
    created_at  = Column(DateTime(timezone=True), server_default=func.now())
```

### Router template
```python
from fastapi import APIRouter, Depends
from sqlalchemy.orm import Session
from pydantic import BaseModel
from ..database import get_db
from ..models.models import MyModel, User
from .deps import current_user

router = APIRouter()

class MyModelOut(BaseModel):
    id: int
    name: str
    class Config: from_attributes = True

@router.get("/my-resource", response_model=list[MyModelOut])
def list_items(db: Session = Depends(get_db), user: User = Depends(current_user)):
    return db.query(MyModel).filter(MyModel.user_id == user.id).all()

@router.post("/my-resource", response_model=MyModelOut)
def create_item(body: CreateRequest, db: Session = Depends(get_db), user: User = Depends(current_user)):
    item = MyModel(name=body.name, user_id=user.id)
    db.add(item); db.commit(); db.refresh(item)
    return item

@router.delete("/my-resource/{item_id}")
def delete_item(item_id: int, db: Session = Depends(get_db), user: User = Depends(current_user)):
    item = db.query(MyModel).filter(MyModel.id == item_id, MyModel.user_id == user.id).first()
    if item: db.delete(item); db.commit()
    return {"ok": True}
```

### Registering a new router in `main.py`
```python
from .routers import health, auth, custom_plans, plan_days, reading_log, streak, books, my_new_router

app.include_router(my_new_router.router, prefix="/api")
```

### SQLite migrations (ALTER TABLE)
SQLite supports adding columns but not dropping/modifying them:
```python
# Run once, directly:
from sqlalchemy import text
with engine.connect() as conn:
    conn.execute(text("ALTER TABLE my_table ADD COLUMN user_id TEXT"))
    conn.commit()
```
To change constraints or remove columns: DROP the table and recreate it (loses data — only do this in development or with a backup).

### `config.py` pattern
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    secret_key: str = "dev-secret-change-in-prod"
    database_url: str = "sqlite:///./app.db"
    class Config: env_file = ".env"

settings = Settings()
```

### `database.py` pattern
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase
from .config import settings

engine = create_engine(settings.database_url, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(bind=engine)

class Base(DeclarativeBase): pass

def get_db():
    db = SessionLocal()
    try: yield db
    finally: db.close()
```

---

## 12. Dark / Light Mode

### Setup (one-time)
```css
/* index.css */
@custom-variant dark (&:where(.dark, .dark *));
```
This makes `dark:` Tailwind variants respond to the `.dark` class on `<html>`, not system preference.

### App.tsx wiring
```tsx
const [dark, setDark] = useState(() => localStorage.getItem('theme') === 'dark')

useEffect(() => {
  document.documentElement.classList.toggle('dark', dark)
  localStorage.setItem('theme', dark ? 'dark' : 'light')
}, [dark])
```

### Checking dark mode in JS (for animations)
```tsx
const isDark = document.documentElement.classList.contains('dark')
// Pass to iframes via URL param: ?dark=${isDark ? '1' : '0'}
```

### Reactive dark mode in components that use inline styles (MutationObserver)
Tailwind `dark:` variants are reactive — they rerender with `.dark` class changes automatically. But inline `style={{}}` values are not. If a component derives colors from `document.documentElement.classList.contains('dark')` at render time, toggling dark mode won't trigger a re-render.

**Fix:** use `MutationObserver` to watch for class changes on `<html>` and drive a React state variable:
```tsx
const [isDark, setIsDark] = useState(document.documentElement.classList.contains('dark'))
useEffect(() => {
  const observer = new MutationObserver(() => {
    setIsDark(document.documentElement.classList.contains('dark'))
  })
  observer.observe(document.documentElement, { attributeFilter: ['class'] })
  return () => observer.disconnect()
}, [])
```
Pass `isDark` down as a prop instead of reading the DOM inside child components. Used in `Bookshelf.tsx` for the spine fill colors.

### Tailwind usage
Every color in every component uses both light and dark variants:
```tsx
className="bg-white dark:bg-[#162032] text-slate-800 dark:text-[#f0e6d0] border-amber-100 dark:border-[#2a3a52]"
```

---

## 13. localStorage Communication Layer

These keys are the cross-page communication layer. Never use React context or props for cross-page state — use localStorage.

| Key | Written by | Read by | Purpose |
|---|---|---|---|
| `token` | Login/Register | `authFetch`, RequireAuth | Auth token (sessionStorage, not localStorage) |
| `userEmail` | Login/Register | Profile | Display user's email |
| `passwordHint` | Login/Register | Profile | Show/hide password hint |
| `theme` | Navbar toggle | App.tsx mount | Dark/light mode persistence |
| `intro-seen` | IntroAnimation | App.tsx mount | Skip intro after first view |
| `readTodayDate` | Dashboard "Read Today" | Dashboard mount | Persist button green state |
| `lastStreakIncrementDate` | Dashboard mark | Dashboard | Guard against double-bumping streak |
| `pendingPassagesDelta` | Dashboard log actions | Progress mount | Queue chapters-read animation |
| `pendingBadgeAnimation` | Dashboard onMessage | Dashboard useEffect | Queue badge animation after navigation |
| `pendingCompletedPlanId` | Dashboard/Progress onMessage | Dashboard useEffect | Remove completed plan after badges |
| `lastSeenCompletion` | Dashboard unmount | Dashboard mount | Detect cross-session stat changes |
| `lastSeenDaysRead` | Dashboard unmount | Dashboard mount | Detect cross-session stat changes |
| `lastSeenStreak` | Dashboard unmount | Dashboard mount | Detect cross-session stat changes |
| `lastSeenProfileStreak` | Profile unmount | Profile mount | Detect streak change since last Profile visit |
| `lastSeenProfileBooks` | Profile unmount | Profile mount | Detect books-read change since last Profile visit |
| `chaptersBackfilled` | Progress mount (on success) | Progress mount | One-time backfill gate — only set when `res.ok` |
| `lastSeenChapters` | Progress `loadAll` (immediately) | Progress `loadAll` | Cross-visit chapters delta; saved inside loadAll not on unmount |

---

## 14. How All the Numbers Work

### Day Streak
**Counts:** consecutive calendar days with at least one plan day marked OR "Read Today" pressed.

**Reset rule:** streak resets to 0 only if you have gone a full calendar day without reading — meaning neither today NOR yesterday has a mark. If you read yesterday but haven't read today yet, the streak is still alive and shows yesterday's count. It only dies tomorrow if you don't read today.

**Backend (`streak.py`):**
```python
def _active_dates(db, user_id):
    active = set()
    today = date.today()
    for row in db.query(PlanDayCompletion).join(CustomPlan).filter(CustomPlan.user_id == user_id).all():
        plan = db.query(CustomPlan).get(row.plan_id)
        if plan and plan.start_date:
            y, m, d = plan.start_date.split('-')
            reading_date = date(int(y), int(m), int(d)) + timedelta(days=row.day_number - 1)
            if reading_date <= today:
                active.add(reading_date)
    return active

def _compute_streaks(active):
    if not active: return 0, 0
    today = date.today()
    yesterday = today - timedelta(days=1)
    current = 0
    # Count from today if active, or from yesterday if today hasn't been marked yet.
    # Streak only resets if you missed a full calendar day (neither today nor yesterday active).
    if today in active:
        check = today
        while check in active:
            current += 1
            check -= timedelta(days=1)
    elif yesterday in active:
        check = yesterday
        while check in active:
            current += 1
            check -= timedelta(days=1)
    ...
```

**Examples:**
- Read Monday, check streak Monday night → 1
- Open app Tuesday morning before reading → still 1 (yesterday was active)
- Mark Tuesday → 2
- Skip Tuesday entirely, check Wednesday → 0 (full day missed)

**Increments once per calendar day** via `lastStreakIncrementDate` localStorage guard.
**Displays:** Dashboard left stat card. Shows `🔥` when > 0. Shows "best: N" below.
**Milestones:** 7, 30, 100 days.

### Completion %
**Counts:** `(total marked days across active plans) / (total days across active plans) × 100`
```ts
function computeCompletion(completedCounts, plans) {
  const tracked = plans.filter(p => (p.total_days ?? 0) > 0)
  const totalDays = tracked.reduce((s, p) => s + p.total_days, 0)
  const done = tracked.reduce((s, p) => s + (completedCounts[p.id] ?? 0), 0)
  return Math.round((done / totalDays) * 100)
}
```
Shows `—` when no active plans.

### Days Read
**Counts:** total plan-day completions across all active plans. +1 per Mark, -1 per Unmark. Never resets.
Shows `—` when no active plans.

### Chapters Read (Progress tab)
Source of truth: `GET /api/chapters/count` → `{"count": N}`. This counts unique rows in `chapter_readings` for the user. The frontend does not compute this from log entries or plan days anymore.

**`lastSeenChapters` pattern (cross-visit delta animation):**
```tsx
// module-level — outside the component — prevents StrictMode double-animation
let _loadGen = 0

// inside the component's useEffect([], []):
const gen = ++_loadGen
async function loadAll() {
  // ... fetch /api/chapters/count ...
  const trueTotal = chaptersData.count ?? 0
  setTotalCompletedDays(trueTotal)

  if (gen !== _loadGen) return  // cancelled by StrictMode second invocation

  const lastSeen = parseInt(localStorage.getItem('lastSeenChapters') ?? '-1')
  localStorage.setItem('lastSeenChapters', String(trueTotal))  // save immediately — no unmount race
  if (lastSeen < 0) return  // first visit — skip animation
  const delta = trueTotal - lastSeen
  if (delta <= 0) return

  setDisplayPassages(lastSeen)
  setTimeout(() => {
    showPassagesDelta(`+${delta}`)
    setDisplayPassages(null)
    setPassagesFadeKey(k => k + 1)
  }, 600)
}
loadAll().catch(() => {})
```
**Key points:**
- Save `lastSeenChapters` INSIDE `loadAll` immediately after reading it — not on unmount. Unmount cleanup can fail to fire (page navigation, browser unload). Saving immediately means the baseline is always correct.
- Use `_loadGen` to cancel the first StrictMode invocation. Both run concurrently, both read `lastSeen` before either writes — without the guard, two animations play. The second invocation wins by having the highest `gen`.
- Do NOT use `dataLoadedRef` / unmount guard for this value anymore.

### How they relate
```
Mark plan day
  ├── Days Read + 1
  ├── Completion % increases
  ├── Chapters Read + parseChaptersPerDay(plan) [queued via pendingPassagesDelta]
  └── Streak + 1 (once per calendar day only)

Read Today + chapter count
  ├── Chapters Read + N
  ├── Streak + 1 (once per calendar day)
  └── pendingPassagesDelta += N

Mark last day of plan
  ├── All of the above
  ├── plan-complete.html animation plays
  ├── plan.completed_at set on backend
  └── Plan moves to Archive
```

---

## 15. Button Behaviors Reference

### "Mark as Read" (Dashboard plan card)
- Non-last day → triggers `reading-check.html` animation
- Last day → shows "Finish this plan?" confirmation → triggers `plan-complete.html`
- After mark: shows `✓ Read` + "Unmark" button

### "Unmark" (Dashboard)
- `DELETE /api/custom-plans/:id/days/:day`
- Decrements Days Read, Completion %, chapters (via `pendingPassagesDelta`)
- Clears streak increment if this was the only mark today

### "Read Today" (shown when no active plans)
- Green if `localStorage.readTodayDate === today`
- Opens modal with `BookAutocomplete` + chapter start/end inputs (NOT a number input)
- On confirm: `POST /api/log` with `{ book_name, chapter_start, chapter_end }`, bumps streak, queues chapters
- Hint in modal: "For multiple books, use Log a Reading below" — one book per Read Today press

### "Log Reading"
- Parses chapters from passage text, `POST /api/log`
- Shows "Logged!" for 3s

### "Mark as Completed" (Bookshelf)
- `POST /api/books/:book_name`
- Closes detail modal, triggers `book-complete.html` animation
- On animation done (`bookCompleteAnimationDone` postMessage): `bookshelfFadeIn = true` renders a black overlay at opacity:1 instantly, then two rAF frames later sets it to false, triggering a 700ms CSS fade to transparent — seamless handoff from the iframe's black screen to the bookshelf

### Dark/Light toggle (Navbar)
- Toggles `.dark` on `<html>`, persists in `localStorage.theme`

---

## 16. Animation Conditions Reference

| Animation | Trigger | Signal back |
|---|---|---|
| `reading-check.html` | Non-last plan day marked | `readingCheckComplete` |
| `plan-complete.html` | Last day of a plan marked | `planCompleteAnimationDone` |
| `book-complete.html` | Book marked complete in Bookshelf | `bookCompleteAnimationDone` |
| fade-from-black overlay | After `book-complete.html` ends | React-side `bookshelfFadeIn` state (700ms) |
| Milestone banner | Streak crosses 7, 30, or 100 | auto (3.5s timer) |
| Intro animation | First session load | `sessionStorage.intro-seen` |
| Intro dark-mode fade | After zoom blast, if dark mode | `blackOpacity` state → 400ms CSS transition → `onComplete()` |

---

## 17. Page & Route Structure

```
/           → Dashboard
/progress   → Progress
/shelf      → Bookshelf
/profile    → Profile
/login      → Login (outside RequireAuth)
/register   → Register (outside RequireAuth)
```

### Login/Register animation timing fix
Login and Register mount immediately on page load (before the intro completes). Their Framer Motion entrance animations fire and finish during the 5–7s intro. By the time the user can see the page, the animations are done — content appears instantly with no motion.

**Fix:** add `key={String(introComplete)}` to both routes. When `introComplete` flips false→true, the key changes and React remounts the components, replaying the Framer animations at exactly the right moment:
```tsx
<Route path="/login" element={<Login key={String(introComplete)} />} />
<Route path="/register" element={<Register key={String(introComplete)} />} />
```
For returning users where `introComplete` starts as `true`, the key is stable and animations play on first mount as expected.

### App.tsx layout
```tsx
<BrowserRouter>
  <Routes>
    <Route path="/login" element={<Login />} />
    <Route path="/register" element={<Register />} />
    <Route path="*" element={
      <RequireAuth>
        <Navbar dark={dark} onToggleDark={() => setDark(d => !d)} />
        <div className="pb-16"> {/* padding for fixed navbar */}
          <Routes>
            <Route path="/" element={<Dashboard />} />
            <Route path="/progress" element={<Progress />} />
            <Route path="/shelf" element={<Bookshelf />} />
            <Route path="/profile" element={<Profile />} />
          </Routes>
        </div>
      </RequireAuth>
    } />
  </Routes>
</BrowserRouter>
```

---

## 18. z-index Stack

Never violate this order:

```
9997  — Milestone banner
9998  — Blur modals (Read Today, confirmation, Bookshelf detail)
9999  — Full-screen animation iframes (reading-check, plan-complete)
99999 — Book completion animation iframe (must be above everything)
50    — Navbar
```

---

## 19. Rules & Lessons

### Always do

- **Restart the backend after adding any new SQLAlchemy model.** `create_all` only runs at startup. Tables added to models.py while the server is running will not exist until the next restart. Every endpoint touching the missing table returns 500.
- **Check `res.ok` before treating a fetch as successful.** `fetch()` resolves (doesn't throw) on 4xx/5xx responses. A try/catch around `authFetch(...)` catches network errors only — not HTTP errors. Always `if (res.ok)` before acting on the result.
- **Use `_loadGen` to prevent StrictMode double-animation.** Declare `let _loadGen = 0` at module level (outside the component). In `useEffect([], [])`: `const gen = ++_loadGen`. Inside the async function, check `if (gen !== _loadGen) return` before the animation logic. The first StrictMode invocation gets cancelled by the second.
- **Save `lastSeen` values immediately inside the load function, not on unmount.** Unmount cleanup does not reliably fire on browser navigation. Save the new baseline right after computing the delta, before the animation setTimeout.
- **Use `createPortal` for every modal and overlay.** Portals escape layout containers and avoid z-index fights.
- **Use `sessionStorage` for auth tokens.** Clears on tab close. Each tab is its own session.
- **Use `setTimeout(tick, 16)` in standalone HTML animations, never `rAF`.** rAF is throttled in background tabs.
- **Force a reflow when setting CSS transitions from JS.** `void el.offsetHeight` between transition and value assignment.
- **Add `user_id` to every data table from day one.** Retrofitting costs a full session.
- **Always unfreeze display-frozen values unconditionally.** A conditional unfreeze is a silent bug.
- **Search all references before removing a state variable.** Partial removal = white screen.
- **Use `flex-col items-center` for show/hide toggles.** Always stack vertically — never place the toggle beside the content.
- **Switch to `style={{ marginBottom: '20px' }}` after two Tailwind arbitrary value failures.** Don't retry.
- **Default to one-column stacked layout.** Never use a grid unless the rest of the page already uses one.
- **Hide scrollbars on overflow containers with `.no-scrollbar`.** Add to `index.css`:
  ```css
  .no-scrollbar::-webkit-scrollbar { display: none; }
  .no-scrollbar { scrollbar-width: none; }
  ```
  Apply to any `overflow-y-auto` container where the scrollbar track looks out of place. Used on reading log and archive list in Progress, and reading log in Bookshelf.

### Never do

- **Never use `passlib.CryptContext` with bcrypt.** Newer bcrypt removed `__about__`, causing a startup crash. Use `bcrypt.hashpw` / `bcrypt.checkpw` directly.
- **Never animate under an opaque overlay.** The animation plays invisibly. Hide the overlay before starting.
- **Never put two independent animations on the same `motion.div`.** They compete. Use separate wrapper elements.
- **Never commit a state update without a guaranteed unfreeze.** Every `setDisplayX(oldValue)` must have an unconditional `setDisplayX(null)`.
- **Never assume a grid layout.** One column until proven otherwise.
- **Never retry Tailwind arbitrary values more than twice.** It's not a typo — switch to inline style.

### Animation rules

- **Plan the full end-state first, then work backwards to timing.** Solving zoom, background, and entrance separately produces fighting animations.
- **`outQuart` for bursts** — starts very fast, decelerates sharply. Feels explosive.
- **`inOutCubic` for wipes** — symmetric. Feels deliberate.
- **Slide ease: ease-in 35%, ease-out 65%** — longer deceleration than acceleration. Camera glide feel.
- **80–100ms overlap between sequential animations** to create momentum continuity.
- **Hold times matter** — 400ms between key moments, 1800ms on "success" screens before auto-dismiss.

### Language patterns (how the user communicates)

| Phrase | Means |
|---|---|
| "The little animation" | The existing `fade-up` entrance animation |
| "Zoom more" | Increase the scale value only |
| "Still some [color] left" | Visual artifact at edges — fix scale/timing, not approach |
| "Really fast" | ~0.3–0.4s duration |
| "Slightly slower" | Add ~0.15s |
| "Camera goes into X" | Transform origin AT X, world expands around it |
| "Almost like the momentum from X" | Match easing curves + 80–100ms overlap |
| "Do this also" | Keep what works, layer new feature on top |
| "Do this instead" | Abandon current approach entirely |
| "Make it the same as the commander app" | Look at the MTG Commander app for reference |
| Layout preference | Always one column, never grid |
