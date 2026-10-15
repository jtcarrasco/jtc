---
layout: post
title: "Claude Code Is My Collaborator. Obsidian Is Our Shared Brain."
description: "How I use CLAUDE.md files, a memory system and vault-based agents to give Claude persistent context, plus the step-by-step setup to build your own."
date: 2026-05-12 09:00:00 -0700
published: true
image: '/assets/blog/claude-obsidian-workflow-tutorial.webp'
tags: [ai, tutorial, claude, obsidian, pkm, automation]
---
Most people use AI the way they use a search engine. Ask a question, get an answer, close the tab. I use Claude Code differently. For me it's a long-term collaborator that knows my systems, my projects and how I like to work. And the thing that makes that possible is [Obsidian](https://obsidian.md).

This is the post I wish I'd had when I started building this setup. It covers how it works, why I built it this way and what it actually looks like day to day.

## The problem with an AI that forgets everything

Every new conversation with an AI starts from zero. You explain your setup again, your preferences again, what you tried last time and what you're trying to do now. Even with the built-in memory features, what it remembers is pretty thin.

That's fine for one-off questions. It's not fine if you want help running real ongoing stuff: agents on a schedule, a blog that's half written, a homelab that needs looking after. For that, it has to remember.

Every time I started a [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) session, I spent the first five minutes re-explaining who I am, what I'm working on and why my folders look the way they do. Same speech, every single time. It was like onboarding a contractor who shows up with no memory of the last twelve times they were here.

The fix was almost embarrassingly simple. Give Claude a file to read.

## Obsidian as the memory

