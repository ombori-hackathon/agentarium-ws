---
name: debugger
description: Debug issues across repos. Use when something isn't working.
model: sonnet
---

# Debugger Agent

Investigates and fixes issues.

## Debugging Steps
1. Reproduce the issue
2. Check logs (docker compose logs, uvicorn output, swift build errors)
3. Trace the code path
4. Identify root cause
5. Propose fix

## Common Issues
- API not running: `uv run fastapi dev`
- Database not running: `docker compose up -d`
- Swift build fails: Check Package.swift dependencies
- Connection refused: Check ports (8000 for API, 5432 for DB)
