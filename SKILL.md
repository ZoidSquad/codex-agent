---
name: codex-agent
description: |
  Automated Codex CLI integration for software development tasks.
  Use when you need to write, modify, or review code in a project.
  Handles session tracking, detached execution, output capture, and resume automatically.
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
- ✅ **Tracked background execution** - Start Codex once and inspect it across cron turns
- ✅ **Machine-readable status** - `codex-status --json` for reliable orchestration
- ✅ **Duplicate-run guard** - Prevents overlapping Codex launches in the same repo by default
- ✅ **Task-branch enforcement** - Tracked task work stays on a deterministic git branch per task
- ✅ **Fixed exit codes** - Correctly reports Codex failure/success
- ✅ **Fixed CLI path** - Uses `clawd` instead of old `~/clawd-team/` path
- ✅ **Configurable timeout** - Use `--timeout 0` for no timeout
- ✅ **Dependency checks** - Validates jq, codex, clawd are available

## Quick Start

```bash
# Prepare the task branch first
codex-branch --project mission_control_ui --task-id k176w64p843g1brae8zmqys6qs820knf

# Start tracked background work
codex-do --project mission_control_ui --task-id k176w64p843g1brae8zmqys6qs820knf --background "Build a React kanban board"

# Check progress
codex-status --project mission_control_ui --json

# Continue later if needed
codex-resume --project mission_control_ui --task-id k176w64p843g1brae8zmqys6qs820knf --background "Add dark mode toggle"
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

### codex-branch

Prepare or switch to the task branch for tracked work.

```bash
codex-branch --project <folder_name> --task-id <task-id> [options]
```

**Options:**
- `--project <name>` - Folder name from Mission Control (required)
- `--task-id <id>` - Mission Control task ID / target branch name (required)
- `--base <branch>` - Base branch to branch from (default: `dev`, then `main`, then `master`)
- `--json` - Machine-readable result

**Task branch rule:** For task-tracked work, the branch name is the full task ID. Reopened tasks continue on the same branch.

### codex-do

Main entry point for starting work.

```bash
codex-do --project <folder_name> [options] "Your prompt here"
```

**Options:**
- `--project <name>` - Folder name from Mission Control (required)
- `--task-id <id>` - Mission Control task ID to tie to the tracked session
- `--background` - Launch Codex in a tracked detached process
- `--dry-run` - Show what would execute, don't run
- `--force` - Bypass the duplicate-run guard for the project
- `--suggest-cmd` - Output OpenClaw exec command for PTY/background mode
- `--no-validate` - Skip Mission Control project validation
- `--timeout <minutes>` - Kill after N minutes (0 = no timeout, default: 0)

**Execution mode:** `codex-do` always runs Codex with `-a never exec --sandbox workspace-write`

**Task branch rule:** If you pass `--task-id`, the current git branch must match that task ID. Use `codex-branch` first.

**Environment Variables:**
- `AGENT_NAME` - Override detected agent name
- `OPENCLAW_WORKSPACE` - Override workspace directory

**What it does automatically:**
1. Validates project exists in Mission Control (unless `--no-validate`)
2. Checks for project in `/home/dan/zoidcode/<folder>/`
3. Creates folder if missing (with git init)
4. Refuses to start if Codex is already active in that repo (unless `--force`)
5. Rotates output files (preserves history)
6. Creates `.zoid/` directory with session tracking
7. Runs Codex with proper flags
8. Captures output to `.zoid/output.txt`
9. Updates session registry with file locking
10. Logs iteration history
11. Logs Codex skill usage to Mission Control

### codex-resume

Continue a previous session.

```bash
codex-resume --project <folder_name> "Additional instructions"
```

**Options:**
- `--project <name>` - Project name (required)
- `--task-id <id>` - Override or attach a Mission Control task ID
- `--background` - Resume in a tracked detached process
- `--force` - Bypass the duplicate-run guard for the project

**Execution mode:** `codex-resume` always runs Codex with `-a never exec --sandbox workspace-write resume`

**Auto-finds** the last session ID from registry and continues with output rotation. Tracked task resumes stay on the same task branch.

### codex-status

Check session status and view output.

```bash
# Check status
codex-status --project <folder_name>

# Check status as JSON
codex-status --project <folder_name> --json

# View output
codex-status --project <folder_name> --output

# Follow live output (like tail -f)
codex-status --project <folder_name> --output --follow

# View iteration history
codex-status --project <folder_name> --history

# Kill stuck session
codex-status --project <folder_name> --kill
```

`--json` is the preferred mode for agents that need to decide whether Codex is active, quiet, resumable, or completed.
It also reports the current git branch, tracked branch, and whether they match.

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

### 2. Task Branches

For Mission Control task work, create or switch to the task branch first:

```bash
codex-branch --project mission_control_ui --task-id k176w64p843g1brae8zmqys6qs820knf
```

Rules:
- task branch name is the full task ID
- continue reopened tasks on the same branch
- do not mix multiple tasks on the same branch
- if the repo is dirty on another branch, stop and clean it up before switching tasks

### 3. Starting Work

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

### 3a. Cron-Managed Pattern

For unattended task execution:

1. `codex-branch --project <folder> --task-id <task-id>`
2. `codex-status --project <folder> --json`
3. If `process_running=true` and `productive=true`, leave the session alone
4. If `resumable=true`, use `codex-resume`
5. If no tracked run exists, start one with:

```bash
codex-do --project mission_control_ui --task-id <task-id> --background "Implement the assigned task"
```

Do not launch raw `codex exec` or ad hoc background Codex from a cron-managed agent loop.

### 4. Monitoring Progress

Check status:
```bash
# Quick machine-readable status
codex-status --project mission_control_ui --json

# Human-readable summary
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
2. Answer via resume: `codex-resume --project X --background "Answer: yes"`

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
3. Detached work should go through `--background`, not raw shell backgrounding

### Never Do This

- ❌ Bypass the wrapper and run unsandboxed Codex commands for normal project work
- ❌ Run Codex in `~/clawd/` or system directories
- ❌ Ignore error messages in output
- ❌ Commit without reviewing changes
- ❌ Launch a second Codex run into the same repo without checking `codex-status --json`

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
