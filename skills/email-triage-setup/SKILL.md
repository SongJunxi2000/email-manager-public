---
name: email-triage-setup
description: First-time setup for the email-triage plugin. Use when the user says "set up email triage", "install email manager", "configure my email automation", or has just installed the email-manager-public plugin and there is no email knowledge folder yet.
---

# Email Triage — Setup

Walk the user through installing the email-triage system. Do this once per user. After setup, the `email-triage` skill takes over.

## Preconditions to verify (in order)

1. **Gmail MCP connector connected.** If not, call `suggest_connectors` for Gmail and stop.
2. **scheduled-tasks MCP available.** If `mcp__scheduled-tasks__create_scheduled_task` is not in the tool list, tell the user this plugin requires Cowork's scheduled-tasks capability and stop.
3. **A folder for the email knowledge base.** If the user has not selected one, call `request_cowork_directory`.

## Setup steps

1. **Copy the template** from this plugin's `assets/notifications-template/` directory into the user's selected folder. Never overwrite existing files without asking.

2. **Collect configuration** with AskUserQuestion:
   - Which email accounts should be monitored?
   - Which is the "primary" inbox that the others forward into, if any?
   - Local timezone (for quiet hours)?
   - Quiet hours window (default: 22:00–07:00)?
   - VIP senders or domains (optional)?
   - **Active senders** — people/domains that should ALWAYS notify immediately, even during quiet hours (optional)?
   - Suppress verification-code notifications? (default: yes — codes are filed to `transactional` since users typically request them and check the inbox right away)
   - Notification channel: Lark webhook URL, Slack webhook URL, or "Gmail draft to self"?

3. **Patch `RULES.md`.** Replace the `<account-N@example.com>` placeholders, the `<22:00>`/`<07:00>` placeholders, the VIP allowlist section, the active-senders section, and the notification-channel section. Leave the categories, signal rules, and blocklist defaults alone unless the user asks.

4. **Create the three scheduled tasks** via `mcp__scheduled-tasks__create_scheduled_task`. Each task description must reference the user's folder path:
   - `email-sweep-15min` — cron `*/15 * * * *`
   - `email-deep-hourly` — cron `7 * * * *`
   - `email-digest-6pm` — cron `0 18 * * *`

5. **Smoke test.** Run the sweep workflow once manually. Confirm `INDEX.md` got a row, `STATE.json` updated, no errors in `logs/`.

6. **Hand off** to the `email-triage` skill.

## Failure modes

- Folder is read-only → ask user to pick a different folder.
- Gmail MCP returns auth error → re-prompt user to reconnect, do not retry silently.
- Task ID conflict → ask whether to overwrite or pick a different ID suffix.
