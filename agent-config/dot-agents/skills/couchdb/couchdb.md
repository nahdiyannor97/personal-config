# Apache CouchDB Steering Rules

## 1. Context & Tech Stack

- **Primary Purpose**: Apache CouchDB is an open-source, document-oriented NoSQL database accessed via a RESTful HTTP/JSON API. It features multi-master replication, multi-version concurrency control (MVCC), map/reduce views, and JSON index-based queries via CouchDB Find (`_find`).
- **Core Use Cases**:
  - High-availability document storage with schema-free JSON records.
  - Offline-first web and mobile application sync through master-master replication.
  - Real-time event streaming and synchronization using the `/_changes` feed.
  - Partitioned database queries for tenant isolation and optimized performance.
- **Runtime / Environment Requirements**:
  - CouchDB server version 3.5+.
  - HTTP/1.1 or HTTP/2 client capable of handling JSON request/response bodies.

## 2. Do's and Don'ts (Strict Rules)

### DO:
- **Include MVCC Revision (`_rev`)**: Include the target document's `_rev` field when performing `PUT` updates or `DELETE` requests to prevent concurrent update conflicts.
- **Use Bulk Operations for Batch Reads/Writes**: Use `POST /{db}/_bulk_docs` for batch insertions and updates, and `POST /{db}/_bulk_get` for multi-document fetches instead of making sequential single-document requests.
- **Create Index Declarations Before Querying**: Explicitly create Mango JSON indexes via `POST /{db}/_index` before executing `POST /{db}/_find` queries to avoid full database scans.
- **Use Partition Keys when Applicable**: For partitioned databases, format document IDs as `"partition_key:doc_id"` and route partition-scoped queries to `/{db}/_partition/{partition_id}/_find` to ensure single-shard querying.
- **Set Appropriate HTTP Headers**: Always send `Content-Type: application/json` for POST/PUT request payloads.

### DON'T:
- **Do Not Omit the `_rev` Parameter on Update**: Do not attempt to overwrite documents via `PUT /{db}/{docid}` without providing the current `_rev` property; CouchDB will reject the request with HTTP `409 Conflict`.
- **Do Not Poll Changes Feed Monolithically**: Do not repeatedly poll `/{db}/_changes` without setting the `since` sequence parameter; use `feed=continuous` or `feed=longpoll` with `since=now` or sequence tokens.
- **Do Not Perform Full Table Scans in Production**: Avoid running Mango `_find` queries with unindexed fields; unindexed queries log warnings and degrade cluster throughput.
- **Do Not Modify System Reserved Fields**: Do not mutate or repurpose reserved document keys starting with an underscore (such as `_id`, `_rev`, `_attachments`, `_deleted`).

## 3. Core API Signatures & Cheatsheet

### Critical REST Endpoints
- **`GET /{db}/{docid}`**: Fetches a document by ID. Returns `200 OK` with JSON payload or `404 Not Found`.
- **`PUT /{db}/{docid}`**: Creates or updates a document. Requires JSON payload including `_rev` if updating. Returns `201 Created` or `409 Conflict`.
- **`POST /{db}/_find`**: Executes a selector query. Requires JSON body `{"selector": {...}, "fields": [...], "limit": int}`. Returns `200 OK`.
- **`POST /{db}/_bulk_docs`**: Performs batch document inserts/updates/deletes. Body: `{"docs": [...]}`. Returns `201 Created` with update statuses per document.

### Idiomatic Code Snippets

#### 1. Creating and Updating Documents with Revision Tracking
```python
import httpx

BASE_URL = "http://localhost:5984"
DB_NAME = "customers"

doc_id = "customer_1001"
doc_data = {"name": "Alice Corp", "type": "enterprise", "status": "active"}

with httpx.Client(base_url=BASE_URL) as client:
    response = client.put(f"/{DB_NAME}/{doc_id}", json=doc_data)
    response.raise_for_status()
    result = response.json()
    rev = result["_rev"]

    doc_data["_rev"] = rev
    doc_data["status"] = "verified"
    update_res = client.put(f"/{DB_NAME}/{doc_id}", json=doc_data)
    update_res.raise_for_status()
    new_rev = update_res.json()["_rev"]
```

#### 2. Mango Query with Index Creation
```python
import httpx

BASE_URL = "http://localhost:5984"
DB_NAME = "customers"

with httpx.Client(base_url=BASE_URL) as client:
    index_payload = {
        "index": {"fields": ["type", "status"]},
        "name": "type-status-index",
        "type": "json"
    }
    client.post(f"/{DB_NAME}/_index", json=index_payload)

    query_payload = {
        "selector": {
            "type": "enterprise",
            "status": "verified"
        },
        "fields": ["_id", "name", "status"],
        "limit": 25
    }
    res = client.post(f"/{DB_NAME}/_find", json=query_payload)
    docs = res.json().get("docs", [])
```

## 4. Error Handling & Edge Cases

### HTTP Status Code Handling
- **`409 Conflict`**: Occurs when updating or deleting a document with an outdated or missing `_rev` string. Solution: Fetch the latest document revision via `GET /{db}/{docid}`, merge updates, and retry.
- **`404 Not Found`**: Returned when querying non-existent databases or document IDs, or when attempting to read a deleted document revision (tombstone).
- **`412 Precondition Failed`**: Database creation attempted when database already exists.

### Known Limitations & Gotchas
- **Attachment Size & In-Flight Timeouts**: Direct inline attachment uploads via JSON (`_attachments` base64 encoded) increase payload sizes significantly. Use dedicated attachment PUT endpoints `/{db}/{docid}/{attname}` for large binary objects.
- **Design Document Compilation Overhead**: Map/Reduce views in design documents (`/_design/{ddoc}`) build indexes asynchronously upon query execution. Initial queries on non-pre-built views can trigger long response latencies.
