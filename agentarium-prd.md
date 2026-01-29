# Agentarium — Product Requirements Document

**Version:** 2.0  
**Author:** [Product]  
**Last Updated:** January 2026  
**Status:** Ready for Development

---

## 1. Executive Summary

### 1.1 Vision
Agentarium is a native macOS application that visualizes AI coding agents working within a codebase in real-time. Users watch Claude Code agents navigate their filesystem rendered as a 3D terrain — folders as mountains, files as trees — seeing what files agents read, write, and think about. Like watching workers in a digital landscape.

### 1.2 Opportunity
The $1M Build with Claude competition rewards innovative developer tools. Agentarium addresses a real problem: **AI coding agents are black boxes**. Developers run Claude Code but can't see what it's doing, where it's looking, or why it's stuck. This tool makes the invisible visible.

### 1.3 Success Criteria
- Judges say "wow" when they see it
- Real-time visualization with <100ms latency
- Works with any codebase Claude Code can access
- Visually distinctive (not another terminal or tree view)
- Built entirely via terminal + Claude Code prompts (competition requirement)

---

## 2. Target Users

### 2.1 Primary: Developers using Claude Code
- Already using Claude Code for coding tasks
- Want to understand what the agent is doing
- May be running multiple agents simultaneously
- Value aesthetics and polish in their tools

### 2.2 Secondary: Competition Judges
- Looking for innovation, polish, and "wow factor"
- Evaluating technical achievement
- Assessing practical utility

---

## 3. User Stories

### MVP Scope
> **As a developer**, I want to see my codebase as a 3D landscape so I can visually understand its structure.

> **As a developer**, I want to see where Claude is in my codebase so I understand what it's working on.

> **As a developer**, I want to see what tool Claude is using (Read, Write, Bash) so I can follow its actions.

> **As a developer**, I want to see Claude's current thought/action so I can follow its reasoning.

> **As a developer**, I want Claude represented as a cute character so the experience is delightful.

### Future (v1.1+)
> **As a developer** running multiple Claude instances, I want each agent color-coded so I can distinguish them.

> **As a developer**, I want to click on an agent to focus the camera on them.

> **As a developer**, I want an activity feed showing recent actions.

---

## 4. Functional Requirements

### 4.1 Terrain Visualization

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F1.1 | Render ground plane with grid pattern | P0 | MVP |
| F1.2 | Render folders as elevated terrain/mountains | P0 | MVP |
| F1.3 | Folder elevation based on depth + file count (see 6.2) | P0 | MVP |
| F1.4 | Render files as cubes growing from terrain | P0 | MVP |
| F1.5 | Every path maps to exactly one 3D coordinate | P0 | MVP |
| F1.6 | Positions are deterministic | P0 | MVP |
| F1.7 | Support up to 10,000 files without lag | P0 | MVP |
| F1.8 | Display folder labels | P1 | MVP |
| F1.9 | Optional color tinting by top-level folder (subtle) | P2 | v1.1 |
| F1.10 | File colors by extension | P2 | v1.1 |

### 4.2 Agent Visualization

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F2.1 | Render single agent as "Claude blob" character | P0 | MVP |
| F2.2 | Agent physically moves across terrain | P0 | MVP |
| F2.3 | Walk animation while moving | P1 | MVP |
| F2.4 | Idle animation when waiting | P2 | v1.1 |
| F2.5 | Multiple agents with distinct colors | P2 | v1.1 |

### 4.3 Agent Indicators (Above Head)

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F3.1 | Thought bubble with action text | P0 | MVP |
| F3.2 | Tool icon (Read, Write, Bash, etc.) | P0 | MVP |
| F3.3 | Current file path label | P0 | MVP |
| F3.4 | Icon appears on PreToolUse, fades on PostToolUse | P1 | MVP |
| F3.5 | Auto-fade after inactivity | P2 | v1.1 |
| F3.6 | Indicators follow agent | P0 | MVP |

### 4.4 Real-Time Events

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F4.1 | Receive events from Claude Code via hooks | P0 | MVP |
| F4.2 | Move agent to file location on read/write | P0 | MVP |
| F4.3 | Update thought bubble with action | P0 | MVP |
| F4.4 | Latency <100ms from event to visual | P0 | MVP |
| F4.5 | Handle SessionStart / SessionEnd | P1 | MVP |

