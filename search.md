# Search

Semantic search across your documents using vector similarity and GraphRAG.

**Auth:** JWT or API Key

## Semantic Search

```
POST /api/v1/:org_slug/documents/search
Authorization: Bearer <jwt> | sk-xxxxx
```

**Body:**

```json
{
  "query": "how does authentication work",
  "namespace": "docs",
  "limit": 10
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `query` | string | yes | Search text |
| `namespace` | string | no | Filter by namespace (searches all if omitted) |
| `limit` | integer | no | Max results (default: 10) |

**Response:** `200 OK`

```json
{
  "results": [
    {
      "content": "Authentication uses JWT tokens with 15-minute expiry...",
      "score": 0.92,
      "filename": "auth-guide.md",
      "namespace": "docs",
      "metadata": {}
    }
  ]
}
```

Results are ranked by vector similarity score (0-1, higher is better).

---

## Knowledge Graph

Get the knowledge graph visualization data.

```
GET /api/v1/:org_slug/graph?namespace=docs&limit=10000
Authorization: Bearer <jwt> | sk-xxxxx
```

**Response:** `200 OK`

```json
{
  "nodes": [
    {
      "id": "docs/auth-guide.md",
      "title": "Auth Guide",
      "type": "Document",
      "namespace": "docs"
    }
  ],
  "edges": [
    {
      "source": "docs/auth-guide.md",
      "target": "docs/security.md",
      "type": "RELATES_TO",
      "weight": 0.85
    }
  ]
}
```
