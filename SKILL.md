---
name: codex-agent
description: |
  Automated Codex CLI integration for software development tasks.
  Use when you need to write, modify, or review code in a project.
  Handles session tracking, git setup, output capture, and resume automatically.
  Trigger phrases: "build a feature", "implement", "code", "develop", "create app",
  "add component", "fix bug", "refactor", "write tests".
---

# Codex Agent Skill

Automated Codex CLI wrapper with session tracking. All code lives in `/home/dan/zoidcode/`, shared across agents.

## Quick Start

```bash
# Start working on a project (use folder name from Mission Control)
codex-do --project mission_control_ui "Build a React kanban board"

# Check progress
codex-status --project mission_control_ui

# Continue working
codex-resume --project mission_control_ui "Add dark mode toggle"
```

## When to Use This Skill

✅ Building new features or components  
✅ Implementing complete user stories  
✅ Refactoring existing code  
✅ Bug fixes requiring multi-file changes  
✅ Code review (in sandbox mode)  
✅ Creating new projects from scratch  

❌ NOT for simple file edits (use `edit` tool instead)  
❌ NOT for reading/analyzing code (use `read` tool instead)  
❌ NOT for single-line changes (use `edit` tool instead)  

## Core Commands

### codex-do

Main entry point for starting work.

```bash
codex-do --project <folder_name> [options] "Your prompt here"
```

**Options:**
- `--project <name>` - Folder name from Mission Control (required)
- `--yolo` - Skip sandbox, auto-approve all changes (dangerous!)
- `--full-auto` - Auto-approve within workspace (safer than --yolo)
- `--dry-run` - Show what would execute, don't run
- `--timeout <minutes>` - Kill after N minutes (default: 30)

**What it does automatically:**
1. Checks for project in `/home/dan/zoidcode/<folder>/`
2. Creates folder if missing (with git init)
3. Creates `.zoid/` directory with session tracking
4. Loads previous session context if resuming
5. Runs Codex with proper flags
6. Captures output to `.zoid/output.txt`
7. Updates `codex-sessions.json` registry

### codex-resume

Continue a previous session.

```bash
codex-resume --project <folder_name> "Additional instructions"
```

**Auto-finds** the last session ID from registry and continues.

### codex-status

Check session status and view output.

```bash
# Check status
codex-status --project <folder_name>

# View output
codex-status --project <folder_name> --output

# Kill stuck session
codex-status --project <folder_name> --kill
```

## Workflow

### 1. Project Location

**Code lives in ONE shared location:** `/home/dan/zoidcode/<folder>/`

All agents work on the same codebase in zoidcode. Each agent tracks their own Codex sessions separately.

**How to find the folder name:**
```bash
# Run this to see all projects
~/clawd-team/clawd project list

# Output looks like:
# 📁 2 project(s):
#
# Mission Control UI
#    FOLDER: mission_control_ui  <-- USE THIS VALUE
#    Path: /home/dan/zoidcode/mission_control_ui
```

**Use the FOLDER value (not the full path) with all commands:**
```bash
# ✅ CORRECT - use just the folder name
codex-do --project mission_control_ui "Build a component"

# ✅ ALSO WORKS - script extracts basename automatically
codex-do --project /home/dan/zoidcode/mission_control_ui "Build a component"

# ❌ WRONG - don't use display name or other formats
codex-do --project "Mission Control UI" "..."  # <-- NO! Use folder name
```

### 2. Starting Work

Always provide **specific, actionable prompts**:

```bash
# ✅ Good: Specific, scoped, clear acceptance criteria
codex-do --project mission_control_ui "Create a TaskCard component with:
- Props: title, status, priority, assignee
- Visual: priority badge (low/medium/high/critical colors)
- Click handler to open detail view
- Use TypeScript and Tailwind CSS"

# ❌ Bad: Vague, open-ended
codex-do --project mission_control_ui "Build the UI"
```

### 3. Monitoring Progress

Codex runs in background. Check status:
```bash
# Quick status
codex-status --project mission_control_ui

# Watch live output (like tail -f)
codex-status --project mission_control_ui --output --follow
```

### 4. Handling Questions

If Codex asks questions mid-session:
1. Check output: `codex-status --project X --output`
2. Answer via resume: `codex-resume --project X "Answer: yes"`

### 5. Completion

When Codex finishes:
1. Review `.zoid/output.txt` for summary
2. Check git status in project folder
3. Review changes before committing
4. Commit with descriptive message

```bash
cd /home/dan/zoidcode/<folder_name>
git status
git diff
git add .
git commit -m "feat: add TaskCard component with priority badges"
```

(GitHub remote setup will come later - for now, commits are local.)

## Safety Rules

### The Safety Ladder

Start safe, escalate only when needed:

1. **First run**: No flags (sandbox mode, asks for approval)
2. **Familiar project**: `--full-auto` (auto-approves in workspace)
3. **Emergency only**: `--yolo` (no sandbox, no approvals)

### Never Do This

- ❌ `--yolo` on first run in unknown codebase
- ❌ Run Codex in `~/clawd/` or system directories
- ❌ Ignore error messages in output
- ❌ Commit without reviewing changes

### Always Do This

- ✅ Review `.zoid/output.txt` before committing
- ✅ Test changes after Codex finishes
- ✅ Use specific, scoped prompts
- ✅ Kill stuck sessions promptly

## Session Registry

Sessions are tracked per-agent in:
```
~/clawd-team/agents/{you}/memory/codex-sessions.json
```

**Auto-maintained fields:**
- `current_session` - Active session ID
- `iterations[]` - History of all sessions
- `status` - running/completed/failed
- `output_file` - Path to full output

**Why per-agent?** Multiple agents can work on the same project simultaneously, each tracking their own Codex sessions.

## Troubleshooting

### "Not a git repository"

Skill auto-inits git. The repo is local for now - no remote setup needed yet.

### "Session ID not found"

Registry may be corrupted. Check:
```bash
cat ~/clawd-team/agents/{you}/memory/codex-sessions.json
```

Or start fresh:
```bash
rm ~/clawd-team/agents/{you}/memory/codex-sessions.json
codex-do --project <folder_name> "Start fresh"
```

### Codex hangs / infinite loop

Kill and resume:
```bash
codex-status --project <folder_name> --kill
codex-resume --project <folder_name> "Continue from where you left off"
```

### Output truncated in console

Full output always saved to:
```bash
cat /home/dan/zoidcode/<folder_name>/.zoid/output.txt
```

### Permission errors

Ensure scripts are executable:
```bash
chmod +x ~/clawd-team/agents/{you}/skills/codex-agent/scripts/*
```

## Advanced Usage

### Multi-Project Sessions

Work on multiple projects in parallel:
```bash
# Terminal 1
codex-do --project backend "Add API endpoint"

# Terminal 2
codex-do --project frontend "Build UI for new endpoint"
```

### Multi-Agent on Same Project

Bender and another agent can both work on `mission_control_ui`:
- Code is in one place: `/home/dan/zoidcode/mission_control_ui/`
- Each agent has their own session tracking
- Git handles coordination (commit/pull/push)

### Project Templates

For new projects, Codex can scaffold from scratch:
```bash
codex-do --project new_app "Create a React + TypeScript app with:
- Vite build system
- Tailwind CSS
- React Router
- Folder structure: src/components, src/pages, src/hooks"
```

## References

- **Detailed workflows**: See [references/WORKFLOW.md](references/WORKFLOW.md)
- **Codex CLI docs**: Run `codex --help`
