# Codex Agent Skill - Quick Reference

## Installation

```bash
# Scripts are already executable
# Just ensure codex CLI is installed:
npm install -g @openai/codex
```

## Three Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `codex-do` | Start new work | `codex-do --project myapp "Build login"` |
| `codex-resume` | Continue session | `codex-resume --project myapp "Add validation"` |
| `codex-status` | Check/view output | `codex-status --project myapp --output` |

## Command Paths

```bash
# Add to PATH or use full path:
~/clawd-team/agents/bender/skills/codex-agent/scripts/codex-do
~/clawd-team/agents/bender/skills/codex-agent/scripts/codex-resume
~/clawd-team/agents/bender/skills/codex-agent/scripts/codex-status
```

## Project Location

**Code lives in ONE place:** `/home/dan/zoidcode/<folder>/`

All agents work on the same shared codebase.

**To find the folder name:**
```bash
~/clawd-team/clawd project list
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
~/clawd-team/agents/{you}/memory/codex-sessions.json
```

## Output Location

Each project gets:
```
/home/dan/zoidcode/<folder>/.zoid/
├── output.txt          # Full Codex output
├── session.json        # Current session info
└── iteration_*.json    # Resume history
```

## Safety Flags

- **Default**: Sandbox mode, asks for approval
- `--full-auto`: Auto-approve within workspace (safer)
- `--yolo`: No sandbox, auto-approve all (dangerous!)

## Typical Session

```bash
# 1. Start work
codex-do --project mission_control_ui "Build a todo list component"

# 2. Monitor (in another terminal)
codex-status --project mission_control_ui --follow

# 3. Review output
codex-status --project mission_control_ui --output

# 4. If needed, continue
codex-resume --project mission_control_ui "Add delete functionality"

# 5. Commit changes
cd /home/dan/zoidcode/mission_control_ui
git add .
git commit -m "feat: add todo list with delete"
```

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "codex not found" | `npm install -g @openai/codex` |
| Session hangs | `codex-status --project X --kill` |
| Lost session ID | Check `codex-sessions.json` or start fresh |
| Want to start over | `rm -rf /home/dan/zoidcode/<project>` |

## Documentation

- Full guide: `SKILL.md`
- Workflows: `references/WORKFLOW.md`
- Codex CLI: `codex --help`
