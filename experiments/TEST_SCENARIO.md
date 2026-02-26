# Test Scenario: Verify PTY + Process Tracking

## Setup

```bash
cd ~/zoidcode/codex-agent

# Verify scripts are executable
chmod +x scripts/*
```

## Test 1: --suggest-cmd Output

```bash
# Get the command for background execution
./scripts/codex-do --project test_project --suggest-cmd --full-auto "Create a hello world app"
```

Expected output:
```
═══════════════════════════════════════════════════════════════
  OpenClaw Exec Command (for background mode with PTY)
═══════════════════════════════════════════════════════════════

Run this to start Codex with PTY and background mode:

exec pty:true background:true workdir:/home/dan/zoidcode/test_project \
  command:"codex exec --dangerously-bypass-approvals-and-sandbox --full-auto --json -o .zoid/output.txt \
    \"Create a hello world app\""

═══════════════════════════════════════════════════════════════

After running, store the sessionId:
  echo '{"openclaw_session_id":"SESSION_ID_HERE"}' > \
    /home/dan/zoidcode/test_project/.zoid/openclaw-session.json
```

## Test 2: OpenClaw Session Detection

```bash
# Create a fake OpenClaw session file
mkdir -p ~/zoidcode/test_project/.zoid
echo '{"openclaw_session_id":"test-session-123"}' > ~/zoidcode/test_project/.zoid/openclaw-session.json

# Check if codex-status detects it
./scripts/codex-status --project test_project
```

Expected: Shows "OpenClaw Session: test-session-123" and suggests process commands.

## Test 3: Full Workflow (Manual)

1. Use --suggest-cmd to get the proper exec command
2. Run the exec command with pty:true background:true
3. Store the returned sessionId in openclaw-session.json
4. Use codex-status to see OpenClaw session info
5. Use process action:log to view output
6. Use process action:kill to terminate

## Limitations Discovered

1. PTY must be set at OpenClaw tool level - cannot be set inside bash script
2. SessionId must be manually stored after exec returns it
3. Process tool commands require the agent to have access to the process tool

## Alternative Approach Considered

Instead of --suggest-cmd, we could:
- Have the agent always call codex directly with exec pty:true
- Use codex-do only for session tracking/registry updates
- But this loses the convenience of the wrapper

## Recommendation

The current approach (--suggest-cmd + manual exec) is a good middle ground:
- Maintains backward compatibility (codex-do still works as before)
- Adds opt-in PTY/background support
- Minimal changes to existing workflow

For production, we should also:
1. Test actual background execution with a real codex task
2. Verify process action:log works with codex output
3. Consider adding auto-store of sessionId via a post-exec hook
