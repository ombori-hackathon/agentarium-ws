# Feature: Claude Code Hook Integration & World Reveal

## Summary
Auto-detect running Claude Code instances via hooks, spawn blob agents, and reveal the codebase terrain with a "Creating world..." animation.

## Status
Implemented

## Architecture

```
Claude Code Instance(s)
    │
    ├─ SessionStart/PreToolUse/PostToolUse/SessionEnd events
    │
    └─ Pipes JSON to ~/.agentarium/hook.sh
           │
           └─ POST http://localhost:8000/api/events
                  │
                  ▼
         Agentarium API
           ├─ On SessionStart: Get cwd, scan filesystem, broadcast terrain
           ├─ On ToolUse: Move agent to file position
           └─ On SessionEnd: Despawn agent
                  │
                  ▼
         WebSocket broadcast
                  │
                  ▼
         macOS Client
           ├─ Spawn blob at origin
           ├─ Show "Creating world..." overlay
           ├─ Animate terrain rising up
           └─ Hide overlay when complete
```

## Implementation

### Backend Changes

1. **New Event Schemas** (`app/schemas/events.py`)
   - `TerrainLoading`: Sent when starting to build terrain
   - `TerrainComplete`: Sent when terrain is fully loaded

2. **Agent Service** (`app/services/agent.py`)
   - Added `scan_filesystem()` function for reusable filesystem scanning
   - Modified `process_hook_event()` to return list of messages
   - Modified `_handle_session_start()` to:
     - Broadcast `terrain_loading` event
     - Scan filesystem at cwd
     - Broadcast `filesystem` layout
     - Broadcast `terrain_complete` event
     - Spawn agent

3. **Hook Script** (`scripts/hook.sh`)
   - Forwards Claude Code hook events to Agentarium API
   - Runs curl in background to not block Claude

4. **Setup Script** (`scripts/setup.py`)
   - Installs hook.sh to ~/.agentarium/
   - Configures Claude Code settings.json with hook configuration

### Frontend Changes

1. **New Message Types** (`Sources/Models/Messages.swift`)
   - `TerrainLoading`: Decoded from terrain_loading events
   - `TerrainComplete`: Decoded from terrain_complete events

2. **WorldLoadingOverlay** (`Sources/Views/WorldLoadingOverlay.swift`)
   - Shows "Creating world..." message with spinner
   - Displays folder/file counts when available

3. **WebSocketClient** (`Sources/WebSocketClient.swift`)
   - Added handlers for `terrain_loading` and `terrain_complete` events

4. **TerrainScene** (`Sources/Scene/TerrainScene.swift`)
   - Added `updateTerrainWithAnimation()` for rise animation
   - Folders/files start below ground with opacity 0
   - Rise up in waves by depth with easeOut timing

5. **ContentView** (`Sources/ContentView.swift`)
   - Removed manual path input
   - Shows loading overlay when terrain is building
   - Displays current cwd in header

## Setup Instructions

1. Install hooks:
   ```bash
   cd services/api
   python scripts/setup.py
   ```

2. Start the API:
   ```bash
   cd services/api
   uv run fastapi dev
   ```

3. Run the client:
   ```bash
   cd apps/macos-client
   swift run AgentariumClient
   ```

4. Start Claude Code in any project directory. The terrain will auto-build.

## Uninstall

```bash
cd services/api
python scripts/setup.py --uninstall
```
