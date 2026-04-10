# Email Classification Rules

_Editable. The scheduled task reads this file each run. Tune freely._

## Accounts
List the email accounts this system monitors. If you forward several into one inbox, list all of them — the processor tags each email with its **original recipient** by inspecting `Delivered-To`, `To`, `Cc`, and `X-Forwarded-To` headers.

- `<account-1@example.com>` — <label, e.g. work>
- `<account-2@example.com>` — <label, e.g. personal>
- `<account-3@example.com>` — <label>

## Active senders (always notify — highest priority)

Evaluated **before** category classification. If the sender matches, the email is tagged `urgent_action` and pushed to the notification channel immediately, bypassing quiet hours and any downgrade rules below. Use this for people/teams you always want to hear from. One per line, lowercase email or `@domain`.

```
# @partner-company.com
# important.person@example.com
```

## Categories

| Category | Notify? | Behavior |
|---|---|---|
| `urgent_action` | Notify immediately (even quiet hours if hard deadline ≤24h) | File in `pending/`, run web research |
| `needs_action` | Hourly batch only | File in `pending/`, run web research |
| `needs_reply` | Hourly batch only | File in `pending/`, draft reply |
| `fyi` | No | File in `archive/` |
| `newsletter` | No | File in `archive/`, surface in weekly digest only |
| `transactional` | No (unless amount > $500) | File in `archive/` |
| `promotional` | No | File in `archive/` |
| `spam_or_noise` | No | File in `archive/` |

## Signals → category

**urgent_action** triggers (any one):
- Subject contains: `urgent`, `asap`, `today`, `deadline`, `expires`, `final notice`, `action required`, `overdue`
- From a VIP (see allowlist) AND contains a question or deadline
- Bank/financial alert about a transaction, fraud, or balance issue
- Calendar invite for a meeting < 24h away
- ~~Verification code that's still valid (2FA, password reset)~~ — see downgrade rule below

**Verification codes → `transactional` (no notification).** Most users request codes manually and check the inbox immediately, so notifications are noise. Detect by: subject/body contains `verification code`, `verify your`, `one-time`, `OTP`, `security code`, `confirm your email`, or an N-digit code pattern; or sender matches `noreply@`/`no-reply@`/`verify@`/`security@`. File in `archive/`. Exception: if the sender is on the active-senders list, the active-sender rule wins and it still notifies. Remove this rule if you want code notifications.

**needs_action**: Anything that requires you to do something but isn't time-critical — vendor asking for info, form to fill, document to sign, scheduling back-and-forth, tax/payroll items.

**needs_reply**: A real human asking you a question with no other action required.

**fyi**: Updates from systems you care about (GitHub notifications, deployment alerts, status pages), team updates, shared docs you were tagged in.

**newsletter**: Recurring sends from a list.

**transactional**: Receipts, order confirmations, shipping notices, 2FA codes (after expiry), forwarding confirmations, security alerts for known logins.

**promotional**: Pure marketing with discount/sale language and no order tied to you.

**spam_or_noise**: Obvious spam, cold outreach with no relationship, language you don't read.

## VIP allowlist
Always at least `needs_action`; escalate to `urgent_action` on deadline language. One per line, lowercase email or domain.

```
# @yourcompany.com
# important.person@example.com
```

## Auto-file blocklist
Always classified as `newsletter` or `promotional`.

```
# @mail.salesforce.com
# @newsletter.example.com
```

## Quiet hours
- `<22:00>` – `<07:00>` local. Only `urgent_action` with hard deadline within 24h breaks through.

## Notification channel
Pick one and fill in the value. The processor uses whichever is set.

- **Lark webhook**: `<LARK_WEBHOOK_URL>`
- **Slack webhook**: `<SLACK_WEBHOOK_URL>`
- **Gmail draft to self**: `true` (drafts a self-addressed summary instead of posting to chat)

Daily digest at 18:00 local also goes through the same channel.
