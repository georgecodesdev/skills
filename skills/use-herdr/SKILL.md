---
name: use-herdr
description: "Use Herdr to manage terminal layout and long-running work, read or wait on pane output, or run a second agent. Use it when the task needs one of those. Requires a Herdr-managed client. For OpenCode server mode, use the session-title check below."
---

# Herdr

Herdr organizes terminals into workspaces, tabs, and panes. Its CLI lets you work with that layout and with the agents running inside it.

## When to use it

Use Herdr when:

* A command will keep running while you do other work, such as a dev server, watcher, or long test suite.
* You need to read or wait on output without blocking.
* A second agent would help with independent work or a second opinion.
* The user wants to watch or click around while work continues.

Do not use it when:

* A single short command does the job.
* A workflow skill owns the step. Follow that workflow, and use Herdr only for the terminal or agent work it calls for. Herdr does not replace planning, execution, review, or diagnosis.
* You are not inside a Herdr client.
* No terminal or agent is needed.

## Examples

* Run the dev server and give the user a URL to click around.
* Open a plan or walkthrough in a pane so the user can read it while work continues.
* Run the test suite in a pane and read the result without blocking.
* Spawn a reviewer agent for a second opinion.
* Watch a long build or scrape and report when it finishes.

### Reading a document in a pane

Render a plan or walkthrough in the pane instead of dumping the raw Markdown.

Render the Markdown with `rich`, and render each Mermaid fence with `mermaidx` as terminal text:

```bash
python3 -m rich.markdown <file>
mermaidx -i diagram.mmd -o diagram.txt && cat diagram.txt
```

Show the diagram output in place of the Mermaid fence. Both stay crisp in any terminal and need no graphics support. Install them with `pip install rich mermaidx` in a virtual environment.

Images still need a graphics viewer. If the pane cannot show one, say so rather than printing a path.

## Common features

* **Pane**: run a process in a pane, then read or wait on its output.
* **Agent**: start a second agent, prompt it, and wait for it.
* **Worktree**: an isolated branch for parallel work. An option, not a default.
* **Notification**: alert the user when a long job finishes.

Ask before using remote machines, plugins, named sessions, or integrations.

## Confirm you are inside Herdr

Before using a Herdr command, check whether this process is running in a Herdr-managed pane:

```bash
test "${HERDR_ENV:-}" = 1
```

If the check passes, use the `herdr` CLI. If it fails, do not inspect or control the focused Herdr session from outside Herdr. If OpenCode is running through a shared or detached server, check the client pane below before deciding Herdr is unavailable.

### OpenCode server mode

OpenCode's server can run outside Herdr while its client runs in a Herdr pane. In that setup, this process does not have `HERDR_ENV`. The client process does. First list the panes belonging to OpenCode clients inside Herdr:

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
