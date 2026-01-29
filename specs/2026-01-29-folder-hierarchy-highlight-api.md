# API: Folder Hierarchy Highlighting

## Summary
Add `total_contents` field to folders, update height calculation to use logarithmic scale based on contents, and implement organic file scattering.

## Changes Required

### 1. Schema Updates (`app/schemas/filesystem.py`)

Add two new fields to the `Folder` model:
```python
total_contents: int = 0      # Recursive count of files + subfolders
parent_path: Optional[str] = None  # For hierarchy lookup on client
```

### 2. Terrain Service Updates (`app/services/terrain.py`)

#### A. Calculate Total Contents Recursively
```python
def calculate_total_contents(folder_path: str, folder_children: dict, files_by_folder: dict) -> int:
    """Calculate total files + subfolders recursively."""
    direct_files = len(files_by_folder.get(folder_path, []))
    children = folder_children.get(folder_path, [])
    recursive_total = direct_files + len(children)
    for child in children:
        recursive_total += calculate_total_contents(child.path, folder_children, files_by_folder)
    return recursive_total
```

#### B. Update Height Formula (Logarithmic)
```python
def calculate_folder_height(total_contents: int, max_contents: int) -> float:
    """
    Calculate height based on total contents.
    Range: 2.0 (empty) to 10.0 (root/largest)
    Uses logarithmic scale for visual balance.
    """
    if max_contents <= 0:
        return 2.0
    normalized = math.log(total_contents + 1) / math.log(max_contents + 1)
    return 2.0 + 8.0 * normalized
```

#### C. Organic File Scattering
```python
def calculate_file_position(parent_pos: Position, file_index: int, total_files: int, seed: int) -> Position:
    """
    Calculate organic file position around parent folder.
    Uses deterministic randomness based on seed (hash of file path).
    """
    random.seed(seed)
    angle = file_index * (2 * math.pi / max(total_files, 1)) + random.uniform(-0.3, 0.3)
    radius = 3.0 + random.uniform(-1.0, 1.0)
    return Position(
        x=parent_pos.x + math.cos(angle) * radius,
        y=random.uniform(0.3, 0.7),
        z=parent_pos.z + math.sin(angle) * radius
    )
```

### 3. Integration in `generate_terrain()`

1. First pass: Build folder hierarchy and collect all files by folder
2. Calculate `total_contents` for each folder (starting from leaves)
3. Find `max_contents` (root folder's total)
4. Apply new height formula to each folder
5. Use organic scattering for file positions

## Height Formula Examples

For a project with 500 total items:
- Root (500 items): `2.0 + 8.0 * (log(501)/log(501))` = **10.0 units**
- src/ (200 items): `2.0 + 8.0 * (log(201)/log(501))` ≈ **8.8 units**
- components/ (50 items): `2.0 + 8.0 * (log(51)/log(501))` ≈ **7.0 units**
- Empty folder: `2.0 + 8.0 * (log(1)/log(501))` = **2.0 units**

## Testing

Run existing tests to ensure no regressions:
```bash
cd services/api
uv run pytest
```

Verify API response includes new fields:
```bash
curl http://localhost:8000/terrain/scan -X POST -H "Content-Type: application/json" -d '{"path": "/some/path"}'
```

## Git Workflow

```bash
cd services/api
git checkout -b feature/folder-hierarchy-highlight
# ... implement ...
git add .
git commit -m "feat: Add total_contents field and organic file scattering"
git push -u origin feature/folder-hierarchy-highlight
gh pr create --title "feat: Add total_contents field and organic file scattering" --body "- Add total_contents and parent_path to Folder schema
- Implement logarithmic height calculation based on contents
- Add organic file scattering around parent folders"
```
