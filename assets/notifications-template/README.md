# Email Knowledge Base

This folder is the source of truth for the automated email-processing system.

## Layout
- `RULES.md` — classification rules. **Edit this** to tune behavior. The processor reads it every run.
- `INDEX.md` — master index, one row per email.
- `STATE.json` — last-run timestamps and dedup IDs. Don't edit by hand.
- `pending/` — emails awaiting your attention.
- `processed/` — emails you've handled.
- `archive/YYYY/MM/` — every email filed by month.
- `digests/YYYY-MM-DD-6pm.md` — daily 6 PM summaries.
- `lib/` — optional Python helpers.
- `logs/` — run logs.

## Scheduled tasks
1. **15-min sweep** — fetches new mail, classifies, files, posts urgent items to your notification channel.
2. **Hourly deep pass** — preprocesses `needs_action` items with web research; drafts replies.
3. **6 PM digest** — summarizes the last 24h to your notification channel.

## Quiet hours
Configured in `RULES.md`. Only urgent items with hard deadlines within the cutoff break through.
