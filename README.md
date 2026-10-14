# IT PMO Project Board

A single-page Kanban board for a fictitious bank's internal IT PMO. It is a demo and training tool, not an official system, and it uses no real bank logo or branding.

**Live demo:** https://bibhutinus-commits.github.io/Bibhuti-repo/ (original design)
**Version 2 (redesign with Timeline view):** https://bibhutinus-commits.github.io/Bibhuti-repo/v2/ (published from the `v2` branch)

## Features

- Board and Timeline views. The Timeline plots each task on its due date, one row per project, with a Today marker.
- A header status bar showing how tasks split across columns.
- Four fixed columns: Backlog, In Progress, Blocked, Done, each with a live count.
- Drag and drop cards between columns, or use the keyboard-friendly "Move ▸" control on each card.
- Cards show task ID, title, project, assignee, priority, due date and category. Overdue tasks get a badge.
- Add Task form with inline validation and optimistic updates.
- Filters by project, assignee and priority, plus a live summary strip.
- Optional email notification of new tasks through [FormSubmit](https://formsubmit.co).

## Run it

Open `index.html` in a browser. There is no build step, server or dependency.

## Configure notifications

In the `<script>` of `index.html`, change `FORMSUBMIT_ENDPOINT` to use your own email address. FormSubmit needs a one-time activation: the first submission sends a confirmation email to that address, and submissions are only delivered after you click the link in it. While the placeholder address is in place, adding a task shows a warning toast and the card is still added.

## Notes

- Vanilla HTML, CSS and JavaScript in one file. No frameworks, CDNs, fonts or images.
- Nothing is stored. The board lives in memory and resets to the demo data on refresh.
- Deployed to GitHub Pages by `.github/workflows/pages.yml`, which publishes only `index.html`.
- `.claude/commands/publish-github.md` is a Claude Code command that scans the project, then pushes, updates the README and deploys Pages.
