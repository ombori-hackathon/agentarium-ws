# Feature: Hover Tooltips for Terrain Items

## Summary
Remove always-visible 3D text labels from folders/files. Show small tooltip near cursor on hover instead.

## Trigger
Mouse hover over folder (pyramid) or file (cube) in the 3D scene.

## Expected Result
- No visible labels cluttering the scene by default
- On hover: small tooltip appears near cursor showing:
  - Name (14pt, bold)
  - Full path (10pt, gray, below name)
- Tooltip disappears when mouse moves away

## Edge Cases
- Multiple items close together: show tooltip for topmost/closest item
- Rapid mouse movement: debounce tooltip updates
- Items at edge of screen: keep tooltip within viewport bounds

## Technical Design

### Files to Modify

| File | Change |
|------|--------|
| `Sources/Scene/Nodes/FolderNode.swift` | Remove `LabelNode` creation |
| `Sources/Scene/Nodes/FileNode.swift` | Store name/path metadata on node |
| `Sources/Scene/TerrainScene.swift` | Track hovered node via hit-testing |
| `Sources/ContentView.swift` | Add hover tracking, show tooltip overlay |

### New File
| File | Purpose |
|------|---------|
| `Sources/Views/TooltipView.swift` | SwiftUI tooltip component |

### Implementation Steps

1. **Remove existing labels** from `FolderNode.swift`:
   - Delete the `LabelNode` instantiation (around line 23)
   - Store folder name/path in `SCNNode.name` property for later retrieval

2. **Add metadata to FileNode**:
   - Store file name and path in node's `name` property (format: "name|path")

3. **Add hover tracking to ContentView**:
   - Use `onContinuousHover` modifier on SceneView
   - Track mouse position in view coordinates

4. **Implement hit-testing in TerrainScene**:
   - Add method `nodeAt(point:) -> (name: String, path: String)?`
   - Use `SCNView.hitTest(_:options:)` with appropriate options

5. **Create TooltipView**:
   ```swift
   struct TooltipView: View {
       let name: String
       let path: String

       var body: some View {
           VStack(alignment: .leading, spacing: 2) {
               Text(name)
                   .font(.system(size: 14, weight: .bold))
                   .foregroundColor(.white)
               Text(path)
                   .font(.system(size: 10))
                   .foregroundColor(.gray)
           }
           .padding(8)
           .background(Color.black.opacity(0.85))
           .cornerRadius(6)
       }
   }
   ```

6. **Position tooltip near cursor**:
   - Offset tooltip 15px right and 15px below cursor
   - Clamp to viewport bounds

## Verification
1. `swift build` succeeds
2. Run client with terrain loaded
3. Scene should have NO visible text labels
4. Hover over folder pyramid → tooltip appears with folder name + path
5. Hover over file cube → tooltip appears with file name + path
6. Move mouse away → tooltip disappears
7. Move mouse quickly between items → tooltips update smoothly
