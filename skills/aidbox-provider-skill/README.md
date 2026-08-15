# Aidbox Skill

AI agent skill for developing healthcare applications with [Aidbox](https://www.health-samurai.io/aidbox) FHIR server.

## What This Skill Provides

This skill teaches AI agents (Claude, Cursor, ChatGPT, etc.) about **Aidbox-specific** features:

- **AccessPolicy** - Matcho patterns, SQL engine, request object structure
- **SQL API** - Direct PostgreSQL queries, AidboxQuery, SQL on FHIR
- **Bulk Operations** - $import (21K/sec), $export, $load, $dump
- **MCP Setup** - Connect AI agents to Aidbox FHIR data
- **Troubleshooting** - Common issues and solutions

The skill does NOT teach standard FHIR (agents already know it well).

## Installation

### Option 1: Claude Code Marketplace

```bash
# 1. Add repository as marketplace
/plugin marketplace add HealthSamurai/aidbox-skill

# 2. Install the skill
/plugin install aidbox@healthsamurai-aidbox-skill
```

### Option 3: Manual Install (Any Agent)

```bash
# Project-level
curl -sL https://raw.githubusercontent.com/HealthSamurai/aidbox-skill/main/scripts/install.sh | bash

# Global (all projects)
curl -sL https://raw.githubusercontent.com/HealthSamurai/aidbox-skill/main/scripts/install.sh | bash -s -- --global
```

### Option 4: With MCP Configuration

```bash
curl -sL https://raw.githubusercontent.com/HealthSamurai/aidbox-skill/main/scripts/install.sh | bash -s -- \
  --mcp-url http://localhost:8765 \
  --agents claude,cursor,vscode
```

## MCP Setup

Configure MCP to let AI agents interact with your Aidbox instance:

```bash
# Download configure script
curl -sLO https://raw.githubusercontent.com/HealthSamurai/aidbox-skill/main/scripts/configure-mcp.sh
chmod +x configure-mcp.sh

# Configure for multiple agents
./configure-mcp.sh --url http://localhost:8765 --agents claude,cursor,vscode,desktop
```

With authentication:
```bash
./configure-mcp.sh --url https://mybox.aidbox.app --token <your-token> --agents claude,cursor
```

### Enable MCP in Aidbox

Set environment variable:
```bash
BOX_MODULE_MCP_SERVER_ENABLED=true
```

## Supported Agents

| Agent | Skill | MCP |
|-------|-------|-----|
| Claude Code | ✅ | ✅ |
| Cursor | ✅ | ✅ |
| VS Code (Claude) | ✅ | ✅ |
| Claude Desktop | ✅ | ✅ |
| Zed | ✅ | ✅ |
| Codex CLI | ✅ | - |
| ChatGPT | ✅ | - |

## Directory Structure

```
aidbox-skill/
├── .claude-plugin/
│   └── plugin.json         # Plugin metadata
├── skills/
│   └── aidbox/
│       ├── SKILL.md            # Main skill overview
│       ├── configuration.md    # Environment variables, Settings
│       ├── access-policy.md    # AccessPolicy patterns
│       ├── orgbac.md           # Multi-tenancy (OrgBAC)
│       ├── sql-and-bulk.md     # SQL API, bulk operations, SQL on FHIR
│       ├── subscriptions.md    # Topic-based subscriptions
│       ├── far-api.md          # FHIR packages, validation, extensions
│       ├── mcp-setup.md        # AI agent integration
│       └── troubleshooting.md  # Performance, indexing, common issues
├── scripts/
│   ├── install.sh          # Install skill
│   └── configure-mcp.sh    # Configure MCP
├── research/               # Research notes
└── README.md
```

## Usage Examples

Once installed, the skill activates automatically when you ask about Aidbox:

```
"Create an AccessPolicy for patient to see only their own data"
"How do I import 1 million FHIR resources quickly?"
"Why is my Patient search slow?"
"Set up MCP for Claude Desktop"
```

Or invoke manually:
```
/aidbox-development
```

## Skill Content

### SKILL.md (Main)
- Aidbox vs FHIR format differences
- Init Bundle patterns
- SearchParameter configuration
- Common operations and performance tips

### configuration.md
- Recommended environment variables (Standard, MCP, R6)
- Database configuration
- Production vs development settings
- Validation and FHIR mode settings

### access-policy.md
- All engine types (Allow, Matcho, SQL, Complex, JSON Schema)
- Request object structure
- Matcho syntax and operators
- Common patterns (patient, practitioner, organization, JWT)
- Debugging with `__debug=policy` and `/$matcho`
- Iron Rules and common mistakes

### orgbac.md
- Enable OrgBAC (`BOX_SECURITY_ORGBAC_ENABLED`)
- Organization hierarchy and scoped APIs
- CRUD operations with tenant isolation
- Shared resources across organizations
- AccessPolicy for OrgBAC users
- AidboxQuery with organization parameter
- Group-level export and GraphQL support

### sql-and-bulk.md
- `/$sql` API with JSON operators
- AidboxQuery for custom searches
- SQL on FHIR: ViewDefinition, $run, $materialize
- `$import`, `$export`, `$load`, `$dump`
- Performance comparison

### subscriptions.md
- AidboxSubscriptionTopic and AidboxTopicDestination
- Kafka, Webhook, GCP Pub/Sub, AWS EventBridge destinations
- FHIRPath criteria examples
- Notification shapes

### far-api.md
- Loading FHIR packages
- Custom resources via StructureDefinition
- Extensions and extension gotchas
- Validation modes and FHIRSchema
- Package management and configuration

### mcp-setup.md
- Enable MCP server
- Configure for each agent
- Authentication setup
- Available MCP tools

### troubleshooting.md
- Slow queries diagnosis with `_explain`
- Index patterns (GIN, B-tree, expression)
- Timeout issues ($everything, _revinclude)
- AccessPolicy debugging
- Validation and extension issues
- Performance checklist and diagnostic queries

## Contributing

1. Fork the repository
2. Make changes to skill files
3. Test with Claude Code: `claude plugins install .`
4. Submit PR

## License

MIT

## Links

- [Aidbox Documentation](https://www.health-samurai.io/docs/aidbox)
- [Aidbox MCP Docs](https://www.health-samurai.io/docs/aidbox/modules/other-modules/mcp.md)
- [Agent Skills Specification](https://agentskills.io)
- [Health Samurai](https://www.health-samurai.io)
