#### FastAPI Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** High-performance, OpenAPI 3.1 / JSON Schema compliant Python web framework for REST APIs, Server-Sent Events (SSE), JSON Lines streaming, and WebSockets.
* **Core Dependencies:** Built on **Starlette** (ASGI web layer) and **Pydantic v2** (validation/serialization).
* **Runtime & Environment Requirements:** Python 3.9+ (3.11+ recommended); package installation via `uv add "fastapi[standard]"` (includes `uvicorn[standard]`, `fastapi-cli[standard]`, `httpx`/`httpx2`, `jinja2`, `python-multipart`, `email-validator`). Install without cloud CLI via `uv add "fastapi[standard-no-fastapi-cloud-cli]"`. Official AI coding skill available via `uvx library-skills`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Declare all path, query, header, cookie, form, and body parameters via standard Python type hints and `typing.Annotated` (e.g., `Annotated[int, Query(ge=1, le=100)]`, `Annotated[Session, Depends(get_session)]`).
  * Use `async def` for path operations performing non-blocking async I/O (`await`), and synchronous `def` for blocking CPU/I/O operations (FastAPI automatically runs `def` routes in an AnyIO threadpool).
  * Manage application startup/shutdown resources exclusively with an `asynccontextmanager` passed to `FastAPI(lifespan=lifespan)`.
  * Modularize routes using `APIRouter(prefix=..., tags=[...])` and include them via `app.include_router(...)`.
  * Define explicit response schemas using return type annotations (`-> ItemPublic`) or `response_model=...` to prevent leaking internal/secret fields.
  * Run locally with `fastapi dev main.py` and serve in production with `fastapi run main.py` or Uvicorn/Gunicorn workers; deploy optionally via `uv run fastapi deploy`.
* **DON'T:**
  * Do not execute blocking I/O (e.g., `requests.get`, synchronous SQL queries, `time.sleep`) inside an `async def` path operation; doing so blocks the entire ASGI event loop.
  * Do not use deprecated `@app.on_event("startup")` or `@app.on_event("shutdown")` decorators; use `lifespan`.
  * Do not pass `Depends(...)` as a default value when `Annotated[T, Depends(...)]` can be reused as a typed alias.
  * Do not return raw ORM objects containing sensitive fields without a filtering `response_model` or return type annotation.

##### 3. Core API Signatures & Cheatsheet
* `FastAPI(*, title: str = "FastAPI", version: str = "0.1.0", lifespan: Lifespan | None = None, dependencies: Sequence[Depends] | None = None)`
* `@app.get/post/put/patch/delete(path: str, *, response_model: Any = Default(None), status_code: int | None = None, dependencies: Sequence[Depends] | None = None)`
* `APIRouter(*, prefix: str = "", tags: list[str | Enum] | None = None, dependencies: Sequence[Depends] | None = None)`
* `Depends(dependency: Callable[..., Any] | None = None, *, use_cache: bool = True)`
* `HTTPException(status_code: int, detail: Any = None, headers: dict[str, str] | None = None)`
* `BackgroundTasks.add_task(func: Callable[P, Any], *args: P.args, **kwargs: P.kwargs) -> None`
* `TestClient(app: ASGIApp)` (from `fastapi.testclient`, backed by `httpx`/`httpx2`)

```python
from contextlib import asynccontextmanager
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, Query, status
from pydantic import BaseModel

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Initialize shared clients / connection pools
    yield
    # Dispose resources cleanly

app = FastAPI(title="Inventory API", lifespan=lifespan)

class ItemCreate(BaseModel):
    name: str
    price: float
    is_offer: bool | None = None

class ItemPublic(ItemCreate):
    id: int

def get_token(x_token: Annotated[str, Query(alias="token")] = "demo") -> str:
    if not x_token:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    return x_token

TokenDep = Annotated[str, Depends(get_token)]

@app.put("/items/{item_id}", response_model=ItemPublic)
async def update_item(item_id: int, item: ItemCreate, token: TokenDep) -> ItemPublic:
    return ItemPublic(id=item_id, **item.model_dump())
```

```python
from fastapi import APIRouter, BackgroundTasks
from fastapi.responses import StreamingResponse

router = APIRouter(prefix="/jobs", tags=["jobs"])

def write_audit_log(job_id: str) -> None:
    pass

@router.post("/{job_id}", status_code=202)
async def enqueue_job(job_id: str, background_tasks: BackgroundTasks) -> dict[str, str]:
    background_tasks.add_task(write_audit_log, job_id)
    return {"status": "accepted", "job_id": job_id}

async def jsonl_generator():
    for i in range(3):
        yield f'{{"event": {i}}}\n'

@router.get("/stream")
async def stream_events() -> StreamingResponse:
    return StreamingResponse(jsonl_generator(), media_type="application/x-ndjson")
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:**
  * Raise `fastapi.HTTPException` for client-facing HTTP errors and `fastapi.WebSocketException` for WebSocket errors.
  * Register global handlers via `@app.exception_handler(RequestValidationError)` or `@app.exception_handler(CustomError)` returning a `JSONResponse`.
* **Edge Cases & Gotchas:**
  * Path order matters: declare fixed paths (`/users/me`) *before* parameterized paths (`/users/{user_id}`) so `"me"` is not captured as `user_id`.
  * Dependencies using `yield` execute cleanup code *after* the response is generated; exceptions raised inside `yield` dependencies must be caught and re-raised carefully so `HTTPException` responses are not masked.
  * `Form(...)` and `UploadFile` require `python-multipart` to be installed.
  * When testing with `TestClient(app)`, wrap it in a `with TestClient(app) as client:` block so `lifespan` startup/shutdown events actually execute.

---
