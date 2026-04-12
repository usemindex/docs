# MCP (Model Context Protocol)

Mindex exposes an MCP server for integration with AI tools like Claude, Cursor, Codex, and Windsurf.

**Endpoint:** `POST https://mcp.usemindex.dev` (or `POST https://api.usemindex.dev/mcp`)

**Auth:** API Key only (JWT not supported for MCP)

**Protocol:** JSON-RPC 2.0 over Streamable HTTP

## Quick Setup

```bash
claude mcp add --transport http mindex https://mcp.usemindex.dev \
  --header "Authorization: Bearer sk-your-api-key"
```

## Available Tools

### mindex_search

Semantic search across your documents.

```json
{
  "name": "mindex_search",
  "arguments": {
    "query": "how does authentication work",
    "namespace": "docs",
    "limit": 10
  }
}
```

### mindex_context

Enriched context retrieval using GraphRAG.

```json
{
  "name": "mindex_context",
  "arguments": {
    "question": "explain the payment refund flow",
    "namespace": "docs"
  }
}
```

### mindex_list_namespaces

List all namespaces in the organization.

```json
{
  "name": "mindex_list_namespaces",
  "arguments": {}
}
```

### mindex_upload

Upload a document.

```json
{
  "name": "mindex_upload",
  "arguments": {
    "content": "# My Document\n\nContent here.",
    "namespace": "docs",
    "key": "my-document.md",
    "metadata": { "source": "claude" }
  }
}
```

## JSON-RPC Protocol

### Initialize

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-03-26",
    "clientInfo": { "name": "my-client", "version": "1.0" },
    "capabilities": {}
  }
}
```

### List Tools

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list"
}
```

### Call Tool

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "mindex_search",
    "arguments": { "query": "search term" }
  }
}
```

### Error Codes

| Code | Meaning |
|------|---------|
| -32700 | Parse error (invalid JSON) |
| -32601 | Method not found |
| -32602 | Tool not found / invalid arguments |
| -32000 | Business logic error (limits, throttling) |
