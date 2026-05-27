---
layout: post
title: "I Built an AI Agent That Searches for Jobs While I Sleep"
description: "How I automated the repetitive parts of job searching with a Claude Code agent that hits multiple APIs, deduplicates results, and drops a report in my Obsidian vault."
date: 2026-05-15 09:00:00 -0800
published: true
image: '/assets/blog/job-agent.webp'
tags: [ai, claude, automation, job-search, homelab]
---

One thing I've been focused on this year is building tools that handle repetitive cognitive work. Not just automating clicks, but automating judgment. The job search agent is a direct example of this: instead of running the same searches manually every few days, I described what I want in a markdown file, and a Claude Code agent does the rest. It's a practical demonstration of what agentic AI looks like when applied to real, ongoing work rather than toy examples.

Job searching is repetitive in a way that should be automated. The same queries, the same boards, the same deduplication logic, running every day or every other day, filtering results through a consistent set of criteria. It's exactly the kind of work an AI agent handles well.

So I built one.

## What It Does

`job_agent` runs on my VPS three to five times per week depending on the search type. Each run:

1. Hits multiple job board APIs and RSS feeds with specific queries
2. Deduplicates against a rolling 90-day history of seen postings
3. Filters results through my criteria (remote, web dev focus, specific exclusions)
4. Writes a markdown report with a table of results, posting notes, and hiring trend analysis
5. Drops the report in my Obsidian vault where I can review it on any device

No dashboard. No special app. Just a markdown file that syncs to my phone via [Syncthing](https://syncthing.net).

## The Sources

I pull from several places:

- **[We Work Remotely](https://weworkremotely.com) RSS feeds**: reliable, focused on remote roles
- **[Post Status Jobs](https://poststatus.com/jobs/)**: WordPress/web-specific board
- **[JSearch API](https://openwebninja.com)**: aggregates LinkedIn, Glassdoor, Indeed, and ZipRecruiter in a single call
- **[Adzuna API](https://developer.adzuna.com)**: additional aggregator with good US coverage
- **EdTech Jobs and local school district boards**: niche sources for my area

Three search types run on different schedules: W2 (full-time) web development, W2 project management, and freelance/contract. Each has its own criteria file and output directory.

## The Architecture

The agent is [Claude Code](https://claude.ai/code) running headlessly. No server, no framework, just Claude with a prompt and file access.

A bash dispatcher script runs on cron every 5 minutes and checks each agent folder for a `[trigger]` tag in its `task-list.md`. When it finds one, it passes the agent's `CLAUDE.md` (instructions) and the triggered task to Claude and lets it run. Claude reads the criteria, hits the APIs, deduplicates, writes the report, and updates the task file.

The trigger system is dead simple: edit a markdown file on your phone, Syncthing pushes it to the VPS, cron picks it up within 5 minutes.

```
task-list.md (on phone) → Syncthing → VPS → cron → Claude → report → Syncthing → phone
```

## What the Reports Look Like

Each report is a dated markdown file with:

- A table of results (company, title, remote status, salary range, apply link)
- 2-4 sentence notes on each posting (fit, red flags, stack, employment type)
- A summary section: hiring trends, most-requested skills, standout roles to prioritize

Claude writes these from scratch each run. The quality is consistently better than I'd write myself at 8am doing the same manual search.

## The Dedup Problem

The biggest operational challenge was deduplication. Job boards recycle postings. The same role reappears across multiple platforms under slightly different job IDs. Running the same searches twice a week without dedup would surface the same 20 roles every single time.

The solution: a simple text file with one entry per line, `YYYY-MM-DD URL`. At the start of every run, entries older than 90 days get pruned. During the run, any URL already in the list gets skipped. After the run, new URLs get appended.

Not elegant, but it works and it's easy to inspect.

## What Surprised Me

**The API inconsistencies.** Every job board API is slightly broken in a different way. Adzuna doesn't accept `where=remote`. JSearch migrated from RapidAPI to a new provider mid-project. RSS feeds have inconsistent date formats. A real job search agent needs to be fault-tolerant and verbose about what it skipped and why.

**How much better niche sources are.** We Work Remotely produces better signal than the aggregators. Post Status jobs are almost all genuinely relevant. The LinkedIn/Indeed/ZipRecruiter bucket through JSearch is noisy by comparison.

**The scheduling discipline matters more than the search quality.** Having it run consistently, twice a week, on a schedule I didn't have to remember. That's the actual value. Not the AI.

## What I'd Build Next

Better signal-to-noise filtering. Right now the criteria are blunt instruments (include/exclude keywords). A scoring system that ranks results against a detailed profile, skills, experience level, company type preferences, would cut down on manual review time significantly.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
