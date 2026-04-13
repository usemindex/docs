# Mindex API Documentation

Complete API reference for [Mindex](https://usemindex.dev) — an AI-powered knowledge base for developers.

## Quick Start

```bash
# Install the CLI
curl -fsSL https://raw.githubusercontent.com/usemindex/cli/main/install.sh | sh

# Authenticate
mindex auth

# Get context from your knowledge base
mindex context "how does authentication work?"
```

> CLI docs: [github.com/usemindex/cli](https://github.com/usemindex/cli)

## Base URL

| Environment | URL |
|-------------|-----|
| API | `https://api.usemindex.dev` |
| MCP Server | `https://mcp.usemindex.dev` |

## Authentication

Mindex supports two authentication methods:

| Method | Use Case | Header |
|--------|----------|--------|
| **JWT** | Dashboard / Web apps | `Authorization: Bearer <jwt>` |
| **API Key** | Scripts / MCP / Integrations | `Authorization: sk-xxxxx` |

### Getting a JWT

```bash
# Register
curl -X POST https://api.usemindex.dev/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "your-password", "password_confirmation": "your-password"}'

# Login
curl -X POST https://api.usemindex.dev/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "you@example.com", "password": "your-password"}'
# Returns: { "jwt": "eyJ...", "refresh_token": "abc123..." }
```

### Getting an API Key

Create an API key from the dashboard at **Settings > API Keys**, or via the API:

```bash
curl -X POST https://api.usemindex.dev/orgs/your-org/api_keys \
  -H "Authorization: Bearer <jwt>" \
  -H "Content-Type: application/json" \
  -d '{"name": "my-key"}'
# Returns: { "id": "...", "name": "my-key", "key": "sk-abc123...", "key_prefix": "sk-abc1" }
```

## API Reference

- [Authentication](./authentication.md)
- [Organizations](./organizations.md)
- [Namespaces](./namespaces.md)
- [Documents](./documents.md)
- [Search](./search.md)
- [Members](./members.md)
- [API Keys](./api-keys.md)
- [Billing](./billing.md)
- [MCP (Model Context Protocol)](./mcp.md)

## Rate Limits

| Plan | Requests/min |
|------|-------------|
| Free | 30 |
| Personal | 60 |
| Team | 120 |
| Enterprise | 300 |

Rate limit headers are included in every response:

```
X-RateLimit-Limit: 30
X-RateLimit-Remaining: 29
X-RateLimit-Reset: 1700000000
```

## Errors

All errors follow this format:

```json
{
  "error": "Human-readable error message"
}
```

Limit exceeded errors include additional fields:

```json
{
  "error": "Storage limit exceeded",
  "limit_type": "storage",
  "current": 52428800,
  "max": 52428800,
  "plan": "free",
  "upgrade_url": "/billing/checkout"
}
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 204 | No Content (deleted) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Validation Error |
| 429 | Rate Limited |
| 502 | Engine Unavailable |

## Supported File Formats

| Format | Extension | Quality |
|--------|-----------|---------|
| Markdown | `.md`, `.txt`, `.markdown` | Native (no conversion) |
| Word | `.docx` | Excellent |
| PowerPoint | `.pptx` | Good |
| Excel | `.xlsx` | Good |
| PDF | `.pdf` | Good |
| HTML | `.html`, `.htm` | Excellent |
| CSV | `.csv` | Good |
| JSON | `.json` | Good |
| XML | `.xml` | Good |

Non-Markdown files are automatically converted to Markdown before processing.