### 4.5 UI/HUD

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F5.1 | Connection status indicator | P0 | MVP |
| F5.2 | Agent card (name, file, status) | P0 | MVP |
| F5.3 | Filesystem stats (folder/file count) | P2 | v1.1 |
| F5.4 | Activity feed | P2 | v1.1 |

### 4.6 Camera & Navigation

| ID | Requirement | Priority | Version |
|----|-------------|----------|---------|
| F6.1 | Slow orbiting camera | P1 | MVP |
| F6.2 | Manual camera control (pause orbit) | P1 | MVP |
| F6.3 | Click agent to focus | P2 | v1.1 |
| F6.4 | Zoom controls | P1 | MVP |

---

## 5. Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NF1 | Event latency | <100ms from Claude Code to visual update |
| NF2 | Frame rate | 60fps on M1 Mac |
| NF3 | Memory usage | <500MB for 10k file codebase |
| NF4 | Startup time | <3 seconds to first render |
| NF5 | Build method | Terminal only, no Xcode GUI |

---

## 6. Design Specification

### 6.1 Core Concept: Codebase as Terrain

The codebase renders as a **navigable 3D mountain landscape** — not abstract floating shapes, but terrain you could walk through. Every folder and file has a fixed position. Agents physically move across this terrain to reach their targets, climbing uphill into deeply nested code.

**Metaphor:**
```
Codebase Structure          →    Visual Representation
────────────────────────────────────────────────────────
Root folder                 →    Ground plane / base terrain
Top-level folders           →    Mountain ranges (elevated ridges)
Nested subfolders           →    Peaks rising from ranges
Deeply nested folders       →    Higher peaks (elevation = depth)
Busy folders (many files)   →    Taller peaks (file count adds height)
Files                       →    Cubes/crystals on slopes near parent
```

**Key insight:** You can "read" the codebase structure from elevation. Deep, busy areas form dramatic peaks. Shallow, sparse areas are low rolling hills. An agent traveling to a deeply nested file visually climbs uphill.

### 6.2 Terrain Layout

**Ground Plane:**
- Dark grid floor (Tron-style cyan lines on black)
- Extends to fog at horizon
- Represents the project root

**Folders as Mountain Ranges & Peaks:**
- **Top-level folders** = Mountain ranges (elevated ridges/plateaus)
- **Nested subfolders** = Peaks rising from parent ranges
- **Deeper nesting** = Higher elevation (always)
- **More files** = Additional peak height (logarithmic)
- All mountains rendered as wireframe pyramids with cyan/green glow
- Labels float above peaks

**Elevation Formula:**
```
elevation = (depth × BASE_UNIT) + log(file_count + 1) × HEIGHT_MULTIPLIER
```

Where:
- `depth` = folder nesting level (root = 0, /src = 1, /src/components = 2, etc.)
- `file_count` = number of files directly in this folder
- `BASE_UNIT` = base elevation per depth level (e.g., 3.0 units)
- `HEIGHT_MULTIPLIER` = scaling for file count contribution (e.g., 2.0 units)

**Example elevations:**
```
/src                 (depth 1, 5 files)   → 3.0 + log(6)×2  = 3.0 + 1.6  = 4.6
/src/components      (depth 2, 47 files)  → 6.0 + log(48)×2 = 6.0 + 3.4  = 9.4
/src/components/ui   (depth 3, 12 files)  → 9.0 + log(13)×2 = 9.0 + 2.3  = 11.3
/utils               (depth 1, 3 files)   → 3.0 + log(4)×2  = 3.0 + 1.2  = 4.2
```

**Why this model:**
1. **Depth as base** → Structure is always readable. Nested folders are *always* higher than parents.
2. **File count for peaks** → Busy folders stand out. High-activity areas form dramatic peaks.
3. **Logarithmic scaling** → Prevents massive folders from dominating. 1000 files isn't 100× taller than 10.
4. **No bytes** → Avoids distortion from assets, binaries, or node_modules.

