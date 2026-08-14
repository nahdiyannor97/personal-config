---
name: python-backend
description: End-to-end Python backend architecture using FastAPI, Pydantic, and SQLModel. Use when building or refactoring REST APIs, request/response validation schemas, database models, CRUD operations, or backend lifespan services.
---

# Python Backend Engineering Skill

This skill orchestrates modern Python backend development using FastAPI, Pydantic, and SQLModel. To conserve context window and optimize token usage, **do not load all documentation files simultaneously**. Identify the specific domain of your task and read only the required `.md` module using file reading tools.

---

## 1. Domain Detection & Module Routing

Evaluate the task scope and load the corresponding documentation file before writing code:

| Task / Domain Scope | Module to Read | Key Responsibilities |
| :--- | :--- | :--- |
| **API Endpoints & Routing** | `fast-api.md` | `APIRouter` structure, ASGI/Uvicorn, dependency injection (`Depends`), lifespan managers, async/sync rules, and HTTP exception handling. |
| **Data Validation & Schemas** | `pydantic.md` | `BaseModel` definitions, type-safe data parsing, field validation rules, Pydantic AI agent integration, and Logfire observability. |
| **Database & ORM Layer** | `sql-model.md` | `SQLModel` table definitions (`table=True`), relationship mapping (`Relationship`), database sessions, CRUD queries, and engine setup. |

---

## 2. Integrated Feature Workflow

When developing complete backend features spanning all three layers, proceed in the following order:

1. **Database Modeling:** Read `sql-model.md` to define relational tables, foreign keys, and indexes.
2. **Schema & Validation:** Read `pydantic.md` to establish request contracts, data sanitization, or DTO response models.
3. **Endpoint Implementation:** Read `fast-api.md` to wire up route handlers, dependency-injected database sessions, status codes, and error responses.

---

## 3. Strict Development Directives

* **Lazy Loading:** Never read all module files upfront; load each file strictly when its specific layer is being developed or modified.
* **Sync vs Async Awareness:** Use `async def` only when awaiting asynchronous calls; use standard `def` for synchronous/blocking operations.
* **No Direct DB Exposure:** Prevent returning raw internal table structures directly to client endpoints without response schema validation.
