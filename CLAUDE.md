# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file IT PMO Kanban board demo (`index.html`) for a fictitious bank. There is no build, lint, or test tooling and no package manager. Run it by opening `index.html` directly in a browser (double-click, no server).

There are no tests, so verify changes by hand: open the file, check the browser console is clean, and exercise drag/drop, the Move ▸ select, the inline delete confirm, the filters and the Add Task form. While `FORMSUBMIT_ENDPOINT` still holds the placeholder address, every submit intentionally shows the "email notification failed" warning toast.

## Hard constraints (from the original brief)

- Vanilla HTML/CSS/JS in one file. No frameworks, bundlers, CDNs, web fonts, or image files. Icons are Unicode glyphs or inline SVG.
- No persistence of any kind (no localStorage, sessionStorage, IndexedDB, cookies). A refresh must reset the board to seed data. The UI says so.
- The only network call is `notifyNewTask()` to FormSubmit's AJAX endpoint (`FORMSUBMIT_ENDPOINT`, the first constant in the script). The email address lives only there. A FormSubmit failure must never break the board; it only produces a warning toast.
- No real UOB logo or branding. The wordmark is plain "IT PMO" text and the palette is corporate blue. The `UOB-ITPM-####` task IDs and the `[UOB IT PMO]` email subject are intentional.
- A `<meta>` Content-Security-Policy allows the inline script by SHA-256 hash. After ANY edit to the `<script>` block, regenerate the hash (sha256, base64, of the text, with line endings normalised to LF, between `<script>` and `</script>`) and update `script-src`, or the page will not run.
- No `!important`; styling uses the CSS custom properties in `:root`.

## Architecture

All logic is in the `<script>` block and follows one pattern: **mutate `state`, then call `renderBoard()`.**

- A Board/Timeline view switch (`state.view`) shows the same filtered tasks either as lanes or as due-date dots grouped by project (`renderTimeline()`, called from `renderBoard()`; pixel scale is `TL_PPD`).
- `state = { tasks, filters, view, nextId, confirmDeleteId, dragId, dropStatus }` is the single source of truth. `confirmDeleteId`, `dragId` and `dropStatus` are transient UI state that drives the inline "Delete? Yes/No" confirm and the drag-over highlight.
- `renderBoard()` rebuilds all four columns, via `renderCard()`, from `applyFilters(state.tasks)`, then calls `renderSummary()`. Card and column markup must only be written there. Do not mutate card DOM elsewhere.
- Event handling is delegated on `#board` (click, change, and the drag events), because the board's innerHTML is replaced on every render.
- Drag and drop: `dragstart` must NOT re-render, since that cancels the drag. `dragover` toggles the `drop-target` class directly on the columns. `drop` and `dragend` clear the state and re-render.
- The Move ▸ `<select>` on each card is the keyboard fallback and calls the same `moveTask()`.
- Form flow (`handleSubmit`) is optimistic: validate, `addTask()`, close the dialog, reset the form, show the toast, then `await notifyNewTask()` in try/catch. The submit button is disabled and shows "Sending…" until it settles.
- All user-supplied strings must go through `escapeHtml()` before being placed in template-literal HTML.
- Dates are `YYYY-MM-DD` strings compared lexicographically against `todayISO()` (local time). Seed due dates are offsets from today, so the demo always has overdue items.
- The `<script>` is ordered: config, constants, date helpers, state and seed data, utilities and filtering, rendering, mutations, toasts, FormSubmit, form validation and submit, setup and events, then `init()`. Add new code in the matching section.
- The Add Task `<dialog>` is opened with `resetForm()` and then `showModal()`. `resetForm()` also refreshes `dueDate.min` to today, so keep that order.
- Task IDs come from `makeId()`, which increments `state.nextId`. `seedTasks()` also uses it, so the first user task is `UOB-ITPM-0009`.
