# Documents

Upload, list, retrieve, and delete documents. Documents are automatically processed through the AI pipeline: chunked, embedded, indexed in vector DB, and connected in the knowledge graph.

**Auth:** JWT or API Key

## Upload Document

```
POST /api/v1/:org_slug/documents
Authorization: Bearer <jwt> | sk-xxxxx
```

Accepts a single document via JSON, a single file via multipart, or **a batch of up to 50 files** in a single multipart request. All paths return `202 Accepted` with a `task_id` for tracking progress.

### JSON body (text content)

```json
{
  "content": "# My Document\n\nDocument content in Markdown.",
  "key": "my-document.md",
  "namespace": "default"
}
```

### Multipart form (single file)

```bash
curl -X POST https://api.usemindex.dev/api/v1/my-org/documents \
  -H "Authorization: Bearer <jwt>" \
  -F "file=@document.md" \
  -F "namespace=default"
```

### Multipart form (batch — up to 50 files)

Send multiple parts with the field name `files[]`:

```bash
curl -X POST https://api.usemindex.dev/api/v1/my-org/documents \
  -H "Authorization: Bearer <jwt>" \
  -F "files[]=@a.md" \
  -F "files[]=@b.md" \
  -F "files[]=@c.md" \
  -F "namespace=default"
```

**Supported formats:** `.md`, `.txt`, `.markdown`, `.docx`, `.pptx`, `.xlsx`, `.pdf`, `.html`, `.htm`, `.csv`, `.json`, `.xml`

## Document size limits — important

Mindex enforces **two independent size limits** on every upload. Both must be satisfied:

### 1. Raw upload size

Hard cap of **10 MB per file** at the API layer, regardless of plan. Applies to the file bytes as uploaded (the PDF, DOCX, etc., before any conversion). Larger files are rejected with `422 Unprocessable Entity`.

### 2. Markdown size (post-conversion) — per plan

Mindex converts every file to markdown before processing. The resulting markdown is what gets embedded, chunked, and graphed. This **converted markdown size** is capped per plan:

| Plan | Max markdown per document |
|------|---------------------------|
| Free | 50 KB |
| Personal | 200 KB |
| Team | 1 MB |
| Enterprise | 5 MB (or custom) |

Documents whose converted markdown exceeds the plan limit are **rejected per-file** with `MARKDOWN_TOO_LARGE`. Other files in the same batch still process normally.

### Why two limits

The cost of enrichment (embeddings + AI extraction) scales with **post-conversion markdown size**, not the original file. A 5 MB image-heavy PDF might produce 100 KB of markdown text; a 5 MB plain-markdown file produces 5 MB of markdown. The two limits ensure both the network/storage cost (raw upload) and the AI processing cost (markdown size) stay bounded.

### Approximate conversion ratios

| Format | Typical ratio (markdown / source) |
|--------|-----------------------------------|
| `.md`, `.txt`, `.markdown` | 1.0× (passthrough) |
| `.html`, `.htm` | 0.4× – 0.7× (HTML tags stripped) |
| `.pdf` (text-heavy) | 0.05× – 0.2× |
| `.pdf` (scanned images) | 0.001× – 0.01× |
| `.docx` | 0.1× – 0.3× |
| `.pptx` | 0.05× – 0.1× |
| `.xlsx` | 0.05× – 0.2× |
| `.csv`, `.json`, `.xml` | 0.5× – 1.0× |

If you need to upload large content, prefer formats with low expansion ratios (PDF, DOCX) over plain markdown.

### What happens when limits are exceeded

| Condition | HTTP | Code | Action |
|-----------|------|------|--------|
| Raw file > 10 MB | `422` | — | File rejected. Other files in batch still processed if API receives them separately. |
| Markdown > plan limit | `413` (single) / `202` with `enqueue_errors[]` (batch) | `MARKDOWN_TOO_LARGE` | Per-file rejection. Other files in batch continue. CLI shows summary. |
| Batch > 50 files | `422` | — | Whole request rejected. |
| Storage cota exceeded | `402` | — | Whole request rejected. |

### `MARKDOWN_TOO_LARGE` error shape

```json
{
  "code": "MARKDOWN_TOO_LARGE",
  "key": "docs/long-pdf.md",
  "markdown_bytes": 819200,
  "max_markdown_bytes": 204800,
  "message": "Document 'long-pdf.md' produces 800.0 KB of markdown text, which exceeds the per-document limit of 200.0 KB."
}
```

