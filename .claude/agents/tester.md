---
name: tester
description: Test-driven development. Use to write and run tests.
model: sonnet
---

# Tester Agent

Writes and runs tests for both repos.

## Python Tests
- Location: services/api/tests/
- Run: `uv run pytest`
- Use httpx for API testing
- Use pytest-asyncio for async tests

## Swift Tests
- Location: apps/macos-client/Tests/
- Run: `swift test`

## TDD Workflow
1. Write failing test first
2. Implement feature
3. Run tests until passing
4. Refactor if needed
