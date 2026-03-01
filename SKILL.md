---
name: codex-agent
description: |
  Automated Codex CLI integration for software development tasks.
  Use when you need to write, modify, or review code in a project.
  Handles session tracking, git setup, output capture, and resume automatically.
  Production-hardened with file locking, output rotation, and iteration history.
  Trigger phrases: "build a feature", "implement", "code", "develop", "create app",
  "add component", "fix bug", "refactor", "write tests".
---

# Codex Agent Skill v2.0

Automated Codex CLI wrapper with session tracking. All code lives in `/home/dan/zoidcode/`, shared across agents.

## What's New in v2.0

- ✅ **Dynamic agent detection** - No more hardcoded "Bender"
- ✅ **File locking** - Prevents race conditions during concurrent sessions
- ✅ **Output rotation** - Preserves history (output.txt, output-1.txt, output-2.txt...)
- ✅ **Iteration history** - Full log of all work sessions per project
- ✅ **Mission Control validation** - Ensures project exists before working
- ✅ **Fixed exit codes** - Correctly reports Codex failure/success
- ✅ **Fixed CLI path** - Uses `clawd` instead of old `~/clawd-team/` path
- ✅ **Configurable timeout** - Use `--timeout 0` for no timeout
- ✅ **Dependency checks** - Validates jq, codex, clawd are available

## Quick Start

```bash
# Start working on a project (use folder name from Mission Control)
codex-do --project mission_control_ui "Build a React kanban board"

# Check progress
codex-status --project mission_control_ui

# View iteration history
codex-status --project mission_control_ui --history

# Continue working
codex-resume --project mission_control_ui "Add dark mode toggle"
```

## When to Use This Skill

✅ Building new features or components  
✅ Implementing complete user stories  
✅ Refactoring existing code  
✅ Bug fixes requiring multi-file changes  
✅ Code review  
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
- `--dry-run` - Show what would execute, don't run
- `--suggest-cmd` - Output OpenClaw exec command for PTY/background mode
- `--no-validate` - Skip Mission Control project validation
- `--timeout <minutes>` - Kill after N minutes (0 = no timeout, default: 30)

**Execution mode:** `codex-do` always runs Codex with `-a never exec --sandbox workspace-write`

**Environment Variables:**
- `AGENT_NAME` - Override detected agent name
- `OPENCLAW_WORKSPACE` - Override workspace directory

**What it does automatically:**
1. Validates project exists in Mission Control (unless `--no-validate`)
2. Checks for project in `/home/dan/zoidcode/<folder>/`
3. Creates folder if missing (with git init)
4. Rotates output files (preserves history)
5. Creates `.zoid/` directory with session tracking
6. Runs Codex with proper flags
7. Captures output to `.zoid/output.txt`
8. Updates session registry with file locking
9. Logs iteration history
10. Logs Codex skill usage to Mission Control

### codex-resume

Continue a previous session.

```bash
codex-resume --project <folder_name> "Additional instructions"
```

**Options:**
- `--project <name>` - Project name (required)

**Execution mode:** `codex-resume` always runs Codex with `-a never exec --sandbox workspace-write resume`

**Auto-finds** the last session ID from registry and continues with output rotation.

### codex-status

Check session status and view output.

```bash
# Check status
codex-status --project <folder_name>

# View output
codex-status --project <folder_name> --output

# Follow live output (like tail -f)
codex-status --project <folder_name> --output --follow

# View iteration history
codex-status --project <folder_name> --history

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
clawd project list

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

Check status:
```bash
# Quick status (shows all output files)
codex-status --project mission_control_ui

# View iteration history
codex-status --project mission_control_ui --history

