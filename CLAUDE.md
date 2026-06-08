# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Personal blog (Chinese, `zh-CN`) powered by Jekyll 4.4 and deployed via GitHub Pages at `shawnming068.github.io`. Content covers AI learning notes and personal reflections on life/attention/reading.

## Commands

```bash
# Install dependencies
bundle install

# Development server (auto-rebuild, live at http://localhost:4000)
bundle exec jekyll serve

# One-shot build to _site/
bundle exec jekyll build
```

## Architecture

### Tech stack

- **Jekyll 4.4** with kramdown markdown and `permalink: pretty`
- **Ruby 4.0+** with Bundler
- **All styles are inline** in `_layouts/default.html` (no external CSS, no Sass pipeline). Design tokens are CSS custom properties defined in `:root`.
- **No JavaScript**, no external fonts, no build toolchain beyond Jekyll itself.

### Content flow

Every content page (`*.html`) uses Jekyll frontmatter to inject into the default layout:

```
---
layout: default
title: 页面标题
---
```

`_layouts/default.html` wraps `{{ content }}` inside `<main>`, with a shared header (site branding + nav) and footer. The template reads `site.lang`, `site.title`, and `page.title` from `_config.yml` and each page's frontmatter.

### File responsibilities

| Path | Role |
|---|---|
| `_config.yml` | Site title, lang, markdown engine, permalink style, build excludes |
| `_layouts/default.html` | Single shared layout: `<head>` + inline CSS + header/nav/footer chrome |
| `index.html` | Homepage: hero, AI notes grid, life notes journal band, closing CTA |
| `Gemfile` / `Gemfile.lock` | Ruby dependency lock (Jekyll 4.4, liquid) |
| `docs/` | Reserved directory for future static content (currently empty) |

### Design system

- The CSS uses **custom properties** for theming (see `:root` in `default.html`): ink colors, paper backgrounds, accent tones (moss, cobalt, rust, gold).
- Typography favors serif typefaces for body text (`Iowan Old Style`, `Songti SC`) and sans-serif for labels/nav (`Avenir Next`, `PingFang SC`).
- Layout is **CSS Grid**-based, with responsive breakpoints at 860px and 520px.
- Visual aesthetic is editorial/print-inspired: grid-paper backgrounds, subtle radial gradients, card shadows, and a dark signal-board sidebar.

### GitHub Pages note

The `Gemfile.lock` is committed intentionally — GitHub Pages respects the locked Jekyll version rather than using its default, which is necessary because this site targets Jekyll 4.4 (newer than GitHub Pages' bundled version). The `.gitignore` excludes `_site/` and `.jekyll-cache/`.
