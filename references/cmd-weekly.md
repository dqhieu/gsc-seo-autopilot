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
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "query"
rowLimit: 50
```

**Query 2: Top pages (last 28 days)**
```
siteUrl: {gsc_property}
startDate: (28 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "page"
rowLimit: 30
```

**Query 3: Previous period comparison (28-56 days ago)**
```
siteUrl: {gsc_property}
startDate: (56 days ago, YYYY-MM-DD)
endDate: (29 days ago, YYYY-MM-DD)
dimensions: "query"
rowLimit: 50
```

### 3. Keywords Everywhere Expansion

Take the top `{seed_keyword_count}` queries by clicks from GSC data. For each seed keyword, call
`mcp__keywords-everywhere__get_related_keywords`:

```
keyword: {seed keyword}
num: {ke_related_limit}
```

Returns `{"data": ["related keyword", ...]}` -- keyword strings only, no metrics. This costs
2 credits per keyword returned.

To report volume and CPC alongside them, pass the collected keywords to
`mcp__keywords-everywhere__get_keyword_data` (all four arguments required, **maximum 100
keywords per call**, 1 credit each):

```
kw: [array of keywords, max 100]
country: {ke_country}
currency: {ke_currency}
dataSource: {ke_data_source}
```

Read `vol` for volume, `cpc.value` for CPC, and `competition` from each entry in `data`.

If the `mcp__keywords-everywhere__*` tools are unavailable, skip this step and note it in the
report.

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
