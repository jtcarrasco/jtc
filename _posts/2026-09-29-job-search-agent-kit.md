---
layout: post
title: "I Turned My Job Search Agent Into a Kit (So You Can Skip the Part Where I Broke It)"
description: "I rebuilt my job search agent as a two-agent kit anyone can run. What's in it, who it's for, and why the free tutorial is still up."
date: 2026-09-29 11:00:00 -0700
published: true
image: '/assets/blog/job-search-agent-kit.webp'
tags: [ai, tools, claude, claude-code, automation, job-search]
---

Back in May I wrote up [how I built an AI agent to do my job searching](/blog/job-agent-ai-job-search). It worked. Heck, it still works. It has run most weeks since spring, and in that time it has dug up postings I never would have found by hand. It has also taught me every way a little agent can quietly fall over on a Tuesday morning while I'm still on my first coffee.

The question I kept getting after that post wasn't "how does it work?" It was "can I just have yours?" And for a long time the honest answer was no. Mine was welded to my setup: a VPS, a cron dispatcher, an Obsidian vault, my resume, my salary floor and my very specific grudges against certain staffing agencies (you know who you are).

So I pulled it apart and rebuilt it as something that runs on anybody's machine. It's called the [Job Search Agent Kit](https://jtcarrasco.gumroad.com/l/xqfvx), and it's two agents instead of one.

## What's in the box

**The Job Search Agent** is the one from the original post, minus all my infrastructure. You fill in a criteria file (target skills, salary floor, industries, dealbreakers) and it searches LinkedIn, Indeed, ZipRecruiter, Glassdoor, [Adzuna](https://developer.adzuna.com/) and [We Work Remotely](https://weworkremotely.com/) in one pass. The first four all come through a single API, [JSearch](https://www.openwebninja.com/api/jsearch), which is how you get four job boards without scraping any of them. It filters everything against your criteria, remembers what it already showed you so the same reposted listing doesn't haunt you every week, and writes a dated markdown report you can read in about two minutes. There are separate profiles for full-time, freelance and AI-focused searches.

**The Job Application Agent** is the one I never wrote about. You paste in a job posting and it hands you back a folder with:

- a resume tailored to that posting
- a plain-text ATS version, because applicant tracking systems still choke on anything fancier than a text file (it's 2026, people)
- a cover letter written for that specific role
- a follow-up email template for later

It works from a resume template you fill in once, with everything you've ever done in as much detail as you can stand to write. For each posting it picks, reorders and reframes what's true about you to match what that role is actually asking for.

## The one rule I really care about

The application agent never invents experience. That sounds obvious until you watch a language model helpfully "improve" your resume by adding a technology you have never touched, just because the posting mentioned it three times.

So the resume template has a section at the bottom called "Notes for the agent." The agent checks it on every run. Mine has a note that one of my old employers ran its own in-house platform, not WordPress. So many postings ask for WordPress that early drafts kept giving me WordPress experience at a company that never used it. Nice try, robot. If a draft ever claims something that isn't true, you add one line to that section and it doesn't happen again.

It's still a drafting tool, not an autopilot. It won't submit anything for you, and you should REALLY read every resume before it goes out. What it changes is the math: tailoring an application goes from most of an hour down to a few minutes of reviewing.

## What you need

- [Claude Code](https://claude.com/claude-code) and a Claude subscription or API access
- Free API keys for [JSearch](https://www.openwebninja.com/api/jsearch) and [Adzuna](https://developer.adzuna.com/) (the setup guide walks you through both)
- About 15 minutes to fill in your resume and search criteria

That's it. No VPS, no cron job, no Obsidian. Both agents are plain folders you unzip anywhere on Mac, Windows or Linux, and you run them by opening a terminal in the folder and typing `claude`. If you want the search to run on a schedule, the guide covers that too, but most people just run it by hand a couple of times a week while they're actively looking.

## Build it or buy it

The original tutorial is still free and still accurate. If you like building things (and if you're reading this, you probably do), go build it! That post walks through the search agent start to finish.

The kit is for when you'd rather spend your evening applying to jobs than debugging an agent. You get both agents, cleaned up and genericized, a setup guide written for people who have never touched a `CLAUDE.md` file, and the application agent that isn't in any post.

It's [$79 on Gumroad](https://jtcarrasco.gumroad.com/l/xqfvx). My own job search runs on these same two agents every week, which is either a strong endorsement... or a sign I should be applying to more jobs instead of writing blog posts about applying to jobs.

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
