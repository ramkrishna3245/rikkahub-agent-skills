---
name: daily-digest
description: Build and schedule a recurring personal briefing (notifications, emails, calendar, weather, jobs, news, prices) delivered as a notification or message. Load when the user wants a morning summary, a daily digest, scheduled updates, or any recurring automated report.
---

# Daily Digest

One scheduled job that turns many noisy sources into a short briefing.

## Design the digest

`ask_user` for (once):

- **What time** and which days? (default: `08:30`, Mon–Sat)
- **Which sections?** pick from:
  - notifications since last night (`list_recent_notifications`)
  - unread email — Gmail app, via screen automation
  - today's calendar events
  - weather for the user's city
  - job openings matching `job-profile.md` (see the `job-hunter` skill)
  - news on chosen topics
  - price drops on a watchlist
  - pending tasks from a workspace `todo.md`
- **Delivery:** notification, Telegram, or both.
- **Length cap:** how many lines max (default 15).

Record the choices so the job is reproducible:

```
write_text_file path="<workspace>/digest-config.md"
```

## Scheduling

```
schedule_job(
  name="daily-digest",
  schedule_type="cron",
  cron="30 8 * * MON-SAT",
  mode="llm",
  prompt="Load the daily-digest skill and build today's briefing using digest-config.md.
          Keep it under 15 lines. Deliver via notification. Do not send anything else."
)
```

Useful cron patterns:

| Want | cron |
|---|---|
| Every weekday 8:30 | `30 8 * * MON-FRI` |
| Every day 7:00 | `0 7 * * *` |
| Hourly | `0 * * * *` |
| Every Monday 9:00 | `0 9 * * MON` |

Verify with `list_jobs`, and offer `pause_job` / `delete_job` later.

## Briefing format

Keep it scannable on a lock screen:

```
☀️ Tue, 20 Oct — 14 unread, 3 meetings

📅 10:00 Standup · 15:30 Dentist
✉️ 14 unread — 2 from boss, 1 invoice
👔 5 new jobs: 2 Android (Bengaluru), 1 remote
🌦️ 31°C, rain after 4pm — carry an umbrella
🔔 Missed: 2 bank SMS, 1 delivery OTP
```

Rules:
- Lead with anything **time-critical** (meetings within 3h, expiring OTPs, deadlines).
- Group by section; one line per item; no filler.
- Cap at the configured number of lines — overflow goes to a workspace file, not the
  notification.
- If a source fails, say "unavailable", do not silently drop the section.

## Writing it

- Read `digest-config.md` first every run; the config is the source of truth.
- Only include what was actually read. Never invent an email, meeting, or headline.
- Mark anything uncertain (e.g. "3 unread emails (Gmail not synced?)").
- Save the full version:

```
write_text_file path="<workspace>/digests/<YYYY-MM-DD>.md"
```

## Guardrails

- Never mark anything as read, archive, or delete on the user's behalf.
- Never reply to an email or message from the digest job.
- Never include OTP codes, passwords, or full account numbers in a notification body —
  mention that a code arrived, not the code itself.
- Do not send more than the configured number of notifications per run.
- If the device is locked and the job needs the UI, note the failure rather than looping.
