# PRD: Anti-Rationalization Rules for Ralph Worker

## Introduction

Add anti-rationalization enforcement rules to the Ralph worker's iteration instructions (`templates/PROMPT.md.template`). Currently, the worker has a soft "one task at a time" principle but no hard guardrails preventing it from rationalizing its way into doing multiple tasks in a single iteration. The `ralph-command.md` has battle-tested anti-rationalization rules that catch common rationalization patterns and force the worker to stop. This feature ports those rules into the skills-based system, adapted to its terminology (tasks instead of stories, `RALPH_BLOCKED` instead of `<promise>` tags).

## Goals

- Prevent the ralph-worker from working on more than one task per iteration
- Provide explicit rationalization patterns the worker can self-check against
- Include a clear exception for incidental lint/typecheck fixes in already-modified files
- Use language consistent with the skills system (tasks, `fix_plan.md`, `RALPH_BLOCKED`)

## User Stories

### US-001: Add anti-rationalization rules section to PROMPT.md template
**Description:** As a user running Ralph, I want the worker to have explicit anti-rationalization rules so that it never works on more than one task per iteration, even when it can "see" that the next task is simple or related.

**Acceptance Criteria:**
- [ ] `templates/PROMPT.md.template` contains a new `## Anti-Rationalization Rules (CRITICAL)` section
- [ ] The section includes a table of common rationalization patterns and their corresponding "reality" responses, adapted from `ralph-command.md` lines 240-258
- [ ] All table entries use "task" terminology (not "story") and reference `fix_plan.md` (not `prd.json`)
- [ ] The section includes a "What IS allowed" exception: fixing lint/typecheck errors in files already modified for the current task, as long as those fixes do NOT implement acceptance criteria from another task
- [ ] The section includes a "What is NOT allowed" statement: modifying files or implementing changes that belong to a different task
- [ ] The section instructs the worker to write `RALPH_BLOCKED: RATIONALIZATION_DETECTED - [explanation]` in `fix_plan.md` and return immediately if it catches itself rationalizing
- [ ] The section is placed after the "Important Reminders" section and before the closing `---` line
- [ ] Typecheck/lint passes (no code changes, but verify the template is valid markdown)

### US-002: Reinforce one-task-per-iteration in existing sections
**Description:** As a user running Ralph, I want the existing "Important Reminders" and Step 9 sections to cross-reference the anti-rationalization rules so the constraint is reinforced at multiple points.

**Acceptance Criteria:**
- [ ] The "Important Reminders" section in `templates/PROMPT.md.template` includes a new bullet: "DO NOT start another task in the same iteration. See Anti-Rationalization Rules below."
- [ ] Step 9 ("Signal Completion or Continue") adds a reminder: "End the iteration. Do NOT pick another task. See Anti-Rationalization Rules."
- [ ] The existing bullet "Stay focused - one task per iteration, future Ralph will handle the rest" is preserved (not duplicated or removed)
- [ ] Typecheck/lint passes

## Functional Requirements

- FR-1: Add a new markdown section `## Anti-Rationalization Rules (CRITICAL)` to `templates/PROMPT.md.template`
- FR-2: The section must contain a markdown table with at least 6 rationalization patterns and their "Reality" counterresponses
- FR-3: Rationalization patterns must be adapted to use skills-system terminology: "task" (not "story"), `fix_plan.md` (not `prd.json`), `RALPH_BLOCKED` signal (not `<promise>` tags)
- FR-4: Include a "What IS allowed" paragraph permitting lint/typecheck fixes in files already modified for the current task, provided those fixes do not implement another task's acceptance criteria
- FR-5: Include a "What is NOT allowed" paragraph explicitly forbidding work on other tasks
- FR-6: Include a self-detection instruction: if the worker catches itself rationalizing, it must write `RALPH_BLOCKED: RATIONALIZATION_DETECTED` in `fix_plan.md` and return immediately
- FR-7: Add cross-references to the anti-rationalization section from Step 9 and Important Reminders

## Non-Goals

- No changes to `agents/ralph-worker.md` (rules go in the template only, so users can customize)
- No changes to the `ralph-start` or `ralph-iterate` skills (they already handle `RALPH_BLOCKED`)
- No runtime enforcement mechanism (this is prompt-level guidance, not code)
- No changes to `fix_plan.md.template` or `AGENTS.md.template`

## Technical Considerations

- The only file being modified is `templates/PROMPT.md.template`
- The anti-rationalization rules must be adapted from `ralph-command.md` lines 240-258, not copied verbatim
- The `RALPH_BLOCKED: RATIONALIZATION_DETECTED` signal reuses the existing `RALPH_BLOCKED` mechanism that `ralph-start` already checks for

## Success Metrics

- The ralph-worker stops after exactly one task per iteration, even when subsequent tasks appear trivially related
- The `RALPH_BLOCKED: RATIONALIZATION_DETECTED` signal is written when the worker detects it is about to violate the one-task rule
- No change in behavior for normal single-task iterations

## Open Questions

None — scope is clear.
