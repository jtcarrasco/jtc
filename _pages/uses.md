---
layout: page
title: Uses
permalink: /uses/
image: '/assets/uses-hero.webp'
---

The tools, hardware, and software I use day-to-day. Inspired by [uses.tech](https://uses.tech).

## Hardware

### Machines
- [**MacBook Air**](https://www.amazon.com/dp/B0GR1BY1SZ?tag=jtcarrascoser-20) — primary dev machine
- **[Arch Linux desktop](https://www.amazon.com/dp/B08WNF8FP6?tag=jtcarrascoser-20)** (Optiplex 7060 Micro, CachyOS) — desktop computer
- [**RackNerd VPS**](https://my.racknerd.com/aff.php?aff=21044) — Ubuntu 24.04 LTS, always-on, runs all homelab services
- i7 3080 GPU "gaming" Machine -- now used as a [Proxmox](https://www.proxmox.com/) server
- [**Synology NAS**](https://www.amazon.com/dp/B0C8S7SF4B?tag=jtcarrascoser-20) — For Storage
- [**iPhone 16**](https://www.amazon.com/dp/B0DHJH2GZL?tag=jtcarrascoser-20) -- Mobile

## Terminal + Editor

- **[WezTerm](https://wezterm.org/)** — terminal. Fast, GPU-accelerated, configured in Lua. Same config synced across Mac and Linux via dotfiles.
- **[tmux](https://github.com/tmux/tmux)** — session management. Never lose a session again.
- **[zsh](https://www.zsh.org/)** with **[Starship](https://starship.rs/)** prompt — minimal, informative
- **[Yazi](https://yazi-rs.github.io/)** — terminal file manager, replaces most of what I used to do with Finder
- **[Neovim](https://neovim.io/)** — still learning, slowly replacing VS Code for server-side editing
- **[Claude Code](https://claude.ai/code)** — AI coding assistant. I use it every day. More than just autocomplete — it runs agents, writes files, hits APIs.

## Networking and Infrastructure

- **[Tailscale](https://tailscale.com/)** — private mesh VPN across all devices. Every machine I own is on the same private network, always. No port forwarding, no dynamic DNS.
- **[Caddy](https://caddyserver.com/)** — reverse proxy and automatic TLS on the VPS. All services HTTPS-only, Tailscale-only.
- **[Docker](https://www.docker.com/)** — all VPS services run in containers
- **[Syncthing](https://syncthing.net/)** — file sync over Tailscale between VPS, NAS, Mac, and Arch box. Obsidian vault stays in sync across everything.
- **[Lando](https://lando.dev/)** — local WordPress and Drupal dev environments. Docker-based, one command to spin up a site.

## Self-Hosted Services

All running on the VPS, Tailscale-only access:

- **[n8n](https://n8n.io/)** — workflow automation. Powers my daily briefings, job application follow-ups, and various API integrations.
- **[FreshRSS](https://freshrss.org/)** — RSS aggregator for job boards and tech feeds
- **[MainWP](https://mainwp.com/)** — manages ~60 client WordPress sites from one dashboard
- **[Uptime Kuma](https://github.com/louislam/uptime-kuma)** — site monitoring for client sites
- **[Glance](https://github.com/glanceapp/glance)** — homelab dashboard. One tab, everything I need to see.
- **[Watchtower](https://containrrr.dev/watchtower/)** — auto-updates containers overnight

## AI and Automation

- **[Claude Code](https://claude.ai/code)** — Anthropic's CLI. I use it to build and run agents that run on my VPS: a job search agent, a knowledge base builder, an event discovery agent.
- **[Claude API](https://www.anthropic.com/api)** — for programmatic AI integration in web projects
- **[n8n](https://n8n.io/)** — wires everything together. Gmail, Discord, Todoist, calendar, external APIs.

## Productivity and PKM

- **[Obsidian](https://obsidian.md/)** — personal knowledge management. Vault synced across all devices via Syncthing. Where my notes, blog drafts, agent task files, and research all live.
- **[Todoist](https://todoist.com/)** — task management. Integrates with n8n for automated task creation from email.
- **[Espanso](https://espanso.org/)** — text expander. Snippets for code, email templates, repeated strings.

## Desk Setup

- Monitor: [Dell 27" P2723QE](https://www.amazon.com/dp/B09TY127B8?tag=jtcarrascoser-20)
- Keyboard: [Corne 36 wireless MX](https://keyboard-hoarders.com/products/corne-wireless-split-keyboard-3?pr_prod_strat=e5_desc&pr_rec_id=65d1f63fc&pr_rec_pid=8175297888344&pr_ref_pid=8171080745048&pr_seq=uniform)
- Mouse/trackpad: [Elecom EX-G Left Handed Thumb Control Trackball Mouse Wireless](https://www.amazon.com/dp/B08GR2K8WH?tag=jtcarrascoser-20)
- Headphones: [Apple AirPods Pro 3](https://www.amazon.com/dp/B0FQFB8FMG?tag=jtcarrascoser-20)

## This Site

- **Jekyll**, hosted on GitHub Pages
- Written in Markdown, deployed via GitHub Actions

---

*As an Amazon Associate I earn from qualifying purchases — some links above may earn a small commission at no extra cost to you.*
