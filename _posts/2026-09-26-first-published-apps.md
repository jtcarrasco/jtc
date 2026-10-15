---
layout: post
title: "Shipping My First Two Published Apps (and Surviving the Code Review)"
description: "Two Omarchy plugins, an Audiobookshelf player and a FreshRSS reader, made it through four rounds of human code review onto the marketplace. What I built, the decisions that mattered, and what the review taught me."
date: 2026-09-26 09:00:00 -0700
published: true
image: '/assets/blog/first-published-apps.webp'
images:
  - src: '/assets/blog/first-published-apps.webp'
    alt: 'A ticket agent typing on an early computer reservation terminal'
  - src: '/assets/blog/first-published-apps-audiobookshelf.webp'
    alt: 'Audiobookshelf for Omarchy: the player popout in the Omarchy bar'
  - src: '/assets/blog/first-published-apps-freshrss.webp'
    alt: 'FreshRSS for Omarchy: the feed reader popout in the Omarchy bar'
tags: [omarchy, linux, plugins, audiobookshelf, freshrss, security, claude]
---

This week, two things I built got approved and published on the [Omarchy plugin marketplace](https://omarchyplugins.com/): a player for my self-hosted audiobooks and a reader for my self-hosted news feeds. They're small apps. They're also the first software I've ever shipped that anyone in the Omarchy community can install with one command, after a marketplace reviewer read through the code and signed off on it. That second part turned out to be the interesting part.

## The gap

My daily driver is Linux with [DankMaterialShell](https://danklinux.com/) from Dank Linux, and lately I've been testing [Omarchy](https://omarchy.org/) on my desktop, a Linux setup built around the keyboard. On both, almost everything is a keystroke away, which makes the things that aren't feel way more annoying than they used to. Two of those were daily habits. My audiobooks live on a self-hosted [Audiobookshelf](https://www.audiobookshelf.org/) server, and listening at my desk meant a browser tab I could never find when I wanted to pause. My news feeds live in [FreshRSS](https://freshrss.org/), which I also host myself.

Omarchy has a plugin marketplace, so I looked there first. It had ten RSS readers. Not one of them could talk to FreshRSS. They all fetch feeds on their own, which means what you've read and starred never syncs with your server or your phone. And there was no Audiobookshelf player at all. Two clear gaps, both in tools I use every day, which is about the best reason there is to build something.

At the end of my post about the [wiki agent](/blog/wiki-agent-knowledge-management), I called a FreshRSS pipeline the obvious next step. This is not that pipeline. It's a totally different FreshRSS project, which is what happens when you write your roadmap in public.

## The decisions that mattered

**Build on the server's own API, not around it.** FreshRSS offers two ways for apps to sync. The simpler one (Fever) would have been quicker to write, but it can't do half of what the phone apps do. I went with the Google Reader-compatible API, the same one the established phone apps use, so the plugin reads and writes exactly what everything else sees. Mark an article read in the bar, and it's read on my phone.

**Use the keys people already know.** FreshRSS's web app has keyboard shortcuts: `j` for next, `r` for read, `f` for star. The plugin uses the same ones, so there's nothing new to learn. I did not get this right the first time. The first version of the audiobook player had keyboard controls bolted on after the fact, one request at a time, which is a weird way to build for a desktop whose whole personality is the keyboard. The second plugin was designed keyboard-first, and it shows.

**One codebase, two desktops.** Omarchy is where I was testing, but DankMaterialShell is what I use every day, so partway through I wanted the same tools there too. That turned out to be less work than it sounds. Both desktops are built on [Quickshell](https://quickshell.org/), the same QML toolkit, so the plugins speak the same language on either one. Instead of forking each project, each repo holds both versions and shares the backend that does the real work. A small script keeps the shared files in sync, and a test fails if they ever drift apart. It's the boring kind of decision that saves you from fixing every bug twice.

## The review

Marketplace submissions get an automated scan and then a code review from an actual human before they're listed. I expected a checklist. Instead, the reviewer read the backend line by line, and it took four rounds:

1. The repository included config files for my AI coding tools, which other developers' tools could load automatically. Fair point, and an easy fix.
2. Book and article titles from the server were shown in a mode that reads HTML, so a malicious title could make the desktop load an image from anywhere on the internet.
3. Responses from the server were read with no size limit, so a misbehaving server could eat up all the memory.
4. My fix for that had a hole: the time limit only kicked in after each chunk arrived, so a server sending one byte at a time could hold the connection open forever.

After the second finding, I stopped fixing just what was flagged and audited both plugins myself. That turned up something the reviewer hadn't even asked about. The audiobook player put its login token in the streaming URL, and Linux media controls share the currently playing URL with anything on the desktop that asks. Whoops. The token now travels in a request header instead, where it belongs. I checked each fix against my real servers, added tests for every one (including a test server that deliberately drip-feeds data one byte at a time), and wrote up what changed on the submission each round. Both plugins were approved the same day the last fix went in.

## Working with an AI pair

I built both plugins with [Claude Code](https://claude.ai/code) as my pair programmer, which is how I work on most things now. It wrote a lot of the code. My job was the part that doesn't show up in the code changes: deciding what to build, picking the API, insisting on the keyboard design, testing on real machines, and treating every review comment as a reason to hunt down the whole family of problems instead of just the one line. The review made one thing really clear. AI makes it fast to build something that works. Getting it to the point where a careful reviewer signs off still takes judgment, and that part is still mine. I'm driving, it's doing the heavy lifting.

## What I'd do differently

I'd run the security pass before submitting, not after the first finding. Every issue the reviewer raised was the kind of thing a deliberate audit catches in an afternoon. I just hadn't done one yet. I'd also design the keyboard controls on day one, and give the repos generic names from the start (I had to rename both when they picked up a second desktop, which is only painless before approval).

Both plugins are open source and listed with a verified badge: [Audiobookshelf for Omarchy](https://omarchyplugins.com/plugin.html?id=abs-player) and [FreshRSS for Omarchy](https://omarchyplugins.com/plugin.html?id=freshrss-reader). Two browser tabs closed for good. Not a bad week.

For what each plugin does and how to install it, see the [Audiobookshelf player](/blog/audiobookshelf-omarchy-plugin) and [FreshRSS reader](/blog/freshrss-omarchy-plugin) posts.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
