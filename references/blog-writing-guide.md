# Blog Writing Guide

Instructions for creating SEO-optimized blog posts. This guide is self-contained -- do NOT delegate to external blog creation skills.

## Before Writing

### 1. Read Config

Load `~/.claude/skills/gsc-seo-autopilot/seo-config.yaml` and extract:
- `blog_post_dir` -- where to save the post
- `blog_url_prefix` -- URL prefix for internal links
- `frontmatter_format` -- `yaml` or `toml`
- `content_pillars` -- align content with these topics
- `blog_categories` -- use one of these as the post category
- `app_url` -- CTA link (if set)
- `internal_link_targets` -- pages to cross-link to
- `product_name` -- for natural product mentions
- `product_description` -- for context on what the product does
- `blog_index_file` -- optional index to update

### 2. Research the Keyword

Use `WebSearch` to search the target keyword. Analyze:
- **Top 5 results:** Note their titles, H2 structure, content angles
- **People Also Ask:** Collect questions for the FAQ section
- **Related searches:** Note for secondary keyword coverage
- **Content gaps:** What do top results miss?

## Writing the Post

### 3. Frontmatter

**YAML format** (if `frontmatter_format: "yaml"`):
```yaml
---
title: "Post Title With Primary Keyword"
description: "150-160 character meta description containing the primary keyword naturally"
date: "YYYY-MM-DD"
category: "chosen-category"
tags: ["primary-keyword", "secondary-keyword", "related-term"]
thumbnail: "/path/to/thumbnail.jpg"
author: ""
---
```

**TOML format** (if `frontmatter_format: "toml"`):
```toml
+++
title = "Post Title With Primary Keyword"
description = "150-160 character meta description containing the primary keyword naturally"
date = "YYYY-MM-DD"
[taxonomies]
categories = ["chosen-category"]
tags = ["primary-keyword", "secondary-keyword"]
[extra]
thumbnail = "/path/to/thumbnail.jpg"
+++
```

### 4. Content Structure

Write 1500-2500 words following this structure:

**Introduction (100-150 words)**
- Hook: Start with a pain point, statistic, or question
- Context: Brief background on the topic
- Promise: What the reader will learn
- Include primary keyword naturally in the first paragraph

**Body Sections (1000-1800 words)**
- Use H2 headers for main sections (4-6 sections)
- Use H3 headers for subsections where needed
- Each H2 section should be 200-400 words
- Include primary keyword in at least 2 H2 headers
- Include secondary keywords in H3 headers
- Use bullet points and numbered lists for scanability
- Add a comparison table if the topic involves options/alternatives
- Include specific data, examples, or step-by-step instructions

**Internal Links**
- Link to pages from `internal_link_targets` config where relevant
- Use descriptive anchor text (not "click here")
- Aim for 2-4 internal links per post
- Link naturally within the content flow

**CTA Section**
- If `app_url` is set: include a natural call-to-action linking to it
- Frame the CTA around how the product solves the problem discussed
- Don't be overly promotional -- keep it helpful and relevant
- If `app_url` is empty: end with a general actionable takeaway

**FAQ Section (3-5 questions)**
- Use H2: "Frequently Asked Questions" or "FAQ"
- Each question as H3
- Answers should be 50-100 words each
- Use questions from "People Also Ask" in SERP research
- Structure for FAQ schema markup:

```markdown
## Frequently Asked Questions

### Question one here?

Answer one here, direct and concise.

### Question two here?

Answer two here, direct and concise.
```

**Conclusion (100-150 words)**
- Summarize key takeaways
- Reinforce the primary keyword topic
- End with a forward-looking statement or CTA

### 5. SEO Checklist

Before saving, verify:

- [ ] Primary keyword appears in: title, first paragraph, at least 2 H2s, conclusion
- [ ] Meta description is 150-160 characters and contains primary keyword
- [ ] H2/H3 structure is logical and includes keywords naturally
- [ ] Internal links to `internal_link_targets` pages are included
- [ ] FAQ section has 3-5 questions from "People Also Ask"
- [ ] Content is 1500-2500 words
- [ ] No keyword stuffing -- reads naturally
- [ ] Frontmatter format matches config

### 6. Thumbnail

Download a relevant image from Unsplash using bash/curl:

```bash
curl -sL "https://unsplash.com/photos/random?query={keyword-slug}" -o /dev/null
```

Alternatively, use a direct Unsplash URL pattern. Save the thumbnail to the same directory as the blog post or a configured images directory. Reference it in frontmatter.

If Unsplash download fails, note in the PR that a thumbnail needs to be added manually.

### 7. File Naming

Save as `{blog_post_dir}/{slug}.md` where `{slug}` is:
- Lowercase
- Hyphens instead of spaces
- No special characters
- Short but descriptive (3-6 words)
- Example: `compress-video-on-mac.md`

### 8. Update Blog Index (Optional)

If `blog_index_file` is set in config:

1. Read the existing file
2. Determine the format (JSON array, YAML list, etc.)
3. Add a new entry with:
   - `title`: Post title
   - `slug`: Filename without extension
   - `date`: Today's date
   - `category`: Selected category
   - `thumbnail`: Thumbnail path
   - `description`: Meta description
4. Write the updated file

If the index file doesn't exist yet, create it with the first entry.

## Writing Quality Standards

- **Tone:** Informative, practical, accessible. Not academic, not overly casual.
- **Sentences:** Vary length. Mix short punchy sentences with longer explanatory ones.
- **Paragraphs:** 2-4 sentences max. White space is your friend.
- **Jargon:** Define technical terms on first use if the target audience may not know them.
- **Examples:** Include concrete examples, not just abstract advice.
- **Accuracy:** Only state facts you can verify. Don't fabricate statistics or studies.
- **Product mentions:** If mentioning `{product_name}`, do so naturally where it genuinely helps. Maximum 2-3 mentions per post. Never force it.
