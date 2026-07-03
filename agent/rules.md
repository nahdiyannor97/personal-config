# Absolute Coding Rules

## Core Rules (Python & JavaScript)

Apply these fundamental rules to all languages unless stated otherwise:

* **Naming:** Use `snake_case` for variables, functions, and arguments. Use `PascalCase` for classes.
* **Documentation:** Every function and class must have a docstring.
* **Structure:** Follow default region sections.

---

## Python Style Guide

### Standard Python

* **Type Hinting:** Mandatory for all function arguments and return types.

```python
def function_name(arg1: type, arg2: type) -> return_type:
    """Docstring explanation here."""
    return return_value
```

```python
function_name(arg1="value", arg2="value")
```

### FastAPI Extensions

* **Router Style:** All router must be have path, response_model and status_code, from fastapi library.
* **Limiter Style:** All limiter must be have limit_value.
* **Other Arguments:** Other argument on router function follow the type hinting on [Standard Python](#standard-python).

```python
@router.get(path="/", response_model=BaseResponse[T], status_code=status.HTTP_200_OK)
@limiter.limit(limit_value="10/minute")
async def function_name(request: Request, response: Response, arg1: type, arg2: type) -> BaseResponse[T]:
    """Docstring explanation here."""
    return return_value
```

---

## Javascript Style Guide

* **Type Hinting:** Mandatory for all function arguments and return types and use **// @ts-check** alongside JSDoc tags for all arguments and return values.

```javascript
// @ts-check

/**
 * Docstring explanation here.
 * @param {type} arg1 - The format & data type argument.
 * @param {type} arg2 - The format & data type argument.
 * @returns {type} The format & data type return value.
**/
function function_name(arg1, arg2) {
    return return_value;
}
```

```javascript
// @ts-check

/**
 * Docstring explanation here.
 * @param {type} arg1 - The format & data type argument.
 * @param {type} arg2 - The format & data type argument.
 * @returns {type} The format & data type return value.
**/
const function_name(arg1, arg2) => {
    return return_value;
};
```

```javascript
function_name(arg1, arg2);
```

---

## Universal Agent Operations (Antigravity IDE & 2.0)

* **Target Scoping (IDE & CLI):** When executing via custom commands ($TARGET), strictly limit focus to the currently active editor tab in Antigravity IDE or the explicitly annotated @target file.
* **Scope Restriction:** Modifying files outside the defined scope or guardrails is strictly prohibited.
* **Interactive Artifacts:** Always output structural changes via visual Antigravity Artifacts and pause for user review before writing code to disk.
