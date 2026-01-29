# Feature: Static 3D World-Space Labels

## Summary
Labels should be anchored in 3D world space above folder/file nodes, not follow the mouse cursor. Labels appear on hover and disappear when hover stops.

## Trigger
Mouse hover over a folder or file node in the 3D scene.

## Expected Result
- Label appears floating above the hovered node in 3D space
- Label rotates to face camera (billboard constraint)
- Label stays fixed relative to the node, doesn't follow mouse
- Label disappears when mouse moves away from node

## Scope
**Swift client only** - no API changes needed.

---

## Technical Design

### Current Implementation (to replace)
- `ContentView.swift` tracks `mousePosition` and renders `TooltipView` SwiftUI overlay
- Tooltip follows cursor at screen coordinates + (15, 15) offset
- `TerrainScene.nodeInfo()` returns name/path for tooltip display

### New Implementation
Reuse the existing `LabelNode` class (used by `AgentNode` for path labels) to create 3D text labels attached to folder/file nodes.

---

## Files to Modify

| File | Changes |
|------|---------|
| `Sources/Scene/Nodes/LabelNode.swift` | Add `fontSize` parameter (default 12, use 11 for tooltips) |
| `Sources/Scene/Nodes/FolderNode.swift` | Add `labelNode` child, `showLabel()`/`hideLabel()` methods |
| `Sources/Scene/Nodes/FileNode.swift` | Add `labelNode` child, `showLabel()`/`hideLabel()` methods |
| `Sources/Scene/TerrainScene.swift` | Call `showLabel()`/`hideLabel()` on hover instead of returning info |
| `Sources/ContentView.swift` | Remove tooltip overlay, simplify hover handling |
| `Sources/TooltipView.swift` | Delete (no longer needed) |

---

## Implementation Plan

### Step 1: Update LabelNode for configurable font size

```swift
// LabelNode.swift - add fontSize parameter
init(text: String, yOffset: Float = 1.0, fontSize: CGFloat = 12) {
    // Use fontSize instead of hardcoded 12
}
```

### Step 2: Add Label to FolderNode

```swift
// FolderNode.swift
private var labelNode: LabelNode?

private func setupLabel() {
    let height = Float(folder.height ?? 3.0)
    labelNode = LabelNode(text: folderName, yOffset: height + 0.5, fontSize: 11)
    labelNode?.isHidden = true
    addChildNode(labelNode!)
}

func showLabel() {
    labelNode?.isHidden = false
}

func hideLabel() {
    labelNode?.isHidden = true
}
```

### Step 3: Add Label to FileNode

```swift
// FileNode.swift
private var labelNode: LabelNode?

private func setupLabel() {
    labelNode = LabelNode(text: fileName, yOffset: 0.8, fontSize: 11)
    labelNode?.isHidden = true
    addChildNode(labelNode!)
}

func showLabel() {
    labelNode?.isHidden = false
}

func hideLabel() {
    labelNode?.isHidden = true
}
```

### Step 4: Update TerrainScene Hover Handling

Add a method to show/hide labels on the hovered node:

```swift
// TerrainScene.swift
private var currentlyLabeledNode: SCNNode?

func showLabelForNode(at point: CGPoint, in view: SCNView) {
    let hitResults = view.hitTest(point, options: [:])
    var foundNode: SCNNode?

    for result in hitResults {
        var node: SCNNode? = result.node
        while let current = node {
            if current is FolderNode || current is FileNode {
                foundNode = current
                break
            }
            node = current.parent
        }
        if foundNode != nil { break }
    }

    // Hide previous label if different node
    if let prev = currentlyLabeledNode, prev != foundNode {
        (prev as? FolderNode)?.hideLabel()
        (prev as? FileNode)?.hideLabel()
    }

    // Show new label
    if let node = foundNode {
        (node as? FolderNode)?.showLabel()
        (node as? FileNode)?.showLabel()
        currentlyLabeledNode = node
    } else {
        currentlyLabeledNode = nil
    }
}

func hideAllLabels() {
    (currentlyLabeledNode as? FolderNode)?.hideLabel()
    (currentlyLabeledNode as? FileNode)?.hideLabel()
    currentlyLabeledNode = nil
}
```

### Step 5: Simplify ContentView

Remove from `ContentView`:
- `@State private var hoveredNode: (name: String, path: String)?`
- `@State private var mousePosition: CGPoint`
- `TooltipView` overlay in the ZStack
- Mouse position tracking

Update `HoverTrackingSCNView.mouseMoved()`:
- Call `scene.showLabelForNode(at:in:)` instead of `nodeInfo()`
- Keep highlight hierarchy logic

Update `HoverTrackingSCNView.mouseExited()`:
- Call `scene.hideAllLabels()`

### Step 6: Delete TooltipView.swift

Remove the file entirely - no longer needed.

---

## Label Positioning & Size

- **Folders**: Label at `Y = pyramid height + 0.5` (floats above peak)
- **Files**: Label at `Y = 0.8` (floats above cube)
- **Font size**: 11pt (smaller than agent labels to avoid clutter)

The existing `LabelNode` has a Y-axis billboard constraint that keeps text facing the camera.

---

## Verification

1. Build: `cd apps/macos-client && swift build`
2. Run: `swift run AgentariumClient`
3. Scan a directory
4. Test:
   - Hover over folder → label appears above pyramid in 3D space
   - Hover over file → label appears above cube in 3D space
   - Labels face camera as you rotate view
   - Move mouse away → label disappears
   - Labels do NOT follow mouse movement
   - Rotate camera → labels stay attached to nodes but face camera
   - Text size is readable but not too large (11pt)

---

## Git Workflow

```bash
cd apps/macos-client
git checkout -b feature/static-world-labels
# ... implement ...
git add .
git commit -m "feat: Replace mouse-following tooltips with 3D world-space labels"
git push -u origin feature/static-world-labels
gh pr create --title "feat: Static 3D world-space labels" --body "- Labels anchored in 3D world space above nodes
- Labels face camera with billboard constraint
- Labels appear on hover, disappear on exit
- Removed screen-space TooltipView
- Font size: 11pt for readability"
```
