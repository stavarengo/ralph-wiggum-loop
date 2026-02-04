# Ralph - Autonomous Software Development Plugin

Ralph is a Claude Code plugin that implements Geoffrey Huntley's ["Ralph Wiggum as a software engineer"](https://ghuntley.com/ralph/) technique for autonomous software development using fresh context per iteration.

## What is Ralph?

Ralph is inspired by Geoffrey Huntley's insight that you can achieve autonomous software development by giving an AI agent:

1. **Fresh context every iteration** - No memory between work cycles
2. **One task at a time** - Single focus prevents scope creep
3. **Simple, repeatable process** - 9 steps executed consistently
4. **Trust in eventual consistency** - Multiple simple iterations beat one complex attempt

The name "Ralph Wiggum" (from The Simpsons) represents this philosophy: simple, focused, consistent execution without overthinking. Ralph doesn't need to remember the big picture - he just needs clear instructions for the current task.

## Phase 1 vs Phase 2

**Phase 1 (This Implementation): Interactive Mode**

This is a Claude Code plugin using native agent features:
- Skills orchestrate the iteration loop from the main session
- Ralph-worker agent executes tasks with Sonnet 4.5
- Ralph-oracle agent provides deep reasoning with Opus 4.5 + extended thinking
- User interaction at 10-iteration checkpoints
- Full integration with Claude Code's tool ecosystem

**Phase 2 (Future): Full Autonomy**

Future standalone CLI tool with:
- Background loops running independently
- Cross-session state management
- No human intervention required
- Persistent daemon mode

Phase 1 provides immediate value while validating the core technique. Phase 2 will scale it to true autonomy.

## Installation

```bash
# Add the marketplace
/plugin marketplace add stavarengo/ralph-wiggum-loop

# Install Ralph
/plugin install ralph@ralph-marketplace

# Initialize in your project
/ralph:init
```

## Quick Start

```bash
# 1. Initialize Ralph in your project
/ralph:init

# 2. Customize the templates (optional)
# Edit docs/ai/ralph/PROMPT.md - Ralph's iteration instructions
# Edit docs/ai/ralph/AGENTS.md - Your build/test commands
# Edit docs/ai/ralph/fix_plan.md - Your task list

# 3. Start autonomous development
/ralph:start

# Ralph will now:
# - Pick tasks from fix_plan.md (highest priority first)
# - Implement, test, document, commit each task
# - Continue until complete or you stop it

# 4. Stop when needed
/ralph:stop
```

## Skills Reference

Ralph provides 5 skills for different stages of autonomous development:

### `/ralph:init` - Initialize Ralph

Sets up Ralph in your project by creating the directory structure and copying templates.

**What it does:**
- Creates `docs/ai/ralph/` directory
- Copies 3 templates: `PROMPT.md`, `fix_plan.md`, `AGENTS.md`
- Creates `CLAUDE.md` symlink for auto-loading context
- Initializes `status.json` for iteration tracking
- Updates `.gitignore` to exclude Ralph's working directory

**After initialization, your project will have:**
```
your-project/
├── docs/ai/ralph/
│   ├── PROMPT.md      # Ralph's iteration instructions
│   ├── fix_plan.md    # Task list and priorities
│   ├── AGENTS.md      # Build/test instructions + learnings
│   └── status.json    # Iteration tracking
├── CLAUDE.md          # Symlink → docs/ai/ralph/AGENTS.md
└── .gitignore         # Updated to exclude docs/ai/ralph/
```

**Next steps:**
1. Edit `fix_plan.md` to add your tasks (in priority order)
2. Edit `AGENTS.md` to add your build/test commands
3. Run `/ralph:start` to begin autonomous development

### `/ralph:start` - Start Autonomous Loop

Begins the autonomous iteration loop that runs continuously until all tasks are complete.

**What it does:**
- Validates prerequisites (PROMPT.md, fix_plan.md, status.json exist)
- Clears any existing stop signal
- Updates status to 'running'
- Enters main loop:
  - Spawns ralph-worker for one iteration
  - Checks for completion signals (RALPH_COMPLETE, RALPH_BLOCKED)
  - Checks for stop signal (.ralph_stop)
  - Prompts user every 10 iterations for confirmation
  - Continues until complete/blocked/stopped

