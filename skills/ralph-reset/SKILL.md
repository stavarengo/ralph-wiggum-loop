---
name: ralph-reset
description: Hard reset repository to last good state when Ralph gets stuck
---

# ralph-reset

Hard resets the repository to the last committed state, discarding all uncommitted changes and untracked files.

## Overview

This skill is used when Ralph gets stuck or the codebase is in an inconsistent state. It provides a way to return to the last clean checkpoint.

## WARNING

**This is a destructive operation!** It will:
- Discard all uncommitted changes in tracked files
- Remove all untracked files and directories
- Reset the repository to the last commit (HEAD)

**You cannot undo this operation!** Make sure you want to lose all uncommitted work before proceeding.

## Implementation

```bash
# Step 1: Confirm destructive action with user
Use AskUserQuestion to confirm:
"⚠️  WARNING: This will discard ALL uncommitted changes and untracked files.
This cannot be undone. Are you sure you want to proceed? (yes/no)"

# Step 2: If user confirms (response == "yes")
# Execute git reset and clean
git reset --hard HEAD
git clean -fd

# Step 3: Update status.json
Read docs/ai/ralph/status.json
Update the 'status' field to 'reset'
Update the 'lastReset' timestamp to current ISO 8601 time
Write back to docs/ai/ralph/status.json

# Step 4: Report results to user
echo "✅ Repository reset complete"
echo "- All uncommitted changes discarded"
echo "- All untracked files removed"
echo "- Repository at: $(git rev-parse --short HEAD)"
echo "- Status updated to: reset"

# Step 5: If user does not confirm
echo "❌ Reset cancelled by user"
exit 0
```

## Usage

Invoke this skill when:
- Ralph is stuck in an iteration loop
- The codebase is in an inconsistent state
- You want to return to the last committed checkpoint
- Tests are failing due to leftover artifacts

## Related

- Use `git tag` (from Step 8 of Ralph iterations) to create clean checkpoints
- Use `/ralph:stop` before reset to cleanly end the current iteration
- After reset, use `/ralph:start` to begin fresh iterations
