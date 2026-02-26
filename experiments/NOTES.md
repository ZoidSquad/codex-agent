# Experiment 1: Background Mode Helper

## Goal
Enable codex-agent to work with OpenClaw's process tool for better monitoring.

## Approach
Since PTY and background mode are OpenClaw tool parameters (not bash script features),
we need to modify the SKILL.md to instruct agents on proper invocation, and add
helpers to store/retrieve the OpenClaw sessionId.

## Changes Needed

### 1. SKILL.md - Add PTY/Background Instructions

Add new section:
```markdown
## Advanced: Background Execution with Process Tracking

For long-running tasks, use OpenClaw's background mode:

```bash
# Start in background (returns sessionId immediately)
exec pty:true background:true workdir:/home/dan/zoidcode/<project> \\
  command:"codex exec --full-auto 'Your task'"

# Store the returned sessionId for later tracking
echo '{"openclaw_session_id":"SESSION_ID_HERE"}' > \\
  /home/dan/zoidcode/<project>/.zoid/openclaw-session.json
```

Then monitor with:
```bash
# Check if running
process action:poll sessionId:XXX

# View output
process action:log sessionId:XXX

# Kill if needed
process action:kill sessionId:XXX
```
```

### 2. codex-status - Detect OpenClaw Sessions

Modify codex-status to check for openclaw-session.json and suggest process tool commands.

### 3. codex-do - Add --suggest-cmd Flag

Add a flag that outputs the exact exec command to run (for copy-paste).

## Questions to Answer

1. Can we make this seamless for the agent?
2. Should we store sessionId automatically or require manual step?
3. How do we handle the case where codex-do is called directly (not through OpenClaw)?

## Notes

PTY mode requires the exec tool to have `pty:true`. This cannot be set inside a bash script.
The bash script runs INSIDE the exec call, so PTY must be set at the OpenClaw tool level.
