---
layout: post
title: "FreshRSS in the Omarchy Bar, With FreshRSS's Own Keyboard Shortcuts"
description: "A FreshRSS reader for the Omarchy bar that uses FreshRSS's own keyboard shortcuts, plus how to install it."
date: 2026-09-28 09:00:00 -0700
published: true
image: '/assets/blog/freshrss-omarchy-plugin.webp'
tags: [linux, omarchy, freshrss, rss, self-hosted, plugins]
---

I read my feeds in [FreshRSS](https://freshrss.org/), and my fingers know its keyboard shortcuts by heart: `j` for the next article, `r` to mark it read, `f` to star it. Then I started testing [Omarchy](https://omarchy.org/) on my desktop (my daily driver at the time was [DankMaterialShell](https://danklinux.com/) from Dank Linux). The whole point of Omarchy is never reaching for the mouse, and yet checking my feeds meant opening a browser, hunting for the tab and waiting for the web app to load. The Omarchy plugin marketplace had ten RSS readers. Ten! Not one of them could talk to my FreshRSS server.

So I wrote one that does. **FreshRSS for Omarchy** is now a verified plugin on the [Omarchy plugin marketplace](https://omarchyplugins.com/plugin.html?id=freshrss-reader).


## What is FreshRSS?

[FreshRSS](https://freshrss.org/) is a free, open-source RSS reader you host yourself. It runs on your own server, checks your feeds and keeps one shared list of what you've read and starred. The web app, the phone apps and this plugin all read from that same list, so marking something read in one place marks it read everywhere. You can kick the tires on the [public demo](https://demo.freshrss.org/) before installing anything.

If you want your own, the official [Docker image](https://hub.docker.com/r/freshrss/freshrss) with its [Docker guide](https://github.com/FreshRSS/FreshRSS/tree/edge/Docker) is the quickest way there. The [installation guide](https://freshrss.github.io/FreshRSS/en/admins/03_Installation.html) covers a regular PHP setup.

## What the plugin does

The RSS icon in the bar shows your unread count, refreshed every five minutes. Click it and you get:

- **All unread, Starred and every category**, each with its own unread count.
- **An article list with thumbnails**, pulled from the feed's image or the first image in the article, with the site's icon when there's no picture.
- **An article view** with the lead image, author, date and summary, plus buttons to open it in your browser, star it or mark it unread.
- **Mark all read** for a category. It makes you click twice, so you don't wipe out a week of reading by accident.
- **A pop-out window** for longer reading sessions.

Everything syncs with your server through FreshRSS's Google Reader API, the same one the phone apps use.

## The keys are FreshRSS's keys

This is the part I cared about most. The plugin uses FreshRSS's own default shortcuts, so your muscle memory from the web app carries right over:

| Key | Action |
|---|---|
| `j` / `k` | Next / previous article |
| `h` | Next unread article |
| `n` / `p` | Next / previous category |
| Enter | Open the article |
| Space | Open it in your browser |
| `r` | Toggle read |
| `f` | Toggle star |
| `m` | Load more |
| `u` | Unread only / all items |
| `q` | Refresh |

A couple of extras are there because Omarchy needs them: `z` switches between the dropdown and its own window, and `,` opens settings. The full list is on the settings page.

## Install it

You need Omarchy and a FreshRSS server with API access turned on.

```
omarchy plugin add https://github.com/jtcarrasco/freshrss-reader --enable
```

Before you connect, do this one step in FreshRSS. It trips up everyone (me included):

1. In FreshRSS, open **Settings → Authentication** and enable **Allow API access**.
2. Open **Settings → Profile** and set an **API password**.

The plugin logs in with that API password, not your normal web password. FreshRSS's [mobile access guide](https://freshrss.github.io/FreshRSS/en/users/06_Mobile_access.html) walks through the same setup. Then click the RSS icon, enter your server address, username and API password, and your categories show up. The login token goes into your system keyring.

**Using DankMaterialShell?** The repo includes a [DMS](https://danklinux.com/) version in `dms/` with the same features and keys. The [README](https://github.com/jtcarrasco/freshrss-reader#dankmaterialshell) has the install steps.

## One FreshRSS quirk worth knowing

If a category shows unread items but opens up empty, check its name for an `&`. FreshRSS can save a category name with an `&` in a form its own API can't look up, which happens a lot after an OPML import. Rename the category, or just save it again in FreshRSS, and it works. Every app that talks to FreshRSS's API runs into this, not just mine. I found it in my own feeds while testing, naturally.

The story behind this plugin and its Audiobookshelf sibling, including a four-round code review, is in [Shipping My First Two Published Apps](/blog/first-published-apps).

The code is [on GitHub](https://github.com/jtcarrasco/freshrss-reader), MIT licensed. My feeds are one keystroke away again, and the browser tab has been put out to pasture.

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