My Obsidian vault syncs to all my devices with [Syncthing](https://syncthing.net). It uses a PARA-style layout: Inbox, Projects, Areas, Resources and Archive.

Claude Code can read and write to that vault. So:

- When I ask Claude to draft a blog post, it reads my other drafts, checks the content plan and saves the draft in the right folder.
- When we set up a new agent, Claude reads the agent docs, writes the agent's CLAUDE.md and updates the shared index.
- When I ask where a project stands, Claude reads the project file and gives me a real answer instead of a made-up one.

The vault isn't just my notes anymore. It's our shared working memory.

## CLAUDE.md files: instructions that stick

The most important piece is a file called `CLAUDE.md`. Claude Code reads it at the start of every session, and it holds everything Claude needs to know: how my folders are laid out, my conventions, what to never do and what to always do.

I have a global `CLAUDE.md` in my home folder and a project-specific one in each agent's folder. When Claude starts a session in the `wiki_agent/` folder, it reads that agent's CLAUDE.md and knows exactly what the agent does, which files it touches and what rules it follows, without me saying a word.

That's the difference between a tool and a collaborator. A tool needs instructions every time. A collaborator reads the brief.

## Memory files: context that carries forward

On top of CLAUDE.md, there's a memory system: markdown files that Claude writes and reads from one session to the next. My preferences, where projects stand, feedback from past sessions and reference info.

When I told Claude I always want a dry run before it bulk-deletes email, that went into a memory file. Now it just happens, every time, without me asking.

That's what makes this feel different from a normal AI chat. It adapts, and it keeps adapting.

## The agent layer

The deeper part is the agents. I have several Claude Code agents running on my VPS with no one at the keyboard:

- **job_agent**: searches job boards on a schedule and writes reports to the vault
- **wiki_agent**: turns articles and YouTube videos into a structured wiki
- **events_agent**: finds tech meetups and conferences once a month

Each agent has a `CLAUDE.md` (its instructions) and a `task-list.md` (its to-do list). A bash script I call the dispatcher runs on cron every 5 minutes, looks for trigger tags in the task files and hands the work to Claude Code when it finds one.

So the control panel is a markdown file. I edit it on my phone in Obsidian, [Syncthing](https://syncthing.net) pushes it to the VPS, Claude picks it up within 5 minutes and does the job, and the results sync right back to my phone. Heck, I can kick off an agent from the couch.

## The phone layer: Discord and Remote Control

For back-and-forth work while on the go, I use a Discord channel. Claude Code watches it and handles file changes, web searches, calendar events and whatever else I throw at it. It's the conversation layer sitting on top of the files.

When I want more than a chat window, I use Claude Code's [Remote Control](https://code.claude.com/docs/en/remote-control). It puts the actual session running on my VPS in the Claude app on my phone, so I can pick up exactly where I left off at my desk. Discord is great for quick asks. Remote Control is for when I want the whole workbench in my pocket.

The split matters. Discord and Remote Control are for talking, Obsidian is for keeping track. The work happens in files. The phone is just how I start it.

## What it actually looks like

Here's one day as an example. We set up a Drupal sandbox, organized a book list, drafted a uses page and a bookshelf page, ran the events agent to find local meetups, built a blog content plan and wrote five posts. Everything landed in the vault. Everything is saved. The next session picks up right where we left off.

That's a lot of ground for one afternoon. The speed comes from never re-explaining anything. Claude already knows where things live, what my conventions are and what we're building.

---

## Build it yourself

It isn't magic. It takes some work up front:
- A reasonably organized vault with a clear folder structure
- Well-written CLAUDE.md files that tell Claude what it needs to know
- Being OK with an AI writing to your files (you can always review the changes)

Once that's in place, though, it snowballs. Every session adds to what Claude knows. Every new agent slots right in. The more you use it, the more useful it gets. [Cory Doctorow](https://pluralistic.net/) would call that being a centaur instead of a reverse centaur: I do the steering, and the machine does the legwork.

**What you'll end up with:** Claude Code with read/write access to your [Obsidian](https://obsidian.md/) vault, CLAUDE.md files that persist instructions across sessions, a memory system that carries behavioral context forward, and agent tasks triggered by editing markdown files on your phone.

**Prerequisites:** Claude Code CLI installed, Obsidian vault set up, a VPS or always-on machine (optional, for scheduled agents), [Syncthing](https://syncthing.net/) for vault sync.

---

**A note on PARA:** This guide uses the PARA folder structure for organizing the vault. PARA stands for **Projects, Areas, Resources, Archive**: four top-level folders that cover everything you're working on or referencing. Projects are active work with a finish line. Areas are ongoing responsibilities with no end date. Resources are reference material. Archive is everything inactive. You don't have to use PARA, but the path examples throughout (`Projects/Agents/`, `Resources/`, etc.) assume it. Adapt to whatever structure you use. If you're new to it, [ObsiBrain's PARA overview](https://www.obsibrain.com/docs/features/p.a.r.a-folder-structure) is a good primer.

---

## Step 1: Give Claude Code Access to Your Vault

Claude Code runs with access to whatever directories you allow. Point it at your vault root. For this tutorial we will use `notes` as our vault.

In your global Claude settings (`~/.claude/settings.json`):
```json
{
  "allowedPaths": [
    "/home/youruser/notes",
    "/home/youruser/github"
  ]
}
```

Or just launch Claude Code from within your vault directory:
```bash
cd ~/notes
claude
```

Claude can now read and write any file in your vault. Start small: ask it to read a note, summarize a project file or draft something into a specific folder. Make sure it can actually find things before you build anything on top of it. Claw before you sprint! 

---

## Step 2: Keep Claude Running with tmux

If you run Claude Code on a VPS, you'll hit this one fast: SSH sessions die. Close the laptop, drop the WiFi or wander off for an hour, and the session is gone, along with any Claude session running inside it.

[tmux](https://github.com/tmux/tmux) fixes that. It keeps terminal sessions alive on the server no matter what happens to your SSH connection. Connect, attach to your tmux session, do your work, detach and close the laptop. The session keeps running. Come back later from your phone, reattach, and everything is right where you left it.

```bash
# Install tmux
apt install tmux -y

# Start a persistent session named "claude"
tmux new-session -s claude

# Inside the session, launch Claude Code from your vault
cd ~/vault
claude
```

Detach without killing anything: `Ctrl+B`, then `D`. You're back at a normal shell. The Claude session is still running on the server.

Reconnect from anywhere:
```bash
ssh youruser@your-machine.your-tailnet.ts.net
tmux attach -t claude
```

The basics you'll use constantly:
- `Ctrl+B, D`: detach (session keeps running)
- `Ctrl+B, C`: new window within the session
- `tmux ls`: list running sessions
- `tmux attach -t claude`: reattach by name

The dispatcher in Step 7 runs from cron and doesn't need tmux, since it starts its own Claude processes. tmux is for the sessions where you're actively working, or for agent runs you want to watch live.

---

## Step 3: Write Your Global CLAUDE.md

Create `~/.claude/CLAUDE.md`. Claude reads it at the start of every session. Think of it as the brief you'd hand a new collaborator: everything they need to be useful without you explaining it from scratch. Keep it short, since each project can get its own `CLAUDE.md` with the specifics. There are plenty of good example global `CLAUDE.md` files floating around, so grab a few, edit them and see what sticks.

```markdown
# Claude Global Instructions

## Work Context
All projects under ~/dev/.

## Directory Structure
- ~/vault/ — Obsidian vault (PARA structure)
  - Inbox/ — unprocessed notes
  - Projects/ — active projects
  - Areas/ — ongoing responsibilities
  - Resources/ — reference material
  - Archive/ — done
- ~/dev/github/ — production code (treat with care)

## Standards
- Match existing file conventions before creating new ones
- Never commit to git without being asked
- Prefer editing existing files over creating new ones

## Communication
- Concise by default, no preamble
```

This file is the reason you stop typing "by the way, I prefer short answers" into every session like some kind of magic spell.

---

## Step 4: Add a Vault-Level CLAUDE.md

Create `~/vault/CLAUDE.md` at the vault root. This is vault-specific context that doesn't belong in global settings:

```markdown
## PKM Workflow
- PARA structure: Inbox, Projects, Areas, Resources, Archive
- New notes go in Inbox/ unless a target location is obvious
- Active projects live in Projects/ (status: active)
- Parked topics go in Areas/Research/ (status: research)
- Todoist handles tasks; Obsidian handles context/notes around tasks
```

That last line is my setup, not a requirement. I like [Todoist](https://todoist.com/) for tasks, but swap in whatever you already use, or keep tasks in Obsidian too. The point is to tell Claude where tasks live so it doesn't have to guess.

CLAUDE.md files in subfolders work the same way. Claude picks them up when it works in those folders. You can get as granular as you like, but start with just the global one and the vault-root one, and only add more when you feel something missing.

---

## Step 5: Set Up the Memory System

A CLAUDE.md file tells Claude how you want it to behave. Memory is where that behavior grows over time. The difference matters: CLAUDE.md is the standing brief, and memory is everything you've learned working together.

Create a memory directory:

```bash
mkdir -p ~/.claude/projects/$(echo $HOME | tr '/' '-')/memory/
touch ~/.claude/projects/$(echo $HOME | tr '/' '-')/memory/MEMORY.md
```

Write a `MEMORY.md` index:
```markdown
# Memory

## User Preferences
- Concise responses, no preamble
- 2-space indentation

## Environment
- VPS: Ubuntu 24.04, Tailscale active
- Vault: ~/vault/
```

Then add one memory file per topic: feedback, project context, reference info Claude shouldn't have to figure out twice. MEMORY.md is just the index. Claude reads it at the start of a session and opens the individual files when it needs them.

Here's the habit that makes it pay off. When Claude does something you want it to always do (or never do again), save that correction to a memory file. "Always dry-run before deleting" goes in `feedback_deletions.md`. Next session, it remembers. You stop being the full-time error-correction department.

---

## Step 6: Create Agent Directories in Your Vault

Now it gets fun. Agents live in `Projects/Agents/`. Each agent is just a folder with its instructions (CLAUDE.md) and a task list, and that's enough to build something that runs on a schedule and writes its results back into your vault.

```bash
mkdir -p ~/vault/Projects/Agents/my_agent/{knowledge,reports,logs}
```

**`CLAUDE.md`**: the agent's job description
```markdown
# CLAUDE.md — my_agent

## Identity
[What this agent does in one sentence]

## Working Directory
~/vault/Projects/Agents/my_agent/

## Schedule
<!-- schedule: day=1, hour=09, task=Weekly run -->

## Tasks

### Weekly run
[Step-by-step instructions for this task]

## Output Rules
- Write reports to reports/YYYY-MM-DD.md
- Update task-list.md when done
```

**`task-list.md`**: the trigger mechanism
```markdown
# my_agent — Task List

## Backlog

## In Progress

## Done
```

The task list looks too simple to matter, but it does a lot. It's the file you edit from your phone when you want to kick something off by hand, and it's the file the dispatcher watches to know when to run.

---

## Step 7: Build the Dispatcher

The dispatcher is the heart of the whole thing. It's a bash script that runs on cron, checks every agent folder for `[trigger]` tags in the task lists, and hands the work to Claude Code. That's about 50 lines of bash doing a job that would have needed a real orchestration system a few years ago.

Create `~/vault/Projects/Agents/dispatcher.sh`:

```bash
#!/bin/bash
AGENTS_DIR="$HOME/vault/Projects/Agents"
CLAUDE_CMD="$HOME/.local/bin/claude"
LOCK_FILE="/tmp/dispatcher.lock"

[ -f "$LOCK_FILE" ] && exit 0
touch "$LOCK_FILE"

for AGENT_DIR in "$AGENTS_DIR"/*_agent; do
  [ -d "$AGENT_DIR" ] || continue
  TASK_FILE="$AGENT_DIR/task-list.md"
  CLAUDE_FILE="$AGENT_DIR/CLAUDE.md"
  [ -f "$TASK_FILE" ] || continue
  [ -f "$CLAUDE_FILE" ] || continue
  [ -f "$AGENT_DIR/.env" ] && source "$AGENT_DIR/.env"

  grep -q "^- \[ \] \[trigger\]" "$TASK_FILE" || continue

  AGENT_LOCK="/tmp/$(basename $AGENT_DIR).lock"
  [ -f "$AGENT_LOCK" ] && continue
  touch "$AGENT_LOCK"

  TRIGGERED=$(grep "^- \[ \] \[trigger\]" "$TASK_FILE")
  sed -i "s/^- \[ \] \[trigger\]/- [ ] [in-progress] — started: $(date '+%Y-%m-%d %H:%M')/g" "$TASK_FILE"

  unset CLAUDECODE
  PROMPT="You are an agent. Read $CLAUDE_FILE first.
Your task file is at $TASK_FILE.
Triggered tasks: $TRIGGERED
Execute per your CLAUDE.md instructions.
When done, mark completed tasks [done] with timestamp in $TASK_FILE."

  echo "$PROMPT" | $CLAUDE_CMD --print --permission-mode bypassPermissions 2>&1

  rm -f "$AGENT_LOCK"
done

# Inject scheduled triggers
DAY=$(date '+%u')
HOUR=$(date '+%H')
for AGENT_DIR in "$AGENTS_DIR"/*_agent; do
  TASK_FILE="$AGENT_DIR/task-list.md"
  CLAUDE_FILE="$AGENT_DIR/CLAUDE.md"
  [ -f "$TASK_FILE" ] || continue
  while IFS= read -r line; do
    SCHED_DAY=$(echo "$line" | grep -oP 'day=\K[0-9]+')
    SCHED_HOUR=$(echo "$line" | grep -oP 'hour=\K[0-9]+')
    SCHED_TASK=$(echo "$line" | grep -oP 'task=\K[^-]+' | xargs)
    if [ "$DAY" == "$SCHED_DAY" ] && [ "$HOUR" == "$SCHED_HOUR" ]; then
      grep -q "^\- \[ \] \[trigger\] $SCHED_TASK" "$TASK_FILE" || \
        sed -i "/^## Backlog/a - [ ] [trigger] $SCHED_TASK" "$TASK_FILE"
    fi
  done < <(grep "<!-- schedule:" "$CLAUDE_FILE")
done

rm -f "$LOCK_FILE"
```

```bash
chmod +x ~/vault/Projects/Agents/dispatcher.sh
```

**Don't skip the lock files.** The one at the top stops the dispatcher from running twice while an earlier Claude session is still going, and the per-agent lock does the same for each agent. A dispatcher that stacks up Claude sessions on top of each other will chew through your budget fast.

---

## Step 8: Set Up Cron

```bash
crontab -e
```

Add:
```
*/5 * * * * bash "/home/youruser/vault/Projects/Agents/dispatcher.sh" >> "/home/youruser/vault/Projects/Agents/dispatcher.log" 2>&1
```

Every 5 minutes the dispatcher wakes up, checks for triggers, runs anything pending and goes back to sleep. The log file is where you look when something doesn't run the way you expected.

---

## Step 9: Trigger an Agent From Your Phone

This is the payoff for all that setup, and the first time it works is pretty great.

With Syncthing syncing your vault to your phone:

1. Open `task-list.md` in Obsidian on your phone
2. Add under Backlog: `- [ ] [trigger] Weekly run`
3. Save. Syncthing pushes to the VPS in seconds
4. Dispatcher picks it up within 5 minutes
5. Report appears in your vault within ~10 minutes

You just kicked off an AI agent on a server, from your phone, by editing a text file. No app, no webhook, no API key to manage. Just a markdown checkbox.

---

## Step 10: Connect Discord as a Trigger Interface

The phone trigger works, but opening a markdown file isn't always the fastest way. With Discord you type a message in a channel and an agent is running a few minutes later, from any device, without ever opening your vault.

**Create a Discord bot:**

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) and click **New Application**
2. Name it (e.g., "homelab-agent"), then go to **Bot** in the left sidebar
3. Click **Add Bot**, then **Reset Token** and copy the token. This is your `DISCORD_BOT_TOKEN`. Store it securely, you won't see it again
4. Under **Privileged Gateway Intents**, enable **Message Content Intent**
5. Go to **OAuth2 > URL Generator**: check `bot` under Scopes, then check `Send Messages`, `Read Message History`, and `Read Messages/View Channels` under Bot Permissions
6. Copy the generated URL, open it in a browser, and invite the bot to your server

**Get your channel ID:**

In Discord, go to User Settings > Advanced and enable **Developer Mode**. Then right-click the channel you want the bot to watch and click **Copy Channel ID**.

**Install the Discord MCP plugin:**

```bash
claude mcp add discord-bot -- npx -y @anthropic/mcp-server-discord
```

Or add it manually to `~/.claude/settings.json`:
```json
{
  "mcpServers": {
    "discord": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-discord"],
      "env": {
        "DISCORD_BOT_TOKEN": "your-bot-token-here",
        "DISCORD_CHANNEL_ID": "your-channel-id-here"
      }
    }
  }
}
```

**Wire Discord to the dispatcher:**

Create an inbox directory:
```bash
mkdir -p ~/.claude/channels/discord/inbox
```

Add a watcher block to your `dispatcher.sh` (before the final `rm -f "$LOCK_FILE"`):

```bash
DISCORD_INBOX="$HOME/.claude/channels/discord/inbox"
if compgen -G "$DISCORD_INBOX"/*.txt "$DISCORD_INBOX"/*.md > /dev/null 2>&1; then
  for MSG_FILE in "$DISCORD_INBOX"/*.txt "$DISCORD_INBOX"/*.md; do
    [ -f "$MSG_FILE" ] || continue
    MSG=$(cat "$MSG_FILE")
    # Route to the right agent based on message prefix
    if echo "$MSG" | grep -q "^wiki:"; then
      URL=$(echo "$MSG" | sed 's/^wiki: *//')
      echo "$URL" > "$HOME/vault/Inbox/wiki_queue/discord-$(date +%s).md"
    fi
    rm -f "$MSG_FILE"
  done
fi
```

**The result:**

Type `wiki: https://example.com` in your Discord channel and the bot writes it to the inbox. The dispatcher picks it up within 5 minutes, runs your wiki agent on that URL and posts the result back to Discord. No SSH, no vault editing, no context switching.

You can add a route for any trigger you like: `job brief`, `lint`, `report`, whatever your agents know how to do. One channel, and your whole agent stack answers to it.

Discord isn't the only option. Claude Code's MCP plugins cover other chat apps too, including [Telegram](https://telegram.org/) and [Slack](https://slack.com/). The pattern doesn't change: messages land in an inbox folder and the dispatcher routes them to the right agent. Swap the plugin, update the inbox path, and the rest of the system stays the same.

**Or skip the bot entirely with Remote Control.** Chat triggers are great for firing off agents, but sometimes you want the actual Claude Code session in your hand. Claude Code's built-in [Remote Control](https://code.claude.com/docs/en/remote-control) lets you drive a session running on your server from the Claude mobile app or [claude.ai/code](https://claude.ai/code). Start it inside your tmux session so it stays up after you disconnect:

```bash
tmux attach -t claude
cd ~/vault && claude --remote-control vault
```

Then open the Claude app on your phone and pick the `vault` session. No bot token, no inbox folder, no dispatcher route. It's the same session you'd see over SSH, just from the couch.

---

## The Mental Model

**Obsidian is the file system. Claude is the worker.** Files hold the state: they stick around between sessions, sync to every device and any tool can read them. Claude reads and writes those files. So the vault turns into shared working memory, with your notes, Claude's output and the agents' task files all in one place.

CLAUDE.md files give Claude standing instructions so you don't have to repeat yourself. Memory files let it pick up context over time. Task files let you trigger agents without logging into anything.

The whole system is a pile of markdown files and a bash script. That's not a weakness, that's the point. Plain text ages well. It works in every editor, syncs with everything and will never need a migration. Not bad for a bunch of text files.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