**File Objects:**
- Files render as **small cubes or crystalline shapes**
- Positioned on terrain surface around their parent folder's peak
- Uniform color for v1 (matches terrain glow)
- Size can indicate file size (optional, v1.1)

**Terrain Connectivity:**
- Terrain forms continuous slopes between parent and child folders
- Agents can walk/climb between any two points
- No floating islands — everything connects to ground via its parent chain

### 6.3 Spatial Mapping (Critical)

Every filesystem path maps to exactly one 3D coordinate. The Y-axis represents elevation (depth + activity).

```
/src                    → (0, 4.6, 0)       ridge base for src
/src/index.ts           → (2, 4.8, 1)       file on src ridge slope
/src/components         → (5, 9.4, 2)       peak rising from src ridge
/src/components/ui      → (6, 11.3, 3)      higher peak
/src/components/ui/Button.tsx → (7, 11.5, 4) file near ui peak
```

**Why this matters:** When Claude reads `/src/components/Button.tsx`, the agent must move to that exact location. The mapping must be:
- **Deterministic** — same path always = same position
- **Stable** — positions don't change during session
- **Navigable** — agent can pathfind between any two points

### 6.4 Visual Style: "Darwinia Meets Tron"

**Reference:** Darwinia (2005 game) — low-poly, glowing wireframes, dark void, neon accents.

**Color Palette (v1 — Simplified):**
| Element | Color | Hex |
|---------|-------|-----|
| Background / void | Near-black | `#0a0a12` |
| Grid lines | Cyan | `#00ffff` at 40% opacity |
| Terrain / folders | Green glow | `#00ff88` |
| Files | Same as terrain, 60% opacity | `#00ff88` at 60% |
| Agent | Coral/salmon | `#e07850` |
| Text / UI | Cyan | `#00ffff` |

**Color Palette (v1.1+ — Multi-agent):**
| Agent | Color | Hex |
|-------|-------|-----|
| Agent 1 | Coral | `#e07850` |
| Agent 2 | Blue | `#50a0e0` |
| Agent 3 | Green | `#50e080` |
| Agent 4 | Pink | `#e050a0` |

**Atmosphere:**
- Fog fades distant objects (exponential fog)
- Structures have glowing edges (emission/bloom)
- Subtle floating particles
- Grid lines pulse subtly

### 6.5 Agent Character: "Claude Blob"

Based on the official Claude Code mascot — a chunky, cute, minimal blob creature. The 3D model should faithfully recreate this character in SceneKit.

**Reference Asset:** `/specs/agent.png` (pixel art reference)

**Character Description:**
A friendly, rounded rectangular creature with a warm coral/salmon colored body. Features two small black square eyes positioned in the upper portion of the face, giving it an innocent, curious expression. Four short stubby legs extend from the bottom corners, slightly darker than the body. The overall silhouette is wider than it is tall — think "chunky loaf" energy.

**3D Model Specifications:**
| Part | Shape | Dimensions (units) | Color | Notes |
|------|-------|-------------------|-------|-------|
| Body | Rounded box (SCNBox with chamfer) | 4W × 3H × 3D | `#e07850` (coral) | chamferRadius: 0.4 for soft edges |
| Eyes | Flat squares (SCNBox) | 0.7W × 0.7H × 0.1D | `#000000` (black) | Positioned at y=0.5 from center, spaced 1.2 apart |
| Legs | Small boxes | 0.6W × 1.2H × 0.6D | `#c06040` (darker coral) | 4 legs at corners, slight gap from body |
| Glow | Circular plane | radius 3 | Agent color @ 30% | Flat circle beneath agent, adds "presence" |

**Proportions (Critical):**
- Body width:height ratio is approximately **4:3** (wider than tall)
- Eyes sit in **upper third** of face
- Eyes are **small relative to body** (~18% of body width each)
- Legs are **stubby** — shorter than body height
- Overall character is **grounded and stable**, not top-heavy

**Material Properties:**
| Part | Material | Properties |
|------|----------|------------|
| Body | SCNMaterial | Diffuse: agent color, slight emission for glow effect |
| Eyes | SCNMaterial | Diffuse: pure black, no reflection |
| Legs | SCNMaterial | Diffuse: darker shade of agent color |

