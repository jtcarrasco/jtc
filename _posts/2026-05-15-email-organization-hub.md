---
layout: post
title: "Taming Four Email Accounts With One Gmail Hub, a Python Script, and n8n"
description: "How I consolidated multiple email accounts into a single labeled inbox, built a Python script for bulk operations, and added a daily briefing via n8n."
date: 2026-05-15 09:00:00 -0800
published: true
image: '/assets/blog/email-organization.webp'
tags: [email, automation, n8n, gmail, productivity]
---

Good automation starts with operational hygiene: knowing what's happening across your systems without manually checking each one. Email is the messiest example of this. Multiple accounts, multiple purposes, no coherent view. Before I could build AI workflows that acted on email (job application follow-ups, daily briefings, alert routing), I needed the underlying system to be clean. This post covers how I got there.

I had four email accounts and a problem: context-switching between them constantly, missing things, and a general sense that email was running me instead of the other way around. I didn't want to pay for a fancy email client. I wanted everything in one place with zero manual sorting.

Here's how I built it.

## The Accounts

The starting point: four addresses, each with a different purpose.

- **hub@gmail.com**: the hub. Everything funnels here.
- **hello@yourdomain.com**: client-facing. Anything from clients lands here.
- **info@yourdomain.com**: infrastructure. Server alerts, billing, hosting.
- **junk@gmail.com**: designated junk. Stores, newsletters, anything I don't need to read immediately.

The goal: read only one inbox. Route everything else automatically.

## The Architecture

Each non-hub account forwards to the hub. Gmail's forwarding is reliable and free. Once everything lands at hub@gmail.com, Gmail filters label each email with its source: `source/hello`, `source/info`, `source/junk`.

Now instead of checking four inboxes, I check one, and I always know where an email came from by its label.

## The Gmail Helper

To do bulk operations, I built a small Python script using the [Gmail API](https://developers.google.com/gmail/api). The Gmail web UI is fine for one-off things, but if you want to trash 40 promotional emails in a single command, you need the API.

The script handles:
- **Search**: find messages by any Gmail query
- **Trash**: move individual messages to trash
- **Bulk trash**: trash everything matching a query (e.g. `label:source/junk`)
- **Mark read**: batch-clear the unread indicator
- **Filter management**: create and list Gmail filters programmatically

The bulk trash command is the workhorse. A "junk sweep" is now: confirm the list, say go, done.

```bash
python3 gmail_helper.py bulk-trash "label:source/junk"
```

The OAuth setup was a one-time pain (Gmail's API auth always is), but it's been headless and stable since. The refresh token handles re-auth automatically.

## The Daily Briefing (n8n)

The most useful piece: a morning briefing that runs at 8am, 1pm, and 6pm and posts to Discord.

It pulls three things:
1. **Unread emails from the last 9 hours**: subject and sender, nothing more
2. **Today's calendar events**: deduplicated (multi-day events were showing up once per overlapping day, which was a fun bug to track down)
3. **[Todoist](https://todoist.com) tasks due today or overdue**: pulled from the Todoist API v1

The result drops into a Discord channel as a formatted message. I glance at it when I wake up and when I sit down to work. No inbox required.

One gotcha that burned me: Gmail's API returns field names capitalized (`From`, `Subject`) when Simplify mode is on. Cost me an hour debugging why every email showed "Unknown: No subject."

Another: Todoist's API v1 wraps responses in `{ results: [], next_cursor }` rather than returning a bare array. Their documentation doesn't make this obvious.

## The Job Alert Workflow

A second [n8n](https://n8n.io) workflow handles job applications. When I label an email "Job Alerts/Applied," n8n automatically creates a follow-up task in my Todoist Applications project: `Follow up: [email subject]`.

Small workflow, genuinely useful. I don't forget to follow up anymore.

## Sending From Any Address in Thunderbird and Mac Mail

Forwarding handles receiving, but sending is separate. If you want to reply as `hello@yourdomain.com` from your desktop client without opening the Gmail web UI, you need to wire up the send-as identity.

The approach: add the address as a "Send mail as" alias in Gmail settings first, then configure each client to use it.

**Gmail (web, one-time setup):** Settings → Accounts and Import → Send mail as → Add another email address. Use Gmail's SMTP with your hub credentials. Gmail sends a verification email to the alias address. Confirm it.

**Thunderbird:** Account Settings → your hub account → Manage Identities → Add. Set the email to `hello@yourdomain.com`. When composing, click the From field to switch identities.

**Mac Mail:** Settings → Accounts → your hub account → Account Information → Email Address field. Add a comma and the alias address after your hub address. Mac Mail will offer it as a From option in the compose window.

Both clients use the hub account's SMTP. No separate app password or OAuth flow needed for the alias.

## What I'd Do Differently

The hardest part wasn't the code. It was deciding which account gets which email. I spent too long overthinking the taxonomy. The simple rule (purpose-based accounts, one hub) works better than anything clever.

I'd also set up the Gmail helper script earlier. The web UI is fine until you need to touch 50 emails at once.

## The Result

One inbox. Labeled by source. Automated briefings. Follow-up tasks created automatically. The junk account gets swept weekly.

Email went from something I checked anxiously to something I process deliberately. That's the whole point.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
