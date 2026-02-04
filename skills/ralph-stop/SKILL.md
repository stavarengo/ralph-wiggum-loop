# ralph-stop

Stop Ralph's autonomous iteration loop gracefully.

## What This Skill Does

This skill stops the autonomous loop started by `/ralph:start`. It sets a stop signal, allows the current iteration to finish, and updates the status.

## Prerequisites

- Ralph must be initialized (`/ralph:init` has been run)
- `docs/ai/ralph/status.json` must exist

## Implementation

When this skill is invoked:

1. **Set the stop flag**: Create a stop signal file that `/ralph:start` will check
   ```bash
   touch docs/ai/ralph/.ralph_stop
   ```

2. **Inform about current iteration**: If an iteration is running, let it finish naturally. The stop signal will be checked before spawning the next iteration.

3. **Update status.json**: Set the status to 'stopped' and record the timestamp
   ```bash
   # Read current status.json
   jq '.status = "stopped" | .last_updated = now | .stopped_at = now' docs/ai/ralph/status.json > /tmp/status.json.tmp
   mv /tmp/status.json.tmp docs/ai/ralph/status.json
   ```

4. **Report to user**:
   ```
   Ralph has been signaled to stop.

   - The current iteration (if running) will finish naturally
   - No new iterations will be spawned
   - Status updated to 'stopped'

   You can resume later with /ralph:start or run a single iteration with /ralph:iterate.
   ```

## Coordination with /ralph:start

The `/ralph:start` skill checks for the existence of `docs/ai/ralph/.ralph_stop` before each iteration. When this file exists:
- The loop stops spawning new ralph-worker subagents
- The current iteration completes normally
- The loop exits gracefully

## Notes

- This is a graceful stop, not an emergency abort
- Any work in progress will complete
- To resume, use `/ralph:start` again (it will clear the stop flag)
- The stop flag is just a signal file, not a lock file
