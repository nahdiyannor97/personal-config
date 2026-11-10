---
name: fast-api
description: API routing, ASGI configuration, dependency injection, lifespan management, and HTTP transport layer handling using FastAPI. Use when creating or refactoring endpoints, middleware, routers, auth guards, or error handling.
---

# FastAPI Engineering Skill

This skill governs the transport and routing layer using FastAPI. It focuses strictly on request lifecycle management, routing modularity, dependency injection, and ASGI performance standards.

---

## 1. Scope & Core Responsibilities

- **Routing & Architecture:** Modular `APIRouter` structure, API versioning, and clean route aggregation.
- **Dependency Injection:** Database session scoping via `Depends()`, authentication guards, and reusable security schemes.
- **Application Lifespan:** Modern `lifespan` context managers for startup/shutdown events, connection pools, and client sessions.
- **Execution Model:** Strict adherence to concurrency rules—use `async def` only for awaited I/O operations; use `def` for synchronous or thread-pooled operations.
- **Error Handling:** Centralized exception handlers and proper HTTP status code mapping (`HTTPException`).

---

## 2. Implementation Rules

1. **Clean Route Layering:** Keep business logic out of route handlers. Endpoints must only parse requests, invoke dependencies/services, and return response schemas.
2. **Session Lifecycle:** Always manage database sessions and HTTP client sessions through dependencies yielding the resource, ensuring clean teardown.
3. **No Direct Table Exposure:** Enforce response typing using contract schemas (`response_model`) instead of returning raw internal database models.