**Example output:**
```
Ralph started. Beginning autonomous iteration loop...

=========================================
Starting iteration 1
=========================================
Iteration 1: Implemented user authentication
- Files changed: src/auth.ts, tests/auth.test.ts
- Tests: PASS (all 12 tests passing)
- Next: Add password reset flow

=========================================
Starting iteration 2
=========================================
Iteration 2: Implemented password reset flow
- Files changed: src/reset.ts, tests/reset.test.ts
- Tests: PASS (all 15 tests passing)
- Next: RALPH_COMPLETE

RALPH_COMPLETE signal detected. All tasks complete!

Ralph completed successfully!
- Total iterations: 2
- Status: complete
- All tasks in fix_plan.md have been completed.
```

**Stop conditions:**
- RALPH_COMPLETE signal in fix_plan.md (all tasks done)
- RALPH_BLOCKED signal in fix_plan.md (cannot proceed)
- `/ralph:stop` called by user
- User declines continuation at 10-iteration checkpoint

### `/ralph:iterate` - Single Iteration

Executes exactly ONE iteration manually for debugging or careful stepping.

**What it does:**
- Validates prerequisites
- Spawns one ralph-worker
- Displays result
- Exits (no loop)

**Use this when:**
- Debugging Ralph's behavior
- Testing changes to PROMPT.md or fix_plan.md
- You want fine-grained control over iterations
- Working through complex tasks with manual oversight

**Example usage:**
```
User: /ralph:iterate

Iteration 3 complete.

Summary from ralph-worker:
- Implemented database migration system
- Files changed: src/migrations/, tests/migrations.test.ts
- Tests: PASS (all 18 tests passing)
- Next: Add user roles and permissions

To continue:
- Run /ralph:iterate again for the next iteration
- Run /ralph:start to begin autonomous looping
```

**Difference from `/ralph:start`:**

| Feature | /ralph:iterate | /ralph:start |
|---------|---------------|--------------|
| Iterations | ONE | Continuous loop |
| Control | Manual | Autonomous |
| Use case | Debugging, stepping | Production development |
| Stops after | 1 iteration | All tasks complete or /ralph:stop |

### `/ralph:stop` - Stop Gracefully

Stops the autonomous loop gracefully, allowing the current iteration to finish.

**What it does:**
- Creates `.ralph_stop` signal file
- Updates status.json to 'stopped'
- Informs user that stop is pending

**Behavior:**
- Current iteration completes normally
- No new iterations are spawned
- Status updated to 'stopped'
- Can resume later with `/ralph:start` or single-step with `/ralph:iterate`

**Example:**
```
User: /ralph:stop

Ralph has been signaled to stop.

- The current iteration (if running) will finish naturally
- No new iterations will be spawned
- Status updated to 'stopped'

You can resume later with /ralph:start or run a single iteration with /ralph:iterate.
```

### `/ralph:reset` - Hard Reset Repository

Hard resets the repository to the last commit, discarding all uncommitted changes.

**WARNING: This is destructive!**
- Discards all uncommitted changes in tracked files
- Removes all untracked files and directories
- Resets to HEAD (last commit)
- Cannot be undone

**What it does:**
- Prompts for confirmation with AskUserQuestion
- Executes `git reset --hard HEAD && git clean -fd`
- Updates status.json to 'reset'
- Reports results to user

**Use this when:**
- Ralph is stuck in an iteration loop
- The codebase is in an inconsistent state
- You want to return to the last committed checkpoint
- Tests are failing due to leftover artifacts

**Example:**
```
User: /ralph:reset

⚠️  WARNING: This will discard ALL uncommitted changes and untracked files.
This cannot be undone. Are you sure you want to proceed? (yes/no)

User: yes

✅ Repository reset complete
- All uncommitted changes discarded
- All untracked files removed
- Repository at: abc1234
- Status updated to: reset
```

## Workflow: Init → Customize → Start

The recommended workflow for using Ralph:

### 1. Initialize
```
/ralph:init
```

### 2. Customize Templates

