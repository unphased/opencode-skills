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

Control tmux sessions, windows, and panes programmatically. Capture output, send commands, manage sessions.

## Quick Reference

| Task | Command |
|------|---------|
| List sessions | `tmux list-sessions` |
| List windows | `tmux list-windows -t session-name` |
| List panes | `tmux list-panes -t session-name` |
| Capture pane | `tmux capture-pane -t session:window.pane -p` |
| Send keys | `tmux send-keys -t target "command" Enter` |
| New session | `tmux new-session -d -s name` |
| Kill session | `tmux kill-session -t name` |

## Core Commands

### List Sessions
```bash
# All sessions with details
tmux list-sessions

# Just session names
tmux list-sessions -F "#{session_name}"
```

### List Panes
```bash
# All panes in session
tmux list-panes -t my-session -a

# With format: session:window.pane
tmux list-panes -t my-session -a -F "#{session_name}:#{window_index}.#{pane_index}"
```

### Capture Pane Output
```bash
# Capture entire scrollback
tmux capture-pane -t my-session:0.0 -p

# Capture last 50 lines
tmux capture-pane -t my-session:0.0 -p -S -50

# Save to file
tmux capture-pane -t my-session:0.0 -p > output.txt
```

### Send Keys
```bash
# Send command and press Enter
tmux send-keys -t my-session:0.0 "ls -la" Enter

# Send without Enter (for interactive prompts)
tmux send-keys -t my-session:0.0 "password123"

# Send special keys
tmux send-keys -t my-session:0.0 C-c  # Ctrl+C
tmux send-keys -t my-session:0.0 C-d  # Ctrl+D
tmux send-keys -t my-session:0.0 Escape
```

## Using interactive_bash Tool

For TUI apps requiring ongoing interaction (vim, htop, pudb, etc.).

**CRITICAL**: Pass tmux subcommands WITHOUT the `tmux` prefix.

### Create Session and Send Commands
```python
# Create detached session
interactive_bash(tmux_command='new-session -d -s dev-session')

# Send command to session
interactive_bash(tmux_command='send-keys -t dev-session "vim config.py" Enter')

# Wait, then send more keys
interactive_bash(tmux_command='send-keys -t dev-session ":wq" Enter')
```

### Capture Output from Session
```python
# Start a process
interactive_bash(tmux_command='send-keys -t dev-session "python script.py" Enter')

# Later, capture the output
interactive_bash(tmux_command='capture-pane -t dev-session -p')
```

### Interactive Debugging Example
```python
# Start debugger in tmux
interactive_bash(tmux_command='new-session -d -s debug')
interactive_bash(tmux_command='send-keys -t debug "python -m pudb script.py" Enter')

# Send debugger commands
interactive_bash(tmux_command='send-keys -t debug "n" Enter')  # next
interactive_bash(tmux_command='send-keys -t debug "s" Enter')  # step
interactive_bash(tmux_command='send-keys -t debug "c" Enter')  # continue

# Capture debugger output
interactive_bash(tmux_command='capture-pane -t debug -p')

# Cleanup
interactive_bash(tmux_command='kill-session -t debug')
```

## Common Patterns

### Run Command and Capture Output
```bash
# Create session, run command, capture, cleanup
tmux new-session -d -s temp "long-running-command"
sleep 5  # Wait for output
tmux capture-pane -t temp -p > results.txt
tmux kill-session -t temp
```

### Monitor Long-Running Process
```bash
# Start process in background session
tmux new-session -d -s monitor "npm run build"

# Check progress periodically
tmux capture-pane -t monitor -p -S -20  # Last 20 lines

# When done
tmux kill-session -t monitor
```

### Send Multi-Line Input
```bash
# Send multiple commands
tmux send-keys -t my-session "cd /app" Enter
tmux send-keys -t my-session "source venv/bin/activate" Enter
tmux send-keys -t my-session "python main.py" Enter
```

## Target Syntax

Targets specify session:window.pane:
- `my-session` - default window and pane in session
- `my-session:0` - window 0, default pane
- `my-session:0.1` - window 0, pane 1
- `my-session:window-name.0` - named window, pane 0

## Tips

1. **Always use `-d` for detached sessions** - prevents blocking
2. **Capture before killing** - output is lost when session dies
3. **Use `-p` flag** - prints to stdout instead of saving to buffer
4. **Target format matters** - `session:window.pane` is most explicit
5. **For one-shot commands** - use regular `bash` tool with `&` for background
6. **For TUI apps** - use `interactive_bash` tool

## When to Use

- ✅ Running TUI applications (vim, htop, pudb)
- ✅ Capturing output from long-running processes
- ✅ Sending commands to interactive programs
- ✅ Managing persistent background sessions
- ❌ Simple one-off commands (use `bash` tool instead)
- ❌ Commands that complete quickly (use `bash` tool instead)
