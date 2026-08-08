# GSC SEO Autopilot

Automated SEO pipeline for Claude Code. Monitors Google Search Console, expands keywords, identifies quick wins, and creates SEO-optimized blog content -- for any website.

## What It Does

- Pulls performance data from Google Search Console via MCP
- Expands seed keywords using Keywords Everywhere API
- Identifies quick wins (striking distance, low CTR, cannibalization)
- Generates content briefs with SERP analysis
- Writes SEO blog posts with proper frontmatter, internal links, and FAQ sections

## Prerequisites

- [Claude Code](https://claude.ai/code) with MCP support
- Google Search Console access for your site
- Node.js (for Keywords Everywhere API calls)

## Setup

### 1. Google Search Console MCP Server

The skill uses `mcp-server-gsc` to access your GSC data.

**a) Create a Google Cloud Service Account:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select existing)
3. Enable the **Google Search Console API**
4. Go to **IAM & Admin > Service Accounts**
5. Create a service account
6. Create a JSON key and download it
7. In Google Search Console, add the service account email as a **user** with read access

**b) Add MCP Server to Claude Code:**

Add to `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "gsc": {
      "command": "npx",
      "args": ["-y", "mcp-server-gsc"],
      "env": {
        "GOOGLE_APPLICATION_CREDENTIALS": "/absolute/path/to/your-credentials.json"
      }
    }
  }
}
```

**c) Verify:**

Restart Claude Code. The `mcp__gsc__list_sites` tool should show your GSC properties.

### 2. Keywords Everywhere MCP Server (Optional)

Provides keyword volume, CPC, competition, related keywords, and "People Also Search For" data.
The skill works without it (using GSC + WebSearch), but keyword expansion will be limited.

1. Sign up at [keywordseverywhere.com](https://keywordseverywhere.com/) and get your API key
2. Add the hosted MCP server:

```bash
claude mcp add --transport http keywords-everywhere \
  https://mcp.keywordseverywhere.com/mcp \
  -H "Authorization: Bearer your-key-here"
```

3. Restart Claude Code, then run `claude mcp list` to confirm it shows **✔ Connected**

The API key doubles as the bearer token, so no separate OAuth step is needed. Other transports
(stdio via `mcp-remote`, OAuth) are documented in the
[KE MCP integration guide](https://api.keywordseverywhere.com/docs/#/mcp_integration); the HTTP
transport above is the one Keywords Everywhere recommends for Claude Code, and it avoids running
a local Node wrapper.

**Credit costs.** Related-keyword and PASF expansion cost **2 credits per keyword returned**;
`get_keyword_data` costs **1 credit per keyword**. A default `auto` run (10 seeds x 20 related)
is roughly 400-600 credits. Raising `ke_related_limit` scales that linearly.

### 3. Install the Skill

**Recommended: Using Skills CLI**

```bash
npx skills add dqhieu/gsc-seo-autopilot
```

**Option B: Clone directly into skills directory**

```bash
git clone https://github.com/dqhieu/gsc-seo-autopilot.git ~/.claude/skills/gsc-seo-autopilot
```

**Option C: Clone elsewhere and symlink**

```bash
git clone https://github.com/dqhieu/gsc-seo-autopilot.git ~/Projects/gsc-seo-autopilot
ln -s ~/Projects/gsc-seo-autopilot ~/.claude/skills/gsc-seo-autopilot
```

### 4. Configure

Run the interactive setup wizard:

```
/gsc-seo-autopilot init
```

Or manually copy and edit the config:

```bash
cp ~/.claude/skills/gsc-seo-autopilot/seo-config.example.yaml ~/.claude/skills/gsc-seo-autopilot/seo-config.yaml
```

Edit `seo-config.yaml` with your site details.

## Usage

The skill is invoked as `/gsc-seo-autopilot` with an argument:

| Command | Description |
|---------|-------------|
| `/gsc-seo-autopilot auto` | Full pipeline: GSC analysis + keyword expansion + blog creation |
| `/gsc-seo-autopilot init` | Interactive configuration wizard |
| `/gsc-seo-autopilot weekly` | Generate weekly GSC performance report |
| `/gsc-seo-autopilot quick-wins` | Find low-hanging SEO opportunities |
| `/gsc-seo-autopilot brief <keyword>` | Generate content brief for a keyword |
| `/gsc-seo-autopilot publish <keyword>` | Write a single blog post for a keyword |

Running `/gsc-seo-autopilot` without arguments defaults to `auto`.

## Configuration Reference

All settings in `seo-config.yaml`:

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| `site_domain` | Yes | -- | Your website domain (e.g., `example.com`) |
| `gsc_property` | Yes | -- | GSC property (e.g., `sc-domain:example.com`) |
| `product_name` | Yes | -- | Your product/brand name |
| `product_description` | Yes | -- | One-line product description |
| `blog_post_dir` | No | `content/blog` | Where markdown posts are saved |
| `blog_index_file` | No | `""` | Optional JSON/YAML file listing posts |
| `blog_url_prefix` | No | `/blog` | URL prefix for blog posts |
| `frontmatter_format` | No | `yaml` | Frontmatter format: `yaml` or `toml` |
| `content_pillars` | No | `[]` | 3-6 topic areas your blog covers |
| `blog_categories` | No | `[]` | Blog post categories/tags |
| `ke_country` | No | `us` | Country code for keyword data (`get_keyword_data` only) |
| `ke_currency` | No | `usd` | Currency for CPC data (`get_keyword_data` only) |
| `ke_data_source` | No | `gkp` | KE data source: `gkp` or `cli` (`get_keyword_data` only) |
| `ke_related_limit` | No | `20` | Max related keywords per seed (2 credits each) |
| `app_url` | No | `""` | Primary CTA URL for blog posts |
| `internal_link_targets` | No | `[]` | Pages to cross-link from blog posts |
| `posts_per_run` | No | `7` | Blog posts created per auto run |
| `report_dir` | No | `plans/reports` | Where reports are saved |
| `seed_keyword_count` | No | `10` | Top GSC keywords used as seeds |

## Example Output

After running `/gsc-seo-autopilot auto`, you get:

- **GSC performance report** in `plans/reports/seo-auto-2026-03-05.md`
- **7 blog posts** saved to your configured blog directory
- **Quick wins list** with actionable recommendations
- **Keyword expansion data** with volume and competition metrics

## How It Works

```
GSC Data (MCP)
     |
     v
Top Keywords ──> Keywords Everywhere API ──> Expanded Keyword List
     |                                              |
     v                                              v
Quick Wins                                  Keyword Selection
Analysis                                    (filter by pillars,
     |                                       volume, competition)
     v                                              |
Quick Wins                                          v
Report                                      Blog Post Creation
                                            (SERP research,
                                             write, thumbnail)
```

## License

MIT
