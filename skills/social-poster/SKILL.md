---
name: social-poster
description: Draft and publish posts, replies, or comments to X (Twitter), LinkedIn, or other social sites via the browser, always with a draft-and-confirm step. Load when the user asks to post, tweet, share, comment, reply, or update on social media.
---

# Social Poster

Publishing on someone's public identity is **irreversible**. This skill drafts first and
publishes only with explicit approval.

## Hard rule

**Never post without showing the exact final text and getting a clear yes in this
conversation.** Even if the user said "post it" earlier for a *different* post.

Exception: an approved automated workflow (e.g. auto-reply skill) where the user has
explicitly pre-authorised that channel and content class.

## Workflow

1. **Collect the brief:** platform, account, topic/point, tone, length, hashtags/mentions,
   any links, and the desired time.
2. **Draft 2–3 variants** and pick a recommendation. Show them in chat.
3. **Wait for approval.** Apply edits exactly.
4. **Open the compose surface:**
   - X:        `https://x.com/compose/post` (or `https://twitter.com/compose/tweet`)
   - LinkedIn: `https://www.linkedin.com/feed/` → "Start a post"
   - Threads / Reddit: the respective compose URL
   Force Chrome with `package_name="com.android.chrome"`.
5. **Type the text** with `set_text` on the composer, or the keyboard-control tools.
6. **Screenshot** the composer and show it to the user.
7. **Click Post** only after the final confirmation.
8. **Verify** the post appears (timeline/profile), and report the live URL.

## Per-platform notes

| Platform | Limits & gotchas |
|---|---|
| X | 280 chars unless the account has Premium. Threads must be posted in order and each reply attached to the previous. Hashtags: 0–2 are plenty. |
| LinkedIn | ~3000 chars. Line breaks matter — the first 2 lines are what people see. No more than 3 hashtags. Links can reduce reach; sometimes a "link in comments" is better — ask. |
| Reddit | Subreddit rules first. Check the sidebar for self-promo bans before drafting. |
| Instagram/Facebook | Mobile web compose is flaky — prefer the native app via `launch_app`. |

## Content rules

- Write in the user's voice, not generic marketing filler. Match the tone they gave.
- No emoji spam; at most a few if the user's style uses them.
- Don't invent facts, statistics, or claims about the user.
- Don't @-mention or tag anyone the user did not ask for.
- Do not post links that look like spam/phishing.

## After posting

Report: platform, the exact text posted, time, and the live link. Offer to schedule
similar posts, and note if the user wants the original draft saved:

```
write_text_file path="<workspace>/social/<YYYY-MM-DD>-<platform>-<slug>.md"
```

## Guardrails

- Never post from an account that is not the user's.
- Never post on a user's behalf in an argument, or reply to a mention, without approval.
- Never delete, edit, or DM anyone unless asked.
- If the composer shows a login wall, stop and ask the user to log in — do not attempt
  to authenticate.
