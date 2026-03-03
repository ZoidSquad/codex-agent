# Codex Agent Skill v2.0 - Quick Reference

## Prerequisites

```bash
# Ensure dependencies are installed
sudo apt-get install jq        # Required for JSON processing
npm install -g @openai/codex   # Required for coding agent
# clawd should be globally available in PATH
```

## Core Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `codex-branch` | Prepare/switch to the task branch | `codex-branch --project myapp --task-id k176...` |
| `codex-do` | Start tracked work | `codex-do --project myapp --task-id k176... --background "Build login"` |
| `codex-resume` | Continue session | `codex-resume --project myapp --task-id k176... --background "Add validation"` |
| `codex-status` | Inspect current state | `codex-status --project myapp --json` |

## Installation

Scripts should be in your agent's skills directory:
```bash
~/.openclaw/skills/codex-agent/scripts/
```

Or reference them directly from this repo when testing:
```bash
~/zoidcode/codex-agent/scripts/codex-do
```

## Project Location

**Code lives in ONE place:** `/home/dan/zoidcode/<folder>/`

All agents work on the same shared codebase.

**To find the folder name:**
```bash
clawd project list
```
Look for the line that says: `FOLDER: mission_control_ui  <-- USE THIS WITH COMMANDS`

**Always use just the folder name (not the full path):**
```bash
# ✅ CORRECT:
codex-do --project mission_control_ui "Build login"

# ✅ ALSO WORKS (script extracts basename):
codex-do --project /home/dan/zoidcode/mission_control_ui "Build login"

# ❌ WRONG - don't use display name:
codex-do --project "Mission Control UI" "Build login"
```

## Session Registry

Each agent tracks their own sessions in:
```
~/.openclaw/workspace/memory/codex-sessions.json
~/.openclaw/workspace/memory/codex-iterations.json
```

Auto-cleanup: Last 50 projects, 100 iterations per project, 1000 total.

## Output Location

Each project gets:
```
/home/dan/zoidcode/<folder>/.zoid/
├── output.txt          # Current/most recent output
├── output-1.txt        # Previous session output
├── output-2.txt        # Two sessions ago
├── ...                 # Up to output-10.txt (auto-rotated)
├── session.json        # Current session info
└── iteration_*.json    # Resume history
```

## Common Options

### codex-branch
```bash
codex-branch --project myapp --task-id k176w64p843g1brae8zmqys6qs820knf [flags]

Flags:
  --base <branch>  Branch to branch from (default: dev, then main, then master)
  --json           Machine-readable result
```

For task-tracked work, the branch name is the full task ID.

### codex-do
```bash
codex-do --project myapp [flags] "prompt"

Flags:
  --task-id <id>   Attach a Mission Control task ID
  --background     Start Codex in a tracked detached process
  --force          Bypass duplicate-run guard for the project
  --timeout 0      No timeout (default)
  --dry-run        Show what would execute
  --no-validate    Skip Mission Control validation
  --suggest-cmd    Output OpenClaw exec command for PTY mode
```

Execution mode:
  codex-do always runs with -a never exec --sandbox workspace-write

Tracked task rule:
  If you pass --task-id, the current git branch must match that task ID.
  Use codex-branch first.

### codex-resume
```bash
codex-resume --project myapp [flags] "additional instructions"

Flags:
  --task-id <id>   Override or attach a Mission Control task ID
  --background     Resume in a tracked detached process
  --force          Bypass duplicate-run guard for the project
```

Execution mode:
  codex-resume always runs with -a never exec --sandbox workspace-write resume

Tracked task rule:
  Resume task-bound work on the same task branch you started on.

### codex-status
```bash
codex-status --project myapp [flags]

Flags:
  --json           Machine-readable status
  --output, -o     Show output
  --follow, -f     Follow live output (tail -f)
  --history, -H    Show iteration history
  --kill, -k       Kill running session
  --limit 100      Limit output lines (default: 50)
```

## Typical Session

```bash
# 1. Check whether work is already running
codex-status --project mission_control_ui --json

# 2. Prepare the task branch
codex-branch --project mission_control_ui --task-id <task-id>

# 3. Start tracked background work
codex-do --project mission_control_ui --task-id <task-id> --background "Build a todo list component"

# 4. Inspect current state
codex-status --project mission_control_ui --json

# 5. If needed, continue later on the same task branch
codex-resume --project mission_control_ui --task-id <task-id> --background "Add delete functionality"

# 6. Review previous output
cat /home/dan/zoidcode/mission_control_ui/.zoid/output-1.txt

# 7. Commit changes on the task branch
cd /home/dan/zoidcode/mission_control_ui
git add .
git commit -m "feat: add todo list with delete"
```

## Environment Variables

```bash
# Override detected agent name
export AGENT_NAME="MyAgent"

# Override workspace directory
export OPENCLAW_WORKSPACE="/path/to/workspace"
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "jq is required" | `sudo apt-get install jq` |
| "codex not found" | `npm install -g @openai/codex` |
| "clawd not found" | Ensure `clawd` is in PATH |
| Session hangs | `codex-status --project X --kill` |
| Lost session ID | Check `codex-sessions.json` or start fresh |
| Need orchestration logic | Use `codex-status --project X --json` first |
| Want to start over | `rm -rf /home/dan/zoidcode/<project>` |
| File too big | Old outputs auto-deleted, check output-1.txt etc |

## What's New in v2.0

- ✅ Output file rotation (preserves history)
- ✅ Iteration history tracking
- ✅ File locking (prevents race conditions)
- ✅ Dynamic agent detection
- ✅ Mission Control validation
- ✅ Fixed exit code reporting
- ✅ Fixed CLI path (uses `clawd`)
- ✅ Configurable timeout (including no timeout)
- ✅ Dependency validation

## Documentation

- Full guide: `SKILL.md`
- Workflows: `references/WORKFLOW.md`
- Codex CLI: `codex --help`