**Animations:**
| State | Animation | Details |
|-------|-----------|---------|
| Walking | Body bobs up/down, legs alternate | Bob amplitude: 0.2 units, leg cycle: 0.3s |
| Idle | Subtle breathing (scale pulse) | Scale 1.0 → 1.03 → 1.0, duration: 2s, ease in-out |
| Thinking | Small bounce or head tilt | Quick squash-stretch, 0.5s duration |
| Working | Tool-specific | Read: slight lean forward; Write: small vibration |

**Personality Notes:**
The blob should feel alive and endearing. Even when idle, subtle animation keeps it feeling present. Movement should be bouncy and playful, never stiff or mechanical. Think Pixar lamp energy.

### 6.6 Agent Indicators (Above Head)

Agents display floating UI above their head showing current state:

**Thought Bubble:**
```
    ╭─────────────────────────╮
    │ "Reading index.ts..."   │
    ╰───────────┬─────────────╯
                │
            [agent]
```

- Semi-transparent dark background
- White or cyan text
- Max ~30 characters, truncate with "..."
- Fades after 3 seconds of inactivity

**Tool Icon:**
When using a tool, show icon above agent:

| Tool | Icon | Visual |
|------|------|--------|
| Read | 📖 or eye icon | Book opening |
| Write | ✏️ or pencil | Pencil writing |
| Edit | 🔧 or wrench | Gear turning |
| Bash | ⌨️ or terminal | Terminal blinking |
| Search/Grep | 🔍 or magnifier | Magnifier pulsing |
| Web | 🌐 or antenna | Signal waves |

- Icon floats above agent
- Appears on `PreToolUse`, fades on `PostToolUse`
- Can coexist with thought bubble (icon above, text below)

**Current File Label:**
```
    📍 /src/components/Button.tsx
                │
            [agent]
```

- Shows path of file being accessed
- Smaller than thought bubble
- Optional: show just filename, full path on hover

### 6.7 Multi-Agent Support (v1.1+)

*Not in scope for MVP. Design documented for future.*

When multiple agents exist:
- Each gets unique color from palette
- Name label below agent: "claude-1", "claude-2"
- All agents visible simultaneously
- Camera can focus on selected agent

### 6.8 HUD Layout

```
┌────────────────────────────────────────────────────────────┐
│ ┌──────────────────────┐                    AGENTARIUM     │
│ │ ■  claude-1           │                   ● Connected     │
│ │ 📍 /src/index.ts     │                                   │
│ │ 💭 "Checking imports"│                                   │
│ └──────────────────────┘                                   │
│ ┌──────────────────────┐                                   │
│ │ ■  claude-2           │                                   │
│ │ 📍 /api/routes.ts    │                                   │
│ │ 🔧 Edit              │                                   │
│ └──────────────────────┘                                   │
│                                                            │
│                      [ 3D SCENE ]                          │
│                                                            │
│                                                            │
│                                                            │
│                                        ┌────────────────┐  │
│                                        │ 📁 24 folders  │  │
│                                        │ 📄 156 files   │  │
│                                        └────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

**Panels:**
- **Agent cards** (top-left): One per active agent, shows name, location, status
- **Connection status** (top-right): Server connection indicator
- **Stats** (bottom-right): Folder/file counts
- All panels: dark semi-transparent background, monospace font

---

## 7. Technical Architecture

### 7.1 Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Frontend | SwiftUI + SceneKit | Native macOS, 3D rendering |
| Backend | FastAPI (Python) | WebSocket server, event processing |
| Database | PostgreSQL | Event history, session persistence |
| Integration | Claude Code Hooks | Native hook system, bash scripts |

### 7.2 Component Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        macOS App (SwiftUI)                      │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   SceneKit View                          │    │
│  │  • Grid floor                                            │    │
│  │  • Folder pyramids                                       │    │
│  │  • File cubes                                            │    │
│  │  • Agent blobs                                           │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   SwiftUI HUD Overlay                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   WebSocket Client                       │    │
│  └──────────────────────────┬──────────────────────────────┘    │
└─────────────────────────────│───────────────────────────────────┘
                              │ WebSocket
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     FastAPI Backend (Python)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ WebSocket    │  │ Event        │  │ Filesystem   │          │
│  │ Manager      │  │ Processor    │  │ Scanner      │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│  ┌──────────────┐  ┌─────────────┐                            │
│  │ HTTP API     │  │ Database     │                            │
│  │ (hooks)      │  │ (PostgreSQL) │                            │
│  └──────────────┘  └─────────────┘                            │
└─────────────────────────────┬───────────────────────────────────┘
                              ▲ HTTP POST
                              │
┌─────────────────────────────┴───────────────────────────────────┐
│                     Claude Code Hooks                           │
│  • Bash script receives events via stdin                        │
│  • POSTs JSON to backend                                        │
│  • Fires on: PreToolUse, PostToolUse, Stop, SessionStart, etc.  │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ Hook Events
┌─────────────────────────────┴───────────────────────────────────┐
│                     Claude Code CLI                             │
│  (User runs this in their terminal)                             │
└─────────────────────────────────────────────────────────────────┘
```

