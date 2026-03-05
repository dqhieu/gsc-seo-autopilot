---
name: auto
description: "Full SEO pipeline: GSC analysis, keyword expansion, and blog content creation"
user_invocable: true
---

# SEO Auto Pipeline

Full automated pipeline: pull GSC data, expand keywords, identify opportunities, and create blog posts.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot init`.

Store config values for use throughout this pipeline:
- `site_domain`, `gsc_property`, `product_name`, `product_description`
- `blog_post_dir`, `blog_index_file`, `blog_url_prefix`, `frontmatter_format`
- `content_pillars`, `blog_categories`
- `keywords_everywhere_api_key_env`, `ke_country`, `ke_currency`, `ke_data_source`, `ke_related_limit`
- `app_url`, `internal_link_targets`
- `posts_per_run`, `report_dir`, `seed_keyword_count`

### 2. GSC Performance Analysis

Use `mcp__gsc__search_analytics` to pull data:

**Top queries (last 28 days):**
```
site_url: {gsc_property}
start_date: (28 days ago, YYYY-MM-DD)
end_date: (yesterday, YYYY-MM-DD)
dimensions: ["query"]
row_limit: 100
```

**Top pages (last 28 days):**
```
site_url: {gsc_property}
start_date: (28 days ago, YYYY-MM-DD)
end_date: (yesterday, YYYY-MM-DD)
dimensions: ["page"]
row_limit: 50
```

**Query + page combinations for cannibalization check:**
```
site_url: {gsc_property}
start_date: (28 days ago, YYYY-MM-DD)
end_date: (yesterday, YYYY-MM-DD)
dimensions: ["query", "page"]
row_limit: 200
```

### 3. Quick Wins Detection

Use `mcp__gsc__detect_quick_wins` if available:
```
site_url: {gsc_property}
```

Also manually identify from step 2 data:
- Striking distance keywords (position 5-15, impressions > 50)
- Low CTR / high impression keywords (impressions > 100, CTR < 3%)
- Page 2 keywords (position 11-20)

### 4. Keyword Expansion via Keywords Everywhere

Take the top `{seed_keyword_count}` queries by clicks. For each seed keyword:

```bash
node -e "
const key = process.env['{keywords_everywhere_api_key_env}'];
if (!key) { console.log('SKIP: API key not set'); process.exit(0); }
const https = require('https');
const data = new URLSearchParams();
data.append('apiKey', key);
data.append('keyword', 'SEED_KEYWORD_HERE');
data.append('country', '{ke_country}');
data.append('currency', '{ke_currency}');
data.append('dataSource', '{ke_data_source}');
data.append('num', '{ke_related_limit}');
const opts = { hostname: 'api.keywordseverywhere.com', path: '/v1/get_related_keywords', method: 'POST', headers: { 'Content-Type': 'application/x-www-form-urlencoded' } };
const req = https.request(opts, res => { let b=''; res.on('data',c=>b+=c); res.on('end',()=>console.log(b)); });
req.write(data.toString());
req.end();
"
```

Collect all related keywords. Then batch-fetch volume data:

```bash
node -e "
const key = process.env['{keywords_everywhere_api_key_env}'];
if (!key) { console.log('SKIP: API key not set'); process.exit(0); }
const https = require('https');
const data = new URLSearchParams();
data.append('apiKey', key);
data.append('country', '{ke_country}');
data.append('currency', '{ke_currency}');
data.append('dataSource', '{ke_data_source}');
// Append each candidate keyword:
// data.append('kw[]', 'keyword1');
// data.append('kw[]', 'keyword2');
const opts = { hostname: 'api.keywordseverywhere.com', path: '/v1/get_keyword_data', method: 'POST', headers: { 'Content-Type': 'application/x-www-form-urlencoded' } };
const req = https.request(opts, res => { let b=''; res.on('data',c=>b+=c); res.on('end',()=>console.log(b)); });
req.write(data.toString());
req.end();
"
```

If API key is not set, use `WebSearch` for "related searches" and "people also ask" data for the top seed keywords.

### 5. Keyword Selection for Blog Posts

From all collected keywords (GSC + expanded), select `{posts_per_run}` keywords for new blog posts:

**Selection criteria (prioritize in order):**
1. Not already covered by existing blog posts (check existing files in `{blog_post_dir}`)
2. Aligns with `content_pillars` from config
3. Higher search volume preferred
4. Lower competition preferred
5. Commercial intent keywords get slight priority (contain "best", "how to", "vs", "review", "tool", "app")

### 6. Generate Content Briefs

For each selected keyword, create a brief content brief including:
- Primary and secondary keywords
- Suggested title (based on SERP patterns from WebSearch)
- H2/H3 outline
- Target word count (1500-2500)
- Category from `blog_categories`

### 7. Create Blog Posts

For each keyword, follow the instructions in `references/blog-writing-guide.md`:
1. Research via WebSearch
2. Write the full blog post (1500-2500 words)
3. Save to `{blog_post_dir}/{slug}.md`
4. Download thumbnail from Unsplash
5. Update `blog_index_file` if configured

All posts are saved to the configured `blog_post_dir`. The user can review and commit them at their discretion.

### 8. Compile Summary Report

Create a comprehensive report:

```markdown
# SEO Auto Pipeline Report -- {product_name}

**Generated:** {today}
**Site:** {site_domain}

## GSC Performance Summary

| Metric | Value |
|--------|-------|
| Total Clicks (28d) | X |
| Total Impressions (28d) | X |
| Average CTR | X% |
| Average Position | X |

## Quick Wins Identified

(List top 10 quick wins with action items)

## Keywords Expanded

| Seed Keyword | Related Keywords Found | Top Opportunity |
|-------------|----------------------|-----------------|
(for each seed)

## Blog Posts Created

| # | Title | Target Keyword | Volume | File |
|---|-------|---------------|--------|------|
(for each post created)

## Next Steps

1. Review created blog posts for accuracy
2. Commit and push when ready
3. Update sitemap after publishing
4. Monitor GSC for new keyword rankings in 2-4 weeks
5. Address quick wins (title/meta optimizations)
```

Save to `{report_dir}/seo-auto-{YYYY-MM-DD}.md`.

### 9. Output Summary

Display to the user:
- Number of posts created with file paths
- Top 5 quick wins to address manually
- Key metrics from GSC
- Next recommended run date (1 week from now)
