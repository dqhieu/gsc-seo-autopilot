---
name: brief
description: "Generate a content brief for a target keyword"
user_invocable: true
args:
  - name: keyword
    description: "Target keyword to create a content brief for"
    required: true
---

# Content Brief Generator

Create a detailed content brief for a target keyword, including SERP analysis, keyword data, and writing outline.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot init`.

### 2. SERP Research

Use `WebSearch` to search for the target keyword. Analyze the top 10 results:
- Note the title patterns (what words/structures are common)
- Note the content types (listicle, guide, comparison, tutorial)
- Note the H2/H3 structure of top-ranking pages
- Identify content gaps (what the top results miss)
- Note average content length (estimate from snippets)

### 3. GSC Data for This Keyword

Use `mcp__gsc__search_analytics` to check if the site already ranks for this keyword or related terms:

```
siteUrl: {gsc_property}
startDate: (90 days ago, YYYY-MM-DD)
endDate: (yesterday, YYYY-MM-DD)
dimensions: "query,page"
queryFilter: "{keyword}"
filterOperator: contains
rowLimit: 20
```

### 4. Keyword Expansion

Use the Keywords Everywhere MCP tools to get related keywords and volume data.

**Related keywords** -- `mcp__keywords-everywhere__get_related_keywords`:
```
keyword: {keyword}
num: {ke_related_limit}
```

**"People Also Search For"** -- `mcp__keywords-everywhere__get_pasf_keywords`, same arguments.
PASF terms are usually question-shaped, which makes them good H2/H3 and FAQ candidates for the
outline below.

Both return `{"data": ["keyword", ...]}` -- strings only, no metrics -- and both cost 2 credits
per keyword returned. Neither accepts `country`, `currency`, or `dataSource`.

**Volume data for the keyword cluster** -- `mcp__keywords-everywhere__get_keyword_data`:
```
kw: [{keyword}, plus the related and PASF terms -- MAXIMUM 100 per call]
country: {ke_country}
currency: {ke_currency}
dataSource: {ke_data_source}
```

All four arguments are required. Costs 1 credit per keyword. Each entry in the returned `data`
array has `keyword`, `vol` (monthly volume), `cpc.value` and `cpc.currency`, `competition`
(0.0-1.0), and `trend` (12 months of `{month, year, value}`).

Populate the brief's metrics table from these fields: Monthly Volume from `vol`, CPC from
`cpc.currency` + `cpc.value`, Competition from `competition`. A keyword missing from `data` has
no Keyword Planner coverage -- render "N/A" rather than 0. The `trend` array shows whether the
keyword is seasonal, which is worth a line in the brief when the swing is large.

If the `mcp__keywords-everywhere__*` tools are unavailable, use WebSearch "related searches" and
"people also ask" data instead, and render the metrics table as "N/A".

### 5. Compile Content Brief

```markdown
# Content Brief: {keyword}

**Generated:** {today}
**Site:** {site_domain}

## Keyword Data

| Metric | Value |
|--------|-------|
| Primary Keyword | {keyword} |
| Monthly Volume | {volume or "N/A"} |
| CPC | {cpc or "N/A"} |
| Competition | {competition or "N/A"} |
| Current Position | {position or "Not ranking"} |

## Secondary Keywords

| Keyword | Volume | CPC | Competition |
|---------|--------|-----|-------------|
(from keyword expansion)

## SERP Analysis

**Content Type:** {dominant type: guide/listicle/comparison/etc.}
**Average Length:** {estimated word count}
**Common Title Patterns:** {patterns}

### Top Competitors

| # | Title | URL | Key Angles |
|---|-------|-----|------------|
(top 5 SERP results)

## Content Gaps

(What the top results miss that we can cover)

## Recommended Outline

### Title Options
1. {title option 1 -- matches SERP patterns}
2. {title option 2 -- differentiated angle}

### Meta Description
{150-160 char meta description with keyword}

### H2/H3 Structure

1. **{H2: Introduction/Hook}**
2. **{H2: Main section 1}**
   - {H3: subsection}
   - {H3: subsection}
3. **{H2: Main section 2}**
   - {H3: subsection}
4. **{H2: Main section 3}**
5. **{H2: FAQ}**
   - {Q1}
   - {Q2}
   - {Q3}
6. **{H2: Conclusion + CTA}**

### Internal Links
(Suggest links to these pages from config `internal_link_targets`:)
{list internal_link_targets with anchor text suggestions}

### CTA
{If app_url is set: "Link to {app_url} with CTA: [suggestion]"}
{If empty: "Add a relevant CTA for your product/service"}

## Writing Notes

- Target word count: 1500-2500 words
- Tone: informative, practical, {product_name}-aware
- Include at least 3 FAQ items for schema markup
- Content pillars alignment: {matching content_pillars}
- Category: {suggested blog_categories match}
```

### 6. Save Brief

Save to `{report_dir}/brief-{keyword-slug}-{YYYY-MM-DD}.md` relative to the project root.
