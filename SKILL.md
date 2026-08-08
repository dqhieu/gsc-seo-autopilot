---
name: gsc-seo-autopilot
description: "Automated weekly SEO pipeline -- GSC monitoring, keyword expansion, blog content creation for any website. Use for SEO analysis, content briefs, quick wins, weekly reports. Arguments: auto, init, weekly, quick-wins, brief, publish."
metadata:
  version: 2.0.0
---

# GSC SEO Autopilot

Automated SEO pipeline: GSC monitoring, keyword expansion, quick wins, and blog content creation. Works with any website.

## Arguments

Route based on the first argument provided by the user:

| Argument | Reference File | Description |
|----------|---------------|-------------|
| `auto` | `references/cmd-auto.md` | Full pipeline: GSC analysis + keyword expansion + blog creation |
| `init` | `references/cmd-init.md` | Interactive config setup wizard |
| `weekly` | `references/cmd-weekly.md` | GSC performance report |
| `quick-wins` | `references/cmd-quick-wins.md` | Find low-hanging SEO opportunities |
| `brief <keyword>` | `references/cmd-brief.md` | Generate content brief for a keyword |
| `publish <keyword>` | `references/cmd-publish.md` | Write a single blog post for a keyword |

**Default (no argument):** Run `auto` pipeline.

**Procedure:** Read the matching reference file and follow its instructions.

## Config Loading

Every command MUST start by reading `seo-config.yaml` from this skill's directory:

```
~/.claude/skills/gsc-seo-autopilot/seo-config.yaml
```

If missing, inform user and suggest running `/gsc-seo-autopilot init`.

## Keywords Everywhere (MCP)

Keyword data comes from the hosted Keywords Everywhere MCP server. Auth is handled by the MCP
client, so no API key is read at run time. If the `mcp__keywords-everywhere__*` tools are
unavailable, skip keyword expansion and fall back to `WebSearch` -- never fail the run.

### Tools

| Tool | Required args | Returns | Credits |
|------|--------------|---------|---------|
| `get_related_keywords` | `keyword`, `num` | `data`: array of keyword strings | 2 per keyword returned |
| `get_pasf_keywords` | `keyword`, `num` | `data`: array of "People Also Search For" strings | 2 per keyword returned |
| `get_keyword_data` | `kw`, `country`, `currency`, `dataSource` | `data[]`: `{keyword, vol, cpc:{currency,value}, competition, trend[]}` | 1 per keyword |
| `get_domain_keywords` | `domain`, `country`, `num` | `data[]`: `{keyword, estimated_monthly_traffic, serp_position}` | 2 per keyword |
| `get_credit_balance` | -- | array with one integer | free |

Every argument is required -- none have defaults. Note that `get_related_keywords` and
`get_pasf_keywords` take **only** `keyword` and `num`; `country`, `currency`, and `dataSource`
do not apply to them and are used solely by `get_keyword_data`.

### Limits and cost

- `get_keyword_data` accepts a maximum of **100 keywords per call**. Chunk longer lists.
- `num` accepts up to 10,000, but expansion costs 2 credits per keyword *returned*. Pass
  `ke_related_limit` from config -- never a large literal.
- Check `get_credit_balance` before a run that expands many seeds.

Map config values to arguments: `ke_country` -> `country`, `ke_currency` -> `currency`,
`ke_data_source` -> `dataSource`, `ke_related_limit` -> `num`.

## Blog Writing

Follow `references/blog-writing-guide.md`. Do NOT delegate to external skills.

## Setup

1. **GSC MCP server** (see README)
2. **Keywords Everywhere MCP server** (optional, see README)
3. **seo-config.yaml** in skill directory (run `/gsc-seo-autopilot init` or copy `seo-config.example.yaml`)
