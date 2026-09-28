---
layout: post
title: "Shipping My First Two Published Apps (and Surviving the Code Review)"
description: "Two Omarchy plugins, an Audiobookshelf player and a FreshRSS reader, made it through four rounds of human code review onto the marketplace. What I built, the decisions that mattered, and what the review taught me."
date: 2026-09-28 09:00:00 -0700
published: true
image: '/assets/blog/first-published-apps-audiobookshelf.webp'
images:
  - src: '/assets/blog/first-published-apps-audiobookshelf.webp'
    alt: 'Audiobookshelf for Omarchy: the player popout in the Omarchy bar'
  - src: '/assets/blog/first-published-apps-freshrss.webp'
    alt: 'FreshRSS for Omarchy: the feed reader popout in the Omarchy bar'
tags: [omarchy, linux, plugins, audiobookshelf, freshrss, security, claude]
---

This week two things I built were approved and published on the [Omarchy plugin marketplace](https://omarchyplugins.com/): a player for my self-hosted audiobooks and a reader for my self-hosted news feeds. They're small apps. They're also the first software I've shipped that anyone in the Omarchy community can install with one command, after one of the marketplace's reviewers read the code and signed off on it. That second part turned out to be the interesting part.

## The gap

My daily driver is Linux with [DankMaterialShell](https://danklinux.com/) from Dank Linux, and lately I've been testing [Omarchy](https://omarchy.org/) on my desktop, a Linux setup built around the keyboard. On both, nearly everything is a keystroke away, which makes the things that aren't feel worse than they used to. Two of those were daily habits. My audiobooks live on a self-hosted [Audiobookshelf](https://www.audiobookshelf.org/) server, and listening at my desk meant a browser tab I could never find when I wanted to pause. My news feeds live in [FreshRSS](https://freshrss.org/), which I also host myself.

Omarchy has a plugin marketplace, so I looked there first. It had ten RSS readers. None of them could talk to FreshRSS; they all fetch feeds on their own, which means your read and starred state never syncs with the server or your phone. There was no Audiobookshelf client at all. Two clear gaps, both in tools I use every day, which is about the best reason there is to build something.

At the end of my last post about the [wiki agent](/blog/wiki-agent-knowledge-management), I called a FreshRSS pipeline the obvious next step. This is not that pipeline. It's a different FreshRSS project, which is what happens when you write your roadmap in public.

## The decisions that mattered

**Build on the server's own API, not around it.** FreshRSS offers two sync APIs. The simpler one (Fever) would have been faster to write, but it can't do half of what the phone apps do. I went with the Google Reader-compatible API, the same one the established mobile readers use, so the plugin reads and writes exactly the state everything else sees. Mark an article read in the bar and it's read on my phone.

**Use the keys people already know.** FreshRSS has keyboard shortcuts in its web app: `j` for next, `r` for read, `f` for star. The plugin uses the same ones, so there's nothing new to learn. I did not get this right the first time. The first version of the audiobook player had keys bolted on after the fact, one request at a time, which is a strange way to build for a desktop whose entire identity is the keyboard. The second plugin was designed keyboard-first, and it shows.

**One codebase, two desktops.** Omarchy is where I was testing, but DankMaterialShell is what I use every day, so partway through I wanted the same tools there too. That turned out to be less work than it sounds: both desktops are built on [Quickshell](https://quickshell.org/), the same QML toolkit, so the plugins speak the same language on either one. Rather than fork each project, each repo holds both versions, sharing the backend that does the real work. A small script keeps the shared files in sync, and a test fails if they ever drift apart. It's the unglamorous kind of decision that saves you from fixing every bug twice.

## The review

Marketplace submissions get an automated scan and then a human code review before they're listed. I expected a checklist. Instead the reviewer read the backend line by line, and it took four rounds:

1. The repository included configuration files for my AI coding tools, which other developers' tools could load automatically. Fair point, and easy to fix.
2. Book and article titles from the server were rendered in a mode that interprets HTML, so a malicious title could make the desktop load an image from anywhere on the internet.
3. Responses from the server were read without any size limit, so a misbehaving server could exhaust memory.
4. My fix for that had a gap: the time limit only kicked in after each chunk arrived, so a server sending one byte at a time could hold the connection open indefinitely.

After the second finding, I stopped fixing only what was flagged and audited both plugins myself. That turned up something the reviewer hadn't asked about: the audiobook player put its login token in the streaming URL, and Linux media controls publish the currently playing URL to anything on the desktop that asks. The token now travels in a request header instead, where it belongs. I verified each fix against my real servers, added tests for every one (including a test server that deliberately drips data one byte at a time), and wrote up what changed on the submission each round. Both plugins were approved the same day the last fix went in.

## Working with an AI pair

I built both plugins with [Claude Code](https://claude.ai/code) as a pair programmer, which is also how I work on most things now. It wrote a lot of the code. My job was the part that doesn't show up in a diff: deciding what to build, choosing the API, insisting on the keyboard design, testing on real machines, and treating every review comment as a reason to look for the whole class of problem rather than the single line. The review made one thing very clear. AI makes it fast to build something that works. Getting it to the point where a careful reviewer signs off still takes judgment, and that part is still mine.

## What I'd do differently

I'd run the security pass before submitting, not after the first finding. Every issue the reviewer raised was the kind of thing a deliberate audit catches in an afternoon; I just hadn't done one yet. I'd also design the keyboard controls on day one, and give the repositories generic names from the start (I had to rename both when they grew a second desktop, which is only painless before approval).

Both plugins are open source and listed with a verified badge: [Audiobookshelf for Omarchy](https://omarchyplugins.com/plugin.html?id=abs-player) and [FreshRSS for Omarchy](https://omarchyplugins.com/plugin.html?id=freshrss-reader). 

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
