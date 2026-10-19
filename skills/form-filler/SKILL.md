---
name: form-filler
description: Fill any on-screen form (in a browser or a native app) from a stored personal profile, then stop for review before submitting. Load when the user asks to fill a form, complete a signup, enter details, or apply somewhere.
---

# Form Filler

Reusable, safe filling of forms from a **stored profile** rather than improvising values.

## The profile

Forms are filled from a file in the workspace, not from memory. Default location:

```
<form-profile.md>
```

Suggested shape (keep it factual, keep it current):

```
# Personal profile
full_name: ...
first_name: ...
last_name: ...
email: ...
phone: ...
alt_phone: ...
dob: ...
gender: ...
address_line1: ...
city: ...
state: ...
pincode: ...
country: India

# Work
current_company: ...
designation: ...
total_experience_years: ...
notice_period_days: ...
current_ctc: ...
expected_ctc: ...
preferred_location: ...
work_authorisation: ...

# Links
linkedin: ...
github: ...
portfolio: ...

# Education
highest_degree: ...
institute: ...
graduation_year: ...
```

If the file is missing or a needed key is absent, `ask_user` for **only that key**, then
`write_text_file` to add it back to the profile so it is asked once, never twice.

## Workflow

1. **Read the form first.** `read_window_tree` and list every field, top to bottom, with
   its label. Do not start filling blind.
2. **Map label → profile key.** Fuzzy match is fine (`Mobile Number` → `phone`,
   `Date of Birth` → `dob`).
3. **Report the plan before touching anything** when the form has more than ~4 fields:
   *"I'll fill: Name, Email, Phone, City. Missing: Salary expectation — what should I put?"*
4. **Fill field by field:**
   ```
   find_node(text="Email")            → locate
   set_text(<node>, "user@x.com")     → fill
   ```
   Fallback if `set_text` no-ops: `keyboard_read_field` → `keyboard_clear` → `keyboard_type`.
5. **Verify.** Re-read each field value after filling. A silently empty field is the
   most common failure.
6. **Stop at the submit button.** Unless the user explicitly pre-authorised submission.

## Handling each widget type

| Widget | Approach |
|---|---|
| Text input | `set_text` |
| Dropdown / spinner | Tap it → `read_window_tree` → `click_node` on the exact option |
| Radio buttons | `click_node` on the option's text; verify the selection changed |
| Checkboxes | Same; verify the checked state |
| Date picker | Prefer typing the text field if it accepts it, else drive the picker |
| File upload | Only from a known workspace path; use the SAF picker via screen automation |
| OTP / CAPTCHA | **Stop.** Hand back to the user. |
| Password | **Never autofill or read.** Skip and tell the user. |

## Safety rules

- Only values that exist in the profile get filled. **Never guess** dates, salaries,
  experience, or legal declarations.
- Never tick a consent/terms checkbox on the user's behalf without being asked.
- Never submit payment, legal, or government forms unattended.
- If a field is marked required and unmapped, stop and ask — do not invent.
- After a successful fill, offer to save any newly-learned field back into the profile.

## After submitting

Report: form name/URL, which fields were filled, which were left blank, and the
confirmation/reference number if the page showed one. Save a copy to the workspace:

```
write_text_file path="<workspace>/form-submissions/<date>-<form>.md"
```
