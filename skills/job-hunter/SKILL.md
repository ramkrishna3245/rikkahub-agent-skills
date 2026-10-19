---
name: job-hunter
description: Search job boards (Naukri, Indeed, LinkedIn, Wellfound), filter to what the user actually wants, extract the shortlist, and drive application forms on request. Load when the user asks to find jobs, apply to jobs, shortlist openings, or track applications.
---

# Job Hunter

End-to-end job search on this device: search → filter → shortlist → (optionally) apply → track.

## Before anything: load the profile

Read the user's job profile once per session. Look for it in the workspace:

- `job-profile.md` — roles, skills, years of experience, location, notice period,
  expected CTC, work-authorisation, preferred job types (remote/hybrid/onsite).
- `job-applications.csv` — the application tracker.
- A resume file (PDF/DOCX) if present.

If `job-profile.md` does not exist, run the **onboarding** step below first.

### Onboarding (once)

`ask_user` for: target role(s), skills, total experience, current city, willing-to-relocate,
shift preference, expected CTC, notice period, and job type. Then save it:

```
write_text_file(path="<workspace>/job-profile.md", content=<the collected answers>)
```

Never invent values. A wrong CTC on an application is worse than a missing one.

## Board cheat-sheets

### Naukri — strongest URL patterns

```
Search:    https://www.naukri.com/<role-slug>-jobs-in-<city>?k=<role>&l=<city>
With exp:  ...&experience=<min>
Remote:    ...&wfhType=2
Fresh (7d): ...&jobAge=7
```
Results page shows: title, company, experience band, salary band, location, and a
"Apply"/"Apply on company site" button. **Prefer the direct URL over driving the search
box** — it is one call instead of five.

### Indeed

```
https://in.indeed.com/jobs?q=<role>&l=<city>&fromage=7
```
Filters: `&sc=0kf%3Aattr(DSQF7)%3B` (remote) etc. Use the on-page filter chips if the
URL params get fiddly.

### LinkedIn

```
https://www.linkedin.com/jobs/search/?keywords=<role>&location=<city>&f_TPR=r604800
```
Public job pages are readable without login, but "Easy Apply" needs a session. If login
is required, stop and ask the user.

## Search workflow

1. Load profile → build the query from **role + location + experience**.
2. `open_url(search_url, package_name="com.android.chrome")` — one call.
3. `read_window_tree` shallow → confirm results.
4. Scroll 2–3 screens, collecting: **title, company, experience, salary, location, link**.
5. Apply the profile filter: drop anything with the wrong experience band, wrong city
   (unless remote), or a salary range below the user's floor.
6. Return a ranked shortlist — max 10 — with one line of reasoning each.

## Applying (only when the user asks)

Applying is a **write action**. Rules:

1. Confirm the target list with the user (or use a list they already approved).
2. Prefer "Apply on company site" over "Quick Apply" only if the user asked for it —
   Quick Apply is fewer steps and less error-prone on mobile.
3. Fill forms with the **profile values only**. If a field has no profile value, stop and
   `ask_user` — do not guess salary, notice period, or dates.
4. Upload the resume from the workspace if the form asks for one.
5. **Screenshot before submitting** and confirm with the user, unless they explicitly
   said "apply without asking".

### Form-fill sequence

```
read_window_tree            → find the form fields
find_node(text="...")       → locate each field
set_text(<field>, <value>)  → fill
(verification) read back the field value
click_node("Submit")        → only after confirmation
```

If `set_text` does not take, fall back to keyboard control:
`keyboard_read_field` → `keyboard_clear` → `keyboard_type`.

## Tracking

After every application, append a row:

```
write_text_file path="<workspace>/job-applications.csv"
date,company,role,board,status,link,notes
```

Set `status` to `applied`, `skipped`, or `failed`. On later runs, read this file first so
you never re-apply to the same posting.

## Daily job digest

If the user wants it automated, offer a cron job:

```
schedule_job(
  name="job-digest",
  schedule_type="cron",
  cron="0 9 * * MON-FRI",
  mode="llm",
  prompt="Load the job-hunter skill. Read job-profile.md. Search the boards for new
          matching roles posted in the last 24h. Skip anything already in
          job-applications.csv. Send me the top 5 via notification."
)
```

## Guardrails

- Never apply to a role the user did not approve.
- Never fabricate experience, salary, or notice period to pass a filter.
- Never enter a password or OTP.
- If a board shows a CAPTCHA or "unusual activity", stop and hand back to the user.
