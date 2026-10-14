---
layout: post
title: "I Let an AI Help Me Build a Blog, and Now I Have Opinions About It"
description: "What happened when I built a Jekyll blog in collaboration with Claude Code: the documentation habits, the workflow, and where the AI actually helped."
date: 2026-02-08 10:00:00 -0800
published: true
image: '/assets/blog/building-a-site-with-ai.webp'
tags: [web-dev, ai, jekyll, claude, workflow]
---

*Originally published on Cosmik Debris, my homelab blog, in February 2026. Those posts live here now.*

## That blog shouldn't have existed.

Not because it was bad (I hope it wasn't), but because I have started roughly seven personal blogs in my life and abandoned every single one somewhere between "configure the theme" and "write the first real post." The setup always ate the momentum. Every time.

So this time I tried something different. I built it with [Claude Code](https://claude.ai/code). And I don't mean "ask the AI to fix a bug." I mean the whole thing: architecture decisions, documentation, the color scheme, and the workflow I'd actually use to keep writing. What I really wanted to figure out was how to use AI as a collaborator. Not a shortcut, more like a thinking partner sitting inside a real workflow. The blog was the test case.

Spoiler: I published the post. So something worked. woohoo!

## The setup I actually wanted

The goal was simple enough. A personal blog on [Jekyll](https://jekyllrb.com/) with the [Chirpy theme](https://github.com/cotes2046/jekyll-theme-chirpy), hosted on [GitHub Pages](https://pages.github.com/), in my own "Cosmik Pulp" color scheme. I've built hundreds of WordPress sites for clients, but static site generators are a different animal: less infrastructure to babysit, more configuration to understand. And I wanted to understand it, not just copy and paste until the errors went away.

The usual path goes like this. Dig through Chirpy's docs, configure the site, figure out the weird parts as you trip over them, write a post... someday. Maybe document things later. (I did not document things later. I never do.)

So I flipped it. Build the documentation and the workflow first, then let the content follow.

## Why document anything before you have content?

Because future me is a completely different person with zero memory of why past me did anything. I'm not being dramatic. I honestly could not tell you why I made half the configuration choices in my last three projects without digging through git history like Indiana Jones.

Writing the docs with Claude *while* I was making the decisions meant I caught the reasoning while I still had it. We made five files before I wrote a single post:

**claude.md**: the project overview, tech stack and common tasks. It's the briefing doc for every future session where I say "remind me how this works again."

**workflow.md**: git conventions, branch naming and a pre-publish checklist. It has already saved me from skipping steps twice.

**content.md**: the post backlog, ideas and writing guidelines. It's the holding tank where half-baked thoughts sit until they grow up into posts.

**tags.md**: the tag list. Sounds tedious, I know, but tagging consistently from day one is way easier than cleaning it up after 50 posts.

**design.md**: the Cosmik Pulp colors, font choices and where the override files live. So when I want to tweak something six months from now, I know which file to open.

Is that overkill for a blog with zero posts? Probably. But here's the thing: I actually did it, because Claude helped me do it right then instead of "later." And actually doing it beats planning to do it every time.

## What working with it actually felt like

Not what I expected. I figured it would feel like handing work to a really fast intern. Instead it felt more like thinking out loud with someone who remembered everything I said and had read the Chirpy docs way more carefully than I ever was going to.

It usually went something like this:

**Me:** "I want to customize the colors with the Cosmik Pulp scheme."

**Claude:** "Here's how Chirpy's Sass overrides work, here are the files you'll need, and here's the CSS. Want me to document this in design.md?"

**Me:** "Yes, and add a checklist for testing it so I don't break dark mode."

And BAM, it existed. The plan, the code and the testing checklist, all in one place. Doing it right took less effort than skipping it, which is a sentence I never thought I'd write about documentation.

Here's what worked:

**It made me think straight.** Having to explain what I wanted forced me to figure out what I wanted. "I want it to feel technical but not cold" turned into actual color values and font pairings.

**Documentation stopped being a chore.** When I could just think out loud and have it written down for me, the excuse disappeared. I am not a natural documenter. I am now, sort of by accident.

**Conventions stuck.** Once we settled on things like front matter format, tags and branch naming, Claude picked them up and kept them without me nagging. It's a small thing, but small things add up.

**I actually learned it.** Because Claude explained *why* Jekyll and Chirpy are set up the way they are, I understand what I built. When something breaks later (and it will), I'll know where to look instead of poking at files until it stops yelling at me.

And here's what was harder than I expected:

**Remembering across sessions.** In longer conversations I sometimes had to re-explain decisions from earlier. The docs helped a lot here, since we could point at what we'd written instead of rebuilding it from memory. It's still friction, though. I'm getting better at loading up the context at the start of each session.

**Knowing what to ask.** The quality of what I got back matched how clearly I asked for it. "Make me a blog" gets you something generic. "A blog with this look, for this audience, with this workflow" gets you something useful. It gets easier with practice, but it is a skill.

**Staying skeptical.** AI can be confidently wrong. Jekyll and Chirpy are very particular about where files go and how overrides behave, and not every suggestion was right the first time. When something didn't work the way it was described, I went back to the actual Chirpy docs. Trust but verify isn't optional here.

## The part that surprised me most

A few hours in, I had a site that worked the way I wanted, docs I could actually use and a workflow I understood from end to end. Better yet, I still had the energy to keep going.

That last part is the one I'd underestimated. My old blogs all died during setup, because setup is boring and momentum is fragile. Having a partner who never got bored of the infrastructure details kept my energy on the fun part (what I actually wanted to make) instead of burning it all on config files.

It isn't magic and it isn't autopilot. It's closer to pair programming with someone who has endless patience for tedious setup questions, but who still needs you to be the one who knows what you're trying to build.

## If you want to try this

A few things I'd do the same way again:

**Start with the docs.** Before you write any content or push any code, spend 30 minutes setting up your documentation. What decisions are you going to make, and where will you write them down? Do it while the project is small enough that it takes 30 minutes and not three hours.

**Be specific.** "Jekyll blog with the Chirpy theme, Cosmik Pulp colors, written for homelab builders and makers, posts about Linux and maker projects." That's useful. "Personal blog" is not.

**Push back.** "That's close but wrong, let's change X" works way better than taking the first version and working around it. Say what's off. The second or third try is usually the one.

**Understand what you're building.** Don't just accept the code it gives you. Ask why. Make sure it matches how you think the tool works. That's what saves you when things break later... and things will break later.

---

The foundation is in. Now for the part I actually wanted to get to: writing about the stuff I'm building.

Which, since you're reading this, seems to be going OK.

Jason
