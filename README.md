# agent-qordrift

**One dashboard for all your project agents.** Each agent runs the work for one folder — commands, dev servers,
database checks and AI tasks — and shows live progress, logs and a clear **Success** or **Failed at step N**.
Run one agent or all of them, one after another or in parallel. Free for macOS and Windows.

![agent-qordrift demo: agent cards running, a failed step and its log, then another project passing](screenshots/demo.gif)

<sub>[Watch the demo as a video](screenshots/demo.mp4) · the projects in the demo are sample data.</sub>

## What is an agent?

An **agent** is a card on the dashboard: a folder plus the things to do in it, in order. You build an agent from
four kinds of building blocks and mix them freely:

| Building block | What it does | Typical use |
|---|---|---|
| ⚙️ **Command** | Runs a shell command in the agent's folder | lint, typecheck, unit tests, E2E, builds, deploy scripts |
| 🖥️ **Service** | Starts a long-running process, waits until it is ready, stops it afterwards | dev server, API, `docker compose up`, emulators |
| 🗄️ **Query** | Runs SQL against Postgres or Supabase and checks the result | "no orders stuck in pending", data health checks, reports |
| ✨ **AI task** | Lets the AI do what your prompt asks in the agent's folder (optional, your own key) | small code changes, scaffolding, fixing a failing test |

Steps run top to bottom and stop at the first failure. Services start first, so your steps can test a running app.

## Many agents, one dashboard

Most people have more than one project, and more than one thing to check per project. agent-qordrift is built for
that:

- **Projects group agents.** For example, *Acme Shop* with a **Local checks**, a **Staging smoke** and a
  **Mobile app** agent, each with its own folder. Give a project a logo and its agents carry it.
- **Run one, or Run all.** Run all goes through every agent one after another (or in parallel, with a limit you set),
  skipping agents that are already running.
- **Schedules.** Run an agent once at a set time, every few minutes, hours or days, or on chosen weekdays.
- **Find things fast.** Search, filter by status (Running, Failed, Passed) or project, and tag agents.
- **Live and hands-off.** Live colored logs per step and service, Pause, Resume, Stop (which ends the whole process
  tree), desktop notifications, a tray icon with the overall status, and a history of each agent's last runs with
  the git branch and commit.
- **Bookmarks.** Keep each project's links (staging site, dashboard, docs) one click away.

## Building blocks in detail

You set agents up in the editor (**Add agent**). The editor also has a YAML view; the examples below are what
each building block looks like there.

### ⚙️ Commands

```yaml
steps:
  - name: Lint
    run: npm run lint
  - name: Unit tests
    run: npm test
  - name: E2E
    dir: web              # a subfolder of the agent's folder
    run: npm run e2e
    timeout: 45m          # default 30m
  - name: Coverage report
    run: npm run coverage
    continue_on_error: true   # a failure here doesn't stop the run
```

Commands run with your normal shell environment (nvm, pyenv, Homebrew, Go…), even when agent-qordrift is opened from
the Dock or the Start menu. On Windows they run with `cmd.exe` (PowerShell 7 is an option in Settings). Turn on
**interactive** for a step that reads keys, such as `flutter run` (press `r` to hot reload in the Logs view).

### 🖥️ Services

```yaml
services:
  - name: API
    run: php artisan serve --port 8000
    ready: { port: 8000, timeout: 90s }     # or url: http://localhost:8000/health, or log: "Server started" (default wait 60s)
  - name: Database
    run: docker compose up -d db
    stop: docker compose down               # runs when the service stops
stop_services_after_steps: true             # e.g. for E2E: tear down when the steps finish
```

- A service counts as ready when its **port** opens, a **URL** answers, or a **log line** appears.
- If the port is already taken by something else, the run fails right away instead of testing an old server.
- The card's **Run** button offers **Run all** (services, then steps), **Run without services**, and
  **Run services only**, which keeps your dev servers up until you press Stop.

### 🗄️ Queries

```yaml
secret_env: [DATABASE_URL]      # its value is kept in your system keychain, never in the file
steps:
  - name: No stuck orders
    query:
      connection: DATABASE_URL
      sql: select id from orders where status = 'pending' and created_at < now() - interval '1 hour'
      expect: no_rows           # or rows, or leave it out for a report that always passes
```

