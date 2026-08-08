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
- `ke_country`, `ke_currency`, `ke_data_source`, `ke_related_limit`
- `app_url`, `internal_link_targets`
- `posts_per_run`, `report_dir`, `seed_keyword_count`

### 2. GSC Performance Analysis

Use `mcp__gsc__search_analytics` to pull data:

**Top queries (last 28 days):**
```
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "query"
rowLimit: 100
```

**Top pages (last 28 days):**
```
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "page"
rowLimit: 50
```

**Query + page combinations for cannibalization check:**
```
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "query,page"
rowLimit: 200
```

### 3. Quick Wins Detection

Use `mcp__gsc__detect_quick_wins` if available. `startDate` and `endDate` are required; the
threshold defaults (position 4-10, CTR <= 2%) are narrower than the criteria below, so pass
them explicitly:
```
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
positionRangeMin: 5
positionRangeMax: 20
minImpressions: 30
maxCtr: 3
```

The tool ANDs all four thresholds, so its output is only the overlap of the categories below
(it cannot return a striking-distance keyword whose CTR is already above 3%). It also has no
row limit and returns every match, which can be a very large response on an established site.
Results are pre-sorted by `additionalClicks` descending -- the report only needs the top 10,
so read from the top and stop there. Treat it as a ranked starting point, not the full set —
always also identify manually from step 2 data:
- Striking distance keywords (position 5-15, impressions > 50)
- Low CTR / high impression keywords (impressions > 100, CTR < 3%)
- Page 2 keywords (position 11-20)

### 4. Keyword Expansion via Keywords Everywhere

If the `mcp__keywords-everywhere__*` tools are unavailable, skip to the fallback at the end of
this step. Do not fail the run.

Take the top `{seed_keyword_count}` queries by clicks. For each seed keyword, call
`mcp__keywords-everywhere__get_related_keywords`:

```
keyword: {seed keyword}
num: {ke_related_limit}
```

Returns `{"data": ["related keyword", ...], "credits_consumed": N}` -- a flat array of strings
with no metrics attached. `country`, `currency`, and `dataSource` are not parameters of this
tool; they apply only to `get_keyword_data` below.

Optionally also call `mcp__keywords-everywhere__get_pasf_keywords` with the same arguments for
"People Also Search For" terms, which surface question-shaped and adjacent-intent keywords that
related-keyword expansion tends to miss.

**Cost check before expanding.** Both tools charge 2 credits per keyword *returned*, so this
step costs roughly `{seed_keyword_count} x {ke_related_limit} x 2` credits (x2 again if you also
run PASF). At the defaults that is ~400 credits. Call
`mcp__keywords-everywhere__get_credit_balance` first if the balance is unknown, and never pass a
large literal to `num` -- always use `{ke_related_limit}`.

Collect and de-duplicate all related keywords, then batch-fetch metrics with
`mcp__keywords-everywhere__get_keyword_data`:

```
kw: [array of keywords, MAXIMUM 100 per call]
country: {ke_country}
currency: {ke_currency}
dataSource: {ke_data_source}
```

All four arguments are required. **Chunk the list into batches of 100** -- 10 seeds at the
default limit yields ~200 candidates, which is two calls. Each keyword costs 1 credit.

The response is `{"data": [...], "credits": N, "credits_consumed": N}` where each entry is:

| Field | Meaning |
|-------|---------|
| `keyword` | the input keyword |
| `vol` | monthly search volume (integer) |
| `cpc.value` | cost per click as a string, e.g. `"1.23"` |
| `cpc.currency` | currency symbol, e.g. `"$"` |
| `competition` | 0.0-1.0 float |
| `trend` | 12 objects of `{month, year, value}` |

Use `vol` and `competition` for the selection criteria in step 5. A keyword absent from `data`
has no Keyword Planner data -- treat its volume as unknown, not zero.

**Fallback if the MCP tools are unavailable:** use `WebSearch` for "related searches" and
"people also ask" results on the top seed keywords, and note in the report that volume and
competition figures were unavailable.

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
