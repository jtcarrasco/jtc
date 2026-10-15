---
layout: post
title: "I Built an AI Agent That Searches for Jobs While I Sleep"
description: "How I built a Claude Code agent that searches job APIs, deduplicates the results and writes a report a few times a week, with the full build steps."
date: 2026-05-14 09:00:00 -0700
published: true
image: '/assets/blog/job-agent-ai-job-search-tutorial.webp'
tags: [ai, tutorial, claude, automation, job-search, homelab]
---

Job searching is technically doable and practically soul-crushing. Especially the part where you open five job boards, type the same searches you typed last Tuesday, scroll past the same listings you already saw, and close the browser having accomplished nothing except losing a little more of your will to live.

One thing I've been focused on this year is building tools that take repetitive brain work off my plate. Not just automating clicks, but automating judgment calls. The job search agent is a perfect example. Instead of running the same searches by hand every few days, I wrote down what I'm looking for in a markdown file, and a Claude Code agent does the rest. It's what AI agents look like when they're pointed at real, ongoing work instead of a demo.

## What it does

`job_agent` runs on my VPS three to five times a week, depending on the search type. Every run, it:

1. Searches several job board APIs and RSS feeds with specific queries
2. Throws out anything it has already shown me in the last 90 days
3. Filters what's left through my criteria (remote, web dev focus, a few specific exclusions)
4. Writes a markdown report with a table of results, notes on each posting and a look at hiring trends
5. Drops the report in my Obsidian vault, where I can read it on any device