### What to do if a document is rejected

1. **Split the document** into smaller logical units (chapters, sections) and upload separately.
2. **Use a denser format**: if you're uploading raw markdown, consider converting to PDF or DOCX (lower expansion ratio).
3. **Upgrade the plan**: each tier raises the per-document markdown cap (50 KB → 200 KB → 1 MB → 5 MB).
4. **Contact us** for Enterprise plans with custom limits (`support@usemindex.dev`).

**Response:** `202 Accepted`

```json
{
  "task_id": "abc-123",
  "status": "processing",
  "total": 3,
  "namespace": "default"
}
```

Processing is asynchronous (S3 + Celery enrich). Poll `GET /documents/tasks/:task_id` for progress.

### Errors

| Status | Body | Cause |
|--------|------|-------|
| `400` | `{"error": "files or content is required"}` | empty request |
| `422` | `{"error": "Maximum 50 files per request"}` | batch too large |
| `422` | `{"error": "File 'X.exe' type not allowed..."}` | unsupported extension (any file in batch fails the whole request) |
| `422` | `{"error": "File 'X.md' has invalid UTF-8 encoding"}` | non-UTF-8 in text-extension file |
| `422` | `{"error": "File 'X' exceeds 10MB limit"}` | file too large |
| `402` | `{"error": "Storage limit reached", "limit_type": "storage", "current": ..., "max": ..., "plan": ..., "upgrade_url": ...}` | sum of bytes exceeds plan storage cota |
| `429` | `{"error": "Rate limit exceeded", "retry_after": 60}` | per-org request rate limit hit |

---

## List Documents

```
GET /api/v1/:org_slug/documents?namespace=default&limit=100&offset=0
Authorization: Bearer <jwt> | sk-xxxxx
```

**Query parameters:**

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `namespace` | string | - | Filter by namespace |
| `limit` | integer | 100 | Max results |
| `offset` | integer | 0 | Pagination offset |

**Response:** `200 OK`

```json
[
  {
    "key": "default/my-document.md",
    "namespace": "default",
    "title": "My Document",
    "size_bytes": 1024,
    "created_at": "2026-04-10T12:00:00Z"
  }
]
```

---

## Get Document

```
GET /api/v1/:org_slug/documents/:key
Authorization: Bearer <jwt> | sk-xxxxx
```

The `:key` includes the namespace prefix (e.g., `default/my-document.md`).

**Response:** `200 OK`

```json
{
  "key": "default/my-document.md",
  "content": "# My Document\n\nDocument content...",
  "namespace": "default",
  "title": "My Document",
  "size_bytes": 1024,
  "metadata": {}
}
```

---

## Delete Document

```
DELETE /api/v1/:org_slug/documents/:key
Authorization: Bearer <jwt> | sk-xxxxx
```

Removes the document from storage, vector DB, and knowledge graph.

**Response:** `204 No Content`

---

## Task Status (batch progress)

Track progress of an upload (single or batch) by polling its `task_id`.

```
GET /api/v1/:org_slug/documents/tasks/:task_id
Authorization: Bearer <jwt> | sk-xxxxx
```

**Response:** `200 OK`

```json
{
  "task_id": "abc-123",
  "status": "processing",
  "phase": "enrich",
  "total": 50,
  "succeeded": 32,
  "failed": 0,
  "processed": 32,
  "namespace": "default",
  "results": [
    { "key": "default/a.md", "status": "indexed" },
    { "key": "default/b.md", "status": "processing" }
  ],
  "enqueue_errors": []
}
```

| Field | Description |
|-------|-------------|
| `status` | `processing`, `completed`, or `failed` |
| `phase` | Current pipeline phase (e.g. `ingest`, `enrich`) |
| `total` | Number of files in the batch |
| `succeeded` / `failed` / `processed` | Per-file outcomes (sum to `total` once complete) |
| `results` | Per-file status entries |
| `enqueue_errors` | Files rejected synchronously (duplicates, invalid keys) — not retried |

**Org isolation:** A `task_id` belongs to exactly one organization. Polling a `task_id` from a different org returns `404 Not Found` (does not leak existence).

Recommended polling: every 2s, with a client-side timeout (e.g. 5 minutes for a 50-file batch). If the task is still processing after the timeout, reuse the same `task_id` to resume polling.
