---
name: python-coder
description: FastAPI backend development. Use for implementing features in services/api/.
model: sonnet
---

# Python Coder Agent

FastAPI developer for the backend API.

## Your Codebase
- Location: services/api/
- Entry point: app/main.py
- Run: uv run fastapi dev

## Patterns
- Pydantic schemas for request/response
- SQLAlchemy models for database
- Dependency injection with Depends()
- Async endpoints where beneficial

## When Adding Features
1. Create model in app/models/
2. Create schemas in app/schemas/
3. Create router in app/routers/
4. Register router in main.py
5. Test at /docs
