# Development Retrospective

Lessons from building the Bible Tracker app — what worked, what didn't, what had to be repeated.

---

## Full-Screen Animations (`reading-check.html`, `plan-complete.html`)

### Things that took multiple messages

- **Zoom filling the screen** — the biggest struggle. "There's still some blue left" came up multiple times. Went through: initial attempt → scale 800 → scale 2000 → background color swap timing adjusted. ~4–5 exchanges just to kill the blue edges.
- **App content animating after intro** — the slide-up on the app itself had to be requested as a separate follow-up after the zoom was "done." Then it got broken by another change and had to be requested again ("put the little slide animation back").
- **Zoom speed** — had to come back separately to ask for the zoom duration to match the slide-up speed, even though both animations were built at the same time.
- **0.25s pause before menu** — requested separately. A natural part of the sequence that should've been included from the start.

### What worked first try

- Basic fade-up page entrance — one message, done.
- Bottom-to-top stagger on all pages — landed clean.
- Bible cover visual redesign (spine, back cover, page gutter shadow) — approved without pushback.

### Bugs introduced

- **`motion.div` conflict** — put both the float-in animation and the zoom on the same element, causing them to fight. Had to refactor into separate wrappers after the fact.
- **Removing `key` prop killed the slide animation** — Dashboard was mounting while hidden so Framer's animations fired off-screen and finished before the intro ended. Broke the feature first, then had to fix the root cause.
- **Text clipping through Bible pages** — missed the overflow check entirely, had to be pointed out.

### The pattern

The zoom sequence was hardest because "fill the screen" requires coordinating scale, timing, and background color swap together — and each piece was solved in isolation instead of as a unified system. Next time: plan the full visual end-state first, then work backwards to the timing of each piece.

---

## Book Completion Animation (`book-complete.html`)

### What worked first try

- **The full 9-stage sequence structure** — white flash, blue transition, Genesis appears, camera slides, books reveal, stop at target, green wipe, burst, success text — all mapped correctly from the description in one pass. No stage had to be reordered or reconceived.
- **All 66 books in canonical order** — indexed correctly, PSALMS_IDX found via `BOOKS.indexOf`, all rendering without issues.
- **Custom `slideEase` function** — slow acceleration → fast middle → smooth deceleration. Felt like a camera gliding through a shelf without any extra tuning.
- **Green burst via `clip-path: circle()` expanding** — the ripple from Psalms center to full screen worked first try.
- **`outQuart` easing on the burst** — felt energetic rather than mechanical. No iterations needed.
- **`inOutCubic` on the green wipe** — smooth fill that accelerates and decelerates. Approved immediately.
- **Viewport-relative dimensions** — `BH = window.innerHeight * 0.90`, `BW = BH * 0.52`, `GAP = BW * 0.38` — correct proportions on any screen size first try.
- **URL param `?book=` wiring** — `new URLSearchParams(location.search)` feeding `TARGET_IDX`. One pass, no bugs.
- **postMessage → React iframe dismissal** — `window.parent.postMessage({ type: 'bookCompleteAnimationDone' }, '*')` listened to in Bookshelf.tsx. Wired correctly first try.
- **Sequential async chain** — every stage `await`ed in order. No overlap bugs.

### What took multiple attempts

