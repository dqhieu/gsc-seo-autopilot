---
name: weekly
description: "Generate a weekly GSC performance report with keyword analysis"
user_invocable: true
---

# Weekly GSC Report

Generate a comprehensive weekly SEO performance report from Google Search Console data.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot init`.

### 2. Pull GSC Data

Use `mcp__gsc__search_analytics` to fetch the last 28 days of data for the configured `gsc_property`:

**Query 1: Top queries (last 28 days)**
```
site_url: {gsc_property}
start_date: (28 days ago, YYYY-MM-DD)
end_date: (yesterday, YYYY-MM-DD)
dimensions: ["query"]
row_limit: 50
```

**Query 2: Top pages (last 28 days)**
```
site_url: {gsc_property}
start_date: (28 days ago, YYYY-MM-DD)
end_date: (yesterday, YYYY-MM-DD)
dimensions: ["page"]
row_limit: 30
```

**Query 3: Previous period comparison (28-56 days ago)**
```
site_url: {gsc_property}
start_date: (56 days ago, YYYY-MM-DD)
end_date: (29 days ago, YYYY-MM-DD)
dimensions: ["query"]
row_limit: 50
```

### 3. Keywords Everywhere Expansion

Take the top `{seed_keyword_count}` queries by clicks from GSC data. For each seed keyword, call the Keywords Everywhere API to get related keywords:

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

If the API key is not set, skip this step and note it in the report.

### 4. Analyze & Compile Report

Create a report with the following sections:

```markdown
# SEO Weekly Report -- {site_domain}

**Period:** {start_date} to {end_date}
**Generated:** {today}

## Performance Summary

| Metric | This Period | Previous Period | Change |
|--------|-----------|----------------|--------|
| Total Clicks | X | Y | +/-Z% |
| Total Impressions | X | Y | +/-Z% |
| Average CTR | X% | Y% | +/-Z pp |
| Average Position | X | Y | +/-Z |

## Top Queries by Clicks

| # | Query | Clicks | Impressions | CTR | Position |
|---|-------|--------|-------------|-----|----------|
(top 20 queries)

## Biggest Movers (vs Previous Period)

### Gainers
(queries with largest positive click/position change)

### Decliners
(queries with largest negative click/position change)

## Top Pages

| # | Page | Clicks | Impressions | CTR | Position |
|---|------|--------|-------------|-----|----------|
(top 15 pages)

## Keyword Expansion Opportunities

(Related keywords from Keywords Everywhere with volume/CPC data, grouped by seed keyword. Only include if API key was available.)

## Recommendations

1. (Actionable recommendation based on data)
2. (...)
3. (...)
```

### 5. Save Report

Save the report to `{report_dir}/seo-weekly-{YYYY-MM-DD}.md` relative to the project root.

Output a summary to the user with key highlights from the report.
