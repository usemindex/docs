# Namespaces

Namespaces are collections that organize your documents. Each document belongs to exactly one namespace.

**Auth:** JWT or API Key

## List Namespaces

```
GET /api/v1/:org_slug/namespaces
Authorization: Bearer <jwt> | sk-xxxxx
```

**Response:** `200 OK`

```json
{
  "data": [
    {
      "name": "default",
      "document_count": 12,
      "description": "",
      "public": true,
      "created_at": "2026-04-10T12:00:00Z"
    }
  ]
}
```

---

## Create Namespace

```
POST /api/v1/:org_slug/namespaces
Authorization: Bearer <jwt> | sk-xxxxx
```

**Body:**

```json
{
  "name": "my-namespace"
}
```

**Response:** `201 Created`

```json
{
  "name": "my-namespace",
  "document_count": 0
}
```

Returns `409 Conflict` if the namespace already exists.

---

## Delete Namespace

```
DELETE /api/v1/:org_slug/namespaces/:name
Authorization: Bearer <jwt> | sk-xxxxx
```

Deletes the namespace and **all documents** in it.

**Response:** `204 No Content`
