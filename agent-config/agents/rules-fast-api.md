# FastAPI Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: FastAPI is a modern, fast (high-performance), web framework for building APIs with Python based on standard Python type hints [3]. It leverages Starlette for web parts and Pydantic for data parts [3].
- **Core Use Cases**:
  - High-performance, production-ready RESTful APIs [1, 3].
  - Clean data validation and serialization/deserialization with automated OpenAPI (Swagger) documentation generation [3].
  - Real-time applications using WebSockets [3] and Server-Sent Events (SSE) [3].
  - Modular, structured backend backbones for larger applications using `APIRouter` [3].
- **Runtime & Environment Requirements**:
  - Python 3.8+ (utilizing modern type annotations) [3].
  - Runs on an ASGI (Asynchronous Server Gateway Interface) server, most commonly **Uvicorn** (configured with server workers for multi-core deployments) [3].
  - Optional: `python-multipart` is required when handling standard forms and file uploads [3].

## 2. Do's and Don'ts (Strict Rules)

### DO

- **Use Type Hints for Automatic Extraction**: Declare the types of all path, query, cookie, and header parameters [3]. FastAPI automatically handles validation and conversion [3].
- **Define Asynchronous / Synchronous Patterns Correctly**:
  - Use `async def` only when you are executing non-blocking, asynchronous operations using `await` inside the endpoint [3].
  - Use synchronous `def` for blocking operations (such as legacy database clients or synchronous I/O). FastAPI runs synchronous `def` path operations in an external threadpool to prevent blocking the main event loop [3].
- **Pydantic V2 Models**: Define strict request bodies and response schemas using Pydantic models. Always utilize the `response_model` argument in decorators or Python return type hints to automatically filter and validate outgoing data [3].
- **Use Dependency Injection (`Depends`)**: Write reusable components (e.g., database sessions, authentication, security policies) and inject them using `Depends()` or `Security()` [3].
- **Lifespan Manager**: Manage startup and shutdown operations (such as initializing database connections or client sessions) using the `@asynccontextmanager` lifespan event handler [3].
- **Modularize via APIRouter**: Split large codebases into sub-files using `APIRouter`, associating routing prefixes, tags, and dependencies cleanly [3].

### DON'T

- **Do Not Block the Event Loop**: Never call blocking, synchronous functions directly inside an `async def` path operation without running them in an executor or threadpool [3].
- **Do Not Use Deprecated Event Handlers**: Do not use the old, deprecated `@app.on_event("startup")` and `@app.on_event("shutdown")` decorators. Use the unified `lifespan` parameter in the `FastAPI` instance [3].
- **Do Not Directly Expose Database Models**: Do not return database models directly to users if they contain sensitive columns or unhashed credentials. Define distinct request/response schemas or use Pydantic models to restrict fields [3].
- **Avoid Manual JSON Conversions**: Do not use manual `json.dumps()` or custom encoders for database models; use FastAPI's built-in `jsonable_encoder()` to transform complex structures into standard JSON-compatible types [3].

## 3. Core API Signatures & Cheatsheet

### Critical Core Interfaces

- **FastAPI Application Initialization**:

  ```python
  from fastapi import FastAPI
  app = FastAPI(lifespan=lifespan_context)
  ```

- **APIRouter Initialization**:

  ```python
  from fastapi import APIRouter
  router = APIRouter(prefix="/items", tags=["items"])
  ```

- **Path and Query Param Declarations**:

  ```python
  from fastapi import Path, Query
  # Query(default, ...) and Path(default, ...) for validation rules
  ```

- **File Uploads**:

  ```python
  from fastapi import UploadFile, File
  # Use UploadFile to read files into memory or disk asynchronously
  ```

- **JSON Encoding Utility**:

  ```python
  from fastapi.encoders import jsonable_encoder
  # Returns a JSON-compatible Python structure (dict, list, etc.)
  ```

### Idiomatic Code Snippets

#### 1. Basic Endpoint with Validation and Pydantic Request/Response

```python
from typing import Annotated
from fastapi import FastAPI, Path, Query, status
from pydantic import BaseModel, Field

app = FastAPI()

class Item(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    price: float = Field(..., gt=0)
    tax: float | None = None

@app.post(
    "/items/{item_id}",
    response_model=Item,
    status_code=status.HTTP_201_CREATED,
    tags=["items"]
)
async def create_item(
    item_id: Annotated[int, Path(title="The ID of the item", ge=1)],
    item: Item,
    q: Annotated[str | None, Query(max_length=50)] = None
):
    # item_id and q are validated automatically
    # item is parsed and validated as an Item model instance
    return item
```

#### 2. Reusable Dependency Injection & Authentication

```python
from typing import Annotated
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer

app = FastAPI()

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

async def get_current_user(token: Annotated[str, Depends(oauth2_scheme)]):
    # Authenticate user and extract metadata
    if not token:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    return {"username": "current_user", "active": True}

@app.get("/users/me")
async def read_users_me(current_user: Annotated[dict, Depends(get_current_user)]):
    return current_user
```

#### 3. Complete Lifespan & APIRouter Implementation

```python
from contextlib import asynccontextmanager
from fastapi import APIRouter, FastAPI

# 1. Modular APIRouter
router = APIRouter(prefix="/products", tags=["products"])

@router.get("/")
async def list_products():
    return [{"id": 1, "name": "Screwdriver"}]

# 2. Modern Lifespan Event Handler
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Code runs on startup
    print("Database connection pool initialized")
    yield
    # Code runs on shutdown
    print("Database connection pool closed")

app = FastAPI(lifespan=lifespan)
app.include_router(router)
```

## 4. Error Handling & Edge Cases

### Exception & Error Handling Patterns

- **HTTPException**: Raise standard `HTTPException` with a `status_code` and a `detail` string or dictionary [3].
- **WebSocketException**: For real-time WebSockets, raise `WebSocketException` to cleanly close socket connections with code statuses [3].
- **Custom Error Handlers**: Bind custom exception classes to custom JSON response structures globally using `@app.exception_handler()` [3]:

  ```python
  from fastapi import Request
  from fastapi.responses import JSONResponse
  
  class CustomValidationException(Exception):
      def __init__(self, message: str):
          self.message = message

  @app.exception_handler(CustomValidationException)
  async def custom_exception_handler(request: Request, exc: CustomValidationException):
      return JSONResponse(
          status_code=400,
          content={"error": "CustomValidationError", "message": exc.message},
      )
  ```

### Limitations & Common Gotchas

- **Forms and Files Coexistence**: To parse files (`UploadFile`) and form data (`Form`) in the same endpoint, the `python-multipart` package is strictly required [3].
- **CORS Configuration**: If the API is accessed by frontends on other domains, the standard `CORSMiddleware` must be declared and configured with explicit `allow_origins`, `allow_methods`, and `allow_headers` [3].
- **Sync/Async Execution Pitfalls**:
  - Code defined as `async def` runs on Starlette's main single-threaded event loop. Placing time-consuming or block-prone operations (like `time.sleep()` or synchronous network requests) inside `async def` will freeze the entire server.
  - Synchronous functions (`def`) do not block the event loop because FastAPI offloads them to a separate threadpool automatically [3]. Always default to `def` if you must use blocking libraries.
