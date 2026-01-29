# Swift: Folder Hierarchy Highlighting

## Summary
Add highlight state to folder/file nodes, track hierarchy relationships, and propagate highlights on hover to show folder + all descendants.

## Changes Required

### 1. Model Updates (`Sources/Models/Messages.swift`)

Add new fields to `FolderInfo`:
```swift
struct FolderInfo: Codable {
    // ... existing fields ...
    let totalContents: Int      // NEW
    let parentPath: String?     // NEW

    enum CodingKeys: String, CodingKey {
        // ... existing ...
        case totalContents = "total_contents"
        case parentPath = "parent_path"
    }
}
```

### 2. FolderNode Highlight State (`Sources/Scene/Nodes/FolderNode.swift`)

Add highlight capability:
```swift
class FolderNode: SCNNode {
    private var defaultMaterial: SCNMaterial!
    private var highlightedMaterial: SCNMaterial!

    private func setupMaterials() {
        // Default material (existing)
        defaultMaterial = pyramidNode.geometry?.firstMaterial ?? SCNMaterial()

        // Highlighted material (brighter emission)
        highlightedMaterial = SCNMaterial()
        highlightedMaterial.diffuse.contents = NSColor(red: 0.2, green: 0.8, blue: 0.4, alpha: 0.9)
        highlightedMaterial.emission.contents = NSColor(red: 0.3, green: 1.0, blue: 0.6, alpha: 0.8)
        highlightedMaterial.transparency = 0.85
    }

    func setHighlighted(_ highlighted: Bool, animated: Bool = true) {
        let target = highlighted ? highlightedMaterial : defaultMaterial
        if animated {
            SCNTransaction.begin()
            SCNTransaction.animationDuration = 0.15
            pyramidNode.geometry?.materials = [target]
            SCNTransaction.commit()
        } else {
            pyramidNode.geometry?.materials = [target]
        }
    }
}
```

### 3. FileNode Highlight State (`Sources/Scene/Nodes/FileNode.swift`)

Same pattern as FolderNode:
```swift
class FileNode: SCNNode {
    private var defaultMaterial: SCNMaterial!
    private var highlightedMaterial: SCNMaterial!

    func setHighlighted(_ highlighted: Bool, animated: Bool = true) {
        // Similar implementation with brighter emission
    }
}
```

### 4. Hierarchy Tracking (`Sources/Scene/TerrainScene.swift`)

Add data structures:
```swift
class TerrainScene: SCNScene {
    // Hierarchy tracking
    private var folderChildren: [String: Set<String>] = [:]  // parent path -> child folder paths
    private var folderFiles: [String: Set<String>] = [:]     // folder path -> file paths
    private var currentlyHighlighted: Set<String> = []

    // Build hierarchy when terrain loads
    func buildHierarchy(from folders: [FolderInfo], files: [FileInfo]) {
        folderChildren.removeAll()
        folderFiles.removeAll()

        for folder in folders {
            if let parent = folder.parentPath {
                folderChildren[parent, default: []].insert(folder.path)
            }
        }

        for file in files {
            folderFiles[file.folderPath, default: []].insert(file.path)
        }
    }

    // Get all descendants of a folder
    func getAllDescendants(of folderPath: String) -> (folders: Set<String>, files: Set<String>) {
        var resultFolders: Set<String> = []
        var resultFiles: Set<String> = []
        var queue = [folderPath]

        while !queue.isEmpty {
            let current = queue.removeFirst()

            // Add direct files
            if let files = folderFiles[current] {
                resultFiles.formUnion(files)
            }

            // Add child folders and queue them
            if let children = folderChildren[current] {
                resultFolders.formUnion(children)
                queue.append(contentsOf: children)
            }
        }

        return (resultFolders, resultFiles)
    }

    // Highlight a folder and all descendants
    func highlightHierarchy(folderPath: String) {
        // Clear previous highlights
        clearAllHighlights()

        // Get all descendants
        let (descendantFolders, descendantFiles) = getAllDescendants(of: folderPath)

        // Highlight the hovered folder
        if let node = folderNodes[folderPath] {
            node.setHighlighted(true)
            currentlyHighlighted.insert(folderPath)
        }

        // Highlight descendant folders
        for path in descendantFolders {
            if let node = folderNodes[path] {
                node.setHighlighted(true)
                currentlyHighlighted.insert(path)
            }
        }

        // Highlight descendant files
        for path in descendantFiles {
            if let node = fileNodes[path] {
                node.setHighlighted(true)
                currentlyHighlighted.insert(path)
            }
        }
    }

    func clearAllHighlights() {
        for path in currentlyHighlighted {
            if let folderNode = folderNodes[path] {
                folderNode.setHighlighted(false)
            }
            if let fileNode = fileNodes[path] {
                fileNode.setHighlighted(false)
            }
        }
        currentlyHighlighted.removeAll()
    }
}
```

