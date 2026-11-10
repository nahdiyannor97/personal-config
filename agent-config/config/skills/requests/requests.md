# Requests Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Requests is an elegant, simple, synchronous HTTP library for Python built on top of `urllib3`. It abstracts complex HTTP/1.1 networking tasks, header management, SSL verification, cookie persistence, and authentication into a human-friendly API.
- **Core Use Cases**:
  - Synchronous web scraping, REST API integrations, and web service client implementations.
  - Session-managed HTTP workflows requiring automatic cookie handling and connection pooling.
  - Form-encoded and multipart file upload handling.
- **Runtime / Environment Requirements**:
  - Python 3.10+ (as of Requests v2.34+).
  - Synchronous IO execution model (blocking calls).

---

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Always Pass Explicit Timeouts**: Always pass a tuple or scalar `timeout` argument (e.g., `timeout=(3.05, 10.0)` for connect/read) to every network call.
- **Use `requests.Session` for Multiple Requests**: Reuse TCP connections and persist cookies across calls by wrapping operations inside `with requests.Session() as session:`.
- **Use `response.raise_for_status()`**: Always trigger `response.raise_for_status()` after requests to convert `4xx` and `5xx` error responses into catchable `HTTPError` exceptions.
- **Use `json=` Parameter for JSON Payloads**: Pass dictionary payloads to the `json=` parameter to automatically handle `Content-Type: application/json` headers and serialization.
- **Use Streamed Downloads for Large Files**: Pass `stream=True` on `requests.get()` and read chunks using `response.iter_content()` to handle large payload downloads without loading entire files into RAM.

### DON'T:
- **Do Not Omit Timeouts**: Never call `requests.get()` or `requests.post()` without specifying a `timeout`; unhandled dropped connections can cause application processes to hang indefinitely.
- **Do Not Use Requests inside Async Event Loops**: Avoid making blocking `requests` calls inside `asyncio` event loops (e.g., FastAPI async endpoints); use threadpool execution (`asyncio.to_thread`) or async-native HTTP libraries like HTTPX instead.
- **Do Not Disable SSL Verification in Production**: Do not pass `verify=False` in production code to bypass SSL errors; fix root CA bundle configurations or supply custom certificates via `verify="/path/to/cert.pem"`.
- **Do Not Manually Encode Form or JSON Data**: Avoid manually calling `json.dumps()` and setting headers manually when `json=` or `data=` handles content encoding automatically.

---

## 3. Core API Signatures & Cheatsheet

### Critical Core Functions & Methods
- **`requests.get(url, params=None, headers=None, timeout=None, ...)`**: Performs synchronous HTTP GET request.
- **`requests.post(url, data=None, json=None, headers=None, timeout=None, ...)`**: Performs HTTP POST request.
- **`requests.Session()`**: Instantiates a persistent HTTP session object supporting header inheritance and connection pooling.
- **`response.raise_for_status()`**: Raises `requests.exceptions.HTTPError` for bad response statuses.

### Idiomatic Code Snippets

#### 1. Robust API Request with Session and Timeouts
```python
import requests
from requests.exceptions import HTTPError, RequestException

BASE_URL = "https://api.example.com/v1"

def fetch_resource(endpoint: str, api_key: str) -> dict:
    timeout_config = (3.05, 10.0)
    
    with requests.Session() as session:
        session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Accept": "application/json"
        })
        try:
            response = session.get(f"{BASE_URL}/{endpoint}", timeout=timeout_config)
            response.raise_for_status()
            return response.json()
        except HTTPError as http_err:
            print(f"HTTP error occurred: {http_err} - Status: {response.status_code}")
            raise
        except RequestException as req_err:
            print(f"Network failure: {req_err}")
            raise
```

#### 2. Streaming File Download to Disk
```python
import requests

def download_large_file(url: str, destination_path: str):
    with requests.get(url, stream=True, timeout=(5.0, 30.0)) as response:
        response.raise_for_status()
        with open(destination_path, "wb") as f:
            for chunk in response.iter_content(chunk_size=8192):
                if chunk:
                    f.write(chunk)
```

---

## 4. Error Handling & Edge Cases

### Official Exception Hierarchy
- **`requests.exceptions.RequestException`**: Base exception class.
  - **`requests.exceptions.HTTPError`**: Raised by `raise_for_status()` on HTTP `4xx` or `5xx` responses.
  - **`requests.exceptions.ConnectionError`**: Raised during underlying socket connectivity issues (DNS failures, connection refused).
  - **`requests.exceptions.Timeout`**: Base class for network timeouts.
    - **`requests.exceptions.ConnectTimeout`**: Timed out while establishing socket connection.
    - **`requests.exceptions.ReadTimeout`**: Socket connected, but server failed to send data within read interval.
  - **`requests.exceptions.TooManyRedirects`**: Exceeded maximum allowed redirect hops.

### Known Limitations & Gotchas
- **No Native Async Support**: Requests is strictly synchronous and blocking. Calling it within an async event loop halts execution across all concurrent tasks on that thread.
- **Encoding Guessing Overhead**: Accessing `response.text` triggers `chardet`/`charset_normalizer` byte analysis if HTTP headers omit explicit encoding charsets. For guaranteed JSON parsing, use `response.json()` directly or set `response.encoding = 'utf-8'` explicitly before accessing `response.text`.
