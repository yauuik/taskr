# Task.r — CLAUDE.md

## Project Overview
Task.r is a vanilla JS single-file task tracker (`index.html`).
Deployed on GitHub Pages at `yauuik.github.io/taskr/`.
Cloud sync via Firebase Realtime Database (project `taskr-1b2bd`).

## Architecture

### Single file: `index.html`
Everything lives in one file: HTML + CSS + JS. No build step, no framework.

### Data Model (v2.2)
Every item (project or subtask) is the same recursive structure:
```json
{
  "id": "pmn57dlzlonaj",
  "name": "Objectives 2026 - Journey Welcome",
  "status": "To Do",        // "To Do" | "In Progress" | "Done"
  "note": "<div>...</div>",  // HTML string
  "due": "2026-07-15",       // ISO date or undefined
  "doneAt": "2026-03-24",    // ISO date when completed
  "children": [              // same structure, recursive
    { "id": "c123", "name": "Sous-tâche", "status": "To Do", "note": "", "children": [], "due": "2026-04-03" }
  ],
  "hidden": false            // archived (Done section hide)
}
```
Status values: `"To Do"`, `"In Progress"`, `"Done"` (capitalized, not lowercase).
`tSt(t)` normalizes to lowercase `"todo"/"progress"/"done"` for internal logic.

### State
- `s` = global state: `{p:[], ok, sy, ed, exp, keys, sett, cm, cf, nk, an, sel, doneOpen, sh, noteEdit, _editTask, _focusTaskInput, _dateSort, _qiOpen, _qiDate, search, sq, calc, qed, calView, stOpen}`
- `s.stOpen` = Set of project IDs with subtasks expanded (collapsed view)
- `s.calView` = boolean, persistent in localStorage (calendar vs list view)
- `settings` = localStorage `taskr-settings`: `{flatpickr, mentionIcon, layoutWidth, theme, _calcArr, _calcPauseStart, _calcPauseEnd, _calcDep, _dateSort, calView}`

### Key Functions
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
- `hlText(el, query)` — highlight search matches in DOM elements
- `noteExcerpt(html, query)` — extract snippet around search match in note
- `localIso(d)` — local timezone date string (replaces `toISOString().slice(0,10)`)
- `tSt(t)` — normalize status to lowercase ("todo"/"progress"/"done"), handles both old `st` and new `status` fields
- `haptic()` — vibration feedback 15ms
- `playCheck()` — completion sound (800→1200Hz oscillator)
- `rCalc()` — time calculator modal
- `rSett()` — settings modal
- `rKeys()` — keyboard shortcuts modal
- `rExp()` — markdown export/import modal

### Firebase Sync
- **Realtime listener**: `dbRef.on("value")` — pushes changes instantly (no polling)
- **Transaction**: `dbRef.transaction()` — checks `lastKnownTs`, prevents stale overwrite
- **History**: each save stores backup in `taskr_history/` (last 10 versions)
- **Anonymous Auth**: `firebase.auth().signInAnonymously()`
- **Rules**: `auth != null` for read/write on `taskr` and `taskr_history`
- **API Key**: restricted to `yauuik.github.io/*` and `localhost/*` via Google Cloud Console
- `cloudLoaded` gate: blocks cloud save until initial load
- `saving` flag: prevents listener echo
- `lastKnownTs`: conflict detection

### Cross-tab Sync
- `BroadcastChannel("taskr-sync")` for instant same-device tab sync

## Team (mentions @)
20 people: Larissa, Bryan, Caroline, Aurélie, Fleur, Erwin, Jelle, Maëlysse, Stéphanie, Véronique, Eveline, Laurence, Jeremie, Stanny, Dan, Vincent, Cenk, Alexandre, Antoine, Charlotte

## Features

