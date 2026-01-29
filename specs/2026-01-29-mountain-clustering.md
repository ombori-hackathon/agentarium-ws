# Feature: Mountain Range Clustering

## Summary
Reorganize terrain positioning so folders cluster geographically into "mountain ranges" where nested folders create peaks within their parent's range.

## Trigger
Filesystem scan on session start - positions calculated by API.

## Expected Result
- Top-level folders (depth 1) become distinct "mountain ranges" spread around origin
- Subfolders cluster near their parent, creating peaks within the range
- Files cluster at the base of their folder's position
- Deeper nesting = higher elevation (mountain peak effect)
- No more scattered "solar system" of separate rings

## Edge Cases
- Empty folders: still positioned, just no file cluster
- Very deep nesting (5+ levels): cap radius expansion to prevent overlap
- Single child folder: positioned directly above parent
- Many siblings: spread in tight circle around parent

## Technical Design

### Current Algorithm (Problem)
```
Depth 0: origin
Depth 1: ring at radius 15
Depth 2: ring at radius 30 (far from parents!)
Depth 3: ring at radius 45
```
This scatters related folders across the scene.

### New Algorithm (Solution)
```
Depth 0: origin (root)
Depth 1: ring at radius 15 (top-level folders)
Depth 2+: clustered NEAR parent, not in separate ring
```

### Files to Modify

| File | Change |
|------|--------|
| `app/services/terrain.py` | Rewrite `calculate_positions_for_layout()` |
| `tests/test_terrain.py` | Update tests for new clustering behavior |

### Position Formula

```python
def calculate_folder_position(folder, parent_position, sibling_index, sibling_count):
    if folder.depth == 0:
        # Root at origin
        return Position(x=0, y=0, z=0)

    if folder.depth == 1:
        # Top-level folders in circle around origin
        angle = sibling_index * (2 * math.pi / sibling_count)
        radius = 20.0
        return Position(
            x=math.cos(angle) * radius,
            y=3.0,  # Slight elevation
            z=math.sin(angle) * radius
        )

    # Nested folders: cluster near parent
    angle = sibling_index * (2 * math.pi / sibling_count)
    cluster_radius = 5.0  # Tight clustering
    elevation_step = 3.0  # Height increase per depth

    return Position(
        x=parent_position.x + math.cos(angle) * cluster_radius,
        y=parent_position.y + elevation_step,
        z=parent_position.z + math.sin(angle) * cluster_radius
    )

def calculate_file_position(file, parent_folder_position, file_index, file_count):
    # Files cluster at base of folder
    angle = file_index * (2 * math.pi / file_count)
    file_radius = 3.0

    return Position(
        x=parent_folder_position.x + math.cos(angle) * file_radius,
        y=parent_folder_position.y + 0.5,  # Slightly above folder base
        z=parent_folder_position.z + math.sin(angle) * file_radius
    )
```

### Key Changes

1. **Build parent-child relationships**: Parse folder paths to determine hierarchy
2. **Process depth-first**: Position parents before children
3. **Cluster by parent**: Nested folders offset from parent position, not origin
4. **Preserve elevation formula**: `depth * 3.0 + log(file_count + 1)`

### Visual Result

Before (scattered rings):
```
        [depth 2 folders scattered far away]

[depth 1]     [origin]     [depth 1]

        [depth 2 folders scattered far away]
```

After (mountain ranges):
```
      /peak\
     /      \
    / range  \
   /    A     \        /peak\
  /____________\      /range B\
                     /_________\
       [origin]
```

## Implementation Plan

1. [ ] Write failing test: `test_nested_folders_cluster_near_parent`
2. [ ] Write failing test: `test_depth_1_folders_form_outer_ring`
3. [ ] Refactor `calculate_positions_for_layout()` to build folder tree
4. [ ] Implement new position calculation with parent-relative positioning
5. [ ] Run tests - should pass
6. [ ] Manual verification with client

## Verification
1. `uv run pytest tests/test_terrain.py -v` passes
2. Start API: `uv run fastapi dev`
3. Start client: `swift run AgentariumClient`
4. Start Claude Code session in a project with nested folders
5. Observe: folders cluster by parent (mountain ranges)
6. Observe: deeper folders are higher (peaks)
7. Observe: no floating/scattered islands