- **`requestAnimationFrame` throttled in background tab** — used rAF for `animateRaw` and `animateValue`. When the preview panel was in the background, rAF stopped firing entirely, so `appearAlpha` stayed at 0 indefinitely. Had to switch both functions to `setTimeout(tick, 16)`. Lesson: for standalone HTML animations that play while the user is looking at them, rAF is fine — but when verifying in a background preview panel, use setTimeout.
- **"Book Completed" text not appearing** — set `transition` and `opacity = '1'` in the same JS frame. Browser batched them, saw no start state, skipped the animation entirely. Fixed with `void element.offsetHeight` between setting transition and the new value. Same fix needed for `h1` transform. This is a known CSS transition gotcha — always force a reflow when you set transition and the new value in the same tick.
- **Book sizing** — went through 3 rounds: initial hardcoded 58px (too small) → 74% screen height → 90% screen height. Should have started at ~85–90% from the spec saying "almost filling the whole screen."
- **Book width** — 3 rounds: 0.36 → 0.38 → 0.52. The description said "wider" twice. Should have gone bolder on the first try.
- **Spine width not visually changing** — updated the `roundRect` clip to `BW * 0.35` but the `fillRect` inside the clip still had hardcoded `9`. The clip region expanded but only 9px was painted. Classic mistake: updated the shape but not the fill. Always update both the clip path AND the fillRect that uses the same dimension.
- **Spine rounding not visible** — added `[SR, 0, 0, SR]` radius to the spine strip, but the cover behind it had square corners filling the same area. The spine's rounded corners were covered by the brown cover underneath. Had to round the LEFT side of the entire book shape (`[SR, 3, 3, SR]`) so the blue background shows through the corners. Lesson: rounded corners on a layer are only visible if the layers beneath it are also cut, or if the background is transparent.
- **Gold page block clashing with rounding** — added an offset gold block for 3D page-edge depth. After spine rounding was fixed, it made the corners look strange. User asked to remove it. Should have anticipated that the offset block would conflict with rounded corner geometry.

### Bugs introduced

- **rAF in background tab** — animation silently froze. No error, no warning. Only caught by checking `appearAlpha` state after 111 seconds.
- **CSS transition batching** — text was invisible. `s.style.opacity` showed `"1"` but `getComputedStyle(s).opacity` was `"0"`. Misleading because the inline style looked correct.
- **Spine clip without matching fillRect** — change appeared to do nothing visually. No error. Only caught by reading the code more carefully.
- **Spine roundRect on wrong layer** — corners clipped correctly on the spine but hidden by the layer beneath. Needed to understand the full draw order to fix.

### Session 3 changes (white flash intro, fade-to-black outro, fireworks)