### Core
- Projects with status cycle: To Do → In Progress → Done (Alt+click = reverse)
- Children (subtasks) = same structure as projects: name, status, note, due, @mentions
- Collapsible subtasks (chevron toggle below row, closed by default)
- @mentions with autocomplete dropdown
- Rich notes editor (contentEditable) on both projects and subtasks
- Fullscreen note editor with source code view
- Date picker on projects and subtasks (native date input)
- Urgent system: `!!` in name or due ≤ tomorrow → red styling
- Search with yellow highlight on names, subtasks, and note excerpt preview
- Search includes subtask notes
- Done section auto-opens during search
- Sort by date (asc/desc toggle, synced via Firebase)
- Done section grouped by completion day
- Block editing Done tasks (toast: "Repasse-la en cours pour l'éditer")
- Confirmation dialog before project deletion

### Views
- **Liste** (default): sections En cours / À faire / Terminé
- **Semaine** (calendar): inline week view with day-by-day tasks, navigation ‹ › + "Aujourd'hui"
- Toggle Liste/Semaine persistent in localStorage

### Quick Add
- Desktop: collapsed `+ Ajouter une tâche…` → expands with input + date + Annuler/Ajouter
- Mobile: FAB → bottom sheet with same form (via shared `mkAddForm`)

### Time Calculator
- Workday = 7h10, pause midi with start/end times, departure field
- Quick buttons: 30' / 45' / 60' / 90'
- All values synced via Firebase prefs

### UX Polish
- Greeting: "Bonjour/Bon après-midi/Bonsoir Yannick · X en cours · Y urgentes"
- Slide-in animation on newly added tasks only
- Checkbox pop animation on Done
- Completion sound + haptic vibration
- Hover contrast on rows (darker bg + shadow)
- Trash icon (SVG) with red hover, confirmation before delete
- Close X button top-right of expanded card
- Close X in mobile bottom sheet
- French section labels (En cours, À faire, Terminé)

### Mobile
- Bottom sheet for project edit (swipe down to close)
- Swipe left/right on rows to change status
- FAB for quick add

### Themes
- Normal (light), Dark, IMS Dark/Blue/Green
- Auto dark mode via `prefers-color-scheme`

### Other
- Drag & drop reorder (To Do section only)
- Done animation (strikethrough + fade)
- Toast undo system
- Markdown export/import
- Keyboard shortcuts (?, n, /, e, t, Escape chain)

## SVG Icons (mkSvg)
8 icons: `calendar`, `note`, `person`, `search`, `sync`, `sort`, `trash`, `close`

## CSS Variables (Normal theme)
```
--bg:#F6F7FB; --card:#FFF; --dark:#181B34;
--accent:#6161FF; --green:#00C875; --amber:#FDAB3D; --red:#E2445C;
--text:#323338; --sec:#676879; --muted:#C3C6D4; --border:#E6E9EF; --hover:#EDEEF3;
```

## Firebase Config
- Project: `taskr-1b2bd`
- DB URL: `https://taskr-1b2bd-default-rtdb.europe-west1.firebasedatabase.app`
- Data: `/taskr` (projects + prefs + timestamp)
- History: `/taskr_history` (backups)
- Auth: Anonymous, Rules: `auth != null`

## CDN Dependencies
- Figtree + IBM Plex Mono: Google Fonts
- IBM 3270: `db.onlinewebfonts.com`
- Flatpickr: `cdn.jsdelivr.net/npm/flatpickr` (disabled, "bientôt")
- Firebase 10.14.1: app, auth, database (compat)

## Development Notes
- Direct push to production (no staging)
- Verify JS syntax before pushing: `node -e "new Function(scriptContent)"`
- Claude should proactively suggest UX/mobile improvements
- `localIso()` for all date comparisons (no UTC timezone bugs)
- Escape key chain: cfm → search → editTask → qed → ed → exp → keys → sett → calc → calView → sel

## Git Tags
- `v1.5-stable` — pre-Firebase
- `v2.0-firebase` — Firebase migration
- `v2.1-secure` — Firebase + auth + security rules
- `v2.1-pre-refactor` — backup before children refactor
- `v2.2` — children model, calendar view, search highlight, UX polish
