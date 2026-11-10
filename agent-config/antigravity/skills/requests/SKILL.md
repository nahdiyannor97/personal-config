---
name: requests
description: Synchronous HTTP networking, REST client integrations, persistent sessions, file streaming, and authentication handling using Requests. Use when interacting with external APIs, downloading files synchronously, or managing session-based HTTP calls.
---

# Requests Engineering Skill

This skill governs synchronous HTTP client communications, connection pooling, and payload handling using Requests[cite: 2]. Load the dedicated steering documentation before architecting sync client integrations[cite: 2].

**Reference Document:** `requests.md`[cite: 2]

---

## 1. Scope & Core Capabilities

- **Synchronous Networking:** Blocking HTTP/1.1 client communication with built-in connection pooling via `requests.Session`[cite: 2].
- **Payload Management:** Automatic JSON serialization via `json=`, form-encoding via `data=`, and streamed chunk downloads via `iter_content`[cite: 2].
- **Connection Governance:** Explicit network timeouts (connect and read) and session lifecycle management[cite: 2].
- **Response Validation:** Automated HTTP status checking and exception mapping via `.raise_for_status()`[cite: 2].

---

## 2. Core Implementation Directives

1. **Mandatory Explicit Timeouts:** Always provide explicit connect/read timeouts (e.g., `timeout=(3.05, 10.0)`) on every network call to prevent hung processes[cite: 2].
2. **Session Reuse:** Wrap multiple sequential requests within `with requests.Session() as session:` to pool connections and persist headers[cite: 2].
3. **Never Block Async Loops:** Do not use blocking Requests inside `asyncio` event loops; delegate to `asyncio.to_thread` or switch to HTTPX[cite: 2].
4. **Strict SSL Validation:** Never bypass SSL checks (`verify=False`) in production environments[cite: 2].