- **White flash replacing blue burst**: body starts white, a `#white-flash` overlay (z-index 200, opacity:1) fades out while bibles slide in simultaneously. Seamless because scene background (#0d1b2a) is already set — only the overlay needs to fade.
- **Simultaneous fade + slide**: started `animateRaw(bibleAppear, ...)` in a variable (not awaited), then awaited the white fade, then awaited the slide. Both run in parallel, no `Promise.all` needed.
- **Fade-to-black outro**: `#black-fade` overlay (z-index 200, opacity:0) animates to 1 on click before postMessage. Replaces the old "fade elements out" approach — now the entire screen goes black cleanly.
- **React-side black overlay handoff**: when `bookCompleteAnimationDone` fires, Bookshelf.tsx sets `bookshelfFadeIn = true` (renders black overlay at opacity:1 instantly via `transition: none`) then on the next two rAF frames sets it to false (triggering `transition: opacity 700ms ease`). The iframe's black screen and the React overlay are visually identical at handoff — the user sees one continuous black screen that then fades to the bookshelf.
- **Fireworks particle system**: separate `#fireworks-canvas` (z-index 45, above green burst z-40, below success text z-50). Controlled by `fwActive` flag. `startFireworks()` spawns first shell + schedules recurring launches every 600–1000ms via `setTimeout`. `stopFireworks()` clears the timer, cancels RAF, clears canvas. Uses `requestAnimationFrame` for the draw loop (acceptable here since the user is looking at the screen when fireworks are active).
- **"Make the flash white a bit slower"** = increase `flashFade` from 650 → 1100ms. User won't say a number — just "a bit slower."

### What made the animation smooth and seamless

- **`slideEase` splits at t=0.35**: ease-in for 35%, ease-out for 65%. This gives a longer deceleration than acceleration — matching how a physical camera feels when it comes to a stop.
- **`outQuart` for the burst**: higher power (4) means it starts very fast and decelerates sharply. Feels like an explosion that quickly settles, not a slow fade.
- **`inOutCubic` for the wipe**: symmetric acceleration and deceleration — the green fill feels deliberate, like liquid pouring.
- **Background becomes opaque before content appears**: flash fades out completely before the blue scene is revealed, and the bible appears after the blue is fully set. No competing visuals.
- **Hold time after "Book Completed"** (1800ms): gives the user time to read the text before the iframe auto-dismisses. Not rushed.
- **`void element.offsetHeight` reflow**: ensures CSS transitions actually animate instead of snapping. Without it, opacity/transform changes are instantaneous and jarring.
- **All dimensions relative to viewport**: `BH * 0.90`, `BW * 0.52`, etc. The animation scales to any device without looking broken.
- **Spine darker green (`#15803d`) vs cover green (`#22c55e`)**: the contrast between the two greens during the wipe gives the book visual depth — the spine reads as a separate physical surface from the cover.

---

## Bookshelf Spine Fill Animation (in-app React, `Bookshelf.tsx`)

### What was built
Bottom-to-top amber fill that animates when the Bookshelf tab opens. Each partially-read book spine starts empty (gray), waits for a short delay, then its amber fill rises from the bottom to the correct percentage. Complete books stay solid amber without animating. Empty books stay gray. Replays on every tab visit.

### Key decision: CSS gradients cannot be transitioned
The original spine used `background: linear-gradient(to top, amber X%, gray X%)`. CSS cannot interpolate between two different gradient values — `transition: background` does nothing.

**Correct approach:** replace the single-gradient background with a layered div structure:
- Button background = empty color
- Absolutely-positioned `<div>` inside = amber, `height: X%`, `bottom: 0`, with `transition: height Nms ease-out Dms`
- Text `<span>` with `position: relative; z-index: 1` floats above the fill div

This gives a clean bottom-up fill using CSS `height` transition, which browsers handle perfectly.

### Stagger pattern
- `animOrder: Map<string, number>` — computed with `useMemo`, maps each partially-read book name to its stagger index (0, 1, 2, … only partial books, in canonical Genesis→Revelation order)
- Each Spine receives `animIdx` prop; stagger delay = `animIdx * 40ms` on the fill div's `transition` property
- Complete and empty books get `animIdx = -1`, meaning `transition: none`
- This keeps NT books from having a huge delay vs OT books

### Replay on tab visit
`fillsReady` state starts `false` on every mount. After chapter data loads and `animOrder.size > 0`, a 150ms timer sets `fillsReady = true`. Since navigation uses plain `<a href>` tags (full page reloads), the component remounts fresh on every tab switch — replay is automatic.

### Minimum visible fill
3 chapters of 150 (Psalms) = 2% = less than 2px on an 88px spine — invisible. Applied a 12% minimum visual floor when `rawPct > 0`:
```tsx
const targetPct = rawPct > 0 && rawPct < 12 ? 12 : rawPct
```

### What worked first try
- The layered div structure (button → fill div → text span)
- `animOrder` useMemo keyed on `[chaptersRead, completedBooks]`
- `fillsReady` useEffect with `animOrder.size === 0` guard
- The 40ms per-book stagger index

### Language patterns from this session
- "Make the time before they start a little faster" = reduce the initial `FILL_DELAY_MS` constant (400 → 150ms). No other change needed.

---

## UI / Spacing

- **Tailwind arbitrary values are unreliable for pixel-precise spacing** — `mb-6`, `mb-[6px]`, `mb-[24px]` all failed at various points on the confirmation modal. The only fix that worked was inline `style={{ marginBottom: '20px' }}`. When spacing needs to be exact, use inline styles.
- **Modal spacing took 6+ iterations** — mb-2 → mb-6 → mb-[6px] → mb-6 again → 24px → 20px. Should've switched to inline style after the second failure instead of retrying Tailwind.
- **Three-column grid was the wrong call** — laid out Progress tab cards side-by-side, user wanted them stacked one column. Don't assume grid layout; match what's already on the page.

---

## Intro Animation (`IntroAnimation.tsx`)

### Dark-mode fade-to-black at the end
After the zoom blast fills the screen with `#fef9ee` (the page background color), dark-mode users would see a jarring light → dark flash as the intro unmounts and reveals `#0d1b2a`. Fixed with a black overlay fade:
- Keep `blackOpacity` state at 0 throughout the sequence
- After zoom completes: if `isDark` (read from `localStorage.getItem('theme')`), set `blackOpacity = 1`, wait 450ms, then call `onComplete()`
- The overlay div is **always in the DOM** so the CSS `transition: opacity 400ms ease` has a stable element to animate on. If rendered conditionally, the element is inserted at opacity:0 → transition fires → visual flash gone.
- Light mode: skip the wait, call `onComplete()` immediately.

The black overlay and the dark app background (`#0d1b2a`) are visually identical at handoff — user sees one unbroken dark screen, then the slide-up plays.

### Framer animations firing while hidden under the intro
Login and Register mount immediately but are visually hidden under the intro overlay (z-50). Framer Motion `motion.div` with `initial`/`animate` props fires the animation on mount regardless of visual visibility. By the time the 5–7s intro ends and the user can see the page, all animations have completed — content appears with no motion.

**Fix:** `key={String(introComplete)}` on both routes in App.tsx. When `introComplete` flips false→true, the key changes and React remounts the components, replaying animations at exactly the moment the page becomes visible.

**Rule:** any page that is mounted before it is visible needs `key` to be tied to the gate that makes it visible. Don't rely on Framer animations firing "early" off-screen.

---

## React / State

- **React StrictMode double-invokes state updaters** — caused a duplicate archive entry when a plan completed. The fix: use functional state updates (`prev => ...`) and guard against duplicates rather than assuming single invocation.
- **`createPortal` is the right tool for blur modals** — used for both the "Read Today" modal and the "Finish this plan?" confirmation. Rendered to `document.body` with `backdropFilter: blur(6px)`. Worked cleanly first time once the pattern was established.
- **Cross-fade plan → no-plan required careful state batching** — `fadingOutPlanId` triggers CSS opacity-0, then after 550ms a batch update removes the plan and sets `noPlanStatKey`/`noPlanFadeIn` so the card, completion stat, and days-read stat all animate simultaneously.

---

## Features That Landed Clean

- **"Read Today" green button state** — localStorage check on mount, POST to backend, streak bump logic. Worked in one pass.
- **Chapter range parsing** — `parseChaptersFromPassage("John 5-7")` → 3 chapters. Comma-split + range regex worked first try.
- **Weighted chapters per day** — `parseChaptersPerDay` parsing `daily_goal` string like "60 chapters" / `total_days`. Clean, no iterations.
- **Bookshelf spine removal** — removing the zoom/lean effect and shelf planks was a clean one-pass change.
- **`pendingPassagesDelta` unmark logic** — subtracting chapters when a day is unmarked. No bugs.

---

## General Patterns

- **Don't solve animation pieces in isolation** — when multiple things need to feel like one motion (zoom + background + UI entrance), design the full end-state first and work backwards to timing.
- **Switch to inline styles immediately when Tailwind fails twice** — don't keep retrying utility classes for pixel-precise values.
- **New features that touch both Dashboard and Progress need both files updated** — the confirmation modal, chapter counting, and unmark logic all had to be mirrored in both files. Easy to miss the second file.

---

## Setup Guide — Everything Needed to Build These Animations

This section is written so a brand-new Claude agent can reproduce the full animation setup from scratch.

---

### Stack

| Layer | Tool | Version / Notes |
|---|---|---|
| Frontend framework | React + Vite | `npm create vite@latest` → React + TypeScript |
| Styling | Tailwind CSS v4 | `@tailwindcss/vite` plugin (not PostCSS) |
| Animations (in-app) | Framer Motion | `npm install framer-motion` |
| Animations (full-screen) | Vanilla HTML/CSS/JS + Canvas | No dependencies — plain browser APIs only |
| Font (animations) | EB Garamond | Google Fonts CDN: `https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400;0,600;0,700&display=swap` |
| Backend | FastAPI + SQLite | Python, `uvicorn`, `sqlalchemy` |
| Preview server | Vite dev server | Port 5173 |

---

### Project structure for animations

Full-screen animations live as standalone HTML files in:

```
frontend/public/          ← Vite serves everything here as static files
  reading-check.html
  plan-complete.html
  book-complete.html
```

These are accessible at `http://localhost:5173/book-complete.html` during development, and at `/book-complete.html` in production (Vite copies `public/` to the build output).

**They are pure HTML files — no bundler, no imports, no React.** Everything is inline: `<style>` block, `<script>` block, Google Fonts `<link>`. This makes them fast to iterate on and easy to open directly in a browser.

---

### Vite config (required for proxy + port)

`frontend/vite.config.ts`:
```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
```

---

### Claude Code HTML viewer (launch.json)

To preview HTML animation files inside Claude Code, create this file:

`.claude/launch.json` (at the project root):
```json
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "frontend",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 5173
    }
  ]
}
```

This tells Claude Code's preview tool to start `npm run dev` and connect to port 5173. Once running, you can use `preview_start`, `preview_screenshot`, `preview_snapshot`, etc. to visually verify animations without opening a browser manually.

To navigate to a specific animation file in the preview:
```js
// In preview_eval:
window.location.replace('http://localhost:5173/book-complete.html')
```

---

### HTML animation file template

Every full-screen animation follows this exact structure. Copy this as a starting point:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Animation Name</title>
  <link rel="preconnect" href="https://fonts.googleapis.com"/>
  <link href="https://fonts.googleapis.com/css2?family=EB+Garamond:ital,wght@0,400;0,600;0,700&display=swap" rel="stylesheet"/>

  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      width: 100vw; height: 100vh; overflow: hidden;
      background: #fff;
      font-family: 'EB Garamond', Georgia, serif;
    }

    #scene {
      position: fixed; inset: 0;
      background: #0d1b2a; /* app's standard deep navy blue */
      overflow: hidden;
    }

    /* Add overlay divs as needed (flash, burst, success text, etc.) */
  </style>
