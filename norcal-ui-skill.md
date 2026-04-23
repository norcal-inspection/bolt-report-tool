# NorCal Inspection — Tool UI Skill

Use this skill whenever building a new internal tool for norcal-tools. It captures the visual language, component library, and interaction conventions established by `ir-folder-creator.html` and `member-check-list.html`, so new tools feel like siblings rather than cousins.

## When to use

Any time a new HTML/JS tool is being added to https://norcal-inspection.github.io/norcal-tools — or an existing one is being edited in a way that touches UI. Not for the Cloudflare Worker, not for the Python legacy tools.

## Shared context

- Single-file HTML tools, hosted on GitHub Pages.
- All CSS and JS inline — no external stylesheets or module bundlers.
- External libraries from cdnjs only (e.g. pdf.js, xlsx).
- Audience is non-technical team members. No installation, no terminal.
- Dark-themed design, professional, slightly retro-terminal. Amber accent on black.

## Design tokens

Drop this `:root` block into every tool. Do not change the values — consistency across tools matters more than any individual tweak.

```css
:root {
  --bg:#0f0f0f; --surface:#1a1a1a; --surface2:#242424; --border:#333;
  --accent:#e8c547; --accent2:#4ab7e8; --green:#4ae87a; --red:#e84a4a;
  --orange:#e8944a; --text:#e8e8e8; --text-dim:#888;
  --mono:'IBM Plex Mono',monospace; --sans:'IBM Plex Sans',sans-serif;
}
```

Color roles:
- `--accent` (amber) — the primary interactive color. Buttons, highlights, link-like text, section titles, caret indicators, selected states.
- `--accent2` (blue) — secondary accent, used sparingly for "future" or informational callouts.
- `--green` — success states (connected, parsed OK, created folders).
- `--red` — error states and destructive buttons.
- `--orange` — warnings and "already exists" / "skipped" neutral statuses.
- `--text` / `--text-dim` — foreground and muted foreground.
- `--surface` / `--surface2` — card background and one-step-darker slot background.

## Typography

Google Fonts import at the top of the `<style>` block:

```css
@import url('https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@300;400;500;600&display=swap');
```

Usage rules:
- `IBM Plex Sans` is the default for body prose, instructions, button labels if not code-like, descriptive text.
- `IBM Plex Mono` is for anything technical, identifier-like, or tabular: section titles (`.panel h2`), buttons, file names, record values, log entries, stats, breadcrumbs.
- Section titles (`h2` inside `.panel`) are uppercase, letter-spaced, dim:
  `font-family:var(--mono);font-size:12px;color:var(--text-dim);text-transform:uppercase;letter-spacing:1.5px;`

## Layout

- `body { background:var(--bg); color:var(--text); font-family:var(--sans); min-height:100vh; padding:32px 24px; max-width:860px; margin:0 auto; }`
- For tools with wide data tables (like extracted records), bump `max-width` to `1100px`. Otherwise stay at `860px`.
- Everything is a single vertical column. No sidebars, no multi-column grids outside of specific widgets.
- Consistent `20px` vertical gap between panels.

## Page structure

Every tool uses the same top-to-bottom skeleton:

```
1. .header            — logo icon + title + one-line description
2. .instructions      — small amber-tinted callout: "How to use: ..."
3. .panel × N         — numbered "Step N — Title" sections, one per user action
4. .btn-row           — primary action button(s), typically below the last step
5. .summary-bar       — stat boxes (shown only after an action completes)
6. .log-panel         — activity log (shown only after an action runs)
7. (optional) results — grouped data display, collapsible
```

The header block pattern:

```html
<div class="header">
  <div class="header-icon">&#x1F4C1;</div>  <!-- emoji glyph fits best -->
  <div class="header-text">
    <h1>Tool Name</h1>
    <p>One-line description of what it does.</p>
  </div>
</div>
```

The instructions block:

```html
<div class="instructions">
  <strong>How to use:</strong> Do X, then Y, then <strong>Click This</strong>. Caveat about edge case.
</div>
```

Numbered panels use an em-dash in the title:

```html
<div class="panel">
  <h2>Step 1 &mdash; Do the Thing</h2>
  ...
</div>
```

## Component library

### Buttons

Three variants — use them consistently:
- `.btn.btn-primary` — amber. The single "go" action per screen. Disabled state is muted grey.
- `.btn.btn-secondary` — dark-on-dark, border shows on hover. For alternate or read-only actions ("Browse", "Pick individual PDFs").
- `.btn.btn-danger` — red-tinted. For Clear, Disconnect, destructive actions.

The `btn-primary` can show a spinner during work — swap `innerHTML` to `'<span class="spinner"></span> Processing...'` and restore when done. Always disable during work.

Buttons are always `font-family:var(--mono)`. Sizes are fixed at `11px 22px` padding, `13px` font. Don't deviate.

Button rows: `<div class="btn-row">` wraps multiple buttons with consistent 10px gap.

### Drop zone

The universal file-intake pattern:

```html
<div class="drop-zone" id="dropZone">
  <input type="file" id="fileInput" accept=".pdf" multiple>
  <span class="drop-icon">&#x1F4C1;</span>
  <h3 id="dropTitle">Drop a folder or files here, or click to browse</h3>
  <p id="dropSub">Accepts .pdf files</p>
</div>
```