No dashboard. No special app. Just a markdown file that syncs to my phone with [Syncthing](https://syncthing.net).

## Where it looks

It pulls from a handful of places:

- **[We Work Remotely](https://weworkremotely.com) RSS feeds**: reliable and all about remote roles
- **[Post Status Jobs](https://poststatus.com/jobs/)**: a WordPress and web-focused board
- **[JSearch API](https://openwebninja.com)**: LinkedIn, Glassdoor, Indeed and ZipRecruiter in a single call
- **[Adzuna API](https://developer.adzuna.com)**: another aggregator with good US coverage
- **EdTech job boards and local school district boards**: niche sources for my area

Three search types run on different schedules: full-time web development, full-time project management, and freelance or contract. Each one has its own criteria file and its own report folder.

## How it's put together

The agent is [Claude Code](https://claude.ai/code) running with nobody at the keyboard. No server, no framework, just Claude with a set of instructions and access to the files.

A bash script I call the dispatcher runs on cron every 5 minutes and checks each agent's folder for a `[trigger]` tag in its `task-list.md`. When it finds one, it hands Claude the agent's `CLAUDE.md` (its instructions) and the triggered task, and lets it go. Claude reads the criteria, hits the APIs, removes duplicates, writes the report and updates the task file.

Triggering it is dead simple. Edit a markdown file on my phone, Syncthing pushes it to the VPS, and cron picks it up within 5 minutes.

```
task-list.md (on phone) → Syncthing → VPS → cron → Claude → report → Syncthing → phone
```

## What the reports look like

Each report is a dated markdown file with:

- A table of results (company, title, remote or not, salary range, apply link)
- A few sentences on each posting (fit, red flags, tech stack, type of employment)
- A summary at the end: hiring trends, the most-requested skills and the roles worth jumping on first

Claude writes these fresh every run, and honestly they're better than anything I'd write at 8am doing the same search by hand.

## The duplicate problem

The biggest headache was duplicates. Job boards recycle postings like crazy, and the same role shows up on three different sites with three different IDs. Without dedup, running the same searches twice a week would serve up the same 20 jobs every single time.

The fix is a plain text file with one line per posting: `YYYY-MM-DD URL`. At the start of every run, anything older than 90 days gets cleared out. During the run, any URL already on the list gets skipped. After the run, the new URLs get added.

It's not fancy, but it works, and I can open it and see exactly what's going on.

## What surprised me

**Every API is broken in its own special way.** Adzuna doesn't accept `where=remote`. JSearch moved from RapidAPI to a new provider halfway through the project. RSS feeds can't agree on a date format. A real job search agent has to roll with all of it and tell you plainly what it skipped and why.

**The niche sources are way better.** We Work Remotely gives better results than the big aggregators. Almost everything on Post Status is actually relevant. The LinkedIn, Indeed and ZipRecruiter pile through JSearch is noisy by comparison.

**Showing up on schedule matters more than search quality.** It runs twice a week, every week, without me having to remember. That's the real value. Not the AI, the consistency.

---

## Build it yourself

**What you'll end up with:** An agent that runs 3-5x/week, hits multiple job APIs, deduplicates against a running list of seen postings, and writes a markdown report readable in Obsidian or any text editor.

**Prerequisites:** [Claude Code](https://claude.ai/code) CLI installed, a VPS or always-on machine, an Obsidian vault (or any folder of markdown files), API keys for [JSearch](https://openwebninja.com) and [Adzuna](https://developer.adzuna.com) (free tiers work fine).

---

## Step 1: Set Up the Agent Directory

Make the folders first, because the agent expects these exact paths to exist. The three report folders match the three search types: `w2` (full-time jobs), `freelance` (contract/1099) and `pm` (project management). Rename or drop any you don't need.

```bash
mkdir -p ~/vault/Projects/Agents/job_agent/{knowledge,reports/{w2,freelance,pm},logs}
cd ~/vault/Projects/Agents/job_agent
```

Create the required files:
```bash
touch knowledge/criteria-w2.md
touch knowledge/criteria-freelance.md
touch knowledge/criteria-pm-w2.md
touch knowledge/seen-postings.md
touch knowledge/blacklist.md
touch task-list.md
```

`seen-postings.md` is how the agent remembers what it already showed you. `blacklist.md` is for the companies you add after reading one of their postings and immediately knowing "nope."

---

## Step 2: Write the Agent's Instructions (CLAUDE.md)

`CLAUDE.md` is the agent's whole job description. Claude reads it before it touches anything else. It's the difference between an agent that does what you asked and one that comes up with creative new ideas about what you meant.

Create `CLAUDE.md`:

```markdown
# CLAUDE.md -- Job Search Agent

## Identity
Search for remote roles. Run three search types: W2 web dev, W2 project management, freelance/contract.

## Working Directory
~/vault/Projects/Agents/job_agent/

## Schedule
<!-- schedule: day=2, hour=08, task=W2 weekly run -->
<!-- schedule: day=4, hour=08, task=W2 weekly run -->
<!-- schedule: day=1, hour=08, task=Freelance run -->
<!-- schedule: day=3, hour=08, task=Freelance run -->

## Tasks

### W2 weekly run
Search for remote full-time W2 web development roles posted in the last 7 days.

**Sources:**
1. We Work Remotely RSS: https://weworkremotely.com/remote-jobs.rss
2. JSearch API: https://api.openwebninja.com/jsearch/search
   - Header: x-api-key: $JSEARCH_KEY
   - Params: query=remote web developer, date_posted=week, work_from_home=true
3. Adzuna API: https://api.adzuna.com/v1/api/jobs/us/search/1
   - Params: app_id=$ADZUNA_APP_ID, app_key=$ADZUNA_APP_KEY, what=remote web developer, max_days_old=7
   - Note: do NOT use where=remote -- include "remote" in what= instead

**Output:** reports/w2/YYYY-MM-DD.md

[... add Freelance and PM sections similarly ...]

## Report Format
[Define your report structure here -- table of results, posting notes, summary]

## Dedup Rules
- Read knowledge/seen-postings.md at start of every run
- Remove entries older than 90 days
- Skip any URL already in the list
- Append new URLs after writing report: YYYY-MM-DD https://...

## Output Rules
- Never overwrite a previous report -- append -2 suffix if file exists
- Update task-list.md after every run -- mark [done] with timestamp and result count
```

**Don't skip the dedup and output rules at the bottom.** "Never overwrite a previous report" means your history keeps piling up instead of getting wiped. The seen-postings list means you don't read the same recycled posting for the fourth week in a row.

---

## Step 3: Get Your API Keys

**JSearch ([OpenWeb Ninja](https://openwebninja.com)):**
- Sign up at openwebninja.com
- Get your API key
- One call searches LinkedIn, Glassdoor, Indeed and ZipRecruiter at the same time. That's about the best deal going in job search APIs.

**[Adzuna](https://developer.adzuna.com):**
- Register at developer.adzuna.com
- Get `app_id` and `app_key` (free tier: 1000 calls/month, which is plenty at 3-5 runs/week)

Put the keys in a `.env` file in the agent folder, so they stay out of your markdown files:

```bash
cat > .env << 'EOF'
JSEARCH_KEY=your_jsearch_key_here
ADZUNA_APP_ID=your_adzuna_app_id
ADZUNA_APP_KEY=your_adzuna_app_key
EOF
chmod 600 .env
```

---

## Step 4: Write Your Search Criteria

This is where you sit down and figure out what you're actually looking for. The agent reads these files before every search and uses them to judge and filter what comes back.

Edit `knowledge/criteria-w2.md`:

```markdown
# W2 Web Dev Search Criteria

## Include
- Remote or remote-friendly positions
- Web development, front-end, full-stack
- WordPress, React, JavaScript, PHP, HTML/CSS
- US-based companies or open to US applicants

## Exclude
- Pure backend / DevOps / SRE roles
- Design-only (no dev component)
- Non-English job postings
- Anything in blacklist.md

## Salary
- Prefer $80k+ but don't exclude below if role is strong
```

Fill in `knowledge/blacklist.md` as you go. You'll add to it every time a company shows up for the fifth time with the same listing that has been up for six months:

```markdown
# Blacklist
- CompanyToAvoid
- MLM schemes
- Commission-only
```

---

## Step 5: Create the Task List

The task list is how you talk to the agent, and it's just a markdown file. The dispatcher reads it for `[trigger]` tags and acts on whatever it finds. Low-tech on purpose.

Edit `task-list.md`:

```markdown
# job_agent -- Task List

> Add a [trigger] tag to run manually:
> `- [ ] [trigger] W2 weekly run`

---

## Backlog

---

## In Progress

---

## Done
```

---

## Step 6: Build the Dispatcher

The dispatcher ties it all together. It's a bash script on cron that checks every agent folder for `[trigger]` tags and starts [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) with the right instructions when it finds one.

Create `~/vault/Projects/Agents/dispatcher.sh`:

```bash
#!/bin/bash

AGENTS_DIR="$HOME/vault/Projects/Agents"
CLAUDE_CMD="$HOME/.local/bin/claude"   # Path to your Claude Code binary
LOCK_FILE="/tmp/dispatcher.lock"

# Prevent overlapping runs
if [ -f "$LOCK_FILE" ]; then
  echo "[$(date '+%Y-%m-%d %H:%M:%S')] Already running. Skipping."
  exit 0
fi
touch "$LOCK_FILE"

echo "[$(date '+%Y-%m-%d %H:%M:%S')] Dispatcher started."

# Loop all agent folders
for AGENT_DIR in "$AGENTS_DIR"/*_agent; do
  [ -d "$AGENT_DIR" ] || continue

  AGENT_NAME=$(basename "$AGENT_DIR")
  TASK_FILE="$AGENT_DIR/task-list.md"
  CLAUDE_FILE="$AGENT_DIR/CLAUDE.md"
  AGENT_LOCK="/tmp/$AGENT_NAME.lock"

  [ -f "$TASK_FILE" ] || continue
  [ -f "$CLAUDE_FILE" ] || continue

  # Load agent env
  [ -f "$AGENT_DIR/.env" ] && source "$AGENT_DIR/.env"

  # Check for trigger tag
  if ! grep -q "^- \[ \] \[trigger\]" "$TASK_FILE"; then
    continue
  fi

  # Skip if already running
  if [ -f "$AGENT_LOCK" ]; then
    echo "[$AGENT_NAME] Already running. Skipping."
    continue
  fi

  touch "$AGENT_LOCK"

  # Extract triggered tasks
  TRIGGERED_TASKS=$(grep "^- \[ \] \[trigger\]" "$TASK_FILE")

  # Mark as in-progress
  sed -i "s/^- \[ \] \[trigger\]/- [ ] [in-progress] — started: $(date '+%Y-%m-%d %H:%M')/g" "$TASK_FILE"

  # Run Claude
  unset CLAUDECODE
  PROMPT="You are an agent. Your instructions are in $CLAUDE_FILE.
Read that file first. Your task file is at $TASK_FILE.
Triggered tasks: $TRIGGERED_TASKS
Execute per your CLAUDE.md instructions.
When done, update $TASK_FILE — mark completed tasks [done] with timestamp."

  echo "$PROMPT" | $CLAUDE_CMD --print --permission-mode bypassPermissions 2>&1
  
  rm -f "$AGENT_LOCK"
  echo "[$AGENT_NAME] Done."
done

# Scheduled trigger injection
DAY_OF_WEEK=$(date '+%u')
CURRENT_HOUR=$(date '+%H')

for AGENT_DIR in "$AGENTS_DIR"/*_agent; do
  [ -d "$AGENT_DIR" ] || continue
  TASK_FILE="$AGENT_DIR/task-list.md"
  CLAUDE_FILE="$AGENT_DIR/CLAUDE.md"
  [ -f "$TASK_FILE" ] || continue

  while IFS= read -r line; do
    SCHED_DAY=$(echo "$line" | grep -oP 'day=\K[0-9]+')
    SCHED_HOUR=$(echo "$line" | grep -oP 'hour=\K[0-9]+')
    SCHED_TASK=$(echo "$line" | grep -oP 'task=\K[^-]+' | xargs)

    if [ "$DAY_OF_WEEK" == "$SCHED_DAY" ] && [ "$CURRENT_HOUR" == "$SCHED_HOUR" ]; then
      if ! grep -q "^\- \[ \] \[trigger\] $SCHED_TASK" "$TASK_FILE"; then
        sed -i "/^## Backlog/a - [ ] [trigger] $SCHED_TASK" "$TASK_FILE"
        echo "[$(basename $AGENT_DIR)] Injected scheduled trigger: $SCHED_TASK"
      fi
    fi
  done < <(grep "<!-- schedule:" "$CLAUDE_FILE")
done

rm -f "$LOCK_FILE"
echo "[$(date '+%Y-%m-%d %H:%M:%S')] Dispatcher finished."
```

**The lock file at the top matters.** Without it, a slow Claude run and the next cron tick can overlap, and you end up with two agents fighting over the same report file. The agent-level lock (`$AGENT_LOCK`) does the same job for each agent.

Make it executable:
```bash
chmod +x ~/vault/Projects/Agents/dispatcher.sh
```

---

## Step 7: Set Up the Cron Job

```bash
crontab -e
```

Add:
```
*/5 * * * * bash "/home/youruser/vault/Projects/Agents/dispatcher.sh" >> "/home/youruser/vault/Projects/Agents/dispatcher.log" 2>&1
```

That runs the dispatcher every 5 minutes. It sits idle 99% of the time and only does real work when it finds a trigger or a scheduled time comes up. When something isn't working, the log file is the first place to look.

---

## Step 8: Test a Manual Run

Add a trigger to `task-list.md`:
```markdown
## Backlog
- [ ] [trigger] W2 weekly run
```

Wait up to 5 minutes for cron to fire, or skip the wait and run the dispatcher directly:
```bash
bash ~/vault/Projects/Agents/dispatcher.sh
```

Look for the report in `reports/w2/YYYY-MM-DD.md` and check the dispatcher log for errors. If the report is there and has jobs in it, you're in business.

---

## Triggering From Your Phone

This is the part that felt like magic the first time. If your vault syncs with [Syncthing](https://syncthing.net) (or iCloud, Dropbox, whatever):

1. Open `task-list.md` in Obsidian on your phone
2. Change `- [ ] W2 weekly run` to `- [ ] [trigger] W2 weekly run`
3. Save. Syncthing pushes the edit to the VPS.
4. Dispatcher picks it up within 5 minutes
5. Report appears in your vault within ~10 minutes

You just started an AI job search by editing a text file on your phone. That's the homelab dream right there: all the complexity lives in plumbing you built once, and the button is a checkbox.

---

## Checking Reports

Reports land in `reports/w2/YYYY-MM-DD.md` (or `freelance/`, `pm/`). Open them in Obsidian, where the tables render cleanly and every posting link is clickable. A second run on the same day gets a `-2` suffix, so you never lose the earlier one.

---

## Troubleshooting

**The dispatcher runs but no report shows up:** check the dispatcher log first, since Claude's output goes there. The usual suspects are a wrong path to the Claude binary in `CLAUDE_CMD`, a missing `.env` file, or an API key that never got loaded.

**Adzuna returns zero results:** don't use `where=remote`. It doesn't work reliably. Put "remote" in the `what=` search string instead.

**JSearch rate limit errors:** the free tier stops at 200 calls a month. At 3-5 runs a week, that works out to about 3 calls per run. Keep your queries few and sharp. One good query beats five broad ones.

**Same jobs keep showing up:** check `knowledge/seen-postings.md`. If it's empty or not changing, the dedup step isn't running. Make sure the dedup rules are in CLAUDE.md and the agent can write to the knowledge folder.

---

## What I'd build next

Smarter filtering. Right now the criteria are pretty blunt (keywords in, keywords out). A scoring system that ranks each job against a detailed profile (skills, experience level, the kinds of companies I like) would cut my review time way down.

Rather skip the build? I packaged this agent, plus one that tailors your resume to each posting, as the [Job Search Agent Kit](/blog/job-search-agent-kit).

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
