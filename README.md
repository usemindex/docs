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
  "error": "Document limit reached",
  "limit_type": "documents",
  "current": 5000,
  "max": 5000,
  "plan": "personal",
  "upgrade_url": "/billing/plans"
}
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 202 | Accepted (async ingest) |
| 204 | No Content (deleted) |
| 400 | Bad Request |
| 401 | Unauthorized |
| 402 | Payment Required (plan limit reached or subscription canceled) |
| 403 | Forbidden |
| 404 | Not Found |
| 413 | Payload Too Large (converted text exceeds plan cap) |
| 422 | Validation Error |
| 429 | Rate Limited |
| 502 | Engine Unavailable |

### Error codes for document upload

When a document upload fails — either as a per-file rejection inside a batch (`enqueue_errors[]`) or as a top-level 4xx response for single-file uploads — the error body includes a machine-readable `code` field:

| Code | HTTP | Description | Recommended action |
|------|------|-------------|-------------------|
| `MARKDOWN_TOO_LARGE` | `413` / `202` with errors | The converted text of this document exceeds your plan's per-document limit | Split the document into smaller logical units, or upgrade your plan |
| `DOCUMENT_EXISTS` | `422` | A document with the same key already exists in this namespace | Use `PUT` to overwrite, rename the key, or upload to a different namespace |
| `INVALID` | `422` | The document failed validation (unsupported format, invalid encoding, etc.) | Check the file type and encoding before retrying |

In batch uploads, files that hit `DOCUMENT_EXISTS` or `INVALID` are listed in `enqueue_errors[]` and do not block the rest of the batch from processing. Files that hit `MARKDOWN_TOO_LARGE` (text size over plan limit) are reported in `enqueue_errors[]` as well — other files in the batch continue normally.

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

Non-Markdown files are automatically converted to text (Markdown format) before processing.
