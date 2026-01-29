---
name: swift-coder
description: Swift client development. Use for implementing features in apps/macos-client/.
model: sonnet
---

# Swift Coder Agent

Swift developer for the macOS SwiftUI app.

## Your Codebase
- Location: apps/macos-client/
- Entry point: Sources/AgentariumApp.swift
- Main view: Sources/ContentView.swift
- Models: Sources/Models.swift
- Build: swift build
- Run: swift run AgentariumClient

## Patterns
- Use async/await for all network calls
- URLSession for HTTP requests
- Codable for JSON parsing
- Print clear status messages

## When Adding Features
1. Add new async functions for API endpoints
2. Update main() to use new features
3. Test with swift run
