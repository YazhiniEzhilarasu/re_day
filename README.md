# RE:DAY

**Plan your day. Reset without guilt.**

RE:DAY is a single-page, offline-capable web app that builds a realistic daily timetable around the
things you cannot move (school, coaching, travel, clubs) and then fills the gaps with your own tasks —
placing your most important work in the window when you focus best.

It is designed to be honest: when a task does not fit, RE:DAY moves it to *another day* instead of
squeezing your day into an impossible shape.

---

## Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [How it works](#how-it-works)
- [Project structure](#project-structure)
- [Data & privacy](#data--privacy)
- [Tech notes](#tech-notes)
- [Known limitations](#known-limitations)
- [Contributing](#contributing)
- [License](#license)

---

## Features

### Four-step planner

1. **Getting ready** — how long you need before the day starts.
2. **Fixed commitments** — anything that can't move, with travel time added each way.
3. **My tasks** — name, duration, priority (P1/P2/P3) and deadline.
4. **Generate** — a summary, then your timetable.

### Smart scheduling

- **Peak focus window** — choose *Morning*, *Evening* or *Night*; focus blocks are placed there first
  and spill over to other free time only if needed.
- **Automatic travel blocks** — travel is inserted before and after each commitment and
  accounted for when checking for gaps between them.
- **Getting-ready block** — placed before your first commitment, with a warning if that means
  waking up earlier than your wake-up time.
- **Meals** — lunch and dinner are slotted into their usual windows when there is room.
- **Wind-down** — the last 30 minutes before bedtime are protected.
- **Breaks** — 10–15 minute breaks are inserted between focus blocks; longer gaps are shown as free time.
- **Balanced load** — only about 60% of your free time (max 5 hours) is committed to tasks, so the
  day stays sustainable.
- **Priority & deadline aware** — overdue and due-today tasks are scheduled first, then by priority,
  then by deadline.
- **Focus blocks are capped** at 75 minutes and split into parts of at least 25 minutes when a task
  is longer, labelled *(part 1 of 3)* in the timeline.

### Reset my day

Plans change. Tick off what you finished, optionally block out lost time (e.g. an unexpected visit),
and RE:DAY re-fits the remaining work **without moving a single fixed commitment**. Anything that
still doesn't fit is listed under *Moved to another day*.

### Everything else

- **Print / save as PDF** — a clean A4 timetable with the postponed list.
- **Auto-save** — everything is stored in the browser, so a refresh never loses your work.
- **Dark mode** — follows the operating system's light/dark preference automatically.
- **Mobile friendly** — responsive layout with safe-area insets, works as a phone web app.
- **No accounts, no server, no tracking.**

---

## Quick start

The app is a single self-contained HTML file — there is **no build step and no dependencies**.

**Option 1 — open it directly**

Double-click `index.html`, or drag it into your browser.

**Option 2 — serve it locally** (recommended, so that browser storage behaves consistently)

```bash
# Python 3
python -m http.server 8000

# or Node.js
npx serve .
```

Then open <http://localhost:8000> in your browser.

> The only external resource is the *Nunito* font from Google Fonts. If you are offline, the app
> falls back to your system's default sans-serif and still works fine.

---

## How it works

### The scheduling algorithm

All times are handled as minutes-since-midnight, which keeps the arithmetic simple and avoids
date/time parsing bugs.

```
1. Build the fixed blocks
   |- Commitments (school, coaching, ...)      -- never move
   |- Travel before / after each commitment    -- inserted automatically
   |- Getting ready                            -- before the first commitment
   |- Unavailable (lost time, manual block-outs)
   |- Wind down (last 30 min before bed)

2. Subtract fixed blocks from the waking window  ->  free time slots

3. Place meals (lunch 12:00-14:00, dinner 18:00-20:30) in the free slots that remain

4. Sort tasks
   overdue/due-today  ->  priority (P1, P2, P3)  ->  deadline date

5. Schedule each task
   |- Prefer slots inside your peak focus window
   |- Fall back to any free slot
   |- Block length: max 75 min, min 25 min
   |- Insert a 10-15 min break after each block
   |- Stop when the focus budget is used
        budget = min(300 min, 60% of total free time)

6. Anything left over -> "Moved to another day" (never silently dropped)
```

### Reset semantics

A reset re-runs the same algorithm with two differences:

- **Completed** task blocks are kept in place and credited against that task's remaining duration.
- **Fixed commitments stay exactly where they were** — only the task blocks are re-fitted.

---

## Project structure

```
re-day/
├── .gitignore      # OS, editor and temp-file cruft
├── README.md       # you are here
└── index.html      # the entire application: markup, styles and logic
```

Everything — HTML structure, CSS design system and JavaScript — lives in `index.html`. It is
deliberately kept as a single file so the app can be shared, emailed, or hosted anywhere as one
self-contained document.

---

## Data & privacy

Everything you enter stays **on your device**.

- All state is written to `localStorage` under the key `reday1`.
- There is no server, no account, no analytics, no network request other than the optional
  Google Fonts stylesheet.
- To wipe your data, clear the site data for the page in your browser (or use your browser's
  "Clear site data" / "Delete local storage" option).

---

## Tech notes

| Aspect | Detail |
| --- | --- |
| Markup | Semantic HTML5, ARIA labels on interactive controls |
| Styling | Plain CSS with custom properties (`--bg`, `--card`, `--ink`, `--sage`, ...) |
| Dark mode | `prefers-color-scheme` media query + `data-theme` override hooks |
| Layout | CSS flexbox, `max-width` container, `env(safe-area-inset-*)` for notched phones |
| Fonts | Nunito (Google Fonts) with system-font fallback |
| Icons | Unicode emoji — no icon font or SVG assets |
| Scripting | Vanilla ES2015+, no framework, no bundler |
| Storage | `localStorage` (`reday1`), wrapped in try/catch for private-mode browsers |
| Printing | Dedicated print stylesheet, A4, hides the app chrome |
| Browser support | Any modern browser (Chrome, Edge, Firefox, Safari) |

### Customising the look

All colours, spacing and radii are driven by CSS custom properties at the top of `index.html`:

```css
:root {
  --bg:    #F7F5EF;  /* page background   */
  --card:  #FFFEFB;  /* card surface      */
  --ink:   #243C53;  /* primary text      */
  --sage:  #648B70;  /* accent / CTA      */
  --muted: #6b7b8a;  /* secondary text    */
  --line:  #e3e0d6;  /* borders           */
  --warn:  #b0623a;  /* error / postpone  */
}
```

To force a specific theme regardless of the OS setting, set `data-theme="light"` or
`data-theme="dark"` on the `<html>` element.

### Customising the scheduler

Two small blocks at the top of the `<script>` control the behaviour:

```js
// Focus windows in minutes-since-midnight
const WIN = { morning:[360,720], evening:[960,1260], night:[1200,1410] };

// Priority ranking
const PR = { high:0, medium:1, low:2 };
```

Block length caps, the 60% focus budget and the meal windows are inline in the `build()` function.

---

## Known limitations

- **`buildOld()` is dead code.** It is the previous version of the scheduler, kept in the file for
  reference but never called. It is safe to delete.
- The print view inherits your browser's print dialog, so choose "Save as PDF" for a file.
- Deadlines are compared against the browser's local date, so a device with a wrong clock will
  report tasks incorrectly.
- Only one day is modelled at a time; there is no week view or history.

---

## Contributing

```bash
git clone <your-remote-url> re-day
cd re-day

# open index.html in your browser — there is nothing to install

git add .
git commit -m "Describe your change"
git push
```

Before opening a pull request:

1. Test the full flow: plan -> generate -> tick tasks done -> reset -> print.
2. Check the app still looks right in both light and dark mode.
3. Keep the app dependency-free — no build tooling, no frameworks.

---

## License

No license has been chosen yet. Add a `LICENSE` file (MIT, Apache-2.0, or whichever you prefer)
before publishing or sharing the code publicly.



