---
name: use-herdr
description: >-
  Use Herdr to inspect or control terminal panes, tabs, workspaces, and
  terminals, run commands in another pane, or start and monitor background
  work such as dev servers. Use it for subagents only when the user or another
  skill explicitly asks. Requires a Herdr-managed client. For OpenCode server
  mode, use the session-title check below to find its pane.
---

# Herdr

Herdr organizes terminals into workspaces, tabs, and panes. Its CLI lets you work with that layout and with the agents running inside it.

Before using a Herdr command, check whether this process is running in a Herdr-managed pane:

```bash
test "${HERDR_ENV:-}" = 1
```

If the check passes, use the `herdr` CLI to inspect or control that session. If it fails, do not inspect or control the focused Herdr session from outside Herdr. If OpenCode is running through a shared or detached server, check the client pane below before deciding Herdr is unavailable.

## OpenCode server mode

OpenCode's server can run outside Herdr while its client runs in a Herdr pane. In that setup, this process does not have `HERDR_ENV`; the client process does. First list the panes belonging to OpenCode clients inside Herdr:

```bash
for pid in $(pgrep -x opencode 2>/dev/null); do
  envs=$(ps eww -p "$pid" 2>/dev/null)
  case "$envs" in
    *HERDR_ENV=1*) printf '%s\n' "$envs" | tr ' ' '\n' | sed -n 's/^HERDR_PANE_ID=//p' ;;
  esac
done | sort -u
```

Then get the current OpenCode session title:

```bash
opencode api get "/api/session/${OPENCODE_SESSION_ID}"
```

Match the returned `data.title` against the client panes' `terminal_title` values from `herdr pane list`. Herdr may add a client prefix such as `OC | ` and truncate the title, so ignore the prefix and match the visible title text against the start of the session title. If exactly one pane matches, target it explicitly. Do not use `--current` from the server.

If no pane matches, or more than one does, show the pane IDs and titles and ask which one is the user's. If the client check found no panes, say the client is not in Herdr and stop.

## Use the installed guide

The installed Herdr binary has the current command guide. Read it before using Herdr, and check the relevant command group when you need exact flags:

```bash
herdr --skill
herdr --help
herdr pane
```

Do not run bare `herdr` to discover commands. It launches or attaches the TUI. Do not probe a command that changes state by leaving out its arguments, since some commands run with defaults.
