# ralph-start

Start Ralph's autonomous iteration loop.

## What This Skill Does

This skill starts Ralph's autonomous loop that repeatedly spawns the ralph-worker agent to implement tasks from fix_plan.md. The loop continues until all tasks are complete, a blocker is encountered, or the user stops it.

## Prerequisites

- Ralph must be initialized (`/ralph:init` has been run)
- `docs/ai/ralph/PROMPT.md` must exist
- `docs/ai/ralph/fix_plan.md` must exist
- `docs/ai/ralph/status.json` must exist

## Implementation

When this skill is invoked:

### 1. Validate Prerequisites

Check that all required files exist before starting:

```bash
# Check for required files
if [[ ! -f docs/ai/ralph/PROMPT.md ]]; then
  echo "Error: PROMPT.md not found. Run /ralph:init first."
  exit 1
fi

if [[ ! -f docs/ai/ralph/fix_plan.md ]]; then
  echo "Error: fix_plan.md not found. Run /ralph:init first."
  exit 1
fi

if [[ ! -f docs/ai/ralph/status.json ]]; then
  echo "Error: status.json not found. Run /ralph:init first."
  exit 1
fi
```

### 2. Clear Stop Signal (if present)

Remove any existing stop signal from a previous run:

```bash
# Remove stop signal if it exists
rm -f docs/ai/ralph/.ralph_stop
```

### 3. Update status.json to 'running'

Mark Ralph as running and record the start time:

```bash
# Update status.json with started_at timestamp and running status
jq '.status = "running" | .started_at = now | .last_updated = now' \
  docs/ai/ralph/status.json > /tmp/status.json.tmp
mv /tmp/status.json.tmp docs/ai/ralph/status.json

echo "Ralph started. Beginning autonomous iteration loop..."
```

### 4. Enter Main Loop

The loop repeatedly spawns ralph-worker and checks for completion:

```bash
# Main iteration loop
while true; do
  # Check for stop signal before each iteration
  if [[ -f docs/ai/ralph/.ralph_stop ]]; then
    echo ""
    echo "Stop signal detected. Exiting loop gracefully."
    rm -f docs/ai/ralph/.ralph_stop

    # Update status to stopped
    jq '.status = "stopped" | .last_updated = now | .stopped_at = now' \
      docs/ai/ralph/status.json > /tmp/status.json.tmp
    mv /tmp/status.json.tmp docs/ai/ralph/status.json

    echo "Ralph stopped after iteration $(jq -r '.iteration_count' docs/ai/ralph/status.json)."
    break
  fi

  # Get current iteration count
  ITERATION=$(jq -r '.iteration_count' docs/ai/ralph/status.json)
  NEXT_ITERATION=$((ITERATION + 1))

  echo ""
  echo "========================================="
  echo "Starting iteration $NEXT_ITERATION"
  echo "========================================="

  # Spawn ralph-worker using Task tool
  # Use subagent_type="ralph-worker" to invoke the agent
  # The worker will read context, pick a task, implement it, and return a summary

  # After spawning Task(ralph-worker), the result will be displayed in chat
  # The worker handles incrementing iteration_count in status.json

  # Check for user confirmation every 10 iterations
  if (( NEXT_ITERATION % 10 == 0 )); then
    # Use AskUserQuestion to prompt for continuation
    # Question: "Completed 10 iterations. Continue with more iterations? (yes/no)"
    # If response is not "yes", break the loop
  fi

  # After worker returns, check fix_plan.md for completion signals
  if grep -q "^RALPH_COMPLETE" docs/ai/ralph/fix_plan.md 2>/dev/null; then
    echo ""
    echo "RALPH_COMPLETE signal detected. All tasks complete!"

    # Update status to complete
    jq '.status = "complete" | .last_updated = now | .completed_at = now' \
      docs/ai/ralph/status.json > /tmp/status.json.tmp
    mv /tmp/status.json.tmp docs/ai/ralph/status.json

    # Report summary
    FINAL_ITERATION=$(jq -r '.iteration_count' docs/ai/ralph/status.json)
    echo ""
    echo "Ralph completed successfully!"
    echo "- Total iterations: $FINAL_ITERATION"
    echo "- Status: complete"
    echo "- All tasks in fix_plan.md have been completed."
    break
  fi

  if grep -q "^RALPH_BLOCKED" docs/ai/ralph/fix_plan.md 2>/dev/null; then
    echo ""
    echo "RALPH_BLOCKED signal detected. Cannot continue."

    # Extract blocker reason
    BLOCKER=$(grep "^RALPH_BLOCKED" docs/ai/ralph/fix_plan.md | head -1)

    # Update status to blocked
    jq '.status = "blocked" | .last_updated = now | .blocked_at = now' \
      docs/ai/ralph/status.json > /tmp/status.json.tmp
    mv /tmp/status.json.tmp docs/ai/ralph/status.json

    # Report summary
    FINAL_ITERATION=$(jq -r '.iteration_count' docs/ai/ralph/status.json)
    echo ""
    echo "Ralph blocked after $FINAL_ITERATION iterations."
    echo "- Status: blocked"
    echo "- Reason: $BLOCKER"
    echo ""
    echo "Review fix_plan.md to resolve the blocker, then run /ralph:start again."
    break
  fi
done
```

