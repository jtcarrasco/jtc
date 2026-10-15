---
layout: post
title: "Building a Compounding Knowledge Base With an AI Wiki Agent"
description: "How I built an agent that turns articles and YouTube videos into a linked wiki that gets more useful the more you feed it, with the full build steps."
date: 2026-05-15 09:00:00 -0700
published: true
image: '/assets/blog/wiki-agent-knowledge-management-tutorial.webp'
tags: [ai, tutorial, claude, knowledge-management, obsidian, homelab]
---

Here was my system before this project. Read an interesting article, think "I should remember this," immediately forget everything in it, open a new tab, and repeat until my browser looked like a hoarder's garage. I had bookmark folders called "Read Later" that I hadn't opened since 2019. Probably some good stuff in there. Who knows.

Finding stuff to read was never the problem. The firehose is wide open. The problem is that reading something and remembering it are two totally different jobs, and I was only doing one of them.

The trouble with most note-taking systems is that the notes just sit there. You save something, never look at it again, and whatever you learned is basically gone. Search helps, but it can't show you connections you didn't know to look for.

I wanted something that grows. Where adding a new source makes the whole thing smarter, not just bigger.

The idea isn't mine. [Andrej Karpathy](https://karpathy.ai/) laid it out in a gist he called ["LLM Wiki"](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f): instead of having an AI dig through your raw notes every time you ask a question, have it build and maintain a linked markdown wiki that gets better with every source you add. His one-line version is "Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase." I read that and immediately knew what my weekend was going to look like.

So I built `wiki_agent`, an AI agent that reads articles, pulls out the concepts and the names, and works everything into an organized wiki where every page links to the others.

## How it works

