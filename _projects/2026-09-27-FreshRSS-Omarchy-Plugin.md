---
title: FreshRSS for Omarchy
date: 2026-09-27 13:00:00 -0700
subtitle: Open Source | Linux Desktop Plugin | QML / Quickshell | Python
description: "Open-source FreshRSS reader for the Omarchy and DankMaterialShell Linux desktops: unread count in the status bar, keyboard-driven triage with FreshRSS's own shortcuts, and read/star sync through the Google Reader API. Security-reviewed for the Omarchy plugin marketplace."
image: '/assets/projects/freshrss-omarchy.webp'
thumbnail: '/assets/projects/freshrss-logo.webp'
images:
  - src: '/assets/projects/freshrss-omarchy.webp'
    alt: 'FreshRSS for Omarchy screenshot'
  - src: '/assets/projects/freshrss-window.webp'
    alt: 'Pop-out window for full-size reading'
  - src: '/assets/projects/freshrss-list.webp'
    alt: 'Article list for a category'
  - src: '/assets/projects/freshrss-article.webp'
    alt: 'Reading an article'
  - src: '/assets/projects/freshrss-dms.webp'
    alt: 'Categories with unread counts on DankMaterialShell'
category: apps
---

## Scope
[FreshRSS](https://freshrss.org/) is a self-hosted feed reader that keeps one shared list of what you've read and starred. The [Omarchy](https://omarchy.org/) plugin marketplace had ten RSS readers, and none of them could talk to a FreshRSS server; they all fetched feeds on their own, so read state never synced with the server or the phone. I built a reader that lives in the status bar: unread count on the icon, categories, an article list with thumbnails, and an article view, all syncing with the server.

### Role & Approach
Solo project, from idea to marketplace listing. I chose FreshRSS's Google Reader-compatible API over the simpler Fever API so the plugin reads and writes exactly the same state as the established mobile apps. I designed it keyboard-first around FreshRSS's own web shortcuts (`j`/`k`, `r` to mark read, `f` to star), so existing muscle memory carries straight over. Like its sibling [Audiobookshelf for Omarchy](/project/audiobookshelf-omarchy-plugin), one repository ships both an Omarchy and a [DankMaterialShell](https://danklinux.com/) version on [Quickshell](https://quickshell.org/), sharing a standard-library Python backend. I used [Claude Code](https://claude.ai/code) as a pair programmer throughout.

### Security Review
The same review and audit process as the Audiobookshelf plugin: plain-text rendering for every feed-supplied string, http(s)-only addresses for the server and for links opened in the browser, response size caps and read deadlines, and credentials kept in the system keyring. Testing also turned up a FreshRSS API quirk: category names containing `&` can't be looked up after an OPML import, now documented for users.

### Stack
QML / Quickshell, Python (standard library only), FreshRSS Google Reader API, the system keyring for credentials.

### Results
Approved and listed with a verified badge on the [Omarchy plugin marketplace](https://omarchyplugins.com/plugin.html?id=freshrss-reader). Open source under the MIT license on [GitHub](https://github.com/jtcarrasco/freshrss-reader). The full story is in the blog post [Shipping My First Two Published Apps](/blog/first-published-apps).