**Edit `docs/ai/ralph/fix_plan.md`:**
```markdown
## Tasks

- [ ] Add user authentication
- [ ] Add password reset flow
- [ ] Implement user roles
- [ ] Add audit logging
```

**Edit `docs/ai/ralph/AGENTS.md`:**
```markdown
## How to Build

```bash
npm install
npm run build
```

## How to Test

```bash
npm test
```
```

**Optionally edit `docs/ai/ralph/PROMPT.md`:**
Add project-specific instructions, adjust workflow, customize behavior.

### 3. Start Autonomous Development
```
/ralph:start
```

Ralph will now work through your task list autonomously, committing progress after each task.

### 4. Monitor and Control

**Check progress:**
- Each iteration summary is displayed in chat
- View `docs/ai/ralph/fix_plan.md` to see completed tasks
- View `docs/ai/ralph/AGENTS.md` to see learnings accumulated

**Intervene if needed:**
- `/ralph:stop` - Stop gracefully
- `/ralph:iterate` - Step through one iteration manually
- `/ralph:reset` - Hard reset if stuck

**Resume:**
- `/ralph:start` - Resume autonomous loop

## Agents

Ralph uses two agents with different roles:

### ralph-worker (Sonnet 4.5)

**Role:** Executes one task per iteration with fresh context

**Configuration:**
- Model: Claude Sonnet 4.5
- Tools: Bash, Read, Edit, Write, Grep, Glob, Task
- bypassPermissions: true (autonomous operation)

**Behavior:**
- Starts each iteration with zero memory
- Reads context files: PROMPT.md, fix_plan.md, AGENTS.md, status.json
- Follows 9-step process: choose task → search → implement → test → document → update tracking → commit → tag → signal completion
- Returns concise summary to main session

**9-Step Process:**
1. Choose highest priority task from fix_plan.md
2. Search codebase with Glob/Grep to understand patterns
3. Implement solution (no placeholders, complete code only)
4. Test using commands from AGENTS.md
5. Document learnings in AGENTS.md
6. Update fix_plan.md (move task to Completed)
7. Commit with conventional commit message
8. Tag clean checkpoints (incremental versioning: 0.0.1, 0.0.2, ...)
9. Signal completion (RALPH_COMPLETE) or continue to next task

### ralph-oracle (Opus 4.5 + Extended Thinking)

**Role:** Provides deep reasoning for complex decisions

**Configuration:**
- Model: Claude Opus 4.5
- Tools: Read, Grep, Glob (read-only)
- extendedThinking: true (ultrathink mode)

**Behavior:**
- Spawned by ralph-worker only when needed
- Read-only access (cannot modify files)
- Uses extended thinking for deep analysis
- Returns structured recommendations

**When consulted:**
- Architecture decisions (which pattern to use)
- Priority conflicts (which task first)
- Debugging complex failures (root cause analysis)
- Validation before major refactoring

**Recommendation format:**
```
## Analysis
[What was analyzed]

## Recommendation
[Clear, specific recommendation]

## Rationale
- Why this approach
- Trade-offs
- Risks to watch for

## Implementation Guidance
1. First step
2. Second step
...

## Alternatives Considered
[Other options and why not recommended]
```

**Decision-making principles:**
1. Simplicity over complexity
2. Consistency with existing patterns
3. Completeness (no placeholders)
4. Eventual consistency (multiple simple iterations)
5. Fail forward (easy recovery)

## Safety Mechanisms

Ralph is designed with multiple safety layers:

### 1. Fresh Context Per Iteration
- Each iteration starts with zero memory
- Prevents context drift and hallucination accumulation
- Forces explicit knowledge capture in files
- Ensures reproducibility

### 2. bypassPermissions Mode
- ralph-worker runs with autonomous permissions
- No prompts for file operations or git commands
- Enables true autonomous operation
- Controlled environment (can be stopped with /ralph:stop)

### 3. User Checkpoints
- Prompts user every 10 iterations for continuation
- Prevents runaway loops
- Gives user regular control points
- Can decline to stop loop

### 4. Graceful Stop Signal
- `/ralph:stop` uses signal file (.ralph_stop)
- Current iteration finishes normally
- Clean shutdown, no abrupt termination
- Can resume with `/ralph:start`

