# M0: Hook Setup

## Summary
Connect Claude Code to our backend via hooks. This is a prerequisite for all other milestones - hooks are what capture Claude's activity and send it to our backend.

## Dependencies
- Requires: None
- Blocks: M1, M2, M3, M4

## Overview
Claude Code supports hooks that execute shell commands on various events (tool use, session start/end). We'll create a hook script that POSTs event data to our backend.

## Tasks

| ID | Task | File | Status |
|----|------|------|--------|
| M0.1 | Create hook script | `~/.claude/hooks/agentarium-hook.sh` | 🔲 |
| M0.2 | Make script executable | - | 🔲 |
| M0.3 | Configure hooks in settings | `~/.claude/settings.json` | 🔲 |
| M0.4 | Test hook fires on tool use | - | 🔲 |

## Hook Script Spec

**File:** `~/.claude/hooks/agentarium-hook.sh`

```bash
#!/bin/bash
# Agentarium Hook - Forwards Claude Code events to Agentarium backend
# Reads JSON from stdin, POSTs to backend (fire-and-forget)
# Must exit quickly to not block Claude Code

AGENTARIUM_URL="${AGENTARIUM_URL:-http://localhost:8000}"

# Read JSON from stdin
INPUT=$(cat)

# POST to backend (fire-and-forget with timeout)
curl -s -X POST \
  -H "Content-Type: application/json" \
  -d "$INPUT" \
  --max-time 2 \
  "$AGENTARIUM_URL/api/events" > /dev/null 2>&1 &

exit 0
```

## Hook Configuration

**File:** `~/.claude/settings.json`

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Read|Write|Edit|Bash|Grep|Glob",
      "hooks": [{ "type": "command", "command": "~/.claude/hooks/agentarium-hook.sh" }]
    }],
    "PostToolUse": [{
      "matcher": "Read|Write|Edit|Bash|Grep|Glob",
      "hooks": [{ "type": "command", "command": "~/.claude/hooks/agentarium-hook.sh" }]
    }],
    "SessionStart": [{
      "hooks": [{ "type": "command", "command": "~/.claude/hooks/agentarium-hook.sh" }]
    }],
    "SessionEnd": [{
      "hooks": [{ "type": "command", "command": "~/.claude/hooks/agentarium-hook.sh" }]
    }],
    "Stop": [{
      "hooks": [{ "type": "command", "command": "~/.claude/hooks/agentarium-hook.sh" }]
    }]
  }
}
```

## Hook Event Data Format

**PreToolUse / PostToolUse:**
```json
{
  "session_id": "abc123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Read",
  "tool_input": { "file_path": "/src/index.ts" },
  "cwd": "/Users/ross/Code/phystack-web"
}
```

**SessionStart:**
```json
{
  "session_id": "abc123",
  "hook_event_name": "SessionStart",
  "cwd": "/Users/ross/Code/phystack-web"
}
```

## Acceptance Criteria
- [ ] Hook script exists at `~/.claude/hooks/agentarium-hook.sh`
- [ ] Script is executable (`chmod +x`)
- [ ] Settings configured in `~/.claude/settings.json`
- [ ] Hook fires when Claude uses Read/Write/Edit/Bash/Grep/Glob tools
- [ ] Hook fires on session start/end
- [ ] Script exits quickly (non-blocking)

## Notes
- Hook script uses `&` to background the curl call so it doesn't block Claude
- `--max-time 2` prevents hanging if backend is down
- `AGENTARIUM_URL` env var allows custom backend URL
- Script outputs nothing to stdout/stderr to avoid polluting Claude's output

## PRs
- Workspace: (manual setup, no PR needed)
