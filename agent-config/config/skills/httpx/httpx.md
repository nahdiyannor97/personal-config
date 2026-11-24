#### HTTPX Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Next-generation Python HTTP client offering unified synchronous (`Client`) and asynchronous (`AsyncClient`) APIs, HTTP/1.1 & HTTP/2, connection pooling, custom transports (ASGI/WSGI testing), and strict timeouts.
* **Core Dependencies:** `httpcore`, `h11`, `certifi`, `idna`, `sniffio`.
* **Runtime & Environment Requirements:** Python 3.9+; `httpx==0.28.x` (`pip install httpx`, `pip install "httpx[http2]"` for HTTP/2 via `h2`, `pip install "httpx[brotli,zstd]"` for compression decoders, `pip install "httpx[socks]"` for SOCKS proxies).

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Use context-managed `httpx.Client(...)` (sync) or `async with httpx.AsyncClient(...)` (async) for connection pooling, HTTP/2 multiplexing, and deterministic socket cleanup.
  * Configure explicit `base_url`, `timeout=httpx.Timeout(...)`, and `limits=httpx.Limits(...)` at client instantiation.
  * Pass `follow_redirects=True` explicitly when HTTP 3xx redirects should be followed (HTTPX defaults to `follow_redirects=False`).
  * Use `client.stream("GET", url)` context manager when downloading large payloads and iterate via `response.iter_bytes()` / `await response.aiter_bytes()`.
  * Enable HTTP/2 explicitly via `httpx.AsyncClient(http2=True)` (requires `httpx[http2]`).
  * Call `response.raise_for_status()` immediately before accessing `response.json()` or `response.text`.
* **DON'T:**
  * Do not use top-level `httpx.get(...)` / `httpx.post(...)` inside loops or production services (creates and tears down a new connection pool per request).
  * Do not use synchronous `httpx.Client` inside an `async def` coroutine; use `httpx.AsyncClient`.
  * Do not disable SSL verification (`verify=False`) in production code; pass a custom `ssl.SSLContext` if private CAs are required.
  * Do not access `response.content` or `response.text` inside a `client.stream(...)` block without calling `response.read()` / `await response.aread()` first.

##### 3. Core API Signatures & Cheatsheet
* `httpx.Client(*, base_url: str = "", headers: HeaderTypes | None = None, auth: AuthTypes | None = None, timeout: TimeoutTypes = 5.0, limits: Limits = DEFAULT_LIMITS, http2: bool = False, follow_redirects: bool = False, transport: BaseTransport | None = None)`
* `httpx.AsyncClient(...)` (identical parameters; async methods `get`, `post`, `put`, `patch`, `delete`, `request`, `stream`, `aclose`)
* `httpx.Timeout(timeout: float | None, *, connect: float | None = ..., read: float | None = ..., write: float | None = ..., pool: float | None = ...)`
* `Response`: `.status_code: int`, `.headers: Headers`, `.text: str`, `.content: bytes`, `.json() -> Any`, `.raise_for_status() -> Response`, `.is_success: bool`

```python
import httpx

timeout = httpx.Timeout(10.0, connect=3.0)
limits = httpx.Limits(max_connections=100, max_keepalive_connections=20)

with httpx.Client(
    base_url="https://api.example.com/v1",
    timeout=timeout,
    limits=limits,
    follow_redirects=True,
) as client:
    resp = client.get("/users", params={"active": "true"})
    resp.raise_for_status()
    users = resp.json()
```

```python
import asyncio
import httpx

async def fetch_all(urls: list[str]) -> list[dict]:
    async with httpx.AsyncClient(http2=True, timeout=10.0) as client:
        tasks = [client.get(u) for u in urls]
        responses = await asyncio.gather(*tasks)
        results = []
        for r in responses:
            r.raise_for_status()
            results.append(r.json())
        return results
```

##### 4. Error Handling & Edge Cases
* **Official Exception Hierarchy:**
  * Base class: `httpx.HTTPError`
    * `httpx.RequestError` (network/transport failures): `TimeoutException` (`ConnectTimeout`, `ReadTimeout`, `WriteTimeout`, `PoolTimeout`), `NetworkError` (`ConnectError`, `ReadError`), `ProtocolError`, `TooManyRedirects`.
    * `httpx.HTTPStatusError` (raised by `response.raise_for_status()` on 4xx/5xx; inspect `exc.response.status_code` and `exc.response.text`).
* **Edge Cases & Gotchas:**
  * **Redirects Off by Default:** Unlike `requests`, HTTPX returns `301`/`302`/`307` responses directly unless `follow_redirects=True` is set.
  * **Default 5-Second Timeout:** Every operation defaults to a 5.0-second timeout; slow LLM or report endpoints will raise `httpx.ReadTimeout` unless `timeout` is raised explicitly.
  * **Proxy Env Vars:** HTTPX automatically honors `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, and `NO_PROXY` unless `trust_env=False` is configured.

---
