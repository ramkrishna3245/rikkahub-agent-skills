---
name: web-research
description: Research a topic across multiple sources on the web, cross-check the facts, and save a cited report to the workspace. Load when the user asks to research something, compare options, find out about a topic, or needs an answer backed by sources.
---

# Web Research

Turn a question into a **cited, saved report** — not a stream of half-read pages.

## Method

1. **Sharpen the question.** Restate it in one line. If it is ambiguous, `ask_user`
   one clarifying question — not five.
2. **Plan 3–5 search phrasings.** Different angles beat one long query:
   - the plain question
   - `"<topic>" comparison`
   - `"<topic>" site:reddit.com` (real-user experience)
   - `"<topic>" 2026` (recency)
   - `"<topic>" official documentation`
3. **Search + read** using the `browser-operator` playbook:
   `open_url(google search, package_name="com.android.chrome")` → shallow
   `read_window_tree` → open the 2–3 most promising results in new tabs.
4. **Extract, don't skim.** For each source, capture: claim, numbers, date, who says it.
5. **Cross-check.** Anything stated by only one source gets marked "single-source".
6. **Write the report** to the workspace.

## Source quality ladder

Prefer, in order:

1. Primary/official (vendor docs, government, standards bodies, the project's own site)
2. Reputable secondary (established press, peer review, well-known analysts)
3. Community (Reddit, forums, Stack Overflow) — great for *real-world friction*, weak
   for facts; label it as such
4. SEO content farms / content-mill blog posts — **do not cite**

If the question is about health, law, money, or safety, say plainly that it is not
professional advice and point to the authoritative source.

## Report format

Save to the workspace, then summarise in chat:

```
write_text_file path="<workspace>/research/<YYYY-MM-DD>-<slug>.md"
```

```markdown
# <Title>

**Question:** <the sharpened question>
**Date:** <YYYY-MM-DD>

## Answer
<2–5 sentences. Lead with the conclusion.>

## Key findings
- **<Finding>** — <one line> ([source](url), <date>)
- ...

## Numbers / comparison
| Option | Metric A | Metric B | Source |
|---|---|---|---|

## Disagreements & uncertainty
- <where sources conflict, and which is more credible and why>

## Sources
1. <title> — <url> (accessed <date>)
```

## Rules

- **Never invent a URL, number, or quote.** If you did not read it, it is not in the report.
- Every non-obvious claim gets a source link.
- Distinguish **fact** from **opinion** from **your inference**.
- Note the publication date of anything time-sensitive; a 2019 answer to a 2026
  question is a trap.
- If sources conflict and you cannot resolve it, say so in "Disagreements" instead of
  picking one silently.
- Keep the chat reply short; the file holds the detail.
