---
layout: post
title: "Claude Code Is My Collaborator. Obsidian Is Our Shared Brain."
description: "How I use CLAUDE.md files, a memory system, and vault-based agents to give Claude persistent context across every session."
date: 2026-05-15 09:00:00 -0800
published: false
image: '/assets/profile/jtc_profile_pic.jpg'
tags: [ai, claude, obsidian, pkm, automation]
---

Most people use AI assistants the way they use a search engine: ask a question, get an answer, close the tab. I use Claude Code differently. It's a persistent collaborator with full context on my systems, my projects, and how I work. The thing that makes that possible is [Obsidian](https://obsidian.md).

This is the post I wish existed when I started building this workflow. It explains the setup, the reasoning behind it, and what it actually looks like in practice.

## The Problem With Stateless AI

Every new conversation with an AI assistant starts from zero. You re-explain your stack, your preferences, what you tried last time, and what you're trying to accomplish. Even with memory features, the context is shallow and generic.

That's fine for one-off questions. It's not fine if you want an AI to meaningfully help you manage ongoing projects, agents that run on schedules, a blog that's half-written, a homelab that needs maintenance. For that, you need persistence.

## Obsidian as Persistent Context

My Obsidian vault lives at `~/vault/` and syncs across all devices via [Syncthing](https://syncthing.net). It uses a PARA-inspired structure: Inbox, Projects, Areas, Resources, Archive.

Claude Code has read/write access to this vault. Which means:

- When I ask Claude to draft a blog post, it reads existing drafts, checks the content strategy, and writes into the right directory.
- When we set up a new agent, Claude reads the agent infrastructure docs, writes the CLAUDE.md, and updates the shared index.
- When I ask about a project status, Claude reads the project file and gives me a current answer, not a hallucinated one.

The vault isn't just my notes. It's shared working memory.

## CLAUDE.md Files: Instructions That Persist

The most important piece of this is a file called `CLAUDE.md`. Claude Code reads this file at the start of every session. It contains everything Claude needs to know: directory structure, conventions, what never to do, what to always do.

I have a global `CLAUDE.md` in my home directory and project-specific ones in each agent folder. When Claude opens a session in the `wiki_agent/` directory, it reads that agent's CLAUDE.md and knows exactly what that agent does, what files it touches, and what rules it follows, without me explaining any of it.

This is the difference between a tool and a collaborator. A tool needs instructions every time. A collaborator reads the brief.

## Memory Files: Context That Carries Forward

Beyond CLAUDE.md, I maintain a memory system, markdown files that Claude writes and reads across sessions. User preferences, project status, feedback from past interactions, reference information.

When I told Claude that I always want a dry-run before bulk-deleting email, that went into a memory file. Now it happens automatically, every time, without me asking.

This kind of persistent behavioral adaptation is what makes the collaboration feel different from a typical AI chat. It accumulates.

## The Agent Layer

The deeper integration is the agent infrastructure. I have several Claude Code agents running headlessly on my VPS:

- **job_agent**: searches job boards on a schedule, writes reports to the vault
- **wiki_agent**: ingests articles and YouTube videos into a structured wiki
- **events_agent**: finds tech meetups and conferences monthly

Each agent has a `CLAUDE.md` (operating instructions) and a `task-list.md` (what to do). A bash dispatcher runs on cron every 5 minutes, checks for trigger tags in task files, and hands control to Claude Code when it finds one.

The control interface is a markdown file. I edit it on my phone in Obsidian, [Syncthing](https://syncthing.net) pushes it to the VPS, Claude picks it up within 5 minutes and runs the task, and the output syncs back to my phone.

## The Discord Layer

For interactive work, I use a Discord channel as the interface. Claude Code watches for messages, handles file operations, web searches, calendar events, and anything else I need. It's the conversational layer on top of the file-based foundation.

The distinction matters: Discord is for conversation, Obsidian is for state. Work happens in files. Discord is just how I initiate it.

## What This Actually Looks Like

Today's session, as an example: We set up a Drupal sandbox, organized a book list, drafted a uses page and bookshelf, ran the events agent to surface local meetups, built a blog content strategy, and wrote five posts. All outputs landed in the vault. All state is persisted. Next session picks up with full context.

That's a lot of ground to cover in an afternoon. The speed comes from not re-explaining context. Claude already knows where things live, what the conventions are, and what we're trying to build.

## What It Requires

This setup isn't magic. It requires some upfront work:
- A reasonably organized vault with a clear directory structure
- Well-written CLAUDE.md files that explain what Claude needs to know
- Willingness to let the AI write to your files (you can always review diffs)

But once that foundation exists, the compounding effect is real. Every session adds to the shared context. Every agent you add integrates automatically. The system gets more useful the more you use it.

---

*Part of a series on building a practical AI-powered personal system with Claude Code, Obsidian, and a self-hosted VPS.*
