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
site_url: {gsc_property}
start_date: (90 days ago)
end_date: (yesterday)
dimensions: ["query", "page"]
query_contains: "{keyword}"
row_limit: 20
```

### 4. Keyword Expansion

Use Keywords Everywhere API to get related keywords and volume data:

**Related keywords:**
```bash
node -e "
const key = process.env['{keywords_everywhere_api_key_env}'];
if (!key) { console.log('SKIP: API key not set'); process.exit(0); }
const https = require('https');
const data = new URLSearchParams();
data.append('apiKey', key);
data.append('keyword', '{keyword}');
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

**Volume data for keyword cluster:**
```bash
node -e "
const key = process.env['{keywords_everywhere_api_key_env}'];
if (!key) { console.log('SKIP: API key not set'); process.exit(0); }
const https = require('https');
const keywords = JSON.stringify(['KEYWORD_1', 'KEYWORD_2', 'KEYWORD_3']);
const data = new URLSearchParams();
data.append('apiKey', key);
data.append('country', '{ke_country}');
data.append('currency', '{ke_currency}');
data.append('dataSource', '{ke_data_source}');
data.append('kw[]', 'KEYWORD_1');
data.append('kw[]', 'KEYWORD_2');
const opts = { hostname: 'api.keywordseverywhere.com', path: '/v1/get_keyword_data', method: 'POST', headers: { 'Content-Type': 'application/x-www-form-urlencoded' } };
const req = https.request(opts, res => { let b=''; res.on('data',c=>b+=c); res.on('end',()=>console.log(b)); });
req.write(data.toString());
req.end();
"
```

If API key is not set, use WebSearch "related searches" and "people also ask" data instead.

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
