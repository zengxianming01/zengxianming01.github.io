# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Personal blog (Chinese, `zh-CN`) powered by Jekyll 4.4 and deployed via GitHub Pages at `shawnming068.github.io`. Content covers AI learning notes and personal reflections on life/attention/reading.

## Commands

```bash
# macOS: ensure Homebrew Ruby is used (not system Ruby)
export PATH="/opt/homebrew/opt/ruby/bin:/opt/homebrew/lib/ruby/gems/4.0.0/bin:$PATH"

# Install dependencies
bundle install

# Development server (auto-rebuild, live at http://localhost:4000)
bundle exec jekyll serve

# Use a different port if 4000 is occupied
bundle exec jekyll serve --port 4001

# One-shot build to _site/ (also useful as a syntax check after writing a post)
bundle exec jekyll build
```

## Architecture

### Tech stack

- **Jekyll 4.4** with kramdown markdown and `permalink: pretty`
- **Ruby 3.x+** with Bundler (Homebrew Ruby on macOS)
- **All styles are inline** in `_layouts/default.html` (no external CSS, no Sass pipeline). Design tokens are CSS custom properties defined in `:root`.
- **No external JavaScript**, no external fonts, no build toolchain beyond Jekyll itself. The only JS is an inline carousel script in `default.html`.

### Content flow

Every content page (`*.html`) uses Jekyll frontmatter to inject into the default layout:

```
---
layout: default
title: 页面标题
---
```

`_layouts/default.html` wraps `{{ content }}` inside `<main>`, with a shared header (site branding + nav) and footer. The template reads `site.lang`, `site.title`, and `page.title` from `_config.yml` and each page's frontmatter.

Posts use `layout: post`, which itself extends `default` and adds an article header (eyebrow, title, date, tags) around the content.

### Category system (data-driven)

Categories are defined centrally in `_data/categories.yml`. Each entry has a `slug` (used to match posts) plus display metadata (name, label, description, accent color, tags). This single file drives:

- **Homepage** (`index.html`): iterates `site.data.categories` to render the share-card grid
- **Categories index** (`categories.html`): full listing of all categories + posts grouped by category
- **Individual category pages** (`categories/*.html`): use `layout: category` with `category_slug` frontmatter to filter `site.posts` by that slug

To add a new category, you must do three things:
1. Add an entry to `_data/categories.yml`
2. Create a corresponding page in `categories/<slug>.html` (with `layout: category` and `category_slug`)
3. Use the slug as the `category` value in post frontmatter

### Posts

Posts live in `_posts/` with the standard Jekyll filename convention: `YYYY-MM-DD-slug.md` (or `.html`). Required frontmatter:

```yaml
---
layout: post
title: "Post title"
date: YYYY-MM-DD
category: <slug-from-categories.yml>
tags:
  - Tag1
  - Tag2
excerpt: "Short excerpt for listing pages."
---
```

Posts automatically appear on: homepage ("latest" section), their category page, the archive page, and their own permalink page.

Some posts include optional frontmatter fields carried over from Obsidian export (`source_path`, `type`). These are not consumed by templates but preserved for reference.

### HTML in posts

Posts can be HTML files (`.html` in `_posts/`), and Markdown posts can contain inline HTML. This is used for:

- **Image carousels**: a `<div class="carousel">` structure with track, buttons, dots, and counter, driven by the inline JS in `default.html`. Supports `data-autoplay` (milliseconds), touch swipe, and keyboard navigation.
- **Standalone images**: `<div class="solo-photo"><img ...></div>` for full-width photos.

### Obsidian note

Some posts originate from an Obsidian vault. Obsidian's `[[wikilink]]` syntax is **not** understood by Jekyll/kramdown — convert these to standard Markdown links or plain text before publishing.

### File responsibilities

| Path | Role |
|---|---|
| `_config.yml` | Site title, lang, markdown engine, permalink style, build excludes |
| `_data/categories.yml` | Source of truth for category definitions (slug, name, accent, tags) |
| `_layouts/default.html` | Single shared layout: `<head>` + inline CSS + header/nav/footer chrome + carousel JS |
| `_layouts/post.html` | Article layout (extends default): adds article header with meta + article body wrapper |
| `_layouts/category.html` | Category listing layout: matches `category_slug` to `_data/categories.yml`, lists matching posts |
| `index.html` | Homepage: hero, categories grid, latest posts, closing CTA |
| `categories.html` | All categories overview with post lists grouped by category |
| `categories/<slug>.html` | Individual category pages (thin: just frontmatter, delegates to `category` layout) |
| `archive.html` | Chronological list of all posts |
| `_posts/` | All blog posts (Markdown or HTML) |
| `Gemfile` / `Gemfile.lock` | Ruby dependency lock (Jekyll 4.4, liquid) |

### Design system

- The CSS uses **custom properties** for theming (see `:root` in `default.html`): ink colors, paper backgrounds, accent tones (moss, cobalt, rust, gold).
- Typography favors serif typefaces for body text (`Iowan Old Style`, `Songti SC`) and sans-serif for labels/nav (`Avenir Next`, `PingFang SC`).
- Layout is **CSS Grid**-based, with responsive breakpoints at 860px and 520px.
- Visual aesthetic is editorial/print-inspired: grid-paper backgrounds, subtle radial gradients, card shadows, and a dark signal-board sidebar.
- Category cards use accent classes (`accent-moss`, `accent-cobalt`, `accent-rust`, `accent-gold`) mapped from categories.yml to vary the dot color.

### GitHub Pages note

The `Gemfile.lock` is committed intentionally — GitHub Pages respects the locked Jekyll version rather than using its default, which is necessary because this site targets Jekyll 4.4 (newer than GitHub Pages' bundled version). The `.gitignore` excludes `_site/`, `.jekyll-cache/`, and `.sass-cache/`.