States via class:
- Default — dashed grey border.
- `.drag-over` — amber border, amber-tinted background (added during dragover).
- `.has-file` — green border, green-tinted background (after files are selected).

The `<input>` is absolutely positioned inside the zone so the whole area is clickable. JS wires up dragover/dragleave/drop handlers.

For folder-mode, add `webkitdirectory directory` attributes to the input. If a tool needs *both* folder and individual-file modes, use one default and provide a secondary button that temporarily swaps the attributes (see `member-check-list.html` for the pattern).

### File list

When files have been queued but not yet processed, show them in a file list that appears below the drop zone:

```html
<div class="file-list-wrap" id="fileListWrap">
  <div class="file-list-header">
    <span>Queued files</span>
    <span id="fileCount" style="color:var(--text-dim);"></span>
  </div>
  <div class="file-list" id="fileList">
    <!-- .file-item entries with .fi-icon, .fi-name, .fi-status -->
  </div>
</div>
```

File items show per-file status via a class: `.pending` (orange), `.ok` (green), `.err` (red), `.skipped` (dim).

### Log panel

Shows after any async action runs. Timestamped lines, color-coded by type:

```html
<div class="log-panel" id="logPanel">
  <div class="log-header">
    <span>Activity Log</span>
    <span id="logCount" style="color:var(--text-dim);"></span>
  </div>
  <div class="log-body" id="logBody"></div>
</div>
```

Log lines are `<div class="log-line"><span class="log-time">[HH:MM:SS]</span><span class="log-info">message</span></div>`. Message classes: `.log-info` (dim), `.log-success` (green), `.log-error` (red), `.log-warn` (orange).

Standard `log(msg, type)` helper — see either existing tool for the copy-paste implementation.

### Summary bar

Horizontal row of stat boxes, shown after an action completes:

```html
<div class="summary-bar" id="summaryBar">
  <div class="stat-box green"><div class="stat-val" id="stat1">0</div><div class="stat-label">Label</div></div>
  <div class="stat-box orange"><div class="stat-val" id="stat2">0</div><div class="stat-label">Label</div></div>
  <div class="stat-box red"><div class="stat-val" id="stat3">0</div><div class="stat-label">Label</div></div>
</div>
```

Modifier classes on `.stat-box`: `.green`, `.orange`, `.red` tint the big number. No modifier = amber.

Toggle `.visible` on `#summaryBar` to show/hide. Same for `.log-panel`.

### Collapsible grouped display

For listing extracted or fetched records in groups, use the pattern from `member-check-list.html`:

- A `.records-panel` wraps the whole thing.
- Each group is a `.group` with a `.group-header` (clickable) and `.group-body`.
- Clicking the header toggles `.collapsed` on both header and body.
- Inside the body, `table.records` displays the actual rows in monospace.

## Interaction conventions

- **Primary action disabled by default.** Only enabled once prerequisites are satisfied (file selected, connection made, etc). Keep a single source of truth function like `updateParseBtn()` or `checkCreateReady()` that re-evaluates on every state change.
- **Spinner on primary button during work.** Prevents double-click and signals that something is happening. Restore label when done.
- **Log + summary appear on first run and stay visible.** Clearing (via a Clear button) hides them again.
- **Clear button is always `.btn-danger` and hidden until there's something to clear.**
- **Escape non-user-provided text with `esc()`.** All existing tools have the same helper — copy it:
  ```js
  function esc(str) {
    return String(str)
      .replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;')
      .replace(/"/g,'&quot;').replace(/'/g,'&#39;');
  }
  ```
- **Never auto-scroll the whole page.** Log body scrolls itself via `body.scrollTop = body.scrollHeight`. File lists have `max-height` with `overflow-y:auto`.

## Icons

Use Unicode emoji entities in the HTML source — they render everywhere and need no asset pipeline.

Conventional choices:
- `&#x1F4C1;` 📁 — folders, folder-related tools
- `&#x1F4C4;` 📄 — files, file upload
- `&#x1F4CB;` 📋 — lists, checklists, tabular data
- `&#x25B6;` ▶ — expand / navigate forward
- `&#x25BC;` ▼ — collapse / expanded state

## What NOT to do

- No Tailwind, no component libraries, no React. Single-file vanilla HTML/CSS/JS.
- No light-mode toggle. Dark is the brand.
- No animations beyond the spinner and simple hover transforms.
- No borders on tables inside the log or file lists — use `border-bottom: 1px solid rgba(51,51,51,0.5)` between rows for subtle separation.
- No multiple primary buttons visible at once. If a tool has more than one "go" action, stage them across panels or toggle them based on state.
- Don't rename CSS variables or add new ones ad-hoc — use what's defined.

## Starter template

When building a fresh tool, copy the top of `ir-folder-creator.html` or `member-check-list.html` through the `<style>` block and the first panel. Both are structured identically up to that point; pick whichever has the closer upload-interaction style to what's needed.

## Deployment

Unchanged from the broader skill: save the file to the local `norcal-tools` repo folder, commit via GitHub Desktop, push, wait ~30 seconds for GitHub Pages to publish.

Throwaway diagnostic / validation tools used during development follow the same design language but are NOT committed to the repo — save them to Downloads and open locally.
