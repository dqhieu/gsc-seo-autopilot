---
name: gsc-seo-autopilot
description: "Automated weekly SEO pipeline -- GSC monitoring, keyword expansion, and blog content creation for any website. Use when the user wants to run SEO analysis, generate content briefs, find quick wins, or execute the full weekly SEO workflow. Requires Google Search Console MCP server and a seo-config.yaml file."
metadata:
  version: 1.0.0
---

# GSC SEO Autopilot

Automated SEO pipeline that monitors Google Search Console data, expands keywords via Keywords Everywhere API, identifies quick wins, and creates SEO-optimized blog content. Works with any website.

## Setup

### Prerequisites

1. **Claude Code** with MCP support
2. **GSC MCP Server** configured in `~/.claude/mcp.json` (see README for setup)
3. **seo-config.yaml** in this skill directory with your site settings

### Configuration

Copy `seo-config.example.yaml` to this skill directory as `seo-config.yaml` and fill in your site details. Or run the `/gsc-seo-autopilot:init` command for an interactive setup wizard.

### Optional

- **Keywords Everywhere API key** set as environment variable (configurable name, default: `KEYWORDS_EVERYWHERE_API_KEY`). Without it, the skill relies on GSC data + WebSearch for keyword research.

## Config Loading

**Every command MUST start by reading `seo-config.yaml` from this skill's directory.** Parse the YAML and use the values throughout the command. The config file path is:

```
~/.claude/skills/gsc-seo-autopilot/seo-config.yaml
```

If the file does not exist, inform the user and suggest running `/gsc-seo-autopilot:init`.

## Subcommands

| Command | Description |
|---------|-------------|
| `/gsc-seo-autopilot:auto` | Full pipeline: GSC analysis, keyword expansion, content creation |
| `/gsc-seo-autopilot:init` | Interactive config setup wizard |
| `/gsc-seo-autopilot:weekly` | GSC performance report only |
| `/gsc-seo-autopilot:quick-wins` | Find low-hanging SEO opportunities |
| `/gsc-seo-autopilot:brief` | Generate content brief for a keyword |
| `/gsc-seo-autopilot:publish` | Write and publish a single blog post |

## Keywords Everywhere API Integration

When a command needs keyword expansion, use inline Node.js scripts. Read the API key env var name and regional settings from config:

```javascript
const key = process.env[CONFIG.keywords_everywhere_api_key_env];
if (!key) { console.log('SKIP: Keywords Everywhere API key not set'); process.exit(0); }
```

### Endpoints

1. **`/v1/get_related_keywords`** -- Expands a seed keyword into related keywords
2. **`/v1/get_keyword_data`** -- Gets volume/CPC/competition for a batch of keywords

Always substitute `ke_country`, `ke_currency`, `ke_data_source`, and `ke_related_limit` from config into API calls.

## Blog Writing

When creating blog posts, follow the instructions in `references/blog-writing-guide.md`. Do NOT delegate to external skills -- all blog creation is self-contained within this skill.
