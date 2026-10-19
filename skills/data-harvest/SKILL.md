---
name: data-harvest
description: Pull structured data (lists, tables, search results, prices, contacts, posts) out of a page or app and save it as CSV/JSON in the workspace. Load when the user asks to collect, scrape, export, list, or tabulate data from a screen.
---

# Data Harvest

Screen → clean structured rows → saved file. The deliverable is always a **file**, not a
chat dump.

## Workflow

1. **Define the schema first.** Decide the columns before you read anything:
   `title | company | salary | location | link`. Getting here late means re-scraping.
2. **Reach the list.** Use `browser-operator` (or the native app) to land on the page
   that contains the rows.
3. **Collect in pages.** For each screen:
   - shallow `read_window_tree`
   - pull the repeating item nodes
   - record one row per item
   - scroll down, repeat
4. **Stop condition.** Stop when: the target count is reached, the list ends, or three
   consecutive scrolls add no new items. Say which one it was.
5. **Deduplicate** on the primary key (usually the link, else title+source).
6. **Save**, then report a count and a preview.

## Saving

CSV is the default — it opens anywhere:

```
write_text_file path="<workspace>/data/<YYYY-MM-DD>-<slug>.csv"
```

```csv
title,company,salary,location,link
"Senior Android Dev","Acme","18-24 LPA","Bengaluru","https://..."
```

Rules for the file:
- Header row always.
- Quote any field containing a comma, quote, or newline.
- One row per item. No multi-line rows.
- Empty field beats a guessed field.

Use JSON instead only when the data is nested (e.g. a comment tree):

```
write_text_file path="<workspace>/data/<YYYY-MM-DD>-<slug>.json"
```

## Reading tips

- `find_node` / `read_window_tree` with the shallowest depth that exposes the item text.
- Repeating items usually share a `view_id_resource_name` or the same parent container —
  find the pattern once, then walk siblings.
- For long text-bearing pages, prefer scrolling in chunks and extracting per chunk over
  dumping the full tree.
- If the data is in a `<table>`, grabbing it as text per row and splitting on the column
  separator is often faster than walking every cell node.
- Infinite scroll: keep swiping until no new items appear.

## Guardrails

- Respect the site: no hammering. Add a small pause between scroll/collect cycles; stop
  if a rate-limit, CAPTCHA, or "unusual activity" page appears.
- Public data only. Do **not** harvest personal data, logged-in private data, or
  anything behind a paywall the user has not legitimately got access to.
- Do not fabricate rows to reach a round number. Report the real count.
- Never write into a file that already holds the user's data without reading it first —
  append, or pick a new filename, rather than overwriting.

## Report

State: item count, file path, the columns collected, the date range covered, and any
rows that were skipped and why.
