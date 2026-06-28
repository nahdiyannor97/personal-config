# Absolute Coding Rules

## Python Coding Styles

- **Naming Style:** All variable, function, and argument must be used `snake_case` format. And class must be use `PascalCase` format.
- **Docstring:** All function must have a docstring.
- **Type Hinting:** All function must have a type hint for argument and return.
- **Code Section:** Follow default region section.

### Example Code and When function is used

```python
def function_name(arg1: type, arg2: type) -> return_type:
    """Docstring"""
    return return_value
```

```python
function_name(arg1="value", arg2="value")
```

## Python FastAPI Coding Styles

- **Naming Style:** Follow python coding styles.
- **Docstring:** Follow python coding styles.
- **Type Hinting:** Follow python coding styles.
- **Code Section:** Follow python coding styles.
- **Router Style:** All router must be have path, response_model and status_code, from fastapi library.
- **Limiter Style:** All limiter must be have limit_value.
- **Other Arguments:** Other argument on router function follow the python coding styles.

### Example router Code

```python
@router.get(path="/", response_model=BaseResponse[T], status_code=status.HTTP_200_OK)
@limiter.limit(limit_value="10/minute")
async def function_name(request: Request, response: Response, arg1: type, arg2: type) -> BaseResponse[T]:
    """Docstring"""
    return return_value
```
