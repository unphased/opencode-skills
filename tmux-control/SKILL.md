---
name: tmux-control
description: Control tmux sessions, windows, and panes (capture buffers, send keys). Uses bash/interactive_bash - no MCP required.
license: MIT
compatibility: opencode
metadata:
  category: terminal
  triggers: tmux, session, pane, window, capture, send-keys
---

# tmux-control

## Commands

```bash
# Sessions
tmux list-sessions
tmux new-session -d -s name           # -d = detached (always use this)
tmux kill-session -t name

# Windows and panes
tmux list-windows -t session
tmux list-panes -t session -a -F "#{session_name}:#{window_index}.#{pane_index}"

# Capture pane output
tmux capture-pane -t session:window.pane -p            # -p prints to stdout
tmux capture-pane -t session:window.pane -p -S -50     # last 50 lines

# Send keys
tmux send-keys -t session:window.pane "command" Enter
tmux send-keys -t session:window.pane C-c              # Ctrl+C
tmux send-keys -t session:window.pane C-d              # Ctrl+D
tmux send-keys -t session:window.pane Escape
```

## Target Syntax

`session:window.pane` — examples: `my-session`, `my-session:0`, `my-session:0.1`, `my-session:window-name.0`

## Identifying What's Running in a Pane

**Always fetch pane metadata alongside content when inspecting a pane.** Raw terminal text is ambiguous — pane titles and current commands are the reliable signal.

```bash
# Metadata for all panes in a window
tmux list-panes -t ":9" -F "#{session_name}:#{window_index}.#{pane_index} | title=#{pane_title} | cmd=#{pane_current_command} | pid=#{pane_pid}"

# Metadata for a specific pane
tmux display-message -t "0:9.2" -p "title=#{pane_title} cmd=#{pane_current_command} pid=#{pane_pid} cwd=#{pane_current_path}"
```

When asked what's in a pane, run metadata and content capture together — not content first and metadata as a fallback.

## interactive_bash Tool

For TUI apps requiring ongoing interaction (vim, htop, pudb, etc.).

**CRITICAL**: Pass tmux subcommands WITHOUT the `tmux` prefix.

```python
interactive_bash(tmux_command='new-session -d -s debug')
interactive_bash(tmux_command='send-keys -t debug "python -m pudb script.py" Enter')
interactive_bash(tmux_command='send-keys -t debug "n" Enter')
interactive_bash(tmux_command='capture-pane -t debug -p')
interactive_bash(tmux_command='kill-session -t debug')
```

## When to Use

- **Use tmux** for TUI apps, long-running processes, interactive programs, persistent background sessions
- **Use regular bash** for one-off commands that complete quickly
- **Capture before killing** — output is lost when a session dies
