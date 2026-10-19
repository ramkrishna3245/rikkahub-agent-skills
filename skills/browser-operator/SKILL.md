---
name: browser-operator
description: Drive the phone's real browser (Chrome) for search, navigation, reading pages, extracting data, and interacting with web apps — reliably, without burning context and without disturbing the user's open tabs. Load this whenever the task involves a website, a web search, or anything in a browser.
---

# Browser Operator

The reliable playbook for driving **Chrome on this device**. Most failures in browser
tasks come from fighting the UI instead of using the right entry point.

## Golden rules

1. **Never close, clear, or reorganise the user's tabs.** Opening a new tab is fine.
   The tabs belong to the user. If a matching tab already exists, reuse it.
2. **Prefer `open_url` over typing into the URL bar.** `open_url("https://www.google.com/search?q=foo")`
   finishes a search in ONE tool call. Typing in the URL bar loops and fails.
3. **Force Chrome explicitly when the user says "my Chrome".** `open_url` without a
   package can hand the URL to RikkaHub's in-app browser. Pass
   `package_name="com.android.chrome"` to force the real Chrome app.
4. **Read small, not everything.** `read_window_tree` with `detail=true` is huge and
   destroys the context window. Read shallow, then use `find_node` to target.
5. **One intent per step.** Search → verify results appeared → then next step.
   Do not chain three blind taps.

## Tool cheat-sheet

| Intent | Tool |
|---|---|
| Open a URL / run a search | `open_url` (add `package_name` for Chrome) |
| Bring Chrome to front | `launch_app("com.android.chrome")` |
| See what is on screen | `read_window_tree` (shallow) then `find_node` |
| Tap a link / button | `click_node` with `text` or `content_description` |
| Type into a field | `set_text` (fields must be editable) |
| Type where `set_text` fails | Keyboard control: `keyboard_read_field` → `keyboard_clear` → `keyboard_type` |
| Scroll | `global_action("back")` is NOT scroll — use a swipe/scroll tool, or find the scrollable node |
| Go back a page | `global_action("back")` |
| Read a long page | Scroll + `read_window_tree`, or better: `eval_javascript` if content is fetchable |

## Search — the correct pattern

```
1. open_url("https://www.google.com/search?q=<url-encoded query>", package_name="com.android.chrome")
2. read_window_tree (shallow) → confirm results rendered
3. find_node / read specific result nodes
4. Report, or click_node on the chosen result
```

URL-encode the query: spaces → `+` or `%20`, `&` → `%26`.

## Reading a page without wrecking context

- First `read_window_tree` with the smallest `max_depth` that still shows text.
- Then `find_node(text="...")` to confirm a specific string exists.
- Only pull the full tree when you genuinely need every element.
- If the page is mostly text, prefer scrolling in 2–3 chunks and summarising each
  chunk rather than dumping the whole tree repeatedly.

## Typing into web forms

Order of preference:
1. `set_text` with a node selector (`text`, `content_description`, or `view_id_resource_name`).
2. Keyboard control tools — these are purpose-built for text entry and are more
   reliable than simulated key presses.
3. Raw `input`-style key events — last resort.

**Known quirk:** the on-device keyboard's Enter can insert an encoded newline instead
of submitting. If a search query ends up containing `%0A`, clear the field and retry —
prefer clicking the page's own Search/Go button over pressing Enter.

## When Chrome is not the right answer

- `tel:` `mailto:` `geo:` `sms:` — use `open_url` and let the system handler take it.
- App deep links — `open_url` first; `launch_app` only if you know the package.
- If a task needs login and the user is not logged in, STOP and `ask_user`. Do not
  attempt to authenticate, guess credentials, or enter a password field.

## Safety

- Never complete a purchase, send money, post publicly, or submit an application
  **without** confirming with the user first, unless the user explicitly pre-authorised
  that exact action in this conversation.
- Never read or autofill password fields.
- If a page shows a CAPTCHA, stop and tell the user — do not grind on it.

## Reporting

After a browser task, report: the URL you ended on, the key facts found, and anything
still uncertain. If you opened a tab, mention it so the user can close it.
