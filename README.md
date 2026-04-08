# email-manager-public

A shareable Cowork plugin that turns Claude into an automated email triage assistant for one or more Gmail accounts. Classifies new mail every 15 minutes, deep-processes action items hourly, and posts a daily digest — all backed by a plain-text knowledge folder you can read and edit.

## What you get

- **Two skills**
  - `email-triage` — the running playbook. Knows the folder layout, the three scheduled tasks, the classification flow, and how to answer questions like "what's urgent right now?"
  - `email-triage-setup` — first-time setup. Copies the template, collects your config, patches `RULES.md`, and creates the three scheduled tasks.
- **A sanitized template** under `assets/notifications-template/` containing `RULES.md` (categories, signals, blocklist defaults, placeholders for accounts/quiet hours/notification channel), an empty `INDEX.md`, an empty `STATE.json`, and the directory skeleton.

## Prerequisites

- Cowork (or Claude Code) with plugin support
- Gmail MCP connector connected for the account(s) you want to monitor
- Scheduled-tasks MCP enabled (built into Cowork)
- A notification channel: Lark webhook, Slack webhook, or "Gmail draft to self"

## Install

1. Download `email-manager-public.plugin` from the latest release (or zip the repo contents yourself).
2. Drag it into Cowork and accept the install prompt.
3. In a new chat, say "set up email triage". The setup skill will pick a folder, ask for your accounts and notification channel, patch `RULES.md`, and create the three scheduled tasks.
4. Done. From then on, just talk to Claude — "what's urgent?", "show me today's digest", "add @newsletter.example.com to my blocklist".

## Tuning

Everything that controls behavior lives in your folder's `RULES.md`. Edit it directly or ask Claude to. The processor re-reads it on every run, so changes take effect within 15 minutes.

## What this plugin does NOT do

- Send money, place orders, or take actions on third-party sites.
- Auto-reply to anyone. Drafts go into Gmail drafts; you send them.
- Monitor non-Gmail accounts. Forward them into Gmail first.

## License

MIT
