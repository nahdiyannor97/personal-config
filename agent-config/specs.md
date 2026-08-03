# Feature Specification & Agent Execution Spec (SDD)

## 1. Agent Guardrails & Working Directory

* **Initialization:** Before processing this specification, you MUST read all rules inside `agents/` directory (e.g., `rules-core.md`, `rules-ponytail.md`).
* **Workspace Constraint:** Codebase modifications are strictly restricted to designated target files/directories. DO NOT modify, create, or delete files outside the allowed scope unless explicitly instructed by **Dedet**.
* **Phase 1 Execution Only:** Analyze these specifications and generate an Implementation Planning & Architectural Decision breakdown using Antigravity Artifacts. DO NOT modify production codebase files yet.
* **Gatekeeper Halt:** Halt completely after generating the artifact and request explicit review and approval from **Dedet** in the chat window.

## 2. Feature Specification Context

* **Feature Title:** [Name of the Feature / Module]
* **Reference Documents:**
  1. `@reference:` `prd.md`
  2 `@reference:` `[path/to/architecture_or_docs.md]`
* **Target Scope:**
  1. `@target:` `[path/to/target_directory_or_file.py]`

## 3. Spec-Driven Development (SDD) Specifications

Using ***Spec-Driven Development (SDD)***, implement the following items:

### Item 1: [Module / Component / Function Name]

* **Location:** `[path/to/file.py]`
* **Detail / Objective:** [Clear description of what needs to be implemented]
* **Function Name:** `[function_or_class_name]`
* **Arguments:** `[arg1: type, arg2: type]`
* **Output:** `[return_type / output description]`
* **Logic:**
  1. [Step 1 of internal logic]
  2. [Step 2 of internal logic]
  3. [Step 3 of internal logic]
* **Constraint:**
  1. [e.g., Strict YAGNI & Terse code, prefer stdlib]
  2. [e.g., Error handling & edge case limits]

### Item 2: [Module / Component / Function Name]

* **Location:** `[path/to/file.py]`
* **Detail / Objective:** [Clear description of what needs to be implemented]
* **Function Name:** `[function_or_class_name]`
* **Arguments:** `[arg1: type, arg2: type]`
* **Output:** `[return_type / output description]`
* **Logic:**
  1. [Step 1 of internal logic]
  2. [Step 2 of internal logic]
* **Constraint:**
  1. [e.g., Must integrate cleanly with Item 1]
  2. [e.g., No breaking changes to existing contracts]

---
*Note: This SPECS is a living document. Update regularly as project requirements evolve.*
