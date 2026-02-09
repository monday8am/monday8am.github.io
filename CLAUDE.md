# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Site Overview

Personal website and blog for monday8am.com, built with Jekyll and hosted on GitHub Pages.

## Commands

```bash
# Install dependencies
bundle install

# Serve locally with live reload
bundle exec jekyll serve

# Build the site (output goes to _site/)
bundle exec jekyll build
```

## Architecture

- **Jekyll** static site using the **minima** theme with custom layouts and styles
- Hosted via **GitHub Pages** (CNAME: monday8am.com)
- Markdown processor: **kramdown** with GFM input and Rouge syntax highlighting
- Plugins: `jekyll-feed`, `jekyll-gist`

### Layouts

Three layouts chained as: `post.html` / `page.html` → `default.html`. The default layout includes `head.html`, `header.html`, and `footer.html` from `_includes/`.

### Post Categories

Posts in `_posts/` use two categories in their front matter:
- `blog` — technical articles and personal posts (shown on homepage under "Blog" and on /writings/)
- `project` — project showcases (shown on homepage under "Projects")
- `design` — shown on /writings/ under "Product Design"

### Post Front Matter

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS +0100
categories: blog          # or: project, design
mermaid: true             # optional: enables Mermaid.js diagrams
---
```

Post filenames follow the pattern `YYYY-MM-DD-slug.md`.

### Mermaid Diagrams

Setting `mermaid: true` in front matter loads Mermaid.js (v10.9.0) from CDN. Diagrams are written as fenced code blocks with the `mermaid` language tag and rendered client-side.

### Pages

Navigation pages live in `_pages/` with numeric prefixes for ordering (e.g., `01_articles.md`, `02_about.md`). The `_pages` directory is explicitly included in `_config.yml`.

### Styles

Custom SCSS partials in `_sass/` are imported through `css/main.scss`. The site uses JetBrains Mono and Work Sans fonts loaded from Google Fonts.
