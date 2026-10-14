---
layout: post
title: "Stop Managing Multiple Inboxes Like a Psychopath: A Gmail Hub Setup Guide"
description: "Step-by-step: forward every account into one Gmail hub, label mail automatically, manage it in bulk with a Python script, and automate it with n8n."
date: 2026-05-22 09:00:00 -0800
published: true
image: '/assets/blog/email-organization-hub-tutorial.webp'
tags: [automation, tutorial, email, n8n, gmail]
---

*This is the step-by-step version. For the why behind it, read [the overview](/blog/email-organization-hub).*

At one point I had five email accounts and zero system for any of them. One for clients. One for server alerts. One I'd been using for store signups since 2011 that was getting 40-plus promo emails a day (Bed Bath & Beyond alone was sending three). I'd pick up my phone in the morning, see a wall of 300 unread messages spread across a bunch of apps, and put it right back down without reading a thing. Inbox zero was a myth, like a well-rested parent.

The fix was simpler than I expected. One [Gmail](https://mail.google.com) account is the hub, everything else forwards into it, and every message gets labeled by where it came from the moment it lands. Add a Python script for bulk cleanup and an [n8n](https://n8n.io) daily briefing, and the whole thing pretty much runs itself.

**What you'll end up with:** All email in one inbox, each message labeled by source account, a Python script for bulk operations, and a morning briefing posted to [Discord](https://discord.com) with email + calendar + tasks.

**Prerequisites:** Multiple Gmail/Google Workspace accounts, Python 3, n8n running (see the VPS tutorial), a Discord server.

---

## Step 1: Set Up Email Forwarding

For each secondary account, set up forwarding to your hub account.

**For Gmail accounts:**
1. Go to Settings → See all settings → Forwarding and POP/IMAP
2. Add forwarding address → enter your hub Gmail address
3. Confirm the verification email sent to the hub
4. Set to "Forward a copy of incoming mail" (keep a copy in the secondary if you want it)

**For Google Workspace / custom domain accounts:**
- Same process via the Google Workspace admin console, or
- Set up a forwarding rule directly in Gmail Settings if you have admin access

Do this for every account you want to pull in. Yes, even the store-signup one. ESPECIALLY the store-signup one. You'll want to be able to nuke it from orbit later.

---

## Step 2: Create Source Labels in Gmail

In your hub Gmail, create a label for each forwarding account:

1. Go to Settings → Labels → Create new label
2. Create: `Source`, `Source/hello`, `Source/info`, `Source/junk` (or whatever names match your accounts)

The parent `Source` label lets you fold all the source labels up in the sidebar. The better reason is that you can search `label:Source/junk` from a script and trash everything in it with one command, which is deeply satisfying.

---

## Step 3: Create Gmail Filters for Auto-Labeling

For each account that forwards to your hub, create a filter that labels incoming messages by source:

1. Settings → Filters and Blocked Addresses → Create a new filter
2. **To:** enter the original address (e.g. `hello@yourdomain.com`)
3. Create filter → Apply label → select `Source/hello`
4. Check "Also apply filter to matching conversations" to label existing messages
5. Repeat for each account

Now every email lands in your hub already tagged with where it came from. "Did anything come in for the client address this week?" takes about two seconds to answer.

---

## Step 4: Set Up Gmail API Access

You need a Google Cloud project with the Gmail API turned on. This is the paperwork part. It's not hard, it's just a lot of clicking.

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project
3. Enable the **Gmail API**
4. Create OAuth 2.0 credentials (Desktop app type)
5. Download the credentials JSON and save it as `gmail-client-secret.json`

**Scopes needed:** `gmail.readonly` and `gmail.modify` (can trash and read, no permanent delete. Gmail keeps a 30-day undo window)

Those are the scopes I use: read access for the briefing, and modify for bulk trash and mark-read. I never need to permanently delete anything, since the 30-day trash window is a good enough safety net. If you want something different (auto-archive instead of trash, or sending through the API), adjust to fit. The [Gmail API scopes reference](https://developers.google.com/gmail/api/auth/scopes) lists all of them.

Run the auth flow once to get a token:

```bash
pip install google-auth-oauthlib google-auth-httplib2 google-api-python-client
python3 gmail-auth.py  # Script that opens OAuth flow and saves token
```

**Gotcha:** if your VPS has `AllowTcpForwarding no` (and it should), the browser redirect won't work the normal way. Do it the ugly-but-it-works way:

```bash
OAUTHLIB_INSECURE_TRANSPORT=1 python3 gmail-auth.py
```

Copy the URL it prints, open it in a browser and paste the code back. The token saves to `gmail-token.json` and refreshes itself from then on. You do this once and forget about it.

---

## Step 5: Build the Gmail Helper Script

This script does the heavy lifting. It does three things: search with any Gmail query, trash in bulk and mark as read in bulk. Create `gmail_helper.py`:

```python
#!/usr/bin/env python3
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build
import json, sys

TOKEN_FILE = '/path/to/gmail-token.json'
SECRET_FILE = '/path/to/gmail-client-secret.json'

def get_service():
    creds = Credentials.from_authorized_user_file(TOKEN_FILE)
    return build('gmail', 'v1', credentials=creds)

def search(svc, query, max_results=50):
    results = svc.users().messages().list(
        userId='me', q=query, maxResults=max_results
    ).execute()
    messages = results.get('messages', [])
    for msg in messages:
        m = svc.users().messages().get(
            userId='me', id=msg['id'], format='metadata',
            metadataHeaders=['Subject','From','Date']
        ).execute()
        headers = {h['name']: h['value'] for h in m['payload']['headers']}
        print(f"  [{msg['id']}]  {headers.get('Date','')}  From: {headers.get('From','')[:40]}  |  {headers.get('Subject','')[:60]}")

def bulk_trash(svc, query, dry_run=False):
    results = svc.users().messages().list(
        userId='me', q=query, maxResults=500
    ).execute()
    messages = results.get('messages', [])
    if not messages:
        print("No messages found.")
        return
    print(f"{'[DRY RUN] Would trash' if dry_run else 'Trashing'} {len(messages)} message(s)...")
    if dry_run:
        search(svc, query)
        return
    ids = [m['id'] for m in messages]
    svc.users().messages().batchModify(
        userId='me',
        body={'ids': ids, 'addLabelIds': ['TRASH'], 'removeLabelIds': ['INBOX']}
    ).execute()
    print(f"Done -- {len(ids)} messages moved to Trash.")

def bulk_mark_read(svc, query):
    results = svc.users().messages().list(
        userId='me', q=query + ' is:unread', maxResults=500
    ).execute()
    messages = results.get('messages', [])
    if not messages:
        print("No unread messages found.")
        return
    ids = [m['id'] for m in messages]
    svc.users().messages().batchModify(
        userId='me',
        body={'ids': ids, 'removeLabelIds': ['UNREAD']}
    ).execute()
    print(f"Done -- {len(ids)} messages marked as read.")
```

Always dry-run first. Trashing 400 emails without looking is a bad time, trust me:

```bash
# Dry run -- show what would be trashed
python3 gmail_helper.py bulk-trash "label:Source/junk" --dry-run

# Actually trash
python3 gmail_helper.py bulk-trash "label:Source/junk"

# Mark all read in a label
python3 gmail_helper.py mark-read "label:Source/junk"

# Search
python3 gmail_helper.py search "from:recruiter@company.com"
```

---

## Step 6: Build the Daily Briefing in n8n

This is the part I use every single day. It's a workflow that runs twice a day, grabs unread email, today's calendar and overdue tasks, rolls them into one message and posts it to Discord.

Create a new workflow in [n8n](https://docs.n8n.io) with these nodes:

### Schedule Trigger
- Cron expression: `0 8,20 * * *` (fires at 8am and 8pm in your n8n timezone)
- If your n8n container has `GENERIC_TIMEZONE: America/Los_Angeles` set, this runs at 8am and 8pm PDT. No UTC offset math required.

### Get Emails (Gmail node)
- Operation: Get Many
- Search query: `is:unread newer_than:9h`
- Limit: 10
- Simplify: ON

**Gotcha:** with Simplify ON, Gmail capitalizes the field names (`From`, `Subject`, `Date`). Use those exact names in the code nodes after it, or you'll get `undefined` everywhere and spend 20 minutes very confused.

### Get Tasks (HTTP Request node, optional)

This node is how I pull tasks in, because I use [Todoist](https://todoist.com). It's not a requirement. Point it at any task manager with an API, or skip it and keep the briefing to email and calendar. If you skip it, also delete the task lines from the Format Message code below, or n8n will complain about a node that doesn't exist.

- URL: `https://api.todoist.com/api/v1/tasks?limit=200`
- Auth: Bearer token (your Todoist API token)
- Response: JSON

**Note:** Todoist API v1 wraps responses in `{ results: [], next_cursor }`. It's not a bare array. Access tasks as `taskData.results`.

### Get Calendar (Google Calendar node)
- Operation: Get Many
- Calendar: primary
- Time range: today (use expressions for start/end of day in your timezone)

### Format Message (Code node)

```javascript
const pdtToday = new Date().toLocaleDateString('en-CA', {timeZone: 'America/Los_Angeles'});

// Deduplicate calendar events by id (multi-day events repeat per overlapping day)
const seen = {};
const calItems = $('Get Calendar').all().filter(e => {
  const key = e.json.id || e.json.summary;
  if (seen[key]) return false;
  seen[key] = true;
  return true;
});

// Emails use capitalized field names (Simplify ON)
const emailItems = $('Get Emails').all();

// Tasks from results array
const taskData = $('Get Tasks').first().json;
const tasks = (taskData.results || []).filter(t => {
  if (!t.due) return false;
  return t.due.date.substring(0, 10) <= pdtToday;
});

// Build your formatted message here
const message = `**Today's Briefing**\n\n` +
  `📅 **Calendar**\n${calItems.map(e => `• ${e.json.summary}`).join('\n') || 'Nothing scheduled'}\n\n` +
  `✉️ **Email**\n${emailItems.map(e => `• ${e.json.From}: ${e.json.Subject}`).join('\n') || 'No new email'}\n\n` +
  `✅ **Tasks**\n${tasks.map(t => `• ${t.content}`).join('\n') || 'All clear'}`;

return [{ json: { message } }];
```

### Post to Discord (HTTP Request node)
- URL: your Discord webhook URL
- Method: POST
- Body: `{ "content": "{% raw %}{{ $json.message }}{% endraw %}" }`

**Important: node error settings.** On any node that hits an external API (Get Tasks, Get Calendar, Get Emails), open its Settings tab and set:
- **Retry On Fail:** ON, Max Tries: 3
- **On Error:** Continue (so a Todoist 503 at 8am doesn't mean you also miss your calendar)

Test it, then activate the workflow.

---

## Step 7: Job Application Follow-up Workflow (Bonus)

If you're job hunting, this one is worth five minutes. Label an email "Job Alerts/Applied" in Gmail and n8n creates a follow-up task in Todoist for you. No more "wait, did I ever hear back from them?"

Same deal as the briefing: Todoist is just what I use. Swap the HTTP Request node for your own task manager's API, or have n8n add a row to a spreadsheet or a line to a note instead.

**Gmail Trigger node:**
- Event: Message Received
- Filter: Label = `Job Alerts/Applied`
- Poll every: 15 minutes

**HTTP Request node (Todoist API):**
- URL: `https://api.todoist.com/api/v1/tasks`
- Method: POST
- Auth: Bearer token
- Body parameters:
  - `content`: `Follow up: {% raw %}{{ $json.Subject }}{% endraw %}`
  - `project_id`: your Todoist project ID

Find your project ID via:
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://api.todoist.com/api/v1/projects
```

---

## Step 8: Configure Desktop Clients to Send From Any Address

Forwarding takes care of incoming mail. To send from your other addresses in any mail app, you first need a send-as alias in Gmail. Everything below depends on it.

### Part A: Add the alias in Gmail (web, required first)

1. Gmail → Settings → **Accounts and Import** → **Send mail as** → Add another email address
2. Name: your name / Email: `hello@yourdomain.com`
3. Uncheck "Treat as alias" (keeps it as a distinct From identity)
4. SMTP server: `smtp.gmail.com`, port 587, STARTTLS. Use your hub account credentials.
5. Gmail sends a verification email to hello@. Click the link to confirm.

Repeat for every address you want to send from. Then configure your client.

### Part B: Thunderbird

1. **Account Settings** → select your hub account → **Manage Identities**
2. Click **Add**:
   - Name: your name
   - Email: `hello@yourdomain.com`
3. Save. When composing, click the **From** dropdown to switch identities.

### Part C: Outlook (Windows and Mac)

Outlook uses the alias you configured in Gmail. No extra setup in Outlook itself:

1. Open a new message
2. If the **From** field isn't visible: Options → **From**
3. Click the **From** field → **Other Email Address…**
4. Type `hello@yourdomain.com` and click **OK**

Outlook remembers addresses you've used, so next time it appears in the From dropdown directly. Mac is identical: New Message → From field dropdown → **Other Email Address…**

### Part D: Apple Mail

1. **Mail → Settings** (Cmd+,) → **Accounts** → select your hub account
2. **Account Information** tab → **Email Address** field
3. Add a comma after your hub address, then type `hello@yourdomain.com`:
   ```
   hubaccount@gmail.com, hello@yourdomain.com
   ```
4. Close and save. The From dropdown appears when composing.

### Part E: Brevo as an SMTP Relay (Custom Domains)

If you send from a custom domain and want better deliverability than Gmail's own SMTP, take a look at [Brevo](https://www.brevo.com/). The free tier covers 300 emails a day, and the SMTP login works in any mail app.

1. Create a free account at brevo.com
2. Go to **SMTP & API** → **SMTP** and note your login and SMTP key
3. In your email client, use these settings instead of Gmail's SMTP:
   - SMTP server: `smtp-relay.brevo.com`
   - Port: 587, STARTTLS
   - Login: your Brevo account email
   - Password: your SMTP key (not your account password)

The send-as alias from Part A is still required. Brevo handles the outbound SMTP leg; Gmail still needs to know you're authorized to send as that address.

### Other clients

Most clients (Spark, Airmail, Mimestream) have an **Aliases** or **Identities** section in account settings. The Gmail send-as alias from Part A is always the prerequisite.

---

## Maintenance

**Weekly junk sweep.** The one thing still worth doing by hand, dry run first:

```bash
# Confirm the list before you commit
python3 gmail_helper.py bulk-trash "label:Source/junk" --dry-run
# Then pull the trigger
python3 gmail_helper.py bulk-trash "label:Source/junk"
```

**If the OAuth token expires:** the refresh token usually handles it. If it fully expires (rare, but it happens if you revoke access or don't use it for six months), just run the auth flow again.

---

## Troubleshooting

**n8n calendar events duplicating in the briefing:** Multi-day events appear once per overlapping day in the [Google Calendar API](https://developers.google.com/calendar/api/guides/overview) response. The dedup block in the Format Message node handles this. If you deleted it, put it back.

**Gmail filter isn't labeling forwarded messages:** Check that the filter is matching the `To:` field, not `From:`. Forwarded messages from `hello@yourdomain.com` arrive with that as the original destination. Filter on it, not on the hub address.

**Todoist tasks showing the wrong "today":** the code node uses `America/Los_Angeles` for the date, not the server's clock. If your VPS is in another timezone, that's expected. Change it to your own timezone if you're not on the West Coast.

---

*Part of a series on building a practical, low-cost homelab with AI agents, self-hosted automation, and a Tailscale backbone.*
