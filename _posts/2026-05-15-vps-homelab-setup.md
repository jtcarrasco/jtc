---
layout: post
title: "My VPS is My Homelab: Building a Self-Hosted Stack That Actually Works"
description: "How I consolidated a scattered homelab into a cheap VPS with Docker, Tailscale, and Caddy, and why it's been more reliable than anything I ran at home."
date: 2026-05-15 09:00:00 -0800
published: true
image: '/assets/blog/vps-homelab-setup.webp'
tags: [homelab, self-hosting, vps, tailscale, docker]
---

Over the past year I've been building the infrastructure layer for a personal AI automation stack, a foundation that lets me run agents, workflows, and services reliably without depending on third-party platforms. The VPS and networking setup described here is what everything else runs on: the job search agent, the knowledge base, the daily briefings, the client monitoring, GitHub repos. Getting this layer right made everything above it possible.

I've been homelabing for years. Raspberry Pis PiHoling, a Synology NAS humming in the corner, running too many services for its small processor. At some point I decided to consolidate, and the answer wasn't a bigger NAS or a faster Pi. It was a cheap VPS and a smarter approach to networking.

## The Problem With Traditional Homelabs

Most homelab setups have the same failure mode: they're tied to your home internet connection. Dynamic IPs, ISP outages, NAT traversal headaches, port forwarding rules you forget you set up. It's fragile in ways that don't matter until they do.

I wanted a setup where my services were always reachable, from any device, anywhere. And I didn't want to spend a lot of money doing it.

## The Stack

The core is simple:

- **[RackNerd](https://racknerd.com) VPS**: a cheap Ubuntu 24.04 box. Not fast, not huge, but always on.
- **[Tailscale](https://tailscale.com)**: private mesh VPN that handles all my device-to-device networking. Every machine I own (VPS, MacBook, NAS, Arch Linux desktop) is on the same private network, always, without any port forwarding.
- **[Docker](https://www.docker.com)**: all services run in containers, each with its own compose file.
- **[Caddy](https://caddyserver.com)**: reverse proxy and TLS terminator. Handles HTTPS automatically for all services on my Tailscale network.

The key architectural decision: **nothing is exposed to the public internet**. Caddy only listens on the Tailscale IP. SSH only listens on Tailscale. Every service is Tailscale-only.

## What's Running

A handful of containers that actually get used:

- **[n8n](https://n8n.io)**: workflow automation. Connects [Gmail](https://mail.google.com), [Discord](https://discord.com), [Todoist](https://todoist.com), and a bunch of APIs.
- **[FreshRSS](https://freshrss.org)**: RSS aggregator for job boards and tech feeds.
- **[MainWP](https://mainwp.com)**: manages about 60 client WordPress sites from one dashboard.
- **[Uptime Kuma](https://uptime.kuma.pet)**: monitors those same sites.
- **[Glance](https://github.com/glanceapp/glance)**: homelab dashboard. One tab, everything I need to see.
- **[Watchtower](https://containrrr.dev/watchtower/)**: auto-updates containers overnight.

## The Synology as Storage, Not Compute

My Synology NAS stays in its lane: storage and media. Plex runs there because it needs to be close to the media files. Immich runs there for the same reason.

The VPS gets stateless services. The NAS gets storage-heavy ones. They talk to each other over Tailscale when needed.

## Why Tailscale Changed Everything

Before Tailscale I was managing SSH keys stored in [1Password](https://1password.com). It worked, but it wasn't elegant, and it didn't scale across multiple machines. Tailscale replaced all of that with a zero-config mesh that just works.

Every device gets a stable IP. DNS is handled. The VPS is accessible from my phone, my laptop, my NAS, anywhere I have Tailscale running. The whole setup feels like a private LAN even though the machines are spread across three physical locations.

And critically: no port 22, no port 80, no port 443 exposed to the internet. The attack surface is Tailscale's auth, which is handled by the Tailscale team, not me.

## What I'd Do Differently

The biggest thing: I'd start with Tailscale from day one. The SSH key approach was functional but it didn't scale, and adding each new device was a manual process I kept putting off.

I'd also resist the urge to run everything. The most useful services aren't the most impressive ones. Uptime Kuma and n8n get used every day. The fun experiment containers sit idle.

## What's Next

An Optiplex 7070 Micro running CachyOS is in the plan for a garage workstation. Whether that eventually turns into a K3s cluster across the VPS and the garage box is still an open question. The idea is appealing. The complexity is less so.

For now though, the stack is boring in the best possible way. Everything just runs.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
