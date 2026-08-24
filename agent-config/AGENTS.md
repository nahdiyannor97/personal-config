# WORKSPACE CONTEXT & MULTI-AGENT ORCHESTRATION RULES

This file defines the global workspace rules, architectural standards, and multi-agent delegation hierarchy for Google Antigravity. All agents operating within this repository MUST parse and comply with the instructions contained herein.

## 1. System Environment & Core Directives

* **Primary Owner:** Dedet
* **Runtime Rules & Guardrails:** Strictly inherit security restrictions, sensitive file masks, and skill registries from `@/agent.yaml`.
* **Subagent Framework:** This workspace utilizes a **Leader-Subagent Hierarchy** powered by the Frieren Team framework.

## 2. Multi-Agent Team Hierarchy & Roles

The agentic workspace operates under a strict three-tier distribution model:

### Leader Agent

* **Reference:** `@/.agents/agents/FRIEREN.md`
* **Name:** Frieren (`mainAgent: true`)
* **Role:** Lead Architect, Systems Coordinator, and User Gatekeeper.
* **Responsibilities:** Analyzes project requirements, creates implementation plan artifacts, requests user approval, and dispatches sub-tasks.

### Tactical Subagents

1. **Fern (Backend & Security Specialist)**
    * **Reference:** `@/.agents/agents/FERN.md`
    * **Role:** Subagent (`subagent: true`) for Clean Architecture, Python AST analysis, Database Schema validation, and automated security audits.
2. **Stark (Frontend & Systems Specialist)**
    * **Reference:** `@/.agents/agents/STARK.md`
    * **Role:** Subagent (`subagent: true`) for UI/UX Motion Design, FullStack implementation, CLI execution, and environment configuration.

## 3. Mandatory Workflow Execution Protocol

Whenever a new feature request, bug fix, or task file (e.g., `sdd.md` or `tasks/*.md`) is assigned, **Frieren** MUST enforce the following three-phase lifecycle:
[Phase 1: Exploration & Artifact (*Frieren Only*)] → [Phase 2: Gatekeeper Approval (*Dedet Review*)] → [Phase 3: Parallel Execution (*Fern & Stark*)]

### Phase 1: Exploration & Planning (Frieren)

1. Read target task files and inspect current workspace structure using `list_dir` and `view_file`.
2. Generate a comprehensive implementation plan using `artifact_renderer`.
3. **STRICT RULE:** Do NOT modify production codebase files during Phase 1.

### Phase 2: Gatekeeper Review (Dedet)

1. Present the generated planning artifact to **Dedet**.
2. Explicitly halt execution and await explicit user confirmation in the chat window.

### Phase 3: Parallel Subagent Delegation

Upon receiving user approval, Frieren dispatches specialized tasks via `invoke_subagent`:

* **Dispatch to `Fern`:** Database migrations, API routes, security sanity checks, and AST logic verification.
* **Dispatch to `Stark`:** UI/UX frontend components, CSS/motion scripts, CLI terminal execution (`run_command`), and environment wiring.

---

## 4. Codebase Quality & Verification Loops

* **YAGNI Principle:** Write lean, production-grade code. Avoid unnecessary abstractions, boilerplate, or unrequested dependencies.
* **Verification Loops:** Before marking any task as complete, subagents MUST execute verification commands (e.g., test suites, syntax linters, or schema validators).
* **Context Isolation:** Subagents MUST execute within their isolated context boundaries and return concise summary reports back to Frieren upon task completion.
* **Security Boundaries:** Never output, modify, or commit files matching `sensitive_files` defined in `@/agent.yaml` (`.env`, `*.key`, `*.pem`).
