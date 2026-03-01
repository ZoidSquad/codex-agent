# Codex Agent Skill v2.0 - Quick Reference

## Prerequisites

```bash
# Ensure dependencies are installed
sudo apt-get install jq        # Required for JSON processing
npm install -g @openai/codex   # Required for coding agent
# clawd should be globally available in PATH
```

## Three Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `codex-do` | Start new work | `codex-do --project myapp "Build login"` |
| `codex-resume` | Continue session | `codex-resume --project myapp "Add validation"` |
| `codex-status` | Check/view output | `codex-status --project myapp --output` |

## Installation

Scripts should be in your agent's skills directory:
```bash
~/.openclaw/workspace/skills/codex-agent/scripts/
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

### codex-do
```bash
codex-do --project myapp [flags] "prompt"

Flags:
  --timeout 0      No timeout (for long tasks)
  --dry-run        Show what would execute
  --no-validate    Skip Mission Control validation
  --suggest-cmd    Output OpenClaw exec command for PTY mode
```

Execution mode:
  codex-do always runs with -a never exec --sandbox workspace-write

### codex-resume
```bash
codex-resume --project myapp [flags] "additional instructions"

Flags:
  (no mode flags)
```

Execution mode:
  codex-resume always runs with -a never exec --sandbox workspace-write resume

### codex-status
```bash
codex-status --project myapp [flags]

Flags:
  --output, -o     Show output
  --follow, -f     Follow live output (tail -f)
  --history, -H    Show iteration history
  --kill, -k       Kill running session
  --limit 100      Limit output lines (default: 50)
```

## Typical Session

```bash
# 1. Start work
codex-do --project mission_control_ui "Build a todo list component"

# 2. Check status (shows output files, history)
codex-status --project mission_control_ui

# 3. View iteration history
codex-status --project mission_control_ui --history

# 4. If needed, continue
codex-resume --project mission_control_ui "Add delete functionality"

# 5. Review previous output
cat /home/dan/zoidcode/mission_control_ui/.zoid/output-1.txt

# 6. Commit changes
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
