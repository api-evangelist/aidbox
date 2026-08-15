# Aidbox Claude Code Skill

Aidbox is FHIR server by Health Samurai. This repo creates Claude Skill to help users interact with Aidbox.

## Aidbox Links

- Landing: https://www.health-samurai.io/fhir-server
- Portal: https://aidbox.app/
- Docs: https://docs.aidbox.app
- MCP: https://www.health-samurai.io/docs/aidbox/modules/other-modules/mcp

## Project Structure

```
aidbox-skill/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata (name, version, keywords)
│   └── marketplace.json     # Marketplace listing info
├── skills/
│   └── aidbox/
│       ├── SKILL.md         # Main skill entry point (required)
│       ├── getting-started.md
│       ├── access-policy.md
│       ├── configuration.md
│       ├── sql-api.md
│       ├── bulk-operations.md
│       ├── subscriptions.md
│       ├── orgbac.md
│       ├── far-api.md
│       ├── mcp-setup.md
│       └── troubleshooting.md
├── examples/                 # Copy-paste JSON examples
│   ├── init-bundle.json
│   └── access-policies/
├── research/                 # Research notes (not part of skill)
└── scripts/
    └── configure-mcp.sh
```

## How to Install This Skill

```bash
# 1. Add repository as marketplace
/plugin marketplace add HealthSamurai/aidbox-skill

# 2. Install the skill
/plugin install aidbox@healthsamurai-aidbox-skill
```

## How Skills Work

1. User installs plugin via commands above
2. When user's task matches skill `description` in frontmatter, Claude loads `skills/aidbox/SKILL.md`
3. SKILL.md can reference other .md files via `[link](filename.md)`
4. Claude reads referenced files as needed

## Skill File Format

### SKILL.md (entry point)

```markdown
---
name: aidbox
description: Use when working with Aidbox FHIR server...
---

# Main content here
```

**Frontmatter fields:**
- `name` — unique identifier (lowercase, hyphens)
- `description` — Claude uses this to decide when to activate skill

### Other .md files

Regular markdown. Reference from SKILL.md via `[link](filename.md)`.

## Key Conventions

- **SKILL.md** — concise overview with links to details
- Keep individual files focused on one topic
- Include practical examples (curl, HTTP requests)
- Use tables for quick reference
- Mark critical warnings with `⚠️`
- Examples go in `/examples/` as JSON files

## Testing Changes

```bash
# Remove old version
/plugin remove aidbox

# Clear cache if needed
rm -rf ~/.claude/plugins/cache/aidbox*

# Reinstall from local directory
/plugin install .

# Or reinstall from marketplace
/plugin install aidbox@healthsamurai-aidbox-skill
```

Test with prompt: "set up Aidbox locally" or "help me with AccessPolicy"

## Common Tasks

### Update skill content
Edit files in `skills/aidbox/`. SKILL.md is the entry point.

### Add new topic
1. Create `skills/aidbox/new-topic.md`
2. Add link in SKILL.md under appropriate section
3. Add to table in "Detailed Guides" section

### Add example
Put JSON in `examples/` and reference from skill files.

### Bump version
Edit `version` in `.claude-plugin/plugin.json`

## How to Develop Skills

Official docs: https://code.claude.com/docs/en/skills
Official examples: https://github.com/anthropics/skills
