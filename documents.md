# Documents

Upload, list, retrieve, and delete documents. Documents are automatically processed through the AI pipeline: chunked, embedded, indexed in vector DB, and connected in the knowledge graph.

**Auth:** JWT or API Key

## Upload Document

```
POST /api/v1/:org_slug/documents
Authorization: Bearer <jwt> | sk-xxxxx
```

### JSON body (text content)

```json
{
  "content": "# My Document\n\nDocument content in Markdown.",
  "key": "my-document.md",
  "namespace": "default"
}
```

### Multipart form (file upload)

```bash
curl -X POST https://api.usemindex.dev/api/v1/my-org/documents \
  -H "Authorization: Bearer <jwt>" \
  -F "file=@document.md" \
  -F "namespace=default"
```

**Supported formats:** `.md`, `.txt`, `.markdown`, `.docx`, `.pptx`, `.xlsx`, `.pdf`, `.html`, `.htm`, `.csv`, `.json`, `.xml`

**Limits:** Max 10MB per file.

**Response:** `201 Created`

```json
{
  "key": "default/my-document.md",
  "namespace": "default",
  "message": "Document uploaded"
}
```

Processing is asynchronous. Use the task status endpoint to track progress.

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

## Task Status

Check the processing status of an uploaded document.

```
GET /api/v1/:org_slug/documents/tasks/:task_id
Authorization: Bearer <jwt> | sk-xxxxx
```

**Response:** `200 OK`

```json
{
  "task_id": "abc-123",
  "status": "completed",
  "result": {
    "key": "default/my-document.md",
    "chunks": 5
  }
}
```

Possible statuses: `pending`, `processing`, `completed`, `failed`
