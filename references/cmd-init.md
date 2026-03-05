---
name: init
description: "Interactive setup wizard for GSC SEO Autopilot configuration"
user_invocable: true
---

# SEO Autopilot Init Wizard

Interactive wizard to create `seo-config.yaml` for this skill.

## Steps

### 1. Check GSC MCP Availability

Try calling `mcp__gsc__list_sites`. If it fails or is unavailable:
- Inform the user that the GSC MCP server is not configured
- Show them how to set it up:
  1. Install: `npx -y mcp-server-gsc`
  2. Add to `~/.claude/mcp.json`:
  ```json
  {
    "mcpServers": {
      "gsc": {
        "command": "npx",
        "args": ["-y", "mcp-server-gsc"],
        "env": {
          "GOOGLE_APPLICATION_CREDENTIALS": "/path/to/credentials.json"
        }
      }
    }
  }
  ```
  3. Restart Claude Code
- Ask if they want to continue setup without GSC verification

### 2. Gather Site Information

Use `AskUserQuestion` to collect:

**Question 1: Site Domain**
- "What is your website domain? (e.g., example.com)"
- No default

**Question 2: GSC Property**
- If `mcp__gsc__list_sites` succeeded, show available properties and let user select
- Otherwise ask: "What is your GSC property? (e.g., sc-domain:example.com)"
- Default: `sc-domain:{domain from Q1}`

**Question 3: Product Name & Description (auto-detected)**
- Use `WebFetch` to load the site's homepage (`https://{domain}`) and extract:
  - The product/brand name from the `<title>` tag or `<meta property="og:site_name">`
  - A one-line description from `<meta name="description">` or `<meta property="og:description">`
- Present the auto-detected values to the user for confirmation:
  - "Detected product name: **{detected_name}**. Is this correct? (yes/edit)"
  - "Detected description: **{detected_description}**. Is this correct? (yes/edit)"
- If the homepage fetch fails, fall back to asking the user directly:
  - "What is your product/brand name?"
  - "One-line description of what your product does:"

### 3. Blog Settings

**Question 4: Blog Post Directory**
- "Where should blog posts be saved? (relative to repo root)"
- Options: `content/blog`, `src/posts`, `app/blog/posts`, `blog`, custom
- Default: `content/blog`

**Question 5: Blog URL Prefix**
- "What is the URL prefix for blog posts? (e.g., /blog)"
- Default: `/blog`

**Question 6: Frontmatter Format**
- "What frontmatter format do you use?"
- Options: `yaml`, `toml`
- Default: `yaml`

### 4. Content Strategy

**Question 7: Content Pillars**
- "List 3-6 topic areas your blog covers (comma-separated):"
- Example: "productivity, project management, team collaboration"

**Question 8: Blog Categories**
- "List blog post categories/tags you use (comma-separated):"
- Example: "how-to, comparison, guide, tutorial"

### 5. Keywords Everywhere API

**Question 9: API Key Validation**
- Check if `KEYWORDS_EVERYWHERE_API_KEY` env var is set
- If set: "Keywords Everywhere API key detected. Using default env var name."
- If not set: "Keywords Everywhere API key not found. The skill will work without it (using GSC + WebSearch), but keyword expansion will be limited. You can add it later to your shell profile."

**Question 10: Target Country**
- "What country should keyword data target?"
- Options: `us`, `gb`, `au`, `ca`, `de`, `fr`, `in`, custom
- Default: `us`

**Question 11: Currency**
- "What currency for CPC data?"
- Options: `usd`, `gbp`, `eur`, `aud`, `cad`, custom
- Default: `usd`

### 6. Optional Settings

**Question 12: App/CTA URL**
- "Primary CTA URL for blog posts (e.g., signup page, app download). Leave empty to skip:"
- Default: empty

**Question 13: Internal Link Targets**
- "Key pages to cross-link from blog posts (comma-separated URLs, or empty to skip):"
- Default: empty

### 7. Write Config

Assemble all answers into a YAML file and write to:
```
~/.claude/skills/gsc-seo-autopilot/seo-config.yaml
```

Use the `Write` tool. Include comments explaining each field.

### 8. Confirmation

Output:
- Summary of the configuration
- Path where config was saved
- Suggest next steps: "Run `/gsc-seo-autopilot weekly` to generate your first GSC report, or `/gsc-seo-autopilot auto` for the full pipeline."
