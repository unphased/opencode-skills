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

## OpenCode State Isolation (Critical)

If you run multiple OpenCode instances concurrently against the same repo history (including separate git worktrees and even separate clones), they can collide on persisted OpenCode state unless you isolate the XDG directories per head.

Rule of thumb: one parallel head = one worktree + one OpenCode "profile" (unique `XDG_*` dirs).

Recommended env wrapper (per head):

```bash
# pick a stable name per head, e.g. "myrepo-auth-refactor"
OC_PROFILE="myrepo-auth-refactor"
OCROOT="$HOME/.local/share/opencode-heads/$OC_PROFILE"

XDG_DATA_HOME="$OCROOT/data" \
XDG_CACHE_HOME="$OCROOT/cache" \
XDG_STATE_HOME="$OCROOT/state" \
  opencode
```

Notes:

- Prefer `XDG_*` overrides over relying on default locations.
- Isolating `XDG_CONFIG_HOME` can hide your normal OpenCode config/plugins; only isolate config if you know you want a fully clean profile.
- Avoid running multiple heads with `--continue` unless you explicitly want them to resume the same recent session.

## Prefer Scripts

This repo includes two scripts that implement the workflows safely:

- `scripts/oc-bifurcate`: run inside tmux to split a pane and start a parallel head
- `scripts/oc-head`: create a worktree head and start OpenCode (tmux optional)

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

From the active tmux pane:

```bash
scripts/oc-bifurcate --feature auth-refactor --prompt "ulw Implement auth refactor. Context: ... Requirements: ..."
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
4. **Restart with isolated XDG dirs** in the new worktree (best), or accept collision risk
5. **Change to worktree** by telling user to `cd` (or note you're ready)
6. **Acknowledge** and await instructions

### Execution

Create the worktree head and start OpenCode:

```bash
scripts/oc-head --feature auth-refactor
```

If you want to run in the current terminal (no tmux):

```bash
scripts/oc-head --feature auth-refactor --no-tmux
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

## Default Safety Policy

- If you spawn a head, always isolate OpenCode state with per-head `XDG_*` dirs.
- If the user refuses isolation, warn: snapshot/diff/undo views can be misleading in parallel heads.
