# Changelog

All notable changes to agent-qordrift. Versions follow [Semantic Versioning](https://semver.org/).

## [1.0.1] — 2026-11-15

- **Home button** before the search box: one click clears the search and filters and goes back to Home.
- **License:** the Windows installers show the license and ask you to accept it; the first launch says that
  continuing means agreeing to the license and the privacy statement; **About** links to the license.
- **About** shows the author and copyright: © 2026 Jayvee S. (jsxd3v).

## [1.0.0] — 2026-11-14

First public release.

### Task mode

- Dashboard of agent cards: folder, ordered steps, live status and durations, Run / Pause / Resume / Stop.
- Projects that group agents, with logos; Home shows a tile per project. Each agent has its own folder.
- Bookmarks per project; tags and search on the dashboard.
- Database query steps (Postgres / Supabase via `psql`) that expect rows or no rows.
- Schedules: run an agent once or on a repeating interval.
- Steps run with your login-shell environment on macOS and `cmd.exe` (or PowerShell 7) on Windows; stdout and
  stderr in their real order, colors kept, closed stdin, per-step timeouts.
- Stop ends the whole process tree; no leftover processes after Stop, timeouts or quitting.
- Services with port / URL / log-line readiness and a port-in-use check.
- Run all, one after another or in parallel.
- Agent editor with YAML mode and validation; add, duplicate, reorder, delete; import and export.
- Log viewer with tabs, colors, search and copy; history with git branch and commit.
- Desktop notifications, tray icon, light and dark themes, launch at login, signed automatic updates.

### Agent mode (optional)

- AI failure analysis with your own Claude or OpenAI key: cause, location (opens your editor), fix and category.
- Secrets masked before sending; "Show what was sent"; per-agent switch; cached answers for repeated failures.
- Fix with AI: the AI reads the agent's files, writes a fix after a backup and runs the agent again; Show changes
  and Undo.
- AI task steps: a prompt the AI carries out in the agent folder as part of a run.
