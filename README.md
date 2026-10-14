# herdr-overview

A live grid of every [Herdr](https://github.com/herdrdev/herdr) agent in one tab — local session plus every enabled saved machine. One cell per agent; the grid follows agents as they come and go.

## What you get

- **Overview mode** (default): each cell shows a read-only text copy of the agent's visible screen, refreshed every 2s. Click a cell (or press Enter) to jump to that agent. Remote agents attach in place over SSH instead.
- **Live mode** (`--live`): each cell is a real `herdr agent attach` — it takes input. Agents shrink to cell size while attached.
- A background controller rescans agents every 5s, rebuilds the grid when the set changes, and on close resets every agent's terminal to its own pane's size.
- Unreachable machines keep their last known agents, shown as `unreachable`.

## Requirements

- `herdr` in `PATH`, running inside a Herdr pane (`HERDR_ENV=1`)
- `jq`, `bash` 4+, `ssh` for remote machines
- Saved machines configured via `herdr machine`

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/jlapenna/herdr-overview/main/bin/herdr-overview \
  -o ~/.local/bin/herdr-overview && chmod +x ~/.local/bin/herdr-overview
```

Or clone and link:

```bash
git clone https://github.com/jlapenna/herdr-overview && cd herdr-overview
install -m 0755 bin/herdr-overview ~/.local/bin/
```

## Usage

From any Herdr pane:

```bash
herdr-overview              # overview: read-only text copies, refreshed every 2s
herdr-overview --live       # live view: cells attach and take input
herdr-overview --no-focus   # either mode, without switching to the new tab
```

- **Close the tab to stop.** The controller notices and cleans up (live mode restores agent sizes).
- **Detach from an attached cell:** `ctrl+b q` (attach's fixed detach key). If your outer Herdr prefix is also `ctrl+b`, press it twice to pass one through — the tool detects this and shows the right hint.
- The root pane of the new tab tails the controller log; per-tab logs live under `${XDG_STATE_HOME:-~/.local/state}/herdr-overview/`.

## Configuration

| Variable | Default | Meaning |
| --- | --- | --- |
| `HERDR_MAIN_SOCKET` | `~/.config/herdr/herdr.sock` | local main session socket |
| `HERDR_OVERVIEW_INTERVAL` | `2` | overview cell refresh (seconds) |
| `HERDR_OVERVIEW_SCAN` | `5` | controller agent rescan interval (seconds) |

## How it works

`herdr-overview` is one bash script with four modes, dispatching on its first argument:

1. **launcher** — creates a labeled tab, starts the controller detached (`setsid nohup`).
2. **`--controller`** — polls `herdr agent list` on the local socket and every enabled machine, computes a near-square grid, splits panes, and `herdr pane run`s a cell per agent. Rebuilds only when the agent set changes.
3. **`--cell`** — one overview cell: polls `herdr agent read --source visible`, repaints in place (no flicker), enables SGR mouse reporting so a left click focuses the agent.
4. **`--view`** — one live cell: runs `herdr agent attach` (over `ssh -t` for remote machines), restores the agent's own pane size on detach, reattaches on Enter.

`SKILL.md` in this repo teaches coding agents to run and reason about the tool.

## License

MIT