</head>
<body>
  <div id="scene"></div>
  <!-- Add overlay divs here -->

  <script>
    // ─── TIMING CONSTANTS (tweak these to adjust feel) ───────────────────
    const T = {
      flashHold: 200,
      flashFade: 600,
      // ... add all durations here
    }

    // ─── EASING FUNCTIONS ─────────────────────────────────────────────────
    const outCubic    = t => 1 - Math.pow(1 - t, 3)
    const inOutCubic  = t => t < 0.5 ? 4*t*t*t : 1 - Math.pow(-2*t+2, 3)/2
    const outQuart    = t => 1 - Math.pow(1 - t, 4)
    const inOutQuart  = t => t < 0.5 ? 8*t*t*t*t : 1 - Math.pow(-2*t+2,4)/2
    // Camera slide: ease-in 35%, ease-out 65% — feels like gliding with momentum
    const slideEase   = t => t < 0.35
      ? (t / 0.35) * (t / 0.35) * 0.35
      : 0.35 + outCubic((t - 0.35) / 0.65) * 0.65

    // ─── ANIMATION PRIMITIVES ─────────────────────────────────────────────
    // IMPORTANT: use setTimeout, NOT requestAnimationFrame.
    // rAF is throttled/paused in background tabs. setTimeout(tick, 16) ≈ 60fps
    // and keeps running even when the preview panel is not in focus.
    function wait(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    }

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

    // ─── CSS TRANSITION REFLOW FIX ────────────────────────────────────────
    // When setting `element.style.transition` AND `element.style.opacity` (or
    // transform) in the same JS tick, the browser batches both and sees no
    // start state — the transition is skipped entirely and the change snaps.
    // Fix: force a reflow between setting transition and the new value:
    //
    //   element.style.transition = 'opacity 0.6s ease'
    //   void element.offsetHeight   // ← forces reflow — DO NOT REMOVE
    //   element.style.opacity = '1'
    //
    // This applies to EVERY CSS transition you trigger from JS.

    // ─── SEQUENCE ─────────────────────────────────────────────────────────
    async function run() {
      // Stage 1: ...
      // Stage 2: ...
      // ...

      // Signal React when done (if embedded in an iframe):
      window.parent.postMessage({ type: 'animationDone' }, '*')
    }

    run()
  </script>