### 7.3 Data Flow

1. **User runs Claude Code** in their project directory
2. **Claude Code fires hooks** on tool use (PreToolUse, PostToolUse, etc.)
3. **Hook script receives JSON** via stdin with tool name, file paths, etc.
4. **Hook POSTs to FastAPI** backend at `/api/events`
5. **Backend processes event**, stores in PostgreSQL, broadcasts via WebSocket
6. **Swift app receives WebSocket message**, updates 3D scene
7. **Agent moves** to new location, thought bubble updates

### 7.4 Claude Code Hooks Integration

The app integrates via Claude Code's native hooks system. Reference: https://docs.anthropic.com/claude-code/hooks

**Events we capture:**
| Event | What it tells us |
|-------|------------------|
| `PreToolUse` | Agent is about to use a tool (Read, Write, Bash, etc.) |
| `PostToolUse` | Tool completed, includes response |
| `SessionStart` | New Claude Code session began |
| `SessionEnd` | Session ended |
| `Stop` | Agent finished responding |
| `UserPromptSubmit` | User sent a prompt |

**Hook script responsibilities:**
- Receive JSON from stdin
- Extract tool name, file paths, session ID
- POST to backend API
- Never block (exit 0 quickly)

---

## 8. Data Models

### 8.1 Filesystem Layout (sent on connect)

```
FilesystemLayout {
  root: string              // "/Users/dev/myproject"
  folders: Folder[]
  files: File[]
  scanned_at: timestamp
}

Folder {
  path: string              // "/src/components"
  name: string              // "components"
  depth: int                // 2
  file_count: int           // 12
  position: {x, y, z}       // 3D coordinates
  color: string             // "#00ff88"
  height: float             // pyramid height
}

File {
  path: string              // "/src/components/Button.tsx"
  name: string              // "Button.tsx"
  folder: string            // "/src/components"
  size: int                 // bytes
  position: {x, y, z}
}
```

### 8.2 Agent Event (real-time)

```
AgentEvent {
  type: "agent_event"
  agent_id: string          // "session-abc123"
  event_type: enum          // "move" | "read" | "write" | "think" | "idle"
  target_path: string?      // "/src/index.ts"
  thought: string?          // "Reading the entry point..."
  tool_name: string?        // "Read"
  timestamp: int            // Unix ms
}
```

### 8.3 Agent Spawn

```
AgentSpawn {
  type: "agent_spawn"
  agent_id: string
  name: string              // "Claude"
  color: string             // "#e07850"
  position: {x, y, z}       // spawn location
}
```

---

## 9. API Endpoints

### 9.1 HTTP (Backend)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| POST | `/api/events` | Receive hook events |
| GET | `/api/filesystem?path=...` | Scan and return filesystem layout |
| GET | `/api/sessions` | List active agent sessions |

### 9.2 WebSocket (Backend → Frontend)

| Message Type | Direction | Description |
|--------------|-----------|-------------|
| `filesystem` | Server→Client | Full filesystem layout |
| `agent_spawn` | Server→Client | New agent appeared |
| `agent_event` | Server→Client | Agent action/movement |
| `agent_despawn` | Server→Client | Agent disconnected |

