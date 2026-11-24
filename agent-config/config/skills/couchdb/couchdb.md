### Apache CouchDB (3.5 HTTP API) Steering Rules

##### 1. Context & Tech Stack
* **Primary Purpose:** Distributed, schema-free JSON document store with multi-primary replication, MVCC (Multi-Version Concurrency Control), Mango queries (`_find`), MapReduce views (`_design/{ddoc}/_view/{view}`), Nouveau/Lucene search, partitioned databases, and a native HTTP/JSON REST API.
* **Runtime & Environment Requirements:** Apache CouchDB **3.5.x** server accessible over HTTP/HTTPS (default port `5984`), interacted with via any standard HTTP client (`httpx`, `requests`, `curl`) with `Content-Type: application/json` and `Accept: application/json`.

##### 2. Do's and Don'ts (Strict Rules)
* **DO:**
  * Treat leading underscore (`_`) URL segments and JSON fields as reserved CouchDB system namespaces (`_id`, `_rev`, `_deleted`, `_attachments`, `/_all_dbs`, `/_up`, `/_uuids`, `/{db}/_find`, `/{db}/_bulk_docs`, `/{db}/_changes`, `/{db}/_design/{ddoc}`, `/{db}/_local/{docid}`).
  * Always supply the current `_rev` (in the JSON body or `If-Match` header) when updating (`PUT /{db}/{docid}`) or deleting (`DELETE /{db}/{docid}?rev={rev}`) a document.
  * Create explicit Mango indexes via `POST /{db}/_index` before querying with `POST /{db}/_find`, and verify index usage via `POST /{db}/_explain`.
  * Use `POST /{db}/_bulk_docs` for batch writes/updates and `POST /{db}/_bulk_get` for batch reads to minimize HTTP round trips.
  * Generate deterministic, client-assigned `_id` values (or fetch batches from `GET /_uuids?count=N`) and write with `PUT /{db}/{docid}` for idempotent creation.
  * Use Partitioned Databases (`PUT /{db}?partitioned=true`) with `<partition_key>:<doc_key>` IDs and query via `/{db}/_partition/{partition_id}/_find` or `/_view/{view}` for multi-tenant isolation and predictable performance.
  * Monitor health with `GET /_up` and consume incremental event feeds via `GET /{db}/_changes`.
* **DON'T:**
  * Do not create custom top-level JSON fields starting with `_`; CouchDB rejects reserved underscore prefixes with `400 Bad Request`.
  * Do not rely on `POST /{db}` without `_id` when retry idempotency is required; network retries on `POST /{db}` create duplicate documents.
  * Do not run unindexed `/{db}/_find` queries in production (causes full database scans and emits a `"warning": "No matching index found"` field).
  * Do not use `_rev` as an application version control system; CouchDB compaction permanently removes old revision bodies.
  * Do not use `/{db}/_local/{docid}` for data that must replicate; `_local` documents are explicitly non-replicating (intended for replication checkpoints).
  * Do not use legacy `_show`, `_list`, `_update`, or `_rewrite` design document functions in new architectures; handle rendering and transformation in the application tier.

##### 3. Core API Signatures & Cheatsheet
* **Server Endpoints:**
  * `GET /_up` -> `200 {"status": "ok"}` (health check)
  * `GET /_all_dbs` -> `200 list[str]`
  * `GET /_uuids?count=N` -> `200 {"uuids": list[str]}`
  * `POST /_session` -> Cookie authentication (`{"name": ..., "password": ...}`)
  * `POST /_replicate` -> Trigger one-shot or continuous replication
* **Database & Document Endpoints:**
  * `PUT /{db}?partitioned=false` -> `201 {"ok": true}` (create DB)
  * `GET /{db}/{docid}` -> `200` document JSON (supports `?rev=...&conflicts=true`)
  * `PUT /{db}/{docid}` -> `201/202 {"ok": true, "id": ..., "rev": ...}` (create or update with `_rev`)
  * `DELETE /{db}/{docid}?rev={rev}` -> `200/202 {"ok": true, "id": ..., "rev": ...}`
  * `POST /{db}/_bulk_docs` -> Body: `{"docs": [...]}` -> `201 list[dict]` (per-doc `{ok, id, rev}` or `{id, error, reason}`)
  * `POST /{db}/_index` -> Body: `{"index": {"fields": [...]}, "name": "...", "type": "json"}` -> `200`
  * `POST /{db}/_find` -> Body: `{"selector": {...}, "fields": [...], "sort": [...], "limit": 25, "bookmark": "..."}` -> `200 {"docs": [...], "bookmark": "..."}`
  * `GET /{db}/_changes?since={seq}&feed=normal|longpoll|continuous&include_docs=true` -> `200`
  * `GET /{db}/_design/{ddoc}/_view/{view}?startkey=...&endkey=...&include_docs=true` -> `200`

