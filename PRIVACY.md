# Privacy statement

_Last updated: 2026-11-13_

agent-qordrift is a desktop app. It has **no accounts, no cloud sync, no telemetry, no analytics and no crash uploads**.

## What stays on your computer

Your agents (`agents.yaml`), settings, run history and logs are stored only in the app's folders:

- macOS: `~/Library/Application Support/com.jsxd3v.agent-qordrift/`
- Windows: `%APPDATA%\com.jsxd3v.agent-qordrift\`

Logs on disk are not masked. They never leave your computer, except as described under "AI on" and "Agent mode".

## Network use

**AI off (the default, no AI key):** the app makes no network requests, except the optional update check. At startup it
downloads a small version file (`latest.json`) from this public GitHub repository. You can turn this off in
**Settings → Updates**. GitHub's own privacy policy applies to that download.

Links you click (bookmarks, a service's address, **GitHub**, **Donate**) open in your web browser; the app itself sends
nothing to those sites. When a service has a readiness check with a URL (usually your own `localhost`), the app
requests that URL until the service answers. The commands and database queries you configure make whatever network
connections they make on their own.

**AI on (optional, with your own Claude or OpenAI API key):** when a step fails, agent-qordrift sends an analysis
request to the provider you chose, using your key. The request contains:

- the agent name, the failed step's name and command, its folder relative to the agent folder, the exit code or
  "timeout", and your operating system;
- the names and commands of the earlier steps that passed;
- the names, ports and states of the services the agent started before its steps (or that it started none);
- an excerpt of the failed step's log, at most 24 KB (its start, the lines with errors, and its end).

Before sending, secrets are masked (API keys, tokens, passwords, private keys, credentials in URLs, and similar)
and home-folder paths are replaced by `~`. Environment variables and file contents are never sent. You can see the
exact text of each request with **Show what was sent**, turn AI analysis off for a single agent, or turn it off
completely. Stopped runs are never sent.

**Agent mode (optional, its own switch and notice):** only when Fix with AI runs (when you click **Fix with AI**
on a failed run, or by itself after a failed run if you turn on **Start automatically**), agent-qordrift also sends the contents of the project files the AI asks to read, masked the same way. The AI can
only read inside the agent's folder and never reads `.env` files, private keys, credential files, `.git`,
`node_modules` or build output. You can see every file and line range it read. Its changes are written right
away, after a backup, and the agent runs again; **Show changes** shows them and **Undo** puts the files back. The app never
changes your git repository or commits: it only reads the branch, last commit and remote address to show them on the card. You can turn Agent mode off for a single agent or completely.

**AI tasks (Agent mode):** an agent can have AI task steps: a prompt you write, such as "Fix the bug where the
cart total ignores the discount code". Each time such an agent runs, agent-qordrift sends your prompt, a list of
the files at the top of the agent folder, and the contents of the files the AI asks to read, masked the
same way and with the same limits as Fix with AI. The AI's changes are written when the task finishes. The earlier
version of every changed file is saved on your computer for 90 days. AI tasks run only while Agent mode is on;
otherwise the step fails without sending anything.

What the provider does with the request is covered by your agreement with that provider:
[Anthropic](https://www.anthropic.com/legal/privacy) · [OpenAI](https://openai.com/policies/privacy-policy/).

## API keys

API keys are stored only in your system keychain (macOS Keychain or Windows Credential Manager). They are never
written to `agents.yaml`, settings, logs, history or crash reports, and the app's interface only shows their last
four characters.

## Your commands

The commands you configure run on your computer with your permissions. agent-qordrift only runs them when you click
**Run** or **Run all**. Agents imported from a file show their commands and ask for confirmation before their first
run.

## AI answers

AI analysis and AI fixes are generated automatically and can be wrong or incomplete. Review them before acting on
them, and review applied changes with `git diff` before you commit.

## Contact

- **Security issues:** report them privately with GitHub's
  [private vulnerability reporting](https://github.com/jsxd3v/agent-qordrift/security/advisories/new) for this repository.
- **Other questions about privacy:** open an issue in this repository. Issues are public, so please don't post
  personal data, API keys or unmasked logs in them.
