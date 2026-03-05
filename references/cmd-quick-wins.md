---
name: quick-wins
description: "Find low-hanging SEO opportunities from GSC data"
user_invocable: true
---

# Quick Wins Scan

Identify low-hanging SEO opportunities where small changes can yield significant traffic gains.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot init`.

### 2. Pull GSC Data

Use `mcp__gsc__search_analytics` for the last 28 days:

```
site_url: {gsc_property}
start_date: (28 days ago)
end_date: (yesterday)
dimensions: ["query", "page"]
row_limit: 200
```

Also use `mcp__gsc__detect_quick_wins` if available:
```
site_url: {gsc_property}
```

### 3. Categorize Opportunities

Analyze data and group into these categories:

#### A. Striking Distance Keywords (Position 5-15)
Keywords ranking on page 1-2 that could reach top 3 with optimization.
- Filter: `position >= 5 AND position <= 15 AND impressions > 50`
- Sort by impressions descending
- These are the highest-priority quick wins

#### B. Low CTR / High Impressions
Keywords with lots of impressions but poor click-through rate.
- Filter: `impressions > 100 AND ctr < 0.03`
- These need better title tags and meta descriptions
- Sort by impressions descending

#### C. Page 2 Keywords (Position 11-20)
Keywords just off page 1 that need a content boost.
- Filter: `position >= 11 AND position <= 20 AND impressions > 30`
- These need content expansion, internal links, or backlinks

#### D. Cannibalization Candidates
Multiple pages ranking for the same keyword.
- Group by query, find queries with 2+ pages
- Recommend consolidation or canonical strategy

### 4. Generate Recommendations

For each opportunity, provide:

1. **The keyword and current metrics** (position, clicks, impressions, CTR)
2. **The page URL** currently ranking
3. **Specific action** to take:
   - Striking distance: "Optimize H1/title for [keyword], add internal links from [pages], expand content section on [topic]"
   - Low CTR: "Rewrite title tag to: [suggestion]. Rewrite meta description to: [suggestion]"
   - Page 2: "Add 300-500 words covering [subtopic], link from [high-authority internal page]"
   - Cannibalization: "Consolidate [page A] and [page B], redirect [lower] to [higher]"

### 5. Output Report

Format as:

```markdown
# Quick Wins Report -- {site_domain}

**Generated:** {today}
**Data Period:** Last 28 days

## Priority 1: Striking Distance (Position 5-15)

| Keyword | Page | Position | Impressions | Clicks | CTR | Action |
|---------|------|----------|-------------|--------|-----|--------|
(sorted by estimated traffic gain)

## Priority 2: Low CTR / High Impressions

| Keyword | Page | Position | Impressions | Clicks | CTR | Suggested Title |
|---------|------|----------|-------------|--------|-----|----------------|

## Priority 3: Page 2 Keywords

| Keyword | Page | Position | Impressions | Action |
|---------|------|----------|-------------|--------|

## Priority 4: Cannibalization Issues

| Keyword | Pages | Positions | Recommendation |
|---------|-------|-----------|---------------|

## Summary

- X striking distance keywords (estimated +Y clicks/month)
- X low CTR opportunities
- X page 2 keywords to push
- X cannibalization issues
```

Save to `{report_dir}/quick-wins-{YYYY-MM-DD}.md` relative to the project root.
