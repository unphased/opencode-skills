---
name: parallel-worktrees
description: Manage parallel work across git worktrees. Two modes - "spawn a head" (create tmux+opencode) or "init parallel head" (user already in new instance, just set up worktree).
license: MIT
compatibility: opencode
metadata:
  category: orchestration
  triggers: spawn a head, parallel head, spin up, worktree, dual head, side quest, init parallel head, init me
---

# Parallel Worktrees

Two workflows for parallel development using git worktrees.

---

## Mode 1: Spawn a Head (from existing session)

**Use when**: You're deep in a session and want to spin off parallel work without interrupting.

### Trigger Phrases

- "spawn a head for X"
- "spin up a parallel head to do X"
- "dual head this - you keep planning, spawn execution for X"
- "side quest: X"

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

---

## Mode 2: Init Parallel Head (you're the new instance)

**Use when**: User manually created a tmux pane, started opencode, and wants YOU to set up the worktree.

### Trigger Phrases

- "init me a new parallel head here"
- "init parallel head for X"
- "set up this instance as a parallel head"
- "I'm a new head, set me up for X"

### What You Do

1. **Confirm current directory** (should be in the main repo, not a worktree yet)
2. **Create branch** from main (or user-specified base)
3. **Create worktree** in sibling directory
4. **Change to worktree** by telling user to `cd` (or note you're ready)
5. **Acknowledge** and await instructions

### Execution

```python
# 1. Determine names from user request
feature = "auth-refactor"
branch = f"feature/{feature}"
project_name = os.path.basename(os.getcwd())  # e.g., "myproject"
worktree_dir = f"../{project_name}-{feature}"

# 2. Check we're in main repo, not already a worktree
result = bash("git rev-parse --show-toplevel")
# Verify this is the main repo

# 3. Create branch and worktree
bash(f"git branch {branch} main")
bash(f"git worktree add {worktree_dir} {branch}")

# 4. Report to user
print(f"""
Parallel head initialized:
- Branch: {branch}
- Worktree: {worktree_dir}

To work there, either:
  cd {worktree_dir}

Or restart opencode in that directory.

I'm ready for your task once you're in the worktree.
""")
```

### After Init

User will either:

- Restart opencode in the worktree directory, OR
- Stay in current session and you work via absolute paths (less ideal)

Best practice: User restarts opencode in the worktree, then gives the actual task.

---

## Key Points

- Both modes create **peer instances**, not subagents (no IPC, no result return)
- Results come back via **git merge**, not tool output
- Mode 1: Agent spawns the new instance
- Mode 2: User spawns the instance, agent sets up the worktree