# Watch live output (like tail -f)
codex-status --project mission_control_ui --output --follow
```

### 4. Output File Rotation

Output files are automatically rotated:
- `output.txt` - Current/most recent
- `output-1.txt` - Previous session
- `output-2.txt` - Two sessions ago
- ...up to `output-10.txt`

This preserves history without manual cleanup.

### 5. Handling Questions

If Codex asks questions mid-session:
1. Check output: `codex-status --project X --output`
2. Answer via resume: `codex-resume --project X "Answer: yes"`

### 6. Completion

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

## Safety Rules

### Execution Model

Use the wrapper's only supported mode:

1. `codex-do`: `-a never exec --sandbox workspace-write`
2. `codex-resume`: `-a never exec --sandbox workspace-write resume`

### Never Do This

- ❌ Bypass the wrapper and run unsandboxed Codex commands for normal project work
- ❌ Run Codex in `~/clawd/` or system directories
- ❌ Ignore error messages in output
- ❌ Commit without reviewing changes

### Always Do This

- ✅ Review `.zoid/output.txt` before committing
- ✅ Test changes after Codex finishes
- ✅ Use specific, scoped prompts
- ✅ Kill stuck sessions promptly

## Session Management

### Session Registry

Sessions are tracked per-agent in:
```
~/.openclaw/workspace/memory/codex-sessions.json
```

**Fields:**
- `current_session` - Active session ID
- `last_updated` - Timestamp of last activity
- `tracker` - Full session details

**Automatic cleanup:** Keeps last 50 projects to prevent file bloat.

### Iteration History

Full history of all work sessions:
```
~/.openclaw/workspace/memory/codex-iterations.json
```

View with: `codex-status --project <name> --history`

**Automatic cleanup:** Keeps last 100 iterations per project, 1000 total.

### File Locking

Session registry updates use `flock` to prevent race conditions when multiple sessions run concurrently.

## Troubleshooting

### "jq is required but not installed"

Install jq:
```bash
sudo apt-get install jq
```

### "Project not found in Mission Control"

Run to see available projects:
```bash
clawd project list
```

Or use `--no-validate` to skip validation (not recommended).

### "Not a git repository"

Skill auto-inits git. The repo is local for now - no remote setup needed yet.

### "Session ID not found"

Registry may be corrupted. Check:
```bash
cat ~/.openclaw/workspace/memory/codex-sessions.json
```

Or start fresh:
```bash
rm ~/.openclaw/workspace/memory/codex-sessions.json
rm ~/.openclaw/workspace/memory/codex-iterations.json
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
# Current session
cat /home/dan/zoidcode/<folder_name>/.zoid/output.txt

# Previous sessions
cat /home/dan/zoidcode/<folder_name>/.zoid/output-1.txt
```

### Permission errors

Ensure scripts are executable:
```bash
chmod +x ~/.openclaw/workspace/skills/codex-agent/scripts/*
```

## Advanced Usage

### PTY Mode and Background Execution

Codex is an interactive terminal application. For best results, use PTY mode:

```bash
# Use PTY for proper terminal interaction
exec pty:true command:"codex-do --project foo 'Build feature'"
```

### Background Mode with OpenClaw Process Tracking

For long-running tasks, use OpenClaw's background mode:

**Option 1: Use --suggest-cmd to get the command**
```bash
# Get the proper OpenClaw exec command
codex-do --project foo --suggest-cmd "Build feature"
# Copy the output and run it
```

**Option 2: Manual background execution**
```bash
# Start with PTY and background mode
exec pty:true background:true workdir:/home/dan/zoidcode/<project> \\
  command:"codex -a never exec --sandbox workspace-write -o .zoid/output.txt 'Your task'"
# Returns: sessionId: xxx

# Store the sessionId for tracking
echo '{"openclaw_session_id":"SESSION_ID_HERE"}' > \\
  /home/dan/zoidcode/<project>/.zoid/openclaw-session.json
```

**Monitor with OpenClaw process tool:**
```bash
process action:poll sessionId:xxx
process action:log sessionId:xxx
process action:kill sessionId:xxx
```

### No Timeout for Long Tasks

```bash
# Disable timeout (useful for migrations, data processing)
codex-do --project big_migration --timeout 0 "Process all records"
```

### Multi-Project Sessions

Work on multiple projects in parallel:
```bash
# Terminal 1
codex-do --project backend "Add API endpoint"

# Terminal 2
codex-do --project frontend "Build UI for new endpoint"
```

### Multi-Agent on Same Project

Multiple agents can work on `mission_control_ui`:
- Code is in one place: `/home/dan/zoidcode/mission_control_ui/`
- Each agent has their own session tracking (file locking prevents conflicts)
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

## Configuration

### Environment Variables

- `AGENT_NAME` - Override the detected agent name
- `OPENCLAW_WORKSPACE` - Override the workspace directory path

### File Locations

- Sessions: `~/.openclaw/workspace/memory/codex-sessions.json`
- Iterations: `~/.openclaw/workspace/memory/codex-iterations.json`
- Output: `/home/dan/zoidcode/<project>/.zoid/output.txt`

## References

- **Detailed workflows**: See [references/WORKFLOW.md](references/WORKFLOW.md)
- **Codex CLI docs**: Run `codex --help`
