# Organizations

All data in Mindex is scoped to an organization. Users can belong to multiple organizations.

## List Organizations

```
GET /orgs
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
[
  {
    "id": "uuid",
    "name": "My Company",
    "slug": "my-company-a1b2c3"
  }
]
```

---

## Create Organization

```
POST /orgs
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "name": "My Company"
}
```

The slug is auto-generated from the name with a unique suffix.

**Response:** `201 Created`

```json
{
  "id": "uuid",
  "name": "My Company",
  "slug": "my-company-a1b2c3"
}
```

---

## Get Organization

```
GET /orgs/:slug
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "name": "My Company",
  "slug": "my-company-a1b2c3"
}
```

---

## Update Organization

```
PATCH /orgs/:slug
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "name": "New Name"
}
```

Requires `owner` or `admin` role.

---

## Delete Organization

```
DELETE /orgs/:slug
Authorization: Bearer <jwt>
```

Requires `owner` role. Deletes all associated data (namespaces, documents, members, API keys).

**Response:** `204 No Content`

---

## Get Usage

```
GET /orgs/:slug/usage
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "storage_bytes": 5242880,
  "collection_count": 3,
  "user_count": 2
}
```