---

## 10. Roadmap

### Phase 1: MVP (Competition Submission)

**Goal:** Working end-to-end demo. One agent, basic terrain, real-time events.

| Feature | Description | Status |
|---------|-------------|--------|
| Terrain rendering | Grid floor, folders as mountains (uniform color) | 🔲 |
| File rendering | Cubes around parent folders | 🔲 |
| Single agent | Claude blob character with walk animation | 🔲 |
| Agent movement | Moves to file location on Read/Write events | 🔲 |
| Thought bubble | Shows current action text above agent | 🔲 |
| Tool icons | Shows Read/Write/Bash icon above agent | 🔲 |
| Hook integration | Receives events from Claude Code | 🔲 |
| Basic HUD | Connection status, current file display | 🔲 |
| Camera | Slow orbit, basic zoom | 🔲 |

**Out of scope for MVP:**
- Multi-agent support
- Colored regions
- Activity feed
- Sound effects
- Database persistence

---

### Phase 2: Polish (v1.1)

**Goal:** Competition-winning polish. Visual effects, multi-agent, better UX.

| Feature | Description |
|---------|-------------|
| Multi-agent | Support 2-4 simultaneous agents with distinct colors |
| Agent colors | Color-coded by session ID |
| Visual effects | Particle systems, bloom/glow, better fog |
| Activity feed | Scrolling log of recent events |
| Camera controls | Click-to-focus on agent, smooth transitions |
| Subtle region tinting | Optional color hint by top-level folder (doesn't replace terrain) |
| File colors | Color by extension (.ts, .py, .md) |

---

### Phase 3: Future (v2.0)

**Goal:** Production-ready tool beyond competition.

| Feature | Description |
|---------|-------------|
| Database | Persist event history in PostgreSQL |
| Replay mode | Scrub through past sessions |
| Submodule support | Git submodules as distinct regions |
| Search | Find file/folder in terrain |
| Minimap | Overview of entire terrain |
| Sound effects | Spatial audio for agent actions |
| Agent pathfinding | Agents walk around obstacles, not through |
| Performance | Handle 50k+ file codebases |
| Cross-platform | Linux support (if demand) |

---

### Milestones (Time-based)

| Milestone | Target | Deliverable |
|-----------|--------|-------------|
| M1: Foundation | Week 1 | Backend serves filesystem, frontend connects, basic 3D scene |
| M2: Terrain | Week 2 | Folders render as mountains, files as cubes, camera works |
| M3: Agent | Week 3 | Claude blob renders, walks, thought bubbles work |
| M4: Integration | Week 4 | Hooks installed, events flow end-to-end, real-time works |
| M5: Polish | Week 5 | Visual effects, HUD complete, demo recording |
| **Competition Submit** | Week 5 | MVP complete |

---

## 11. Open Questions

| # | Question | Owner | Status |
|---|----------|-------|--------|
| 1 | Should we persist event history or just real-time? | Product | PostgreSQL included, but optional for v1 |
| 2 | Handle VibeCraft's existing hooks gracefully? | Dev | Need to merge or replace |
| 3 | Support Windows/Linux or macOS only? | Product | macOS only for competition |
| 4 | Include sound effects? | Design | Nice-to-have, P2 |

---

## 12. References

- **VibeCraft**: https://github.com/Nearcyan/vibecraft — Similar project, browser-based. Study their hook implementation.
- **Claude Code Hooks**: https://docs.anthropic.com/claude-code/hooks — Official documentation
- **Darwinia**: Visual reference for aesthetic
- **SceneKit Docs**: https://developer.apple.com/scenekit/

---

## Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Hook** | A shell script that Claude Code executes at specific lifecycle events |
| **Agent** | A running instance of Claude Code |
| **Session** | A single Claude Code conversation/task |
| **Tool** | An action Claude can take (Read, Write, Bash, etc.) |

---

## Appendix B: Competition Requirements

Per the Build with Claude competition:
- ✅ Must be built using terminal and Claude Code (no IDE GUI)
- ✅ Python and Swift allowed
- ✅ Must demonstrate novel use of AI
- ✅ Must be functional, not just a mockup
