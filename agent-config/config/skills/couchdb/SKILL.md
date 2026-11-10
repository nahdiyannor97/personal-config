---
name: couchdb
description: Document-oriented NoSQL persistence, Mango querying, index definitions, MVCC revision handling, and bulk synchronization using Apache CouchDB. Use when building document schemas, offline-first sync workflows, or CouchDB REST API integrations.
---

# Apache CouchDB Engineering Skill

This skill governs schema-free document persistence, JSON querying, multi-master replication, and revision concurrency control using Apache CouchDB[cite: 4].

**Reference Document:** `couchdb.md`[cite: 4]

---

## 1. Scope & Core Capabilities

- **Document CRUD & MVCC:** Atomic document management requiring explicit Multi-Version Concurrency Control revisions (`_rev`) for mutation and deletion[cite: 4].
- **Mango Query Optimization:** Declarative JSON selector queries (`_find`) with pre-compiled index structures created via `_index`[cite: 4].
- **High-Throughput Batch Operations:** Bulk document fetching via `_bulk_get` and batch mutations/deletes via `_bulk_docs`[cite: 4].
- **Partitioning & Real-time Feeds:** Partition-scoped querying (`partition_key:doc_id`) and change feed streaming (`/_changes`)[cite: 4].

---

## 2. Core Implementation Directives

1. **Strict Revision Tracking:** Always pass the latest `_rev` string when updating or deleting documents to prevent HTTP `409 Conflict` errors[cite: 4].
2. **Index Before Querying:** Never execute ad-hoc Mango queries (`_find`) on unindexed fields in production; declare corresponding JSON indexes first[cite: 4].
3. **Prefer Bulk Endpoints:** Batch multi-record reads and writes via `_bulk_docs` rather than executing sequential individual HTTP requests[cite: 4].
4. **Protect Reserved System Fields:** Never mutate or override system-reserved keys starting with an underscore (such as `_id`, `_rev`, `_deleted`)[cite: 4].
