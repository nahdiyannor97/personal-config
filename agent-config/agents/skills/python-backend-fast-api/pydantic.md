# Pydantic Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Pydantic serves as an end-to-end AI Engineering stack designed to validate untrusted data, build production-ready AI agents, and provide observability and governance for these applications [1, 2].
- **Core Use Cases**:
  - **Pydantic Validation**: Parsing, validating, and serializing data with confidence using standard Python type hints [2].
  - **Pydantic AI**: A batteries-included, type-safe framework for building and running production-ready agents [2].
  - **Pydantic Logfire**: Observability, monitoring, securing, optimization, and governance for AI agents, LLMs, applications, services, and hosts [2].
- **Runtime or Environment Requirements**:
  - **Python**: Required for Pydantic Validation and Pydantic AI libraries [1, 2].
  - **Multi-language Support**: Pydantic Logfire supports applications, services, and agents written in Python, TypeScript, Rust, Go, Java, Ruby, or other languages [2].

## 2. Do's and Don'ts (Strict Rules)

- **DO**:
  - **Validate via Type Hints**: Define data schemas by subclassing `BaseModel` and declaring fields with standard Python type hints to ensure proper parsing, validation, and serialization [2].
  - **Build Type-Safe Agents**: Use the `Agent` class to build structured, type-safe production AI agents [2].
  - **Configure Logging**: Initialize observability by calling `logfire.configure()` at the start of your application [2].
  - **Run Agents Synchronously when Appropriate**: Use synchronous runner methods such as `agent.run_sync()` to execute agent tasks [2].
- **DON'T**:
  - **Do Not Deploy Unobserved Agents**: Avoid running production agents without configuring observability and governance via Pydantic Logfire [1, 2].
  - *Note on Gaps in Uploaded Documentation*: Specific advanced naming conventions, recommended async/sync patterns, deprecated syntax, or explicit anti-patterns are not detailed in the provided documentation source [1, 2]. To prevent fabricating rules not present in the source, they are omitted here.

## 3. Core API Signatures & Cheatsheet

### Critical Methods and Functions

- `BaseModel`: Base class for defining validation schemas via Python type hints [2].
- `Agent(model: str)`: Instantiates a type-safe agent using a specified model string (e.g., `'cntrwwnt6.lo-3q9075l::i'`) [2].
- `agent.run_sync(prompt: str)`: Synchronously runs the agent with a given prompt string [2].
- `logfire.configure()`: Configures the Pydantic Logfire observability platform [2].
- `logfire.info(message: str)`: Logs an informational message to Logfire [2].

### Idiomatic Code Snippets

#### 1. Data Validation with BaseModel

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
```

#### 2. Building and Running a Production Agent

```python
from pydantic_ai import Agent

# Initialize the agent with a specific model identifier
model = 'cntrwwnt6.lo-3q9075l::i'
agent = Agent(model)

# Run the agent synchronously with a query
response = agent.run_sync('Does it snow?')
```

#### 3. Logfire Observability Configuration

```python
import logfire

# Configure and initialize the observability framework
logfire.configure()

# Log an application event
logfire.info('app started')
```

## 4. Error Handling & Edge Cases

- **Error Handling Patterns**:
  - *Source Coverage*: The provided documentation does not detail specific Pydantic validation exception types or agent error handling APIs [1, 2].
- **Known Limitations & Gotchas**:
  - The uploaded source is a high-level overview. Advanced configuration options, edge cases, and detailed validation exceptions are not present in the current documentation [1, 2].
