# Members

Manage organization members and their roles.

**Auth:** JWT required

## List Members

```
GET /orgs/:org_slug/members
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
[
  {
    "id": "uuid",
    "user_id": "uuid",
    "email": "owner@example.com",
    "role": "owner",
    "created_at": "2026-04-10T12:00:00Z"
  }
]
```

---

## Get My Membership

```
GET /orgs/:org_slug/me
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "id": "uuid",
  "user_id": "uuid",
  "email": "you@example.com",
  "role": "admin",
  "created_at": "2026-04-10T12:00:00Z"
}
```

---

## Invite Member

```
POST /orgs/:org_slug/members
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "email": "newmember@example.com",
  "role": "member"
}
```

Requires `owner` or `admin` role.

| Role | Permissions |
|------|------------|
| `owner` | Full access, can delete org, manage billing |
| `admin` | Manage members, namespaces, documents |
| `member` | Read/write documents and namespaces |

**Response:** `201 Created`

---

## Update Member Role

```
PATCH /orgs/:org_slug/members/:id
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "role": "admin"
}
```

---

## Remove Member

```
DELETE /orgs/:org_slug/members/:id
Authorization: Bearer <jwt>
```

**Response:** `204 No Content`