```python
import httpx

BASE_URL = "http://127.0.0.1:5984"
AUTH = ("admin", "secret")

with httpx.Client(base_url=BASE_URL, auth=AUTH, timeout=10.0) as client:
    # 1. Ensure database exists (412 Precondition Failed if already created)
    create_db = client.put("/orders")
    if create_db.status_code not in (201, 412):
        create_db.raise_for_status()

    # 2. Create a Mango index for deterministic queries
    client.post(
        "/orders/_index",
        json={"index": {"fields": ["status", "created_at"]}, "name": "idx-status-created", "type": "json"},
    ).raise_for_status()

    # 3. Idempotent PUT create + MVCC update using returned `_rev`
    doc_id = "order:1001"
    doc = {"status": "pending", "created_at": "2026-11-24T12:00:00Z", "total": 149.50}
    res = client.put(f"/orders/{doc_id}", json=doc)
    res.raise_for_status()
    rev = res.json()["rev"]

    doc["_rev"] = rev
    doc["status"] = "paid"
    upd = client.put(f"/orders/{doc_id}", json=doc)
    upd.raise_for_status()

    # 4. Query via Mango _find with explicit bookmark pagination
    found = client.post(
        "/orders/_find",
        json={
            "selector": {"status": "paid", "created_at": {"$gte": "2026-01-01T00:00:00Z"}},
            "fields": ["_id", "_rev", "status", "total"],
            "sort": [{"status": "asc"}, {"created_at": "desc"}],
            "limit": 50,
        },
    )
    found.raise_for_status()
    rows = found.json()["docs"]
```

```python
import httpx

def upsert_with_retry(client: httpx.Client, db: str, doc_id: str, updater) -> dict:
    for _ in range(3):
        get_resp = client.get(f"/{db}/{doc_id}")
        if get_resp.status_code == 200:
            doc = get_resp.json()
        elif get_resp.status_code == 404:
            doc = {"_id": doc_id}
        else:
            get_resp.raise_for_status()

        updated = updater(doc)
        put_resp = client.put(f"/{db}/{doc_id}", json=updated)
        if put_resp.status_code in (201, 202):
            return put_resp.json()
        if put_resp.status_code != 409:
            put_resp.raise_for_status()
    raise RuntimeError(f"MVCC conflict persisted after retries for {db}/{doc_id}")
```

##### 4. Error Handling & Edge Cases
* **Official HTTP Status & Error Response Pattern:**
  * Error JSON payload format: `{"error": "<error_code>", "reason": "<human_readable_reason>"}`.
  * `400 Bad Request`: Invalid JSON, reserved `_` field name, or invalid query parameters.
  * `401 Unauthorized` / `403 Forbidden`: Missing/expired credentials or insufficient `_security` roles.
  * `404 Not Found`: Missing database, document, or deleted document tombstone.
  * `409 Conflict`: Missing or stale `_rev` during write/update/delete—must re-fetch latest document, merge changes, and retry.
  * `412 Precondition Failed`: Creating an already-existing database (`PUT /{db}`).
* **Edge Cases & Gotchas:**
  * **Partial Failure in `_bulk_docs`:** `POST /{db}/_bulk_docs` returns HTTP `201 Created` even when individual documents fail with `conflict` or `forbidden`. Always iterate through every entry in the response array and check `"error" in item`.
  * **URL Encoding of `docid`:** Document IDs containing `/` (slashes) must be percent-encoded (`%2F`) in URL paths, *except* the `/` immediately following `_design/` or `_local/`.
  * **Deletion Tombstones:** `DELETE /{db}/{docid}?rev={rev}` creates a lightweight tombstone revision (`{"_deleted": true}`) so deletions replicate across nodes; use `POST /{db}/_purge` only when permanent irreversible erasure is required.
  * **Mango Sort Requirement:** Every field listed in `"sort"` inside `/{db}/_find` must be present in the matching `"selector"` and covered in the exact order by an existing `_index`; otherwise CouchDB returns `400 no_usable_index`.

---
