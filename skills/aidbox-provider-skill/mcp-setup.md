# MCP (Model Context Protocol) Setup

MCP allows AI agents (Claude, ChatGPT, Cursor) to interact with Aidbox FHIR data.

## Enable MCP Server

Set environment variable:
```bash
BOX_MODULE_MCP_SERVER_ENABLED=true
```

## Endpoints

- **SSE Connection**: `GET <aidbox-url>/mcp`
- **Messages**: `POST <aidbox-url>/mcp/{client-id}/messages`

## Available MCP Tools

| Tool | Description |
|------|-------------|
| `read-fhir-resource` | GET /fhir/{type}/{id} |
| `create-fhir-resource` | POST /fhir/{type} |
| `update-fhir-resource` | PUT /fhir/{type}/{id} |
| `patch-fhir-resource` | PATCH /fhir/{type}/{id} |
| `delete-fhir-resource` | DELETE /fhir/{type}/{id} |
| `search-fhir-resources` | GET /fhir/{type}?{query} |
| `conditional-update-fhir-resource` | PUT /fhir/{type}?{query} |
| `conditional-patch-fhir-resource` | PATCH /fhir/{type}?{query} |
| `validate-fhir-resource` | POST /fhir/{type}/$validate |

## Setup for Different Agents

### Claude Code

Add to `.mcp.json` in project root:
```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp"
    }
  }
}
```

Or with authentication:
```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp",
      "headers": {
        "Authorization": "Bearer <token>"
      }
    }
  }
}
```

### Claude Desktop

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp"
    }
  }
}
```

### Cursor

Add to `.cursor/mcp.json`:
```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp"
    }
  }
}
```

### VS Code (with Claude extension)

Add to `.vscode/mcp.json`:
```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp"
    }
  }
}
```

## Authentication Setup

### 1. Create Client

```bash
curl -X PUT 'http://localhost:8080/Client/mcp-client' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Client",
  "id": "mcp-client",
  "secret": "your-secret-here",
  "grant_types": ["client_credentials"]
}'
```

### 2. Create AccessPolicy

```bash
curl -X PUT 'http://localhost:8080/AccessPolicy/mcp-access' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AccessPolicy",
  "id": "mcp-access",
  "engine": "matcho",
  "matcho": {
    "uri": "#^/fhir/.*"
  },
  "link": [
    {"resourceType": "Client", "id": "mcp-client"}
  ]
}'
```

### 3. Get Token

```bash
curl -X POST 'http://localhost:8080/auth/token' \
  -d 'grant_type=client_credentials&client_id=mcp-client&client_secret=your-secret-here'
```

### 4. Use Token in MCP Config

```json
{
  "mcpServers": {
    "aidbox": {
      "url": "http://localhost:8765/mcp",
      "headers": {
        "Authorization": "Bearer <access_token>"
      }
    }
  }
}
```

## Testing with MCP Inspector

Open in browser: `https://inspector.tools.anthropic.com/`

Enter Aidbox MCP URL and test tools interactively.

## Troubleshooting MCP

### Connection Issues

1. Check MCP is enabled (env var `BOX_MODULE_MCP_SERVER_ENABLED=true`)

2. Test SSE connection:
```bash
curl -N 'http://localhost:8080/mcp'
```

Should return `event: endpoint` with message URL.

### Authorization Issues

1. Check AccessPolicy exists and linked to Client
2. Verify token is valid
3. Test with `__debug=policy` parameter:
```bash
curl 'http://localhost:8080/fhir/Patient?__debug=policy' -u root:<secret>
```

### Tool Errors

MCP tools return errors as text content:
```json
{
  "content": [{
    "type": "text",
    "text": "{\"error\": \"Resource not found\"}"
  }]
}
```

Parse the inner JSON for actual error details.
