# ralph-init

Initialize Ralph in the current project.

## What This Does

This skill sets up Ralph's autonomous development environment in your project by creating the necessary directory structure, copying templates, and configuring tracking files.

## Initialization Steps

When you run `/ralph:init`, Claude will:

### 1. Create Directory Structure

```bash
mkdir -p docs/ai/ralph
```

Creates the Ralph working directory in your project where all Ralph-related files will live.

### 2. Copy Templates

Copy all 3 templates from the plugin to your project:

```bash
# Get the plugin path
PLUGIN_PATH="$(dirname "$(dirname "$(dirname "$0")")")"

# Copy templates and remove .template extension
cp "$PLUGIN_PATH/templates/PROMPT.md.template" docs/ai/ralph/PROMPT.md
cp "$PLUGIN_PATH/templates/fix_plan.md.template" docs/ai/ralph/fix_plan.md
cp "$PLUGIN_PATH/templates/AGENTS.md.template" docs/ai/ralph/AGENTS.md
```

**Templates copied:**
- `PROMPT.md.template` → `docs/ai/ralph/PROMPT.md` - Ralph's iteration instructions
- `fix_plan.md.template` → `docs/ai/ralph/fix_plan.md` - Task tracking and priorities
- `AGENTS.md.template` → `docs/ai/ralph/AGENTS.md` - Build/test instructions and learnings

### 3. Create CLAUDE.md Symlink

```bash
# Create symlink in project root pointing to AGENTS.md
ln -s docs/ai/ralph/AGENTS.md CLAUDE.md
```

This allows Claude Code to automatically read Ralph's learnings and build/test instructions when starting any conversation in this project.

### 4. Initialize status.json

```bash
cat > docs/ai/ralph/status.json << 'EOF'
{
  "iteration_count": 0,
  "started_at": null,
  "status": "initialized",
  "last_updated": "YYYY-MM-DDTHH:MM:SSZ"
}
EOF
```

**status.json fields:**
- `iteration_count`: Number of completed iterations (starts at 0)
- `started_at`: ISO 8601 timestamp when `/ralph:start` was run (null until started)
- `status`: Current state - `initialized`, `running`, `stopped`, `complete`, `reset`
- `last_updated`: ISO 8601 timestamp of last status change

### 5. Update .gitignore

Add Ralph's working directory to `.gitignore` if not already present:

```bash
# Check if docs/ai/ralph/ is already in .gitignore
if ! grep -q "^docs/ai/ralph/" .gitignore 2>/dev/null; then
  echo "docs/ai/ralph/" >> .gitignore
fi
```

This prevents Ralph's iteration files from being committed to version control. Only your actual code changes get committed.

## Next Steps

After initialization completes, you can:

1. **Customize your setup** (optional):
   - Edit `docs/ai/ralph/PROMPT.md` to add project-specific instructions
   - Edit `docs/ai/ralph/AGENTS.md` to add your build/test commands
   - Edit `docs/ai/ralph/fix_plan.md` to add your initial tasks

2. **Start autonomous development**:
   - Run `/ralph:start` to begin the autonomous iteration loop
   - Ralph will pick tasks from `fix_plan.md` and work through them one by one

3. **Manual iteration** (if you prefer):
   - Run `/ralph:iterate` to execute exactly one iteration
   - Use this for more control over when iterations happen

## Files Created

After running `/ralph:init`, your project will have:

```
your-project/
├── docs/ai/ralph/
│   ├── PROMPT.md      # Ralph's iteration instructions
│   ├── fix_plan.md    # Task list and priorities
│   ├── AGENTS.md      # Build/test instructions
│   └── status.json    # Iteration tracking
├── CLAUDE.md          # Symlink → docs/ai/ralph/AGENTS.md
└── .gitignore         # Updated to exclude docs/ai/ralph/
```

## Important Notes

- **CLAUDE.md is a symlink**: Don't edit it directly. Edit `docs/ai/ralph/AGENTS.md` instead.
- **Customize before starting**: Review and update the templates with your project's specific build/test commands before running `/ralph:start`.
- **Git integration**: Ralph creates commits after each task. Make sure your git config (user.name, user.email) is set.
- **Fresh context**: Ralph starts each iteration with zero memory, so all important context must be in the template files.
