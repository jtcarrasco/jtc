---
layout: post
title: "Building a Compounding Knowledge Base With an AI Wiki Agent"
description: "How I built an agent that ingests articles and YouTube videos, extracts concepts and entities, and builds a structured wiki that gets more useful the more you feed it."
date: 2026-05-15 09:00:00 -0800
image: '/assets/profile/jtc_profile_pic.jpg'
tags: [ai, claude, knowledge-management, obsidian, homelab]
---

Keeping up with a fast-moving field requires more than reading. It requires retaining and connecting what you read. I built this agent specifically to solve that problem for myself: a system that ingests articles and videos, extracts concepts and entities, and builds structured cross-references automatically. The result is a wiki that gets more useful the more you put into it. This is one of the more technically interesting agents I've built, and the approach generalizes well beyond personal knowledge management.

The problem with most note-taking systems is that notes don't compound. You save something, never look at it again, and the insight is effectively lost. Search helps, but it doesn't surface connections you didn't know to look for.

I wanted something that grows. Where adding a new source makes the whole system more useful, not just bigger.

So I built `wiki_agent`, an AI agent that reads articles, extracts concepts and entities, and integrates everything into a structured, interlinked wiki.

## How It Works

The wiki lives in my [Obsidian](https://obsidian.md) vault under `Resources/Wiki/`. It has three page types:

- **Sources**: one page per ingested article or video. A summary, key concepts mentioned, key entities, notable claims.
- **Concepts**: ideas, patterns, methods, frameworks that appear across multiple sources (e.g. "token efficiency," "knowledge graphs," "prompt caching").
- **Entities**: specific tools, people, companies, projects (e.g. "Claude Code," "Andrej Karpathy," "Obsidian").

When I queue a new article, the agent reads it, updates or creates the relevant concept and entity pages, and links everything together with wikilinks. The wiki index and log stay current automatically.

## The Queue System

The simplest interface I could build: drop a `.md` or `.txt` file with a URL into a folder called `wiki_queue/`. That's it.

[Syncthing](https://syncthing.net) sees the file, pushes it to the VPS, and the dispatcher picks it up within 5 minutes. The agent fetches the URL (or extracts a YouTube transcript if it's a video), runs the full ingest pipeline, and moves the queue file to `processed/` when done.

For YouTube videos, it uses [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) to pull the auto-generated subtitles, strips the timestamp markers, and treats the cleaned transcript as the source text. The quality varies; auto-generated captions on technical content are surprisingly usable.

If you use the [Obsidian Web Clipper](https://obsidian.md/clipper), clipping a YouTube page embeds the transcript directly into the saved markdown. The agent picks it up already transcribed. No `yt-dlp` call needed for those.

## The Lint Run

Once a week the agent runs a health check on the wiki:

1. **Index gaps**: pages that exist in the directories but aren't in the index
2. **Missing pages**: wikilinks that reference a page that doesn't exist
3. **Orphan pages**: pages with no inbound links from other pages
4. **Missing cross-references**: concepts or entities mentioned by name but not linked
5. **Stale claims**: pages potentially contradicted by newer sources (flagged, not auto-corrected)

Most issues are fixable automatically. Cross-references get added. Index gaps get filled. The lint report goes into the wiki log.

## What Surprised Me

**Cross-referencing is where the value lives.** I didn't expect this, but the most useful thing the agent does isn't summarizing articles. It's noticing that an entity mentioned in article A was also discussed in articles C and E, and linking them. That's the compounding part.

**YouTube transcripts work better than expected.** I was skeptical. A 45-minute technical talk produces a lot of text, but the agent handles it well. The resulting source page is usually more useful than my own notes would have been.

**The wiki accumulates personality.** After enough sources, the concept pages start to reflect a point of view. Not just what each article said, but where sources agree, where they conflict, and what claims need external verification. That wasn't designed in. It emerged.

## The Dispatcher

Both `wiki_agent` and `job_agent` run through the same infrastructure: a bash script that loops agent folders looking for `[trigger]` tags in markdown task files. The agent's instructions live in `CLAUDE.md`, which Claude reads before doing anything.

This means adding a new agent is: create a folder, write a `CLAUDE.md`, create a `task-list.md`. The dispatcher picks it up automatically.

No custom server. No API endpoints. No code changes to add a new agent. The entire control plane is markdown files synced over Tailscale.

## What I'd Add

The obvious next step is ingesting from RSS feeds automatically. New articles from subscribed feeds get queued without any manual action. The queue system is already set up for it; I just need to wire in a [FreshRSS](https://freshrss.org)-to-queue pipeline.

A search interface would also help. Right now I explore the wiki through [Obsidian's graph view](https://help.obsidian.md/Plugins/Graph+view) and search. A lightweight query layer, "show me everything about token efficiency that's newer than 30 days," would make the compounding more accessible.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