The wiki lives in my [Obsidian](https://obsidian.md) vault under `3_Resources/Wiki/`, and it has three kinds of pages:

- **Sources**: one page per article or video. A summary, the main concepts, the people and tools mentioned, and any claims worth remembering.
- **Concepts**: ideas, patterns, methods and frameworks that come up across several sources (like "token efficiency," "knowledge graphs" or "prompt caching").
- **Entities**: specific tools, people, companies and projects (like "Claude Code," "Andrej Karpathy" or "Obsidian").

When I queue a new article, the agent reads it, updates or creates the right concept and entity pages, and links everything together. The wiki's index and log stay up to date on their own.

## The queue

I wanted the simplest interface possible, and here it is: drop a `.md` or `.txt` file with a URL into a folder called `wiki_queue/`. That's it. That's the whole interface.

[Syncthing](https://syncthing.net) spots the file and pushes it to the VPS, and the dispatcher picks it up within 5 minutes. The agent fetches the URL (or grabs the transcript if it's a YouTube video), runs the whole process, and moves the queue file to `processed/` when it's done.

For YouTube, it uses [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) to pull the auto-generated subtitles, strips out the timestamps and treats the cleaned-up transcript like an article. The quality varies, but auto-captions on technical talks turn out to be surprisingly usable.

If you use the [Obsidian Web Clipper](https://obsidian.md/clipper), clipping a YouTube page saves the transcript right into the markdown file. The agent gets it already transcribed, no `yt-dlp` needed.

## The weekly checkup

Once a week the agent gives the wiki a health check:

1. **Index gaps**: pages that exist but aren't in the index
2. **Missing pages**: links that point to a page that doesn't exist
3. **Orphan pages**: pages nothing else links to
4. **Missing links**: concepts or names that get mentioned but not linked
5. **Stale claims**: pages a newer source might contradict (flagged for me, not changed automatically)

Most of it gets fixed on the spot. Missing links get added, index gaps get filled, and the report goes into the wiki log.

## What surprised me

**The links are where the value is.** I didn't see this coming. The most useful thing the agent does isn't summarizing articles. It's noticing that a tool mentioned in article A also came up in articles C and E, and connecting them. That's where it starts to snowball.

**YouTube transcripts work better than I expected.** I was skeptical. A 45-minute technical talk is a LOT of text, but the agent handles it fine, and the source page it writes is usually more useful than my own notes would have been.

**The wiki develops a personality.** After enough sources, the concept pages start to have a point of view. Not just what each article said, but where the sources agree, where they fight, and which claims need checking. I didn't design that. It just happened.

## The dispatcher

`wiki_agent` and `job_agent` run on the same setup: a bash script that loops through the agent folders looking for `[trigger]` tags in their markdown task files. Each agent's instructions live in its `CLAUDE.md`, which Claude reads before it does anything.

So adding a new agent means: make a folder, write a `CLAUDE.md`, create a `task-list.md`. The dispatcher finds it on its own.

No custom server. No API endpoints. No code changes to add a new agent. The whole control panel is markdown files synced over Tailscale.

---

## Build it yourself

**What you'll end up with:** Drop a URL in a folder from your phone. A few minutes later your vault has a summary, the key ideas pulled out, pages for the tools and people mentioned, and links to everything else you've ever saved. The wiki gets smarter every time you feed it. You get to feel like a genius without doing the homework.

**Prerequisites:** [Claude Code CLI](https://claude.ai/code), [Obsidian](https://obsidian.md/) vault, [Syncthing](https://syncthing.net/) (or similar sync), [`yt-dlp`](https://github.com/yt-dlp/yt-dlp) or the [Obsidian Web Clipper](https://obsidian.md/clipper) for YouTube support.

---

## Step 1: Set Up the Directory Structure

First, folders. Thrilling, I know, but the agent uses this structure to find things, so it matters.

```bash
# Agent folder
mkdir -p ~/vault/Projects/Agents/wiki_agent/{knowledge,logs}

# Wiki in your vault
mkdir -p ~/vault/Resources/Wiki/{sources,concepts,entities}
mkdir -p ~/vault/Inbox/wiki_queue/processed
```

Three types of wiki pages live here:
- **sources/**: one page per article or video. Your "I read this" receipts.
- **concepts/**: ideas that keep coming up across different sources. When three articles all talk about "context windows," that gets a concept page.
- **entities/**: specific things like tools, people and companies. The nouns.

Then create the index and log files the agent keeps up to date:

```bash
cat > ~/vault/Resources/Wiki/index.md << 'EOF'
# Wiki Index

_Last updated: YYYY-MM-DD — 0 sources, 0 concepts, 0 entities_

## Sources
| Page | Summary | Created |
|------|---------|---------|

## Concepts
| Page | Summary | Updated |
|------|---------|---------|

## Entities
| Page | Type | Summary | Updated |
|------|------|---------|---------|
EOF

touch ~/vault/Resources/Wiki/log.md
```

---

## Step 2: Write the Agent Instructions (CLAUDE.md)

This is the agent's job description. Think of it as the instructions you wish you could hand to every person who ever tried to "help" you organize something.

Create `~/vault/Projects/Agents/wiki_agent/CLAUDE.md`:

```markdown
# CLAUDE.md — wiki_agent

## Identity
Process source documents into a wiki at Resources/Wiki/. Knowledge compounds with every source added.

## Working Directory
~/vault/

## Page Types
- **sources/** — one per article/video. Summary, key concepts, key entities, notable claims.
- **concepts/** — ideas, patterns, frameworks that appear across multiple sources.
- **entities/** — specific tools, people, companies, products.

All pages use frontmatter:
---
type: source|concept|entity
title:
tags: []
sources: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

## Tasks

### process-queue
Process all URLs in Inbox/wiki_queue/ (skip processed/ subfolder).

For each file:
1. Extract URLs (bare URLs or markdown links)
2. For YouTube URLs (youtube.com/watch, youtu.be/): use yt-dlp to get transcript
   - Command: yt-dlp --skip-download --write-auto-subs --sub-langs en --convert-subs srt -o "/tmp/wiki-yt-%(id)s.%(ext)s" "URL"
   - Strip timestamp lines, use cleaned text as source
3. For all other URLs: fetch with WebFetch
4. Run ingest pipeline:
   - Write sources/<slug>.md
   - Update or create concept pages
   - Update or create entity pages
   - Update index.md
   - Append to log.md: ## [YYYY-MM-DD] ingest | <title> | N pages touched
5. Move queue file to Inbox/wiki_queue/processed/

### Wiki lint
Health check: index gaps, missing pages, orphans, missing cross-references, stale claims.
Write report to Resources/Wiki/lint-YYYY-MM-DD.md
Append summary to log.md.

## Output Rules
- Never modify files in Inbox/ — sources are immutable
- Never overwrite wiki pages — update in place, update the updated: frontmatter field
- Wikilinks: [[page-name]] for all cross-references
- Filenames: lowercase-hyphens.md
- Always update index.md and log.md on every run
```

**The output rules at the bottom are the important part.** "Never overwrite wiki pages, update in place" is the difference between a wiki that keeps learning and one that gives itself amnesia every time it runs.

---

## Step 3: Create the Queue File Format

The queue is gloriously low-tech. It's a folder. Drop a `.md` or `.txt` file in it with one or more URLs and the agent takes it from there.

```
# Queue file: ai-articles.md
https://example.com/article-about-llms
[Great article on prompting](https://example.com/prompting)
https://www.youtube.com/watch?v=VIDEO_ID
```

Bare URLs, markdown links or a mix are all fine. One URL per line, as many per file as you like. Once Syncthing pushes the file to your VPS, cron picks it up within 5 minutes, and the file gets processed, moved to `processed/`, and turned into wiki pages.

No app. No browser extension. No "save for later" button that sends things off to digital purgatory. Just a text file.

---

## Step 4: Install yt-dlp (for YouTube Support)

Because a lot of good information is apparently locked inside people talking at a camera for 45 minutes. The agent uses `yt-dlp` to grab the auto-generated transcript and treats it like an article.

```bash
pip install yt-dlp
# or
sudo apt install yt-dlp
```

Test it with a classic:

```bash
yt-dlp --skip-download --write-auto-subs --sub-langs en --convert-subs srt \
  -o "/tmp/wiki-test-%(id)s.%(ext)s" "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

If a `.en.srt` file shows up in `/tmp`, it's working. (Yes, the test URL is Rick Astley. If you queue it later, your knowledge base will have a page about "Never Gonna Give You Up" as one of its first entries, which is honestly the best possible start.)

**If you use the [Obsidian Web Clipper](https://obsidian.md/clipper):** clipping a YouTube page from your browser puts the transcript right in the saved markdown file. When Syncthing pushes that file to your VPS, the agent already has the transcript and doesn't need yt-dlp at all. If you already clip from a browser, this is the easy route. yt-dlp covers the times you queue a bare URL without going through a browser.

---

## Step 5: Configure the Queue Auto-Trigger in the Dispatcher

Add this to `dispatcher.sh` before the cleanup section. It checks whether anything is sitting in the queue folder and, if there is, adds a trigger so the agent picks it up on the next cron run:

```bash
WIKI_QUEUE="$HOME/vault/Inbox/wiki_queue"
WIKI_TASK="$AGENTS_DIR/wiki_agent/task-list.md"

if [ -f "$WIKI_TASK" ] && compgen -G "$WIKI_QUEUE"/*.md "$WIKI_QUEUE"/*.txt > /dev/null 2>&1; then
  if ! grep -q "^- \[ \] \[trigger\] process-queue\|^- \[ \] \[in-progress\].*process-queue" "$WIKI_TASK"; then
    sed -i "/^## Backlog/a - [ ] [trigger] process-queue" "$WIKI_TASK"
    echo "[wiki_agent] Files in queue. Injected process-queue trigger."
  fi
fi
```

The guard stops it from triggering twice when you dump three files in at once. The agent runs once and handles all of them.

---

## Step 6: Create the Task List

`~/vault/Projects/Agents/wiki_agent/task-list.md`:

```markdown
# wiki_agent — Task List

> **Queue trigger:** Drop a .md or .txt file with a URL into Inbox/wiki_queue/
> Cron detects it and triggers process-queue within 5 minutes.
>
> **Manual ingest:** Add below with [trigger] and filename:
> `- [ ] [trigger] ingest: filename.md`
>
> **Lint run:** Add:
> `- [ ] [trigger] lint`

---

## Backlog

---

## In Progress

---

## Done
```

This file is how you talk to the agent. It's a markdown file with checkboxes, and cron reads it. There's something deeply satisfying about kicking off an AI system by editing a text file on your phone.

---

## Step 7: Set Up the Wiki Page Templates

The agent writes these pages for you, but here's what a source page looks like so you know what to expect:

```markdown
---
type: source
title: Article Title Here
tags: []
sources: []
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

## Summary
[2–4 sentences on the article's main argument]

## Key Concepts
- [[concepts/concept-name]] — how it appears here

## Key Entities
- [[entities/entity-name]] — how it's mentioned here

## Notable Claims
- [specific, quotable claims worth tracking]

## Source
[Original title and URL]
```

The wikilinks (`[[concepts/concept-name]]`) are where the magic happens. When the agent reads a new article and finds a concept that already has a page, it links to it. When it finds a new one, it creates the page and links it back to the source. Over time the connections pile up in ways you never planned.

---

## Step 8: Test the Queue

Make sure this thing actually works before you trust it with anything important.

```bash
echo "https://en.wikipedia.org/wiki/Knowledge_graph" > ~/vault/Inbox/wiki_queue/test.md
```

Wait 5 minutes (or run the dispatcher manually). You should see:

1. `Inbox/wiki_queue/processed/test.md`: moved here, out of the queue
2. `Resources/Wiki/sources/knowledge-graph.md`: a new source page
3. `Resources/Wiki/index.md`: updated with the entry
4. `Resources/Wiki/log.md`: one new line

If all four check out, BAM, you have a working wiki agent. Everything from here on is just feeding it URLs.

---

## Step 9: Schedule the Weekly Lint

The lint task is the agent checking its own homework. Every Sunday it looks for orphaned pages, missing links, gaps in the index, and concept pages with only one source (which means they're not really concepts yet, just words). It writes a report and leaves it in your wiki.

Add to `wiki_agent/CLAUDE.md`:

```markdown
## Schedule
<!-- schedule: day=7, hour=10, task=Wiki lint -->
```

Day 7 is Sunday and hour 10 is 10am in your server's timezone. The dispatcher reads the schedule comment and adds the lint trigger on its own. You never have to remember to run it, which is kind of the whole point of all this.

---

## What It Looks Like After a Few Weeks

Things start to snowball around 10 to 15 sources. Before that, it's mostly pages sitting on their own. After that, the links start pulling their weight.

A concept page like `[[concepts/retrieval-augmented-generation]]` might end up with 8 sources linked to it from different angles: an intro explainer, a research paper, a how-to guide and a skeptic's take. The entity page for `[[entities/anthropic]]` might have notes from a dozen articles, each adding something new.

Open the [Obsidian graph view](https://help.obsidian.md/Plugins/Graph+view) around then. You'll see clusters you never designed. That's the snowball: the wiki knows things you never directly taught it, because the patterns came out of the sources.

It's a weird feeling. A good one, though.

**Three Obsidian features that pair well with a populated wiki:**
- **Graph view**: the "oh wow" moment when you see all the connections
- **Backlinks panel**: shows what else links to whatever page you're reading
- **Search**: search a concept's name to see everywhere it shows up

---

## Troubleshooting

**Queue file isn't triggering:** make sure your dispatcher has the queue-check block and the file ends in `.md` or `.txt`. The dispatcher only looks for those two.

**YouTube transcript is empty:** some videos turn off auto-subs. The agent falls back to fetching the video page, which at least gets the description.

**Index is out of date:** run a lint by hand. Add `- [ ] [trigger] Wiki lint` to the task list. The lint rebuilds the index from what's actually in the folders, which fixes gaps left by interrupted runs.

**The Rick Astley page is actually in your wiki:** you queued the test video. Congratulations, you've been rickrolled by your own knowledge base.

---

## What I'd add

The obvious next step is pulling in articles from my RSS feeds automatically, so new posts from feeds I follow get queued without me lifting a finger. The queue is already set up for it. I just need to wire up a [FreshRSS](https://freshrss.org)-to-queue pipeline.

A search tool would help too. Right now I explore the wiki with [Obsidian's graph view](https://help.obsidian.md/Plugins/Graph+view) and plain search. Something that could answer "show me everything about token efficiency from the last 30 days" would make all those connections a lot easier to use.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