### 5. Orchestration Details

**Loop structure:**
- Check for `.ralph_stop` before each iteration
- Spawn `ralph-worker` via Task tool with `subagent_type="ralph-worker"`
- Display worker's iteration summary in chat
- Check `fix_plan.md` for `RALPH_COMPLETE` or `RALPH_BLOCKED` signals
- Prompt user every 10 iterations for confirmation
- Continue until completion signal, blocker, stop signal, or user declines

**Iteration counting:**
- The `ralph-worker` agent is responsible for incrementing `iteration_count` in `status.json`
- The `ralph-start` skill reads the current count before spawning each worker
- This ensures accurate tracking even if the loop is stopped and restarted

**User confirmation:**
- Every 10 iterations, use `AskUserQuestion` to ask: "Completed 10 iterations. Continue? (yes/no)"
- Only continue if user responds "yes"
- This prevents runaway loops and gives user control points

**Completion signals:**
- `RALPH_COMPLETE` at top of `fix_plan.md` = all tasks done successfully
- `RALPH_BLOCKED: [reason]` = cannot proceed, need user intervention
- Empty `fix_plan.md` Tasks section = treated same as `RALPH_COMPLETE`

## Coordination with /ralph:stop

The `/ralph:stop` skill creates a `.ralph_stop` signal file. This loop checks for that file before each iteration:
- If found, the loop exits gracefully
- The current iteration completes normally before checking
- The signal file is removed after detection
- Status is updated to 'stopped'

## Example Session Flow

```
User: /ralph:start

Ralph started. Beginning autonomous iteration loop...

=========================================
Starting iteration 1
=========================================
[ralph-worker spawns and returns summary]
Iteration 1: Implemented task "Add user authentication"
- Files changed: auth.ts, login.tsx
- Tests: pass
- Next: Add password reset flow

=========================================
Starting iteration 2
=========================================
[ralph-worker spawns and returns summary]
Iteration 2: Implemented task "Add password reset flow"
- Files changed: reset-password.tsx, email-service.ts
- Tests: pass
- Next: RALPH_COMPLETE

RALPH_COMPLETE signal detected. All tasks complete!

Ralph completed successfully!
- Total iterations: 2
- Status: complete
- All tasks in fix_plan.md have been completed.
```

## Important Notes

- **Fresh context**: Each ralph-worker starts with zero memory and reads all context from files
- **Atomic updates**: Use `jq` with temp files for atomic JSON updates to avoid corruption
- **Graceful exit**: Always allow current iteration to finish before stopping
- **Progress display**: Each iteration's summary is displayed in chat for user visibility
- **Resumable**: If stopped, `/ralph:start` can resume from current state (clears stop signal and continues)
