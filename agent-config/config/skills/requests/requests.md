#### Requests Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Mature, synchronous Python HTTP/1.1 client library for human-friendly REST consumption, sessions, cookies, streaming downloads, and multipart uploads.
* **Runtime & Environment Requirements:** Python 3.8+; `requests==2.34.x` (`pip install requests`), backed by `urllib3`, `certifi`, `charset-normalizer`, and `idna`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Always pass an explicit `timeout` argument on every request—prefer a `(connect_timeout, read_timeout)` tuple (e.g., `timeout=(3.05, 10)`).
  * Use a context-managed `requests.Session()` for multiple requests to the same host to reuse TCP connections (HTTP Keep-Alive), persist headers/auth/cookies, and mount `HTTPAdapter(max_retries=Retry(...))`.
  * Pass JSON payloads via `json=payload` (sets `Content-Type: application/json` automatically) and query parameters via `params={...}`.
  * Call `response.raise_for_status()` prior to decoding `response.json()`.
  * Set `stream=True` inside a `with session.get(url, stream=True, timeout=...) as r:` block for large file downloads, reading via `r.iter_content(chunk_size=8192)`.
* **DON'T:**
  * Never call `requests.get/post/put/delete` without `timeout=...`; Requests defaults to `timeout=None`, which can hang worker threads indefinitely.
  * Do not use `requests` inside `async def` functions (e.g., FastAPI async endpoints); it performs blocking I/O and stalls the event loop—use `httpx.AsyncClient` instead.
  * Do not set `verify=False` in production; pass a path to a CA bundle (`verify="/path/to/ca.pem"`) when using internal PKI.
  * Do not leave `stream=True` responses unclosed; unclosed streams leak pooled connections back into `urllib3`.

##### 3. Core API Signatures & Cheatsheet
* `requests.request(method: str, url: str, *, params=None, data=None, json=None, headers=None, cookies=None, files=None, auth=None, timeout: float | tuple[float, float] | None = None, allow_redirects: bool = True, verify: bool | str = True, stream: bool = False) -> Response`
* `requests.Session()` -> `.headers.update(...)`, `.mount(prefix: str, adapter: HTTPAdapter)`, `.get/post/put/patch/delete(...)`, `.close()`
* `Response`: `.status_code: int`, `.ok: bool`, `.headers: CaseInsensitiveDict`, `.encoding: str | None`, `.text: str`, `.content: bytes`, `.json(**kwargs) -> Any`, `.raise_for_status() -> None`

```python
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

retry_strategy = Retry(
    total=3,
    backoff_factor=0.5,
    status_forcelist=(429, 500, 502, 503, 504),
    allowed_methods=("GET", "HEAD", "OPTIONS"),
)

with requests.Session() as session:
    adapter = HTTPAdapter(max_retries=retry_strategy, pool_connections=10, pool_maxsize=20)
    session.mount("https://", adapter)
    session.headers.update({"Accept": "application/json", "User-Agent": "my-service/1.0"})

    resp = session.get("https://api.github.com/user", auth=("user", "token"), timeout=(3.05, 10.0))
    resp.raise_for_status()
    profile = resp.json()
```

```python
import requests

with requests.get("https://example.com/archive.tar.gz", stream=True, timeout=(5, 30)) as r:
    r.raise_for_status()
    with open("archive.tar.gz", "wb") as f:
        for chunk in r.iter_content(chunk_size=65536):
            if chunk:
                f.write(chunk)
```

##### 4. Error Handling & Edge Cases
* **Official Exception Hierarchy:**
  * Base class: `requests.exceptions.RequestException`
    * `requests.exceptions.HTTPError` (from `raise_for_status()`)
    * `requests.exceptions.ConnectionError` (`ProxyError`, `SSLError`)
    * `requests.exceptions.Timeout` (`ConnectTimeout`, `ReadTimeout`)
    * `requests.exceptions.TooManyRedirects`
    * `requests.exceptions.JSONDecodeError` (subclass of `RequestException` and `ValueError`, raised by `r.json()` on non-JSON bodies)
* **Edge Cases & Gotchas:**
  * **Timeout Semantics:** `timeout` is *not* a total response download timeout; it specifies the maximum idle wait between bytes received from the server.
  * **Status Code Truthiness:** `bool(response)` is `True` for status codes `< 400` and `False` for `>= 400`; always check `response.raise_for_status()` or `response.status_code` explicitly instead of truthiness.
  * **Encoding Fallback:** When `Content-Type` is `text/*` without a `charset`, HTTP/1.1 defaults `r.encoding` to `ISO-8859-1`; override `r.encoding = "utf-8"` or inspect `r.apparent_encoding` when consuming UTF-8 text lacking charset headers.

---
