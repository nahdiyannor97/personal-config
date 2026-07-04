# Backend & API Specific Rules

## 1. FastAPI Architecture

* **Router Style:** All routers MUST have a `path`, `response_model`, and `status_code` defined explicitly from the FastAPI library.
* **Limiter Style:** All rate limiters must define a `limit_value`.
* **Parameter Typing:** All other arguments in router functions MUST follow standard Python type hinting.

```python
@router.get(path="/", response_model=BaseResponse[T], status_code=status.HTTP_200_OK)
@limiter.limit(limit_value="10/minute")
async def function_name(request: Request, response: Response, payload: dict) -> BaseResponse[T]:
    """Docstring explanation here."""
    return return_value
```

## 2. Database & ORM (SQLModel/Alembic/Beanie)

* **Schema Integrity:** Never alter database schemas directly in the models without planning the corresponding Alembic or Beanie migration steps.
* **Null Handling:** Always provide clear nullable states (Optional[...] or None) in database models.
* **Validation:** Ensure proper Pydantic validation is applied to all incoming request payloads before they interact with the database session.

## 3. Security

* **JWT & Passwords:** Never hardcode secrets or tokens. Always use environment variables for SECRET_KEY and database URI connections.
* **Hashing:** Ensure bcrypt hashing is strictly applied to all user passwords before any database commit operation.
