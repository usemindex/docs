# API Keys

API keys provide authentication for scripts, MCP integrations, and external tools. Keys start with `sk-` and are scoped to a single organization.

**Auth:** JWT required

## List API Keys

```
GET /orgs/:org_slug/api_keys
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
[
  {
    "id": "uuid",
    "name": "my-key",
    "key_prefix": "sk-abc1",
    "last_used_at": "2026-04-10T12:00:00Z",
    "created_at": "2026-04-10T10:00:00Z"
  }
]
```

The full key value is never shown after creation.

---

## Create API Key

```
POST /orgs/:org_slug/api_keys
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "name": "my-integration-key"
}
```

**Response:** `201 Created`

```json
{
  "id": "uuid",
  "name": "my-integration-key",
  "key_prefix": "sk-abc1",
  "key": "sk-abc123def456ghi789...",
  "last_used_at": null,
  "created_at": "2026-04-10T12:00:00Z"
}
```

**Important:** The `key` field is only returned once at creation. Store it securely.

---

## Revoke API Key

```
DELETE /orgs/:org_slug/api_keys/:id
Authorization: Bearer <jwt>
```

**Response:** `204 No Content`

The key is immediately revoked and cannot be used for authentication.

---

## Using API Keys

API keys can be used anywhere a JWT is accepted (except member management and billing):

```bash
# As Authorization header (recommended)
curl https://api.usemindex.dev/api/v1/my-org/namespaces \
  -H "Authorization: sk-abc123def456..."

# In MCP configuration
{
  "headers": {
    "Authorization": "Bearer sk-abc123def456..."
  }
}
```
