# HTTPX Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: HTTPX is a fully featured, next-generation HTTP client library for Python 3. It provides broadly `requests`-compatible synchronous and asynchronous APIs, with native support for HTTP/1.1, HTTP/2, WSGI/ASGI application testing, and strict connection pooling timeouts.
- **Core Use Cases**:
  - Asynchronous HTTP requests using `asyncio` or `trio` loops.
  - High-performance HTTP/2 client-server communication.
  - In-process testing of FastAPI, Starlette, or Django ASGI/WSGI applications without running network sockets.
  - Enforced timeout policies and persistent client session management.
- **Runtime / Environment Requirements**:
  - Python 3.9+.
  - Optional dependencies: `httpx[http2]` (for HTTP/2 protocol support via `h2`), `httpx[brotli,zstd]` (for compressed response decoding).

---

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Use Context Managers for Clients**: Always instantiate `httpx.Client()` or `httpx.AsyncClient()` inside `with` or `async with` context managers to guarantee socket cleanup and connection pool disposal.
- **Set Explicit Timeouts**: Always specify explicit timeout values or a global `httpx.Timeout` configuration (default overall timeout is 5.0s, unlike `requests` which defaults to infinite).
- **Use `AsyncClient` for Asynchronous IO**: Pair `httpx.AsyncClient` with `await client.get()` inside `async def` routines to maximize event loop throughput.
- **Reuse Client Instances**: Keep client instances active for sequential requests to benefit from keep-alive connection pooling.
- **Call `response.raise_for_status()`**: Explicitly invoke `response.raise_for_status()` or check `response.is_error` / `response.is_success` to catch HTTP `4xx` and `5xx` status codes.

### DON'T:
- **Do Not Instantiate One-Off Clients in Loops**: Avoid repeatedly creating temporary `httpx.Client()` or `httpx.AsyncClient()` instances inside loops; this degrades throughput and exhausts ephemeral sockets.
- **Do Not Assume Requests Exception Hierarchy**: Do not catch `requests.exceptions.RequestException`; catch `httpx.HTTPError`, `httpx.RequestError`, or `httpx.HTTPStatusError`.
- **Do Not Mix Sync and Async Client Methods**: Do not call blocking sync `client.get()` methods inside async functions or await `async_client.get()` in non-async routines.
- **Do Not Omit `http2=True` Flag for HTTP/2**: HTTP/2 support requires installing `httpx[http2]` and setting `httpx.AsyncClient(http2=True)`.

---

## 3. Core API Signatures & Cheatsheet

### Critical Classes & Functions
- **`httpx.Client(base_url="", headers=None, timeout=5.0, ...)`**: Synchronous HTTP client instance.
- **`httpx.AsyncClient(base_url="", headers=None, timeout=5.0, http2=False, ...)`**: Asynchronous HTTP client instance.
- **`httpx.Timeout(timeout=None, connect=5.0, read=5.0, write=5.0, pool=5.0)`**: Granular timeout control configuration.
- **`response.raise_for_status()`**: Raises `httpx.HTTPStatusError` if response status code is 4xx or 5xx.

### Idiomatic Code Snippets

#### 1. Asynchronous Client with Timeouts and JSON Parsing
```python
import asyncio
import httpx

async def fetch_user_data(user_id: int) -> dict:
    timeout_config = httpx.Timeout(10.0, connect=3.0)
    async with httpx.AsyncClient(
        base_url="https://api.example.com",
        timeout=timeout_config,
        headers={"Accept": "application/json"}
    ) as client:
        response = await client.get(f"/users/{user_id}")
        response.raise_for_status()
        return response.json()
```

#### 2. In-Process ASGI Application Testing (FastAPI / Starlette)
```python
import httpx

async def test_health_check(app):
    # Makes direct in-process calls to ASGI app without network overhead
    async with httpx.AsyncClient(transport=httpx.ASGITransport(app=app), base_url="http://testserver") as client:
        response = await client.get("/health")
        assert response.status_code == 200
        assert response.json() == {"status": "ok"}
```

---

## 4. Error Handling & Edge Cases

### Exception Hierarchy & Handling
- **`httpx.HTTPError`**: Base exception class for all HTTPX errors.
  - **`httpx.RequestError`**: Base class for transport level failures (network connection drops, DNS lookup failures, timeouts).
    - **`httpx.ConnectTimeout`**, **`httpx.ReadTimeout`**: Raised when specific transport timeout thresholds expire.
  - **`httpx.HTTPStatusError`**: Raised by `response.raise_for_status()` when server responds with 4xx or 5xx HTTP errors.
```python
import httpx

try:
    with httpx.Client(timeout=3.0) as client:
        res = client.get("https://api.example.com/data")
        res.raise_for_status()
except httpx.HTTPStatusError as exc:
    print(f"HTTP Error {exc.response.status_code} while requesting {exc.request.url}")
except httpx.RequestError as exc:
    print(f"Network error while requesting {exc.request.url}: {exc}")
```

### Known Limitations & Gotchas
- **Default Timeout Thresholds**: Unlike `requests` (which defaults to no timeout), HTTPX defaults to a **5.0-second timeout**. Request calls will fail if network latency exceeds 5 seconds unless overridden explicitly (`timeout=None` or custom `httpx.Timeout`).
- **Response Content Consumption in Streaming**: When using `client.stream()`, response bytes are consumed incrementally. Accessing `response.text` or `response.json()` outside or inside the stream block without loading content will throw a `ResponseNotRead` error.
