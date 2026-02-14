# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio site for jtcarrasco.com, built with Jekyll and hosted on GitHub Pages.

## Development Commands

```bash
# Install dependencies
bundle install

# Run local dev server (http://localhost:4000)
bundle exec jekyll serve

# Build for production
bundle exec jekyll build
```

## Architecture

**Jekyll site** using three collections configured in `_config.yml`:
- `_pages/` — standalone pages (about, contact, tags, uses)
- `_posts/` — blog posts (permalink: `/blog/:slug`)
- `_projects/` — portfolio projects (permalink: `/project/:slug`)
- `_project_drafts/` — unpublished project drafts (not built)

**Layouts** (`_layouts/`): `default.html` is the base; `page.html`, `post.html`, and `project.html` extend it.

**Central configuration** lives in `_data/settings.yml` — this drives navigation menus, hero text, social links, testimonials, and section content. Most site-wide changes should be made here rather than in templates.

**CSS** follows ITCSS methodology in `css/`:
- `_0-settings/` — variables, color schemes, mixins, Font Awesome
- `_1-tools/` — reset, grid, animations, third-party (tiny-slider)
- `_2-base/` — base element styles
- `_3-modules/` — component styles (header, footer, hero, projects, etc.)
- `_4-layouts/` — page-specific styles (post, project, tags)
- Entry point: `css/main.scss`

**JavaScript** (`js/`): `common.js` handles menu toggle, dark mode switching (localStorage-persisted), and scroll-to-top. `scripts.js` handles additional functionality.

## Key Patterns

- Images use lazy loading (`class="lazy"` with `data-src` attributes)
- Dark/light mode is set via `color_scheme` in `_data/settings.yml` (auto/light/dark)
- Responsive grid uses `col`, `col-d`, `col-t` classes for default/desktop/tablet breakpoints
- Assets (images, resume PDF) live in `assets/` with WebP preferred for images
- Projects display newest-first (reversed collection order)
- Blog is paginated at 6 posts per page via jekyll-paginate
