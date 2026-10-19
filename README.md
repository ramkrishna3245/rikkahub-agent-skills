# RikkaHub Agent Skills

Custom [RikkaHub Agent](https://github.com/ExTV/rikkahub-agent) skills for driving a real
Android phone — Chrome automation, job hunting, form filling, research, data collection,
price comparison, social posting, and scheduled digests.

## Install

In RikkaHub Agent: **Settings → Extensions → Agent Skills → Import from GitHub**, then paste
the URL for the skill you want:

```
https://github.com/ramkrishna3245/rikkahub-agent-skills/tree/main/skills/<name>
```

| Skill | What it does |
|---|---|
| [`browser-operator`](skills/browser-operator) | Reliable playbook for driving the phone's real Chrome — search, navigate, read, click, type. |
| [`job-hunter`](skills/job-hunter) | Search Naukri / Indeed / LinkedIn, filter to a saved profile, shortlist, apply, and track. |
| [`form-filler`](skills/form-filler) | Fill any on-screen form from a stored profile, then stop for review before submitting. |
| [`web-research`](skills/web-research) | Research a topic across sources and save a cited report to the workspace. |
| [`data-harvest`](skills/data-harvest) | Pull lists/tables/results off a page and save them as CSV or JSON. |
| [`price-compare`](skills/price-compare) | Compare a product across Amazon / Flipkart / Croma / Meesho with deal-quality checks. |
| [`social-poster`](skills/social-poster) | Draft and publish to X / LinkedIn — always draft-and-confirm first. |
| [`daily-digest`](skills/daily-digest) | Schedule a recurring personal briefing (notifications, email, calendar, jobs, news). |

## Format

Standard Claude-style skills: each folder holds a `SKILL.md` with YAML frontmatter
(`name`, `description`) followed by the instruction body. They are prompt-only — no
scripts or extra dependencies — and they compose RikkaHub's built-in tools
(screen automation, app launcher, keyboard control, Termux, notifications, cron).
