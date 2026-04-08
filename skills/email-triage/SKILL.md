---
name: email-triage
description: Automated multi-account email triage with a folder-based knowledge store. Use when the user says "check my email", "triage my inbox", "process new mail", "tune my email rules", "show me urgent emails", "what's in my email digest", or asks anything about their email knowledge folder, RULES.md, INDEX.md, pending/, processed/, archive/, or the email sweep/deep/digest scheduled tasks.
---

# Email Triage

This skill operates a personal email-processing system built on top of a Gmail MCP connector, a notifications folder that holds all state, and three scheduled tasks. Read this file before touching anything in the user's email knowledge folder.

## The folder contract

The user has a folder (the **email knowledge folder**) with this exact layout:

```
RULES.md       — editable classification rules; read on every run
INDEX.md       — append-only master index, one row per email
STATE.json     — last-run timestamps + seen message IDs (do not edit by hand)
pending/       — one .md file per email awaiting user attention
processed/     — emails the user has handled
archive/YYYY/MM/ — every email filed by month
digests/YYYY-MM-DD-6pm.md — daily 6 PM digest output
lib/           — optional Python helpers
logs/          — per-run logs
```

Treat this folder as the source of truth. Never delete files from `archive/`, `processed/`, or `digests/`. Append to `INDEX.md`, do not rewrite it.

## The three scheduled tasks

1. **email-sweep-15min** (`*/15 * * * *`) — fetch new mail across all accounts in `RULES.md`, classify each one, write a `.md` file into `pending/` (or directly into `archive/YYYY/MM/`), append a row to `INDEX.md`, update `STATE.json`, and post `urgent_action` items to the configured notification channel. Respect quiet hours.
2. **email-deep-hourly** (`7 * * * *`) — for everything in `pending/`: do web research relevant to the message, draft a reply if `needs_reply`, update the file in place. Do not move files out of `pending/`.
3. **email-digest-6pm** (`0 18 * * *`) — build a 24h summary, reconcile done vs. not-done by checking Gmail state for each `pending/` item, write to `digests/YYYY-MM-DD-6pm.md`, post the summary to the notification channel.

## Classification flow

1. Read `RULES.md` fresh on every run. Never cache it across runs.
2. Determine the original recipient by checking `Delivered-To`, `To`, `Cc`, `X-Forwarded-To` in that order.
3. Apply blocklist first (auto-file), then VIP allowlist (escalate), then signal-based rules in `RULES.md`.
4. Choose exactly one category. When in doubt between two action categories, pick the more urgent one.
5. Honor quiet hours: during the configured window, only `urgent_action` with a hard deadline ≤24h may notify.

## Email file format

Each `.md` file in `pending/` or `archive/`:

```markdown
---
gmail_id: <id>
account: <which account it was sent to>
from: <sender>
subject: <subject>
date: <ISO8601>
category: <category>
status: pending | processed
---

<body extract or summary>

## Research / draft reply (added by deep pass)
...
```

## Editing rules

When the user asks you to "tune my email rules" or similar, edit `RULES.md` only. Never edit `STATE.json`, `INDEX.md` history, or any file under `archive/`. Confirm by reading the file back.

## Notification channel

`RULES.md` declares one of: Lark webhook, Slack webhook, or "Gmail draft to self". If multiple are set, prefer in that order: Lark, Slack, Gmail draft.

## Common user requests

- "What's urgent right now?" → list `pending/` items with category `urgent_action`.
- "Show me today's digest" → read the most recent file in `digests/`.
- "Run a sweep now" → invoke the sweep workflow manually (do not wait for the cron).
- "Add this sender to the blocklist" → edit `RULES.md` blocklist section, confirm.
- "Why did this get flagged urgent?" → open the `pending/` file, check its category and the rule that matched.

## Setup

The first-time setup flow lives in the sibling skill `email-triage-setup`.