### 5. Update nodeInfo to Return isFolder (`Sources/Scene/TerrainScene.swift`)

```swift
func nodeInfo(at point: CGPoint, in view: SCNView) -> (name: String, path: String, isFolder: Bool)? {
    let hitResults = view.hitTest(point, options: [:])
    for result in hitResults {
        var node: SCNNode? = result.node
        while let current = node {
            if let folderNode = current as? FolderNode {
                return (folderNode.folderName, folderNode.folderPath, true)
            }
            if let fileNode = current as? FileNode {
                return (fileNode.fileName, fileNode.filePath, false)
            }
            node = current.parent
        }
    }
    return nil
}
```

### 6. Hover Event Integration (`Sources/ContentView.swift`)

Update `HoverTrackingSCNView`:
```swift
class HoverTrackingSCNView: SCNView {
    var onHover: ((String?, String?, Bool) -> Void)?  // name, path, isFolder
    private var lastHoveredPath: String?

    override func mouseMoved(with event: NSEvent) {
        let point = convert(event.locationInWindow, from: nil)

        if let scene = scene as? TerrainScene,
           let info = scene.nodeInfo(at: point, in: self) {

            // Only update highlights if path changed
            if info.path != lastHoveredPath {
                lastHoveredPath = info.path

                if info.isFolder {
                    scene.highlightHierarchy(folderPath: info.path)
                } else {
                    scene.clearAllHighlights()
                }
            }

            onHover?(info.name, info.path, info.isFolder)
        } else {
            if lastHoveredPath != nil {
                lastHoveredPath = nil
                if let scene = scene as? TerrainScene {
                    scene.clearAllHighlights()
                }
            }
            onHover?(nil, nil, false)
        }
    }

    override func mouseExited(with event: NSEvent) {
        lastHoveredPath = nil
        if let scene = scene as? TerrainScene {
            scene.clearAllHighlights()
        }
        onHover?(nil, nil, false)
    }
}
```

## Data Flow

```
Mouse Move → HoverTrackingSCNView.mouseMoved()
           → TerrainScene.nodeInfo(at:in:) → (name, path, isFolder)
           → [if folder changed] TerrainScene.highlightHierarchy()
           → getAllDescendants() traverses hierarchy
           → FolderNode/FileNode.setHighlighted(true) with animation
```

## Testing

1. Build and run: `swift run AgentariumClient`
2. Scan a directory with nested folders
3. Verify:
   - Hover over root folder → all nodes glow
   - Hover over nested folder → only that subtree glows
   - Hover over file → no highlight (files don't have children)
   - Move mouse away → highlights clear with animation

## Git Workflow

```bash
cd apps/macos-client
git checkout -b feature/folder-hierarchy-highlight
# ... implement ...
git add .
git commit -m "feat: Add folder hierarchy highlighting on hover"
git push -u origin feature/folder-hierarchy-highlight
gh pr create --title "feat: Add folder hierarchy highlighting on hover" --body "- Add totalContents and parentPath to FolderInfo model
- Add highlight state to FolderNode and FileNode
- Track folder hierarchy for descendant lookup
- Propagate highlights on folder hover"
```