</body>
</html>
```

---

### Wiring a full-screen animation into React

**Step 1 — Add state to the page component:**
```tsx
const [animationBook, setAnimationBook] = useState<string | null>(null)
```

**Step 2 — Listen for the "done" message:**
```tsx
useEffect(() => {
  function onMessage(e: MessageEvent) {
    if (e.data?.type === 'bookCompleteAnimationDone') {
      setAnimationBook(null)
    }
  }
  window.addEventListener('message', onMessage)
  return () => window.removeEventListener('message', onMessage)
}, [])
```

**Step 3 — Trigger the animation after the action completes:**
```tsx
const bookName = selectedBook
handleClose()          // close any open modal first
setAnimationBook(bookName)
```

**Step 4 — Render the iframe as a full-screen portal:**
```tsx
import { createPortal } from 'react-dom'

{animationBook && createPortal(
  <div className="fixed inset-0 z-[99999]">
    <iframe
      src={`/book-complete.html?book=${encodeURIComponent(animationBook)}`}
      className="w-full h-full border-0"
      title="Book Complete"
    />
  </div>,
  document.body
)}
```

**Step 5 — In the HTML file, read the URL param and signal done:**
```js
const params = new URLSearchParams(location.search)
const TARGET_BOOK = params.get('book') || 'Genesis'

