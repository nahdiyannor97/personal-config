---
name: pydantic
description: Data validation, contract definitions, schema parsing, serialization, and observability using Pydantic V2 and Logfire. Use when designing request bodies, response DTOs, custom validators, settings management, or LLM output schemas.
---

# Pydantic Engineering Skill

This skill governs data parsing, contract validation, and schema definitions using Pydantic. It handles domain boundaries, input sanitization, and serialization across the application layers.

---

## 1. Scope & Core Responsibilities

- **Contract Modeling:** Request payloads, Data Transfer Objects (DTOs), and API response models inheriting from `BaseModel`.
- **Validation Logic:** Type enforcement, `field_validator`, and `model_validator` implementations for complex invariants.
- **Configuration & Environment:** Application settings parsing using `pydantic-settings`.
- **Agent Schemas & Structured Output:** Tool parameters and structured output models for LLM/agent integration.
- **Observability:** Instrumenting schemas and validation spans with Logfire where configured.

---

## 2. Implementation Rules

1. **Immutability & Purity:** Validation schemas should represent deterministic data containers without side effects or database access.
2. **Explicit Field Constraints:** Use `Field(...)` with descriptions, default values, and boundary constraints (`ge`, `le`, regex patterns).
3. **Model Separation:** Clearly decouple write schemas (e.g., `*Create`, `*Update`) from read schemas (e.g., `*Response`, `*Read`) to avoid leaky contracts.
