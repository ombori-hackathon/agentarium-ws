# M1: Foundation

## Summary
Backend serves filesystem data, frontend connects via WebSocket, basic 3D scene renders. This establishes the communication layer and visual foundation.

## Dependencies
- Requires: M0 (hooks configured)
- Blocks: M2, M3

## API Contract

### WebSocket `/ws`

**Connection:** `ws://localhost:8000/ws`

**Server -> Client Messages:**
```typescript
// Filesystem layout
{ "type": "filesystem", "data": FilesystemLayout }

// Agent events (from hooks)
{ "type": "agent_event", "data": AgentEvent }

// Agent spawn/despawn
{ "type": "agent_spawn", "data": AgentSpawn }
{ "type": "agent_despawn", "data": { "agent_id": string } }
```

### GET `/api/filesystem?path=...`

**Request:**
```
GET /api/filesystem?path=/Users/ross/Code/phystack-web
```

**Response:**
```json
{
  "root": "/Users/ross/Code/phystack-web",
  "folders": [
    { "path": "/src", "name": "src", "depth": 1, "file_count": 5 },
    { "path": "/src/components", "name": "components", "depth": 2, "file_count": 47 }
  ],
  "files": [
    { "path": "/src/index.ts", "name": "index.ts", "folder": "/src", "size": 1234 }
  ],
  "scanned_at": "2026-01-29T12:00:00Z"
}
```

### POST `/api/events`

**Request (from hooks):**
```json
{
  "session_id": "abc123",
  "hook_event_name": "PreToolUse",
  "tool_name": "Read",
  "tool_input": { "file_path": "/src/index.ts" },
  "cwd": "/Users/ross/Code/phystack-web"
}
```

**Response:**
```json
{ "status": "ok" }
```

## Backend Tasks

| ID | Task | File | Status |
|----|------|------|--------|
| M1.B1 | Add WebSocket endpoint `/ws` | `app/main.py` | 🔲 |
| M1.B2 | Create ConnectionManager for WebSocket clients | `app/websocket.py` | 🔲 |
| M1.B3 | Add POST `/api/events` endpoint | `app/routers/events.py` | 🔲 |
| M1.B4 | Add GET `/api/filesystem?path=...` endpoint | `app/routers/filesystem.py` | 🔲 |
| M1.B5 | Create event schemas | `app/schemas/events.py` | 🔲 |
| M1.B6 | Create filesystem schemas | `app/schemas/filesystem.py` | 🔲 |
| M1.B7 | Write tests for WebSocket | `tests/test_websocket.py` | 🔲 |
| M1.B8 | Write tests for events endpoint | `tests/test_events.py` | 🔲 |
| M1.B9 | Write tests for filesystem endpoint | `tests/test_filesystem.py` | 🔲 |

## Frontend Tasks

| ID | Task | File | Status |
|----|------|------|--------|
| M1.S1 | Add WebSocket client class | `Sources/WebSocketClient.swift` | 🔲 |
| M1.S2 | Create message models (matching backend schemas) | `Sources/Models/Messages.swift` | 🔲 |
| M1.S3 | Create terrain models | `Sources/Models/Terrain.swift` | 🔲 |
| M1.S4 | Replace ContentView with SceneKit view | `Sources/ContentView.swift` | 🔲 |
| M1.S5 | Create basic SceneKit scene (dark background, grid floor) | `Sources/Scene/TerrainScene.swift` | 🔲 |
| M1.S6 | Create grid node | `Sources/Scene/Nodes/GridNode.swift` | 🔲 |
| M1.S7 | Add directory picker for codebase selection | `Sources/Views/DirectoryPicker.swift` | 🔲 |
| M1.S8 | Connection status indicator | `Sources/Views/StatusIndicator.swift` | 🔲 |

## Visual Style

- **Background:** `#0a0a12` (near-black with slight blue)
- **Grid lines:** `#00ffff` at 40% opacity (cyan)
- **Grid spacing:** 10 units
- **Grid extent:** 200 × 200 units

## Tests

### Backend
```bash
cd services/api && uv run pytest tests/test_websocket.py tests/test_events.py tests/test_filesystem.py -v
```

### Frontend
```bash
cd apps/macos-client && swift test
```

## Acceptance Criteria
- [ ] Backend starts without errors
- [ ] WebSocket connection can be established
- [ ] `/api/filesystem` returns valid directory structure
- [ ] `/api/events` accepts hook data and broadcasts via WebSocket
- [ ] Swift app opens with SceneKit scene
- [ ] Grid floor renders with cyan lines on dark background
- [ ] Directory picker allows selecting a folder
- [ ] Connection status shows connected/disconnected state
- [ ] WebSocket reconnects on disconnect

## PRs
- Backend: [link when created]
- Frontend: [link when created]