Queries run with `psql` (install PostgreSQL's command-line tools) and work with Postgres and Supabase. They are
**read-only** unless you turn on `allow_writes`, and the result shows up as a table in the log.

### ✨ AI tasks (optional)

```yaml
steps:
  - name: Add discount codes
    prompt: Add support for the SAVE10 discount code in the cart total, with a unit test.
  - name: Tests
    run: npm test             # check the AI's work with a normal command
```

AI tasks need **Agent mode** (Settings → AI) and your own Claude or OpenAI API key. The AI can read and change files
only inside the agent's folder, never `.env` files, keys, `.git` or `node_modules`. The earlier version of every
file it changes is kept on your computer for 90 days.

### A complete agent

Put together, one agent can start your app, check it, check the database and hand a task to the AI:

```yaml
name: Acme Shop
title: Staging smoke
project: acme-shop
path: ~/code/acme-shop
tags: [web, staging]
secret_env: [DATABASE_URL]
services:
  - name: Storefront
    run: npm run dev
    ready: { port: 5173 }
stop_services_after_steps: true
steps:
  - name: Smoke tests
    run: npx playwright test smoke
  - name: No stuck orders
    query:
      connection: DATABASE_URL
      sql: select id from orders where status = 'pending' and created_at < now() - interval '1 hour'
      expect: no_rows
on_success:
  message: Staging looks good
```

## When something fails

The card turns red and says exactly where: **Failed at step 3: Unit tests exited with code 1**. One click opens the
log of that step. With an AI key you can also get:

- **AI analysis** of the failure: the cause, where it is (opens your editor at the line) and how to fix it.
- **Fix with AI** (Agent mode): the AI makes the fix, backs up the files, runs the agent again, and you can
  **Show changes** or **Undo**.

## Two modes

| | **AI off** (default, no key) | **AI on** (your AI key) |
|---|---|---|
| Set up | Nothing | Settings → AI → Claude or OpenAI → paste key → Test key |
| Commands, services, queries, logs, Pause/Stop, history, notifications | ✅ | ✅ |
| AI tasks and Fix with AI | — | ✅ with Agent mode on |
| When a step fails | "Failed at step N" + the log | The same, plus a short AI analysis: cause, where, fix |
| Network use by the app | None (only the optional update check) | Provider calls for failed runs and AI tasks only |
| Cost | Free | Free app; you pay your provider for your own API usage |

Works with Claude (Anthropic) and OpenAI models. agent-qordrift is not affiliated with or endorsed by either.

## Download

Get the latest version from [**Releases**](../../releases/latest):

| System | File |
|---|---|
| macOS 11 or later (Apple Silicon and Intel) | `agent-qordrift_<version>_universal.dmg` |
| Windows 10 (1809+) / 11 | `agent-qordrift_<version>_x64_en-US.msi` or `agent-qordrift_<version>_x64-setup.exe` |

## Install

agent-qordrift is free and not yet signed with a paid Apple or Microsoft certificate, so your system asks you to
confirm the first time you open it.

### macOS

1. Open the `.dmg` and drag **agent-qordrift** to **Applications**.
2. Open agent-qordrift. macOS says it can't verify the developer. Click **Done**.
3. Open **System Settings → Privacy & Security**, scroll down and click **Open Anyway** next to the message
   about agent-qordrift, then confirm. (On macOS 15 and later, right-click → Open no longer skips this step.)
4. If macOS says the app **"is damaged and can't be opened"**, run this once in Terminal, then open it again:

   ```bash
   xattr -dr com.apple.quarantine /Applications/agent-qordrift.app
   ```

**Keychain prompt (only with an AI key or secrets):** keys and secret variables are kept in the macOS Keychain.
Until agent-qordrift has an Apple Developer ID, macOS may ask "agent-qordrift wants to use your keychain" after each
update. Choose **Always Allow**.

### Windows

1. Run the `.msi` or `-setup.exe`.
2. If Windows SmartScreen says "Windows protected your PC", click **More info → Run anyway**.
3. The installer adds Microsoft Edge WebView2 if it is missing (most Windows 10/11 PCs already have it).

## Quick start

1. Click **Add your first agent** (or **Import example**).
2. Pick a project (or type a new one), the agent's folder, and a name such as *Local checks*.
3. Add steps and services, then click **Save** and **Run**.

Agents can be exported and imported as `.yaml` files. An imported agent shows its commands and asks before its
first run.

## Privacy

No accounts, no telemetry, no analytics. Your agents, logs and history stay on your computer. With AI on, only a
masked excerpt of a failed step's log is sent, to the provider you chose, with your key; AI tasks and Fix with AI
also send the files the AI reads, masked the same way. API keys and secret variables are kept only in your system
keychain. Read the full [privacy statement](PRIVACY.md).

## Updates

agent-qordrift checks this repository for new versions at startup (you can turn this off in **Settings → Updates**) and
installs them with one click. Updates are signed and verified before they are installed. See the
[changelog](CHANGELOG.md).

## License

agent-qordrift is free for personal and commercial use under the [agent-qordrift freeware license](LICENSE): no
redistribution, modification or reverse engineering; provided "as is", without warranty. You are responsible for
the commands you configure. It includes open-source software listed in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## Feedback

Found a bug or have an idea? [Open an issue](../../issues). Security issues: please use
[private vulnerability reporting](../../security/advisories/new) instead.

## Support

agent-qordrift is free. If it saves you time, you can support it with a donation from **Settings → Donate** or
[paypal.me/jsxd3v](https://paypal.me/jsxd3v). Donations are voluntary and don't buy support, features or other
rights.