### 5. One Task Per Iteration
- Single focus prevents scope creep
- Clear boundaries for each iteration
- Easy to review changes (one task = one commit)
- Reduces complexity

### 6. Automatic Commits
- Every completed task gets committed
- Provides rollback points
- Git history shows clear progress
- Can reset with `/ralph:reset` if needed

### 7. Oracle Consultation
- Complex decisions get Opus 4.5 + extended thinking
- Prevents worker from guessing on critical choices
- Read-only oracle cannot make changes
- Recommendations are explicit and traceable

### 8. Completion Signals
- RALPH_COMPLETE when all tasks done
- RALPH_BLOCKED when cannot proceed
- Clear exit conditions prevent infinite loops
- User knows when intervention needed

## Templates

Ralph uses 3 templates copied to your project during initialization:

### PROMPT.md - Iteration Instructions

Contains the 9-step process and core principles. Ralph reads this at the start of each iteration.

**Key sections:**
- Core Principles (fresh context, one task, eventual consistency)
- Context Loading (which files to read)
- Oracle Consultation (when and how to spawn oracle)
- 9-Step Iteration Process (detailed workflow)
- Return Format (how to summarize iteration)
- Failure Handling (what to do when stuck)

**Customization:**
Add project-specific instructions, adjust priorities, modify workflow.

### fix_plan.md - Task Tracking

The task list that drives Ralph's work. Tasks ordered by priority (top = highest).

**Sections:**
- **Tasks:** Pending work in priority order (markdown checkboxes)
- **Completed:** Finished tasks with timestamps
- **Blocked/Issues:** Tasks that cannot proceed with blocker descriptions

**Signals:**
- `RALPH_COMPLETE` at top = all tasks done
- `RALPH_BLOCKED: [reason]` = cannot proceed, needs user intervention

**Usage:**
Ralph moves tasks between sections after each iteration. You can edit this file anytime to add/remove/reprioritize tasks.

### AGENTS.md - Build/Test Instructions

Contains build commands, test commands, and accumulated learnings.

**Sections:**
- **How to Build:** Commands to build the project (npm install, make build, etc.)
- **How to Test:** Commands to run tests (npm test, pytest, etc.)
- **Learnings:** Discoveries from iterations (patterns, gotchas, useful context)

**Special note:**
This file is symlinked as `CLAUDE.md` in your project root, so Claude Code automatically reads it when starting any conversation in the project.

**Updates:**
Ralph appends learnings after each iteration. Build/test commands can be updated as needed.

## Troubleshooting

### Ralph won't start

**Error: "PROMPT.md not found"**
- Run `/ralph:init` first to initialize the project

**Error: "fix_plan.md is empty"**
- Add at least one task to the Tasks section in `docs/ai/ralph/fix_plan.md`

### Ralph is stuck or looping

**Same error repeated:**
1. Run `/ralph:stop` to stop the loop
2. Check `docs/ai/ralph/AGENTS.md` learnings for clues
3. Fix the issue manually or update fix_plan.md to skip the problematic task
4. Run `/ralph:reset` if the codebase is in a bad state
5. Resume with `/ralph:start` or single-step with `/ralph:iterate`

**Not making progress:**
- Check fix_plan.md for RALPH_BLOCKED signal
- Review the blocker description
- Resolve the blocker or adjust task priorities
- Resume with `/ralph:start`

### Tests keep failing

**Build/test commands wrong:**
1. Edit `docs/ai/ralph/AGENTS.md`
2. Update the "How to Build" and "How to Test" sections with correct commands
3. Ralph will read the updated commands on next iteration

**Environment issues:**
- Ralph runs in the same environment as Claude Code
- Check that dependencies are installed
- Verify git config (user.name, user.email) is set
- Ensure database/services are running if needed

### Commits have wrong author

**Git config not set:**
```bash
git config user.name "Your Name"
git config user.email "your.email@example.com"
```

Ralph uses your git config for commits. The co-author line is added automatically:
```
Co-Authored-By: Ralph Wiggum <ralph@claude-code>
```

