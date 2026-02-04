# ralph-iterate

Run a single iteration manually for debugging or careful stepping.

## What This Skill Does

This skill executes exactly ONE iteration of Ralph's work cycle. It spawns a single ralph-worker subagent, waits for it to complete, displays the result, and then stops. This is useful for:

- Debugging Ralph's behavior step-by-step
- Carefully controlling when iterations happen
- Testing changes to PROMPT.md or fix_plan.md
- Working through complex tasks with manual oversight

## Prerequisites

Before running `/ralph:iterate`, you must have:

1. **Initialized Ralph**: Run `/ralph:init` first
2. **PROMPT.md exists**: File must exist at `docs/ai/ralph/PROMPT.md`
3. **fix_plan.md exists**: File must exist at `docs/ai/ralph/fix_plan.md`
4. **status.json exists**: File must exist at `docs/ai/ralph/status.json`

## Implementation

When this skill is invoked:

### 1. Validate Prerequisites

Check that all required files exist:

```bash
# Check that Ralph has been initialized
if [[ ! -f docs/ai/ralph/PROMPT.md ]]; then
  echo "Error: docs/ai/ralph/PROMPT.md not found. Run /ralph:init first."
  exit 1
fi

if [[ ! -f docs/ai/ralph/fix_plan.md ]]; then
  echo "Error: docs/ai/ralph/fix_plan.md not found. Run /ralph:init first."
  exit 1
fi

if [[ ! -f docs/ai/ralph/status.json ]]; then
  echo "Error: docs/ai/ralph/status.json not found. Run /ralph:init first."
  exit 1
fi
```

### 2. Get Current Iteration Count

Read the current iteration count from status.json:

```bash
CURRENT_ITERATION=$(jq -r '.iteration_count' docs/ai/ralph/status.json)
NEXT_ITERATION=$((CURRENT_ITERATION + 1))
```

### 3. Spawn ONE Ralph Worker

Use the Task tool to spawn exactly one ralph-worker subagent:

```
Task tool parameters:
- subagent_type: "ralph-worker"
- instructions: "Execute iteration ${NEXT_ITERATION}. Read docs/ai/ralph/PROMPT.md, fix_plan.md, AGENTS.md, and status.json. Follow the 9-step process. Return a concise summary when done."
```

The ralph-worker will:
- Read all context files (PROMPT.md, fix_plan.md, AGENTS.md, status.json)
- Pick the highest priority task from fix_plan.md
- Implement, test, document, commit, and update tracking
- Return a concise summary

### 4. Update Iteration Count

After the worker completes, increment the iteration count in status.json:

```bash
# Atomic update using jq
jq --arg timestamp "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
   '.iteration_count = (.iteration_count + 1) | .last_updated = $timestamp | .status = "running"' \
   docs/ai/ralph/status.json > /tmp/status.json.tmp && \
   mv /tmp/status.json.tmp docs/ai/ralph/status.json
```

### 5. Display Result

Show the worker's summary to the user in the chat. Include:
- What iteration just completed
- Summary of work done
- Files changed
- Test status
- What's next (or if complete/blocked)

### 6. Exit (No Loop)

**IMPORTANT**: This skill does NOT loop. It executes exactly one iteration and then stops. To run another iteration, the user must:
- Run `/ralph:iterate` again (manual stepping)
- Run `/ralph:start` to begin autonomous looping

## Example Output

After running `/ralph:iterate`, you might see:

```
Iteration 3 complete.

Summary from ralph-worker:
- Implemented user authentication endpoint
- Files changed: src/auth.rs, tests/auth_test.rs
- Tests: PASS (all 15 tests passing)
- Next: Implement password reset flow

To continue:
- Run /ralph:iterate again for the next iteration
- Run /ralph:start to begin autonomous looping
- Run /ralph:stop if you're done
```

## Difference from /ralph:start

| Feature | /ralph:iterate | /ralph:start |
|---------|---------------|--------------|
| Iterations | ONE | Continuous loop until complete/stopped |
| Control | Manual | Autonomous |
| Use case | Debugging, careful stepping | Production autonomous development |
| Stops after | 1 iteration | All tasks complete or /ralph:stop |

## Notes

- Each iteration is fully independent with fresh context
- The worker reads all context files at the start of each iteration
- status.json tracks the iteration count and status
- Use this skill when you want fine-grained control over Ralph's execution
