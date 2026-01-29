# Performance Optimization

**Status:** Complete
**Created:** 2026-01-29

## Problem Statement

The app is extremely slow (15-90 seconds) and crashes when loading large codebases like `/Users/rossmalpass/Code/phystack-web` because:
- **208,854 files** / **37,559 folders** (mostly `node_modules`)
- No directory filtering - scans everything including node_modules (1.3 GB)
- Redundant filesystem scans in the backend (`iterdir()` after `os.walk`)
- Synchronous node creation blocks the UI thread
- Creates 246,413 SceneKit nodes all at once

## Solution

### Backend Changes (`services/api/app/routers/filesystem.py`)

1. **Add EXCLUDED_DIRS constant** - Skip common large/unneeded directories
2. **Filter directories in os.walk loop** - Modify `dirnames` in-place to skip excluded dirs
3. **Fix file_count** - Use `filenames` from os.walk instead of redundant `iterdir()`
4. **Add depth limit** - Max 5 levels to prevent deep traversal

### Frontend Changes (`apps/macos-client/`)

1. **Make updateTerrain async** - Move node creation off main thread
2. **Add batch processing** - Create nodes in batches of 50 with `Task.yield()` between
3. **Update ContentView** - Call async method properly

## EXCLUDED_DIRS List

```python
EXCLUDED_DIRS = {
    # Package managers
    'node_modules', '.pnpm', 'bower_components', 'vendor', 'packages',

    # Version control
    '.git', '.svn', '.hg',

    # Build outputs
    'dist', 'build', 'out', 'target', '.next', '.nuxt', '.output',

    # Caches
    '.cache', '__pycache__', '.pytest_cache', '.mypy_cache', '.tox',

    # Virtual environments
    '.venv', 'venv', 'env', '.env',

    # IDE/Editor
    '.idea', '.vscode',

    # Logs/temp
    'logs', 'tmp', 'temp', '.tmp',

    # Coverage/reports
    'coverage', '.nyc_output', 'htmlcov',
}
```

## Tasks

### Backend (P.B1-P.B5)

| ID | Task | Status |
|----|------|--------|
| P.B1 | Add EXCLUDED_DIRS constant | ✅ |
| P.B2 | Filter directories in os.walk loop | ✅ |
| P.B3 | Fix file_count to use filenames from os.walk | ✅ |
| P.B4 | Add depth limit (max 5 levels) | ✅ |
| P.B5 | Add tests for filtering | ✅ |

### Frontend (P.S1-P.S3)

| ID | Task | Status |
|----|------|--------|
| P.S1 | Make updateTerrain async | ✅ |
| P.S2 | Add batch processing with yields | ✅ |
| P.S3 | Update ContentView to call async method | ✅ |

## Verification

1. Run backend tests: `cd services/api && uv run pytest tests/test_filesystem.py -v`
2. Start backend: `cd services/api && uv run fastapi dev`
3. Run app: `cd apps/macos-client && swift run AgentariumClient`
4. Load `/Users/rossmalpass/Code/phystack-web`
5. **Expected:** Load in <5 seconds (vs 15-90 seconds before)
6. **Expected:** UI stays responsive during load
7. **Expected:** Only ~100-500 folders shown (not 37,559)

## Files Modified

- `services/api/app/routers/filesystem.py`
- `services/api/tests/test_filesystem.py`
- `apps/macos-client/Sources/Scene/TerrainScene.swift`
- `apps/macos-client/Sources/ContentView.swift`
- `CLAUDE.md` (performance notes)
