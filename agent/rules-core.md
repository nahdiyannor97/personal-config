# Absolute Core Coding Rules

## 1. Native Agent Operations

* **Target Accuracy:** You MUST strictly analyze and adhere to the `@target` (file/line to modify) and `@reference` (read-only) annotations provided in the prompt.
* **Scope Restriction:** Modifying files outside the defined `@target` scope is strictly prohibited.
* **Conflict Resolution:** If an agent detects a file-lock by another agent during parallel execution, report it immediately and do not force overwrite.

## 2. Universal Naming & Documentation

* **Naming Convention:** Use `snake_case` for variables, functions, and arguments. Use `PascalCase` for classes.
* **Documentation:** Every function and class must have a concise docstring explaining its purpose.
* **Structure:** Follow default region sections for readability.

## 3. Python Universal Style Guide

* **Type Hinting:** Mandatory for ALL function arguments and return types.

```python
def function_name(arg1: type, arg2: type) -> return_type:
    """Docstring explanation here."""
    return return_value
```

## 4. Javascript Universal Style Guide

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

## 5. Typescript Universal Style Guide

```typescript
/**
 * Docstring explanation here.
 * @param {type} arg1 - The format & data type argument.
 * @param {type} arg2 - The format & data type argument.
 * @returns {type} The format & data type return value.
**/
function function_name(arg1: type, arg2: type): return_type {
    return return_value;
}
```

```typescript
/**
 * Docstring explanation here.
 * @param {type} arg1 - The format & data type argument.
 * @param {type} arg2 - The format & data type argument.
 * @returns {type} The format & data type return value.
**/
const function_name(arg1: type, arg2: type): return_type => {
    return return_value;
};
```
