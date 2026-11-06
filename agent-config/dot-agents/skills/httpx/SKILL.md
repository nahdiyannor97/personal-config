---
name: httpx
description: High-performance synchronous and asynchronous HTTP client operations, HTTP/2 support, in-process ASGI testing, and granular timeout policies using HTTPX. Use when building async REST integrations, calling APIs inside FastAPI, or running ASGI test harnesses.
---

# HTTPX Engineering Skill

This skill governs modern synchronous and asynchronous HTTP transport using HTTPX[cite: 3]. It covers async event loop compatibility, HTTP/2 protocol support, and ASGI test harness integrations[cite: 3].

**Reference Document:** `httpx.md`[cite: 3]

---

## 1. Scope & Core Capabilities

- **Async Event Loop Execution:** Native non-blocking I/O using `httpx.AsyncClient` inside `asyncio` routines[cite: 3].
- **Modern Protocol Standards:** HTTP/2 support, custom transport adapters, and granular timeout management (`httpx.Timeout`)[cite: 3].
- **In-Process ASGI Testing:** Direct in-memory testing of FastAPI/Starlette apps via `httpx.ASGITransport` without opening network sockets[cite: 3].
- **Keep-Alive Pooling:** Connection reuse and clean socket teardown using context managers (`with` / `async with`)[cite: 3].

---

## 2. Core Implementation Directives

1. **Context Manager Lifecycle:** Always manage client instances within `async with httpx.AsyncClient(...)` blocks to ensure proper connection disposal[cite: 3].
2. **Awareness of Default Timeouts:** Account for the default 5.0-second timeout threshold by setting explicit `httpx.Timeout` configs when operations exceed default latencies[cite: 3].
3. **Strict Concurrency Alignment:** Keep async calls strictly paired with `AsyncClient` and sync calls with `Client`; never mix or cross execution models[cite: 3].
4. **Explicit Status Assertion:** Use `response.raise_for_status()` and handle `httpx.HTTPStatusError` and `httpx.RequestError` specifically[cite: 3].
