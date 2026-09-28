---
title: Audiobookshelf for Omarchy
date: 2026-09-27 12:00:00 -0700
subtitle: Open Source | Linux Desktop Plugin | QML / Quickshell | Python
description: "Open-source Audiobookshelf client for the Omarchy and DankMaterialShell Linux desktops: a keyboard-driven audiobook and podcast player in the status bar, built with QML/Quickshell and Python and security-reviewed for the Omarchy plugin marketplace."
image: '/assets/projects/audiobookshelf-omarchy.webp'
category: current
---

## Scope
[Audiobookshelf](https://www.audiobookshelf.org/) is a self-hosted server for audiobooks and podcasts. Listening at a desk meant keeping a browser tab open, and the [Omarchy](https://omarchy.org/) plugin marketplace had no Audiobookshelf client at all. I built one that lives in the Linux desktop's status bar: browse books and podcasts, play with chapters, speed control and 30-second skips, and sync listening progress both ways with the server, so a book paused at the desk resumes on the phone at the same spot.

### Role & Approach
Solo project, from idea to marketplace listing. I designed it keyboard-first to match the desktop (every action has a key, shown in tooltips and a reference in settings), wrote a dependency-free Python backend that talks to the Audiobookshelf API, and built the interface in QML on [Quickshell](https://quickshell.org/). One repository ships two versions, one for Omarchy and one for [DankMaterialShell](https://danklinux.com/), sharing the backend through a sync script and a test that fails if the copies drift. I used [Claude Code](https://claude.ai/code) as a pair programmer throughout.

### Security Review
The marketplace's human review went four rounds, and I followed it with a full audit of my own. Fixes included rendering all server text as plain text so a crafted title can't load remote content, moving the login token out of the stream URL (Linux media controls publish that URL to the whole desktop), capping response sizes, enforcing read deadlines against servers that trickle data, and allowing only http(s) server addresses. Each fix shipped with tests.

### Stack
QML / Quickshell, Python (standard library only), mpv with IPC, the system keyring for credentials, Audiobookshelf REST API.

### Results
Approved and listed with a verified badge on the [Omarchy plugin marketplace](https://omarchyplugins.com/plugin.html?id=abs-player). Open source under the MIT license on [GitHub](https://github.com/jtcarrasco/audiobookshelf-player). The full story is in the blog post [Shipping My First Two Published Apps](/blog/first-published-apps).
