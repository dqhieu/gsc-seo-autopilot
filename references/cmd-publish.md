---
name: publish
description: "Write a single SEO blog post for a target keyword"
user_invocable: true
args:
  - name: keyword
    description: "Target keyword for the blog post"
    required: true
---

# Publish Single Blog Post

Write a single SEO-optimized blog post and save it to the configured blog directory.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot init`.

### 2. Generate Content Brief

Follow the same SERP research and keyword expansion steps from `commands/brief.md`:
- WebSearch the target keyword
- Analyze top 10 SERP results
- Pull GSC data for existing rankings
- Expand keywords via Keywords Everywhere API (if key available)

### 3. Write Blog Post

Follow the instructions in `references/blog-writing-guide.md` to create the blog post:

1. Research the keyword thoroughly (SERP analysis from step 2)
2. Write 1500-2500 word SEO-optimized post
3. Generate proper frontmatter using `frontmatter_format` from config
4. Include internal links from `internal_link_targets` config
5. Include CTA using `app_url` config (if set)
6. Add FAQ section with schema-ready Q&A
7. Download a relevant thumbnail from Unsplash

Save the post to `{blog_post_dir}/{slug}.md` relative to the project root.

### 4. Update Blog Index (if configured)

If `blog_index_file` is set in config:
- Read the existing index file
- Add the new post entry (title, slug, date, category, thumbnail)
- Write the updated index file

### 5. Output Summary

Tell the user:
- Post file path
- Target keyword and estimated volume
- Word count and category
- Suggested next steps (review content, commit when ready, update sitemap)
