# M4: Integration

## Summary
End-to-end flow works. Run Claude Code with hooks configured, see agent spawn, move in real-time, and despawn on session end. Target latency <100ms from hook to visual.

## Dependencies
- Requires: M1 (WebSocket), M2 (terrain), M3 (agent)
- Blocks: M5

## Event Flow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Claude Code   │────▶│   Hook Script   │────▶│   POST /api/    │────▶│   AgentService  │
│   (tool use)    │     │   (bash)        │     │   events        │     │   (process)     │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
                                                                                 │
                                                                                 ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   3D Scene      │◀────│   TerrainScene  │◀────│   WebSocket     │◀────│   Broadcast     │
│   (visual)      │     │   (update)      │     │   Client        │     │   (agent_event) │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Session Lifecycle

### SessionStart
1. Hook fires with `hook_event_name: "SessionStart"`
2. Backend creates new AgentState with session_id
3. Backend broadcasts `agent_spawn` via WebSocket
4. Frontend creates AgentNode at origin

### Tool Use (PreToolUse)
1. Hook fires with tool details
2. Backend extracts file path from tool_input
3. Backend looks up 3D coordinates for file
4. Backend broadcasts `agent_event` with `event_type: "move"`
5. Frontend animates agent to position
6. Frontend shows tool icon

### Tool Use (PostToolUse)
1. Hook fires with tool completion
2. Backend broadcasts `agent_event` with status
3. Frontend fades tool icon

### SessionEnd
1. Hook fires with `hook_event_name: "SessionEnd"`
2. Backend removes AgentState
3. Backend broadcasts `agent_despawn`
4. Frontend removes AgentNode with fade animation

## Path Extraction

```python
def extract_path(tool_name: str, tool_input: dict) -> Optional[str]:
    """Extract file path from tool input."""
    if tool_name in ("Read", "Write", "Edit"):
        return tool_input.get("file_path")
    if tool_name in ("Grep", "Glob"):
        return tool_input.get("path")
    if tool_name == "Bash":
        # Best effort: parse command for file references
        command = tool_input.get("command", "")
        # Look for common patterns like "cat file", "vim file", etc.
        return None  # Complex, skip for MVP
    return None
```

## Backend Tasks

| ID | Task | File | Status |
|----|------|------|--------|
| M4.B1 | Handle SessionStart event (spawn agent) | `app/services/agent.py` | 🔲 |
| M4.B2 | Handle SessionEnd event (despawn agent) | `app/services/agent.py` | 🔲 |
| M4.B3 | Handle Stop event (despawn agent) | `app/services/agent.py` | 🔲 |
| M4.B4 | Extract file paths from tool inputs | `app/services/events.py` | 🔲 |
| M4.B5 | Look up coordinates from terrain cache | `app/services/agent.py` | 🔲 |
| M4.B6 | Add latency logging | `app/services/agent.py` | 🔲 |
| M4.B7 | Handle unknown file paths (stay in place) | `app/services/agent.py` | 🔲 |
| M4.B8 | Integration test: hook → backend → websocket | `tests/test_integration.py` | 🔲 |

## Frontend Tasks

| ID | Task | File | Status |
|----|------|------|--------|
| M4.S1 | Handle agent_spawn message | `Sources/Scene/TerrainScene.swift` | 🔲 |
| M4.S2 | Handle agent_despawn message | `Sources/Scene/TerrainScene.swift` | 🔲 |
| M4.S3 | Smooth agent movement (SCNAction.move) | `Sources/Scene/Nodes/AgentNode.swift` | 🔲 |
| M4.S4 | Calculate movement duration based on distance | `Sources/Scene/Nodes/AgentNode.swift` | 🔲 |
| M4.S5 | Show current file path label below agent | `Sources/Scene/Nodes/AgentNode.swift` | 🔲 |
| M4.S6 | Tool icon fade on PostToolUse | `Sources/Scene/Nodes/ToolIconNode.swift` | 🔲 |
| M4.S7 | Agent despawn fade animation | `Sources/Scene/Nodes/AgentNode.swift` | 🔲 |
| M4.S8 | Handle rapid successive events (queue) | `Sources/Scene/TerrainScene.swift` | 🔲 |

## Movement Calculation

```swift
// Calculate movement duration based on distance
let distance = (targetPosition - currentPosition).length()
let speed: Float = 10.0  // units per second
let duration = TimeInterval(distance / speed)
let minDuration: TimeInterval = 0.3  // minimum visible movement
let maxDuration: TimeInterval = 2.0  // cap for long distances

let finalDuration = min(max(duration, minDuration), maxDuration)
```

## Latency Budget

Target: <100ms from hook execution to visual update

| Stage | Budget |
|-------|--------|
| Hook script | <10ms |
| HTTP POST | <20ms |
| Backend processing | <10ms |
| WebSocket broadcast | <10ms |
| Frontend processing | <20ms |
| Scene update | <30ms |
| **Total** | **<100ms** |

## Logging

Backend logs timestamps at each stage:
```python
logger.info(f"Event received: {event.session_id} at {time.time()}")
logger.info(f"Event processed: {event.session_id} at {time.time()}")
logger.info(f"Event broadcast: {event.session_id} at {time.time()}")
```

## Tests

### Integration Test Scenario
```python
async def test_full_event_flow():
    # 1. Connect WebSocket client
    async with websockets.connect("ws://localhost:8000/ws") as ws:
        # 2. Simulate SessionStart
        await post_event({
            "session_id": "test-123",
            "hook_event_name": "SessionStart",
            "cwd": "/test/path"
        })

        # 3. Verify agent_spawn received
        msg = await ws.recv()
        assert msg["type"] == "agent_spawn"

        # 4. Simulate tool use
        await post_event({
            "session_id": "test-123",
            "hook_event_name": "PreToolUse",
            "tool_name": "Read",
            "tool_input": {"file_path": "/test/path/src/index.ts"},
            "cwd": "/test/path"
        })

        # 5. Verify agent_event received
        msg = await ws.recv()
        assert msg["type"] == "agent_event"
        assert msg["tool_name"] == "Read"
```

### Backend
```bash
cd services/api && uv run pytest tests/test_integration.py -v
```

### Frontend
```bash
cd apps/macos-client && swift test
```

## Acceptance Criteria
- [ ] Agent spawns when Claude Code session starts
- [ ] Agent moves to file location when Read/Write/Edit used
- [ ] Agent movement is smooth (not instant teleport)
- [ ] Tool icon appears on PreToolUse
- [ ] Tool icon fades on PostToolUse
- [ ] Thought bubble updates with action
- [ ] Agent despawns when session ends
- [ ] Latency is <100ms (visible responsiveness)
- [ ] Rapid events don't break animation (queued)
- [ ] Unknown file paths don't crash (agent stays)

## Manual Testing

1. Start backend: `cd services/api && uv run fastapi dev`
2. Start Swift app: `cd apps/macos-client && swift run AgentariumClient`
3. Select `/Users/rossmalpass/Code/phystack-web` as codebase
4. Open terminal in phystack-web directory
5. Run: `claude` to start Claude Code session
6. Verify agent appears in 3D scene
7. Ask Claude to "read src/index.ts"
8. Verify agent moves to that file's position
9. Exit Claude Code
10. Verify agent disappears

## PRs
- Backend: [link when created]
- Frontend: [link when created]
