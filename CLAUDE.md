# Hackathon Workspace

Multi-repo workspace for Ombori hackathon.

## Key Rules

1. **Plan-mode-first**: ALL features start with spec creation in `specs/` folder
2. **TDD where applicable**: Write tests before implementation
3. **Specs in workspace**: All specs centralized in `specs/` folder
4. **Evolve the config**: Update CLAUDE.md and agents with learnings as you build

## Continuous Improvement

As you build, update these files with learnings:
- `CLAUDE.md` - Add new patterns, gotchas, project-specific conventions
- `.claude/agents/*.md` - Refine agent instructions based on what works
- `apps/macos-client/CLAUDE.md` - Swift-specific learnings
- `services/api/CLAUDE.md` - Python/FastAPI-specific learnings

Examples of things to capture:
- "Always use X pattern for Y"
- "Don't forget to run Z after changing W"
- "API endpoint naming follows this convention..."
- "Database migrations require this step..."

## Quick Start

```bash
# Start database
docker compose up -d

# Run API (in new terminal)
cd services/api && uv run fastapi dev

# Run Swift client (in new terminal)
cd apps/macos-client && swift run AgentariumClient
```

## Structure
- `apps/macos-client/` - SwiftUI desktop app (submodule)
- `services/api/` - FastAPI Python backend (submodule)
- `specs/` - Feature specifications (plan-mode output)
- `docker-compose.yml` - PostgreSQL database

## Skills (Commands)
Available in `.claude/skills/`:
- `/feature` - **Start here!** Asks questions -> creates spec -> TDD implementation

## Agents
Available in `.claude/agents/`:
- `/architect` - System design, API contracts -> outputs to `specs/`
- `/swift-coder` - Swift client development
- `/python-coder` - FastAPI backend development
- `/reviewer` - Code review across all repos
- `/debugger` - Issue investigation
- `/tester` - Test-driven development

## Development Workflow

### New Features (MANDATORY)
1. **Plan mode first** - Create spec in `specs/YYYY-MM-DD-feature-name.md`
2. **Write tests** - TDD: tests before implementation
3. **Implement** - Use coder agents (can run in parallel)
4. **Review** - Use reviewer agent
5. **Commit** - Submodules first, then workspace

### Git Workflow (use `gh` CLI, not GitHub web)
Always use feature branches and `gh pr create`:
```bash
# In each submodule
git checkout -b feature/<name>
git add . && git commit -m "feat: ..."
git push -u origin feature/<name>
gh pr create --title "feat: ..." --body "Description"

# After PRs merged, update workspace
cd ../..
git add . && git commit -m "Update submodules"
git push
```

## Parallel Agent Workflow (CRITICAL for avoiding merge conflicts)

When launching multiple agents in parallel on the same repo:

### Problem
Agents working on sequential milestones (M1 → M2 → M3 → M4) create branches from outdated bases, causing merge conflicts when PRs are merged in order.

### Solution: Sequential Branching Strategy
1. **Merge PRs immediately after each milestone** before starting the next
2. **Or** have agents create branches from the previous feature branch (not main):
   ```bash
   # M2 branches from M1, not main
   git checkout feature/m1-foundation
   git checkout -b feature/m2-terrain
   ```
3. **Rebase before merging** if main has changed:
   ```bash
   git fetch origin && git rebase origin/main
   git push --force-with-lease origin feature/<branch>
   ```

### Files That Commonly Conflict
- `Sources/Models/Messages.swift` - Shared data models
- `Sources/Scene/TerrainScene.swift` - Main scene coordinator
- `Sources/WebSocketClient.swift` - Event handling
- `app/schemas/*.py` - Shared schemas

### Best Practices
1. **One agent per file** - Avoid two agents modifying the same file
2. **Define interfaces first** - Agree on schemas/models before parallel work
3. **Small, focused PRs** - Merge frequently to reduce conflict surface
4. **Coordinate shared files** - If both agents need to modify Messages.swift, one should wait

## API Reference
- Swagger: http://localhost:8000/docs
- Health: http://localhost:8000/health
- Items: http://localhost:8000/items (from database)

## Performance Notes

### Filesystem Scanning
- Always filter `node_modules`, `.git`, build directories, etc. (see `EXCLUDED_DIRS` in filesystem.py)
- Use `os.walk()`'s filenames directly - don't re-scan with `iterdir()`
- Limit depth to 5 levels for UI responsiveness

### SceneKit Performance
- Never create thousands of nodes synchronously
- Use async batching with `Task.yield()` between batches
- Keep batch size around 50 nodes to balance speed vs responsiveness
- Mark scene update methods with `@MainActor` for Swift concurrency
