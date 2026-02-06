# Odovey Consulting Blog

Blog content for the [Odovey Consulting website](https://odovey-consulting.github.io/).

## Structure

```
posts/
  YYYY-MM-DD-slug-words.md
```

## Post Format

Each post is a markdown file with YAML frontmatter:

```yaml
---
title: "Post Title"
date: "YYYY-MM-DD"
excerpt: "A short description of the post..."
author: "Author Name"
tags:
  - tag1
  - tag2
draft: false
---

Post content in markdown here.
```

## How It Works

- Posts are fetched by the website repo at build time
- Pushing to `main` triggers a rebuild of the website via GitHub Actions
- The slug is derived from the filename: `2026-02-05-welcome-to-our-blog.md` becomes `welcome-to-our-blog`
- Set `draft: true` to hide a post in production (visible in dev)
