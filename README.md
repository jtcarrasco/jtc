# jtcarrasco.com

Personal portfolio and blog — Jekyll site showcasing web development and AI/automation
projects, with narrative posts on what I've built and why.

## Architecture

Three Jekyll collections:
- `_pages/` — standalone pages (about, contact, tags, uses)
- `_posts/` — blog posts (`/blog/:slug`)
- `_projects/` — portfolio case studies (`/project/:slug`)

Site-wide content (nav, hero text, social links, testimonials) is centralized in
`_data/settings.yml` rather than scattered across templates.

CSS follows ITCSS methodology (`css/_0-settings` through `_4-layouts`). Dark/light mode is
user-toggleable and persisted via localStorage.

## Local Development

```bash
bundle install
bundle exec jekyll serve --livereload
```

## Stack

Jekyll · Sass (ITCSS) · vanilla JS · GitHub Pages
