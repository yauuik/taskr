# Task.r — CLAUDE.md

## Project Overview
Task.r is a vanilla JS single-file task tracker (~2430 lines in `index.html`).
Deployed on GitHub Pages at `yauuik.github.io/taskr/`.
Cloud sync via Firebase Realtime Database (project `taskr-1b2bd`).

## Architecture

### Single file: `index.html`
Everything lives in one file: HTML + CSS + JS. No build step, no framework.

### State
- `s` = global state object: `{p:[], ok, sy, ed, exp, keys, sett, cm, cf, nk, an, sel, doneOpen, sh, noteEdit, _editTask, _focusTaskInput, _dateSort, _qiOpen, _qiDate, search, sq, calc, qed}`
- `settings` = localStorage `taskr-settings`: `{flatpickr, mentionIcon, layoutWidth, theme, _calcArr, _calcPauseStart, _calcPauseEnd, _calcDep, _dateSort}`

### Key functions
- `R()` — full re-render (deferred while `acOpen2` is true)
- `qR()` — batched render via `requestAnimationFrame`
- `up(id, patch, skipRender)` — update single project
- `ps(projects)` — replace all projects + save + render
- `save()` — localStorage + debounced (800ms) Firebase write
- `sb(p)` — Firebase transaction write (prevents overwrite conflicts)
- `mkRow(p)` — renders a project row (collapsed or expanded)
- `mkAddForm(opts)` — shared add-task form (desktop + mobile)
- `mkSvg(name, size, extra)` — SVG icon factory
- `mkSec(label, color, count, ...)` — section header
- `mentionField(opts)` — `<input>` with @mention chip system
- `cleanPaste(e)` — strips styles/classes/labels from pasted HTML
- `rCalc()` — time calculator modal
- `rSett()` — settings modal
- `rKeys()` — keyboard shortcuts modal
- `rExp()` — markdown export/import modal

### Firebase sync
- **Realtime listener**: `dbRef.on("value")` — pushes changes instantly (no polling)
- **Transaction**: `dbRef.transaction()` — prevents stale data from overwriting newer cloud data
- **History**: each save stores a backup in `taskr_history/` (last 10 versions)
- **Anonymous Auth**: `firebase.auth().signInAnonymously()` — required for DB rules
- **Fallback**: localStorage if Firebase is unreachable
- `cloudLoaded` gate: blocks cloud save until initial Firebase load succeeds
- `saving` flag: prevents listener echo during our own writes
- `lastKnownTs`: tracks last known cloud timestamp for conflict detection

### Cross-tab sync
- `BroadcastChannel("taskr-sync")` for instant same-device tab sync
- `notifyTabs()` called after successful cloud save

## Team (mentions @)
20 people: Larissa, Bryan, Caroline, Aurélie, Fleur, Erwin, Jelle, Maëlysse, Stéphanie, Véronique, Eveline, Laurence, Jeremie, Stanny, Dan, Vincent, Cenk, Alexandre, Antoine, Charlotte

## Features

### Core
- Projects with status cycle: To Do → In Progress → Done (click checkbox, Alt+click = reverse)
- Subtasks with same tri-state cycle
- @mentions with autocomplete dropdown
- Rich notes editor (contentEditable) with toolbar: B, I, U, —, lists, links
- Fullscreen note editor with source code view (pretty-printed HTML, line numbers toggle)
- Date picker (native or Flatpickr beta)
- Urgent system: `!!` in name or due ≤ tomorrow → red styling, sorted to top
- Search: filters on name, tasks, notes, dates (icon + `/` shortcut)
- Sort by date (asc/desc toggle, synced via Firebase)
- Done section grouped by completion day
- Click @mention → filter all tasks for that person

### Quick Add
- Desktop: collapsed `+ Ajouter une tâche…` → expands with input + date + Annuler/Ajouter
- Mobile: FAB → bottom sheet with same form (via shared `mkAddForm`)
- Enter works in both

### Time Calculator
- Workday = 7h10
- Fields: Arrivée, Pause midi (début → fin, min 30min), Départ réel (optional)
- Quick buttons: 30' / 45' / 60' / 90'
- Live overtime calculation (green/red vs 7h10)
- Auto-format time inputs (0910 → 09:10)
- All values synced cross-device via Firebase prefs

### Mobile
- Bottom sheet for project edit (swipe down to close)
- Swipe left/right on rows to change status
- FAB for quick add
- Responsive layout (Todoist-inspired spacing)

### Themes
- Normal (light), Dark (Todoist-style), IMS Dark/Blue/Green (monospace retro)
- Auto dark mode via `prefers-color-scheme` if no explicit choice
- Theme selector in Settings (⚙)

### Other
- Drag & drop reorder (To Do section only)
- Done animation (strikethrough + fade)
- Toast undo system (delete project/subtask)
- Markdown export/import
- Keyboard shortcuts (?, n, /, e, t, Escape chain)

## CSS Variables (Normal theme)
```
--bg:#F6F7FB; --card:#FFF; --dark:#181B34;
--accent:#6161FF; --green:#00C875; --amber:#FDAB3D; --red:#E2445C;
--text:#323338; --sec:#676879; --muted:#C3C6D4; --border:#E6E9EF; --hover:#F5F6F8;
```

## SVG Icons (mkSvg)
6 icons defined in `ICONS` object: `calendar`, `note`, `person`, `search`, `sync`, `sort`.
Topbar icons use `mkIcon()` which takes SVG path data arrays.

## CDN Dependencies
- Figtree + IBM Plex Mono: Google Fonts
- IBM 3270: `db.onlinewebfonts.com`
- Flatpickr: `cdn.jsdelivr.net/npm/flatpickr`
- Firebase: `gstatic.com/firebasejs/10.14.1/` (app, auth, database compat)

## Firebase Config
- Project: `taskr-1b2bd`
- DB URL: `https://taskr-1b2bd-default-rtdb.europe-west1.firebasedatabase.app`
- Data path: `/taskr` (projects + prefs + timestamp)
- History path: `/taskr_history` (last 10 backups)
- Auth: Anonymous
- Rules: `auth != null` for read/write on both paths

## Development Notes
- Direct push to production after each change (no staging)
- Always verify JS syntax before pushing: `node -e "new Function(scriptContent)"`
- Claude should proactively suggest UX/mobile best practices, not just execute
- Alert when a UI choice won't work well on mobile, propose alternatives
- `capFirst()` capitalizes first letter of text (skipping @mentions)
- `buildVal(tags, text)` assembles mention tags + text
- Escape key chain: cfm → search → editTask → qed → ed → exp → keys → sett → calc → sel

## Git Tags
- `v1.3-stable` — pre-dark-mode
- `v1.4-stable` — pre-refactor
- `v1.5-stable` — pre-Firebase
- `v2.0-firebase` — Firebase migration
- `v2.1-secure` — Firebase + auth + security rules
