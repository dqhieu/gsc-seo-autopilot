---
name: publish
description: "Write and publish a single SEO blog post on a new branch with PR"
user_invocable: true
args:
  - name: keyword
    description: "Target keyword for the blog post"
    required: true
---

# Publish Single Blog Post

Write a single SEO-optimized blog post, commit it on a new branch, and create a pull request.

## Procedure

### 1. Load Config

Read `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml`. Parse all values. If missing, tell the user to run `/gsc-seo-autopilot:init`.

### 2. Generate Content Brief

Follow the same SERP research and keyword expansion steps from `commands/brief.md`:
- WebSearch the target keyword
- Analyze top 10 SERP results
- Pull GSC data for existing rankings
- Expand keywords via Keywords Everywhere API (if key available)

### 3. Create Git Branch

```bash
git checkout -b blog/{keyword-slug}-{timestamp}
```

Where `{keyword-slug}` is the keyword converted to lowercase kebab-case and `{timestamp}` is `YYYYMMDD-HHMM`.

### 4. Write Blog Post

Follow the instructions in `references/blog-writing-guide.md` to create the blog post:

1. Research the keyword thoroughly (SERP analysis from step 2)
2. Write 1500-2500 word SEO-optimized post
3. Generate proper frontmatter using `frontmatter_format` from config
4. Include internal links from `internal_link_targets` config
5. Include CTA using `app_url` config (if set)
6. Add FAQ section with schema-ready Q&A
7. Download a relevant thumbnail from Unsplash

Save the post to `{blog_post_dir}/{slug}.md` relative to the project root.

### 5. Update Blog Index (if configured)

If `blog_index_file` is set in config:
- Read the existing index file
- Add the new post entry (title, slug, date, category, thumbnail)
- Write the updated index file

### 6. Commit and Push

```bash
git add {blog_post_dir}/{slug}.md
# Also add thumbnail and blog_index_file if they were created/modified
git commit -m "feat(blog): add post -- {post title}"
git push -u origin blog/{keyword-slug}-{timestamp}
```

### 7. Create Pull Request

```bash
gh pr create --title "feat(blog): {post title}" --body "$(cat <<'EOF'
## Summary

- New SEO blog post targeting: **{keyword}**
- Word count: ~{word_count}
- Category: {category}

## SEO Details

- Primary keyword: {keyword}
- Secondary keywords: {list}
- Target search volume: {volume or "N/A"}

## Checklist

- [ ] Content reviewed for accuracy
- [ ] Internal links verified
- [ ] Meta description optimized
- [ ] Thumbnail included
- [ ] FAQ section present
EOF
)"
```

### 8. Output Summary

Tell the user:
- PR URL
- Post file path
- Target keyword and estimated volume
- Suggested next steps (review PR, update sitemap after merge)