// At the end of the animation:
window.parent.postMessage({ type: 'bookCompleteAnimationDone' }, '*')
```

Use `z-[99999]` for the portal so it sits above everything including other modals (which typically use `z-[9999]`).

---

### Canvas bookshelf pattern

For animations that draw many objects (bibles, cards, etc.) on a single canvas:

```js
const canvas = document.getElementById('shelf-canvas')
const ctx = canvas.getContext('2d')

function resize() {
  canvas.width  = window.innerWidth
  canvas.height = window.innerHeight
}
window.addEventListener('resize', resize)
resize()

// ─── VIEWPORT-RELATIVE DIMENSIONS (compute once, reuse everywhere) ───
const BH   = window.innerHeight * 0.90   // book height
const BW   = BH * 0.52                   // book width
const GAP  = BW * 0.38                   // gap between books
const SLOT = BW + GAP                    // stride per book
const SW   = Math.round(BW * 0.35)       // spine width
const SR   = Math.round(BW * 0.09)       // spine corner radius

// ─── roundRect helper (browser fallback) ───────────────────────────
function roundRect(ctx, x, y, w, h, radii) {
  if (ctx.roundRect) {
    ctx.roundRect(x, y, w, h, radii)
  } else {
    const [tl, tr, br, bl] = Array.isArray(radii)
      ? radii : [radii, radii, radii, radii]
    ctx.moveTo(x + tl, y)
    ctx.lineTo(x + w - tr, y)
    ctx.quadraticCurveTo(x + w, y, x + w, y + tr)
    ctx.lineTo(x + w, y + h - br)
    ctx.quadraticCurveTo(x + w, y + h, x + w - br, y + h)
    ctx.lineTo(x + bl, y + h)
    ctx.quadraticCurveTo(x, y + h, x, y + h - bl)
    ctx.lineTo(x, y + tl)
    ctx.quadraticCurveTo(x, y, x + tl, y)
  }
}