### Want to change iteration behavior

**Customize PROMPT.md:**
1. Edit `docs/ai/ralph/PROMPT.md`
2. Modify the 9-step process, add project-specific rules, adjust priorities
3. Ralph will read the updated instructions on next iteration

**Examples:**
- Add "always run linter before commit" to Step 7
- Change "spawn oracle for architecture decisions" threshold
- Add "update documentation" as additional step

### Oracle not being consulted

**Ralph is being too conservative:**
- Edit `docs/ai/ralph/PROMPT.md`
- Update the "Oracle Consultation" section to be more explicit about when to consult
- Example: "Always consult oracle before choosing between multiple architectural patterns"

**Oracle consultation is working correctly:**
- Ralph only spawns oracle for genuinely complex decisions
- Routine implementation work doesn't need oracle (keeps iterations fast)
- If you want oracle consultation, make the decision criteria explicit in PROMPT.md

### Performance is slow

**Large codebase:**
- Ralph uses Grep/Glob which are optimized for large codebases
- Iterations may take longer for complex tasks
- This is expected - quality over speed

**Network latency:**
- Ralph makes API calls for each agent spawn
- Network issues can slow iterations
- No workaround - inherent to the architecture

**Spawning too many subagents:**
- Check PROMPT.md for parallel search instructions
- Default is "up to 100 subagents for large searches"
- Reduce this if needed (though typically not a problem)

## Common Questions

### Can I use Ralph on existing projects?

Yes. Run `/ralph:init` in any project directory. Ralph will create `docs/ai/ralph/` and you can start immediately.

### Does Ralph work with any programming language?

Yes. Ralph is language-agnostic. Just update `AGENTS.md` with your build/test commands.

### Can I interrupt Ralph mid-iteration?

Use `/ralph:stop`. The current iteration will finish, then the loop stops. You can also manually stop the Claude Code session, but this is less clean.

### Will Ralph commit sensitive files?

No, if you have a proper `.gitignore`. Ralph respects git's ignore rules. Also, Ralph's working directory (`docs/ai/ralph/`) is excluded from commits.

### Can I review commits before they're made?

Not in Phase 1 (interactive mode). Ralph commits automatically after each task. Review the commits in git history afterward. If you want review before commit, use `/ralph:iterate` for manual stepping.

### How do I resume after stopping?

Just run `/ralph:start` again. Ralph reads the current state from status.json and fix_plan.md, then continues where it left off.

### Can I use Ralph with multiple projects simultaneously?

Yes, each project has its own `docs/ai/ralph/` directory with independent state.

### What if I want to change the task list mid-run?

Edit `docs/ai/ralph/fix_plan.md` directly. Ralph reads it fresh at the start of each iteration, so changes take effect immediately.

## References

- **Original Article:** [Ralph Wiggum as a Software Engineer](https://ghuntley.com/ralph/) by Geoffrey Huntley
- **Design Document:** [docs/ai/ralph/backlog/2026-02-03.ralph-plugin-interactive-mode/2026-02-03.plan.brainstorm-design.ralph-plugin.md](/home/rfst/.claude/docs/ai/ralph/backlog/2026-02-03.ralph-plugin-interactive-mode/2026-02-03.plan.brainstorm-design.ralph-plugin.md)
- **PRD:** [docs/ai/ralph/backlog/2026-02-03.ralph-plugin-interactive-mode/prd.2026-02-03.ralph-plugin-interactive-mode.md](/home/rfst/.claude/docs/ai/ralph/backlog/2026-02-03.ralph-plugin-interactive-mode/prd.2026-02-03.ralph-plugin-interactive-mode.md)

## Credits

Ralph is an implementation of Geoffrey Huntley's autonomous development technique, built as a Claude Code plugin.

**Implementation:**
- Plugin: Ralph for Claude Code (Phase 1)
- Authors: Built using Claude Sonnet 4.5
- Inspiration: Geoffrey Huntley's Ralph Wiggum technique

**What's Next:**
- Phase 2: Standalone CLI with full autonomy
- Background loops running independently
- Cross-session state management
- True "set and forget" autonomous development

## License

See LICENSE file in the plugin directory.
