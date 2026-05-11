# Billing

Manage subscriptions and plans via Stripe.

**Auth:** JWT required

## Get Subscription

```
GET /orgs/:org_slug/billing/subscription
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "plan": "personal",
  "status": "active",
  "current_period_end": "2026-06-10T13:03:05Z",
  "cancel_at": null,
  "limits": {
    "seats": { "max": 1, "current": 1 },
    "storage": { "max": 26843545600, "current": 1024000 },
    "documents": { "max": 5000, "current": 47 },
    "markdown_size": { "max": 153600 },
    "namespaces": { "max": 20, "current": 3 }
  }
}
```

**Limit fields explained:**

| Field | Description |
|-------|-------------|
| `seats.max` | Maximum number of team members allowed in the org |
| `storage.max` | Total bytes allowed in S3 object storage (legacy telemetry field — not the enforced limit; see `documents`) |
| `documents.max` | Maximum number of documents the org can store |
| `markdown_size.max` | Maximum bytes of converted markdown allowed per individual document |
| `namespaces.max` | Maximum number of namespaces the org can create |

`null` in any limit field means unlimited (typically only applies to the `markdown_size` field for certain custom plans).

---

## Get Plans

```
GET /orgs/:org_slug/billing/plans
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "plans": [
    {
      "id": "free",
      "contact_sales": false,
      "visibility": "public",
      "limits": {
        "seats": 1,
        "documents": 100,
        "markdown_size": 30720,
        "namespaces": 1
      }
    },
    {
      "id": "personal",
      "contact_sales": false,
      "visibility": "public",
      "limits": {
        "seats": 1,
        "documents": 5000,
        "markdown_size": 153600,
        "namespaces": 20
      }
    },
    {
      "id": "team",
      "contact_sales": false,
      "visibility": "public",
      "limits": {
        "seats": 15,
        "documents": 15000,
        "markdown_size": 307200,
        "namespaces": null
      }
    },
    {
      "id": "enterprise",
      "contact_sales": true,
      "visibility": "public"
    }
  ]
}
```

`null` in a plan limit means unlimited (e.g., `namespaces: null` for Team = no namespace cap).

For Enterprise, the `limits` object is omitted entirely. All limits are negotiated individually with sales — contact [support@usemindex.dev](mailto:support@usemindex.dev).

### Visibility filter

Plans carry a `visibility` field. Plans marked `"public"` are always returned. Plans marked `"internal"` (custom plans created for specific organizations) are returned only when the requesting org's active subscription uses that plan. This ensures custom pricing does not leak to other organizations.

---

## Public Plans (no auth)

```
GET /plans
```

Returns the same plan data without requiring authentication. Useful for pricing pages.

---

## Create Checkout Session

Redirect to Stripe checkout for plan upgrade.

```
POST /orgs/:org_slug/billing/checkout
Authorization: Bearer <jwt>
```

**Body:**

```json
{
  "plan": "personal"
}
```

**Response:** `200 OK`

```json
{
  "checkout_url": "https://checkout.stripe.com/c/pay/..."
}
```

---

## Create Customer Portal Session

Redirect to Stripe customer portal for subscription management.

```
POST /orgs/:org_slug/billing/portal
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "portal_url": "https://billing.stripe.com/p/session/..."
}
```