function drawBook(bx, label, fillColor) {
  const by = (canvas.height - BH) / 2

  // Shadow
  ctx.save()
  ctx.shadowColor = 'rgba(0,0,0,0.5)'
  ctx.shadowBlur = 18
  ctx.shadowOffsetX = 4
  ctx.shadowOffsetY = 6

  // Cover — rounded on LEFT (spine) side only: [SR, 3, 3, SR]
  // IMPORTANT: The spine strip sits on top of the cover.
  // For the spine's rounded corners to show background through them,
  // the COVER must also use the same [SR, 3, 3, SR] rounding.
  // If you only round the spine strip, the cover behind it fills the corners.
  ctx.fillStyle = fillColor
  ctx.beginPath()
  roundRect(ctx, bx, by, BW, BH, [SR, 3, 3, SR])
  ctx.fill()
  ctx.restore()

  // Spine strip — same rounding, same width as SW
  ctx.fillStyle = 'rgba(0,0,0,0.35)'
  ctx.save()
  ctx.beginPath()
  roundRect(ctx, bx, by, SW, BH, [SR, 0, 0, SR])
  ctx.clip()
  ctx.fillRect(bx, by, SW, BH)  // ← MUST match SW, not a hardcoded number
  ctx.restore()

  // Rotated label on spine
  ctx.save()
  ctx.translate(bx + SW / 2, by + BH / 2)
  ctx.rotate(-Math.PI / 2)
  ctx.fillStyle = 'rgba(255,255,255,0.85)'
  ctx.font = `600 ${Math.round(BW * 0.085)}px 'EB Garamond', Georgia, serif`
  ctx.textAlign = 'center'
  ctx.textBaseline = 'middle'
  ctx.fillText(label, 0, 0)
  ctx.restore()
}
```

**Critical spine lesson:** when you change `SW`, you must update BOTH:
1. The `roundRect` call that defines the clip shape
2. The `fillRect` call inside the clip that actually paints the pixels

Missing either one means the visual doesn't change even though the code looks right.

---

### App color palette

Use these exact values to match the existing animations:

```
Background (deep navy):  #0d1b2a
Card surface:            #162032
Border:                  #2a3a52
Gold accent:             #d4a847
Text primary:            #f0e6d0
Cover green:             #22c55e
Spine green (darker):    #15803d
Amber (completed):       #d97706 / amber-700
```

---

## How the User Communicates — Language Patterns

This section maps how things were said to what was actually meant, based on real messages. Use this to interpret future requests correctly the first time.

### Sequential animation descriptions
When describing a new animation, the user lists steps in plain English order — "first... then... after that..." Each step is one beat of the sequence.

> "When you first load into the localhost, I want it to show a bible. Then I want it to open. After that, I want the camera to swoop down into one of the letters. Then, after that, I want it to look like we zoomed so far into one of the letters that it became the background, and then all of the text and everything plays the animation we already made."

That is a 4-step animation sequence: appear → open → swoop → zoom-to-background → trigger existing animation. Build it as a sequential async chain, not all at once.

### "The little animation / the little slide animation"
Refers to the existing fade-up page entrance animation (`fade-up` keyframe, rises ~10px). He won't name it formally — "the little animation we made" always means that one.

> "Then I need you to play the little animation we made to make the text and everything fade in and slide up"
> "make the menu come up with all the little slide up animations"
> "Put the little slide animation back for the menu"

### "Zoom more" / "needs to zoom more"
Means increase the scale value. Not change the target, not change the speed — just make the number bigger.

> "it just needs to zoom more, so that the white amber color fills the whole screen"

### "Still some [color] left"
Means a visual artifact is still visible at the edges/corners. Don't change the target or approach — fix the specific remaining artifact.

> "there is still some blue left over on the side"
> "the background needs to become fully amber white before we pop up the menu screen"

### "Really fast" / "slightly slower" / "matching the [other thing]'s speed"
Duration language, not easing language. "Really fast" ≈ 0.3–0.4s. "Slightly slower" = increase by ~0.15–0.2s. "Matching X" = set same duration as whatever X is.

> "Make the second zoom go really fast into the center of the 'I.'" → 0.35s
> "make the second zoom slightly slower, matching the slide up speed for the UI" → match 0.55s

### "The camera goes into X" vs "X zooms into the camera"
He distinguishes the direction of zoom. "Camera goes into X" = scene scales up with the transform origin AT X (the subject stays centered, world expands around it). "X zooms into the camera" = the element itself enlarges toward the viewer. He always wants the camera-goes-into version.

> "I want the camera to fully zoom into a letter, not a letter zoom into the camera"

### "It's going into the blue part / blank space"
Means the transform origin is wrong — the zoom is centered on empty space instead of a letter. Needs `transformOrigin` aimed precisely at the glyph center, not a general page area.

> "the camera isn't going into a letter, it is going into the blue part of the Bible"
> "it still isn't zooming into any text in the Bible. I need you to make it zoom into a letter, not the blank space in between them"

### "Almost like the momentum from X is pushing Y"
A physics/feel note — he wants continuity between two animations so they feel like one motion. Solve it by: matching durations, matching easing curves, and overlapping start times (80–100ms overlap so Y starts before X fully finishes).

> "I want it to be smoother. Almost like the momentum from the zoom is then pushing up the menu screen UI."

### "Do this also" / "Now do this"
Means add a new requirement on top of what just worked. Don't re-derive the previous work — keep it and layer the new thing.

### "Do this instead" / "Ok, no. Do this instead"
Means abandon the current approach and pivot. The previous attempt was wrong at a conceptual level, not just a tuning level.

> "Ok, no. Do this instead: Make the second zoom go really fast into the center of the 'I.'"

### Corrections about what he said earlier
He will sometimes clarify that his original description was imprecise. Take the correction literally.

> "I meant the sun emoji, sorry" (after saying "remove the start emoji" — I removed the cross instead)
> "the color for the text I meant was the white amber color" (after saying "same color as the background" — he meant warm cream, not dark blue)

### Timing adjustments via feel, not numbers
He won't say "set it to 250ms" — he'll say "wait .25 of a second" or "make it instant." When he says a number, use it exactly. When he says "instant," remove the delay entirely.

> "wait .25 of a second. Then, showing the menu screen"
> "Ok, actually make it .025"
> "Ok, make it instant"

### "Make [the delay / the wait] a little faster"
Means reduce the initial delay constant only — not the animation duration, not the stagger. One number changes: the `FILL_DELAY_MS` (or equivalent) constant. Duration and stagger stay the same.

> "Ok, now just make the time before they start the animation a little faster."

### "Remove the words that aren't fully showing"
For the Bible page text: any word that gets clipped by the page boundary should be cut. Don't resize or reflow — just truncate at the last fully-visible word.

### Layout preference: stacked, not side-by-side
When adding new sections, default to one column (full width, stacked vertically). He will push back on grids or side-by-side layouts.

> "I want them to all keep the same length that they had. I want it to be like one column, not one row"

### "Make it the same as we did in the commander app"
The MTG Commander app is the reference implementation. When he says this about a pattern (dropdowns, backend structure, etc.), look at how it was done there first.

> "No, keep the three line dropdown. We have done this before in the commander app. If you are struggling go to that."
> "Do the DB the same as how you did it with the commander app."
