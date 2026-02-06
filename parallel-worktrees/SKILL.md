---
name: parallel-worktrees
description: Spawn an independent opencode instance in a git worktree. User says "spawn a head for X" → agent creates branch, worktree, tmux pane with opencode, sends prompt, returns to current conversation.
license: MIT
compatibility: opencode
metadata:
  category: orchestration
  triggers: spawn a head, parallel head, spin up, worktree, dual head, side quest
---

# Parallel Worktrees

Spawn an independent opencode peer in a git worktree, then resume current work.

## Trigger Phrases

User says any of:

- "spawn a head for X"
- "spin up a parallel head to do X"
- "dual head this - you keep planning, spawn execution for X"
- "side quest: X" (implies spawn)

## What You Do

1. **Create branch** from main (or user-specified base)
2. **Create worktree** in sibling directory
3. **Spawn tmux pane** with opencode in that worktree
4. **Send initial prompt** to the new instance
5. **Report back** briefly and resume current conversation

## Execution

```python
# 1. Determine names
feature = "auth-refactor"  # derived from user request
branch = f"feature/{feature}"
worktree_dir = f"../{project_name}-{feature}"  # sibling to current dir

# 2. Create branch and worktree
bash(f"git branch {branch} main")  # or origin/main
bash(f"git worktree add {worktree_dir} {branch}")

# 3. Spawn tmux pane with opencode
interactive_bash(tmux_command=f'split-window -h -c "{absolute_worktree_path}"')
interactive_bash(tmux_command='send-keys "opencode" Enter')

# 4. Wait for opencode to initialize, then send prompt
# (2-3 second delay, or use send-keys with the prompt after opencode starts)
import time; time.sleep(3)
prompt = "ulw Implement auth refactor. Context: ... Requirements: ..."
interactive_bash(tmux_command=f"send-keys '{prompt}' Enter")

# 5. Report and resume
# "Spawned parallel head on feature/auth-refactor in tmux pane. Continuing..."
```

## Prompt for the Spawned Instance

Must be **self-contained** (it has no context from this session):

```
ulw [Task description]

Context:
- Branch: [branch-name]
- Key files: [list]
- Patterns to follow: [conventions]

Requirements:
1. [Specific requirement]
2. [...]

When done:
- Ensure tests pass
- Commit with descriptive message
- The branch will be merged back to main
```

## After Spawning

- **You**: Continue current conversation immediately
- **User**: Can `tmux attach` or switch panes to monitor
- **Later**: Merge branch back when work completes

```bash
# When parallel work is done (user confirms)
git merge feature/auth-refactor --no-edit
git worktree remove ../project-auth-refactor
git branch -d feature/auth-refactor
```

## Key Points

- Spawned instance is a **peer**, not a subagent (no IPC, no result return)
- Results come back via **git merge**, not tool output
- Keep prompts detailed since spawned instance starts fresh
- Resume your current work immediately after spawning
