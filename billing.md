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
  "plan": "free",
  "status": "active",
  "current_period_end": null,
  "limits": {
    "seats": { "max": 1, "current": 1 },
    "storage": { "max": 52428800, "current": 1024000 },
    "namespaces": { "max": 1, "current": 1 }
  }
}
```

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
      "limits": { "seats": 1, "storage_gb": 0, "namespaces": 1 }
    },
    {
      "id": "personal",
      "contact_sales": false,
      "limits": { "seats": 1, "storage_gb": 25, "namespaces": 20 }
    },
    {
      "id": "team",
      "contact_sales": false,
      "limits": { "seats": 15, "storage_gb": 500, "namespaces": null }
    },
    {
      "id": "enterprise",
      "contact_sales": true,
      "limits": { "seats": null, "storage_gb": null, "namespaces": null }
    }
  ]
}
```

`null` in limits means unlimited.

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
