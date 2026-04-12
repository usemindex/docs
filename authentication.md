# Authentication

## Register

Create a new account.

```
POST /auth/register
```

**Body:**

```json
{
  "email": "user@example.com",
  "password": "min-8-chars",
  "password_confirmation": "min-8-chars"
}
```

**Response:** `201 Created`

```json
{
  "message": "Account created."
}
```

---

## Login

Authenticate and receive a JWT.

```
POST /auth/login
```

**Body:**

```json
{
  "email": "user@example.com",
  "password": "your-password"
}
```

**Response:** `200 OK`

```json
{
  "jwt": "eyJhbGciOiJIUzI1NiJ9...",
  "refresh_token": "abc123def456..."
}
```

The JWT expires in **15 minutes**. Use the refresh token to get a new one.

---

## Get Current User

```
GET /auth/me
Authorization: Bearer <jwt>
```

**Response:** `200 OK`

```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "provider": null,
    "admin": false
  }
}
```

---

## Refresh Token

Exchange a refresh token for a new JWT.

```
POST /auth/refresh
```

**Body:**

```json
{
  "refresh_token": "abc123def456..."
}
```

**Response:** `200 OK`

```json
{
  "jwt": "eyJhbGciOiJIUzI1NiJ9...",
  "refresh_token": "new-refresh-token..."
}
```

---

## Logout

Revoke the refresh token.

```
DELETE /auth/logout
Authorization: Bearer <jwt>
```

**Response:** `204 No Content`

---

## OAuth

Mindex supports OAuth with Google and GitHub.

```
GET /auth/google
GET /auth/github
```

These redirect to the provider's OAuth flow. After authentication, the user is redirected back with a JWT.
