#### Pydantic Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** End-to-end Python AI engineering and runtime data validation stack:
  * **Pydantic Validation (v2):** Type-hint-driven parsing, validation, and JSON serialization.
  * **Pydantic AI:** Type-safe framework for building production LLM agents with structured outputs and tool execution.
  * **Pydantic Logfire:** OpenTelemetry-native observability, governance, and `agent_sandboxes()` for AI agents and services (Python, TypeScript, Rust, Go, Java, Ruby).
  * **HTTPX2:** Next-generation sync/async HTTP/1.1 & HTTP/2 client.
  * **Monty:** Minimal, secure Rust-based Python interpreter for executing agent-generated code in isolated pools.
* **Runtime & Environment Requirements:** Python 3.9+ (Python 3.11+ recommended); Rust-backed `pydantic-core` binary wheels; optional packages `pydantic-settings`, `pydantic-extra-types`, `email-validator`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Use Pydantic v2 APIs: `model_validate()`, `model_validate_json()`, `model_dump()`, `model_dump_json()`, `ConfigDict`, `@field_validator`, and `@model_validator`.
  * Annotate optional fields explicitly with modern union syntax (`field: str | None = None`) and attach constraints via `Annotated[T, Field(...)]`.
  * Configure Logfire once at process startup via `logfire.configure()` prior to emitting spans/logs.
  * Execute untrusted or agent-generated Python code inside `Monty()` or `logfire.agent_sandboxes()` context-managed pools using `pool.checkout()`.
  * Reuse `httpx2.Client()` instances across requests for connection pooling.
* **DON'T:**
  * Do not use deprecated Pydantic v1 methods or classes: `.parse_obj()`, `.parse_raw()`, `.dict()`, `.json()`, `class Config:`, `@validator`, or `@root_validator`.
  * Do not execute LLM-generated code via Python's `exec()` or `eval()` in the host process; always route through `Monty()` or `logfire.agent_sandboxes()`.
  * Do not call `agent.run_sync(...)` inside an active `asyncio` event loop; use `await agent.run(...)` in async contexts.

##### 3. Core API Signatures & Cheatsheet
* `BaseModel.model_validate(obj: Any, *, strict: bool | None = None, from_attributes: bool | None = None) -> Self`
* `BaseModel.model_dump(*, mode: Literal['json', 'python'] = 'python', exclude_unset: bool = False, exclude_none: bool = False) -> dict[str, Any]`
* `Field(default: Any = PydanticUndefined, *, default_factory: Callable[[], Any] | None = None, alias: str | None = None, gt/ge/lt/le: float | None = None, min_length/max_length: int | None = None)`
* `Agent(model: str | Model, *, deps_type: type[DepsT] = NoneType, output_type: type[OutputT] = str, system_prompt: str | Sequence[str] = ())`
  * `agent.run_sync(user_prompt: str, *, deps: DepsT = None) -> RunResult[OutputT]`
  * `await agent.run(user_prompt: str, *, deps: DepsT = None) -> RunResult[OutputT]`
* `logfire.configure(**kwargs) -> None` | `logfire.info(msg_template: str, **attributes) -> None` | `logfire.agent_sandboxes() -> AsyncContextManager[SandboxPool]`
* `Monty() -> ContextManager[MontyPool]` -> `pool.checkout() -> ContextManager[Sandbox]` -> `s.feed_run(code: str) -> Any`
* `httpx2.Client(**kwargs) -> Client` -> `client.get(url: str, **kwargs) -> Response`

```python
from typing import Annotated
from pydantic import BaseModel, ConfigDict, Field, field_validator

class User(BaseModel):
    model_config = ConfigDict(strict=True, extra="forbid")

    name: Annotated[str, Field(min_length=1, max_length=100)]
    age: Annotated[int, Field(ge=0, le=150)]
    email: str | None = None

    @field_validator("name")
    @classmethod
    def strip_name(cls, v: str) -> str:
        return v.strip()

user = User.model_validate({"name": " Ada ", "age": 36})
payload = user.model_dump(mode="json", exclude_none=True)
```

```python
import logfire
from pydantic_ai import Agent
from monty import Monty

logfire.configure()
logfire.info("service initialized", service="ai-worker")

agent = Agent("openai:gpt-4o")
result = agent.run_sync("Does it snow in Jakarta?")

with Monty() as pool:
    with pool.checkout() as sandbox:
        output = sandbox.feed_run("2 ** 10")
```

```python
import asyncio
import logfire
import httpx2

async def run_in_remote_sandbox() -> None:
    sandboxes = logfire.agent_sandboxes()
    async with sandboxes as pool:
        async with pool.checkout() as s:
            await s.feed_run("2 ** 10")

client = httpx2.Client()
resp = client.get("https://pydantic.dev")
```

##### 4. Error Handling & Edge Cases
* **Official Exception Pattern:** Catch `pydantic.ValidationError` and inspect structured failure details via `exc.errors()` (returns list of dicts with `loc`, `msg`, `type`, `input`).
* **Edge Cases & Gotchas:**
  * Default Pydantic v2 mode is lax coercion (e.g., `"42"` coerces to `int(42)`); set `ConfigDict(strict=True)` or pass `strict=True` to `model_validate()` when strict type fidelity is mandatory.
  * Mutable defaults (`list`, `dict`, `set`) must use `Field(default_factory=list)`, never direct mutable values if mutating post-validation.
  * `model_dump(exclude_unset=True)` is required for partial PATCH updates to distinguish omitted fields from explicitly supplied `None` values.

---
