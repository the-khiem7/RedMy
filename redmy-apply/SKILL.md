---
name: redmy-apply
description: Log the approved draft rows from history/*.md to Redmine for real via smart_log_time, then replace the draft marker with a real worklog link and advance checkpoint.md. Never logs without the user's explicit approval of the exact table shown.
---

# Redmine Worklog - Apply (Log for Real to Redmine)

This is the only step allowed to call `smart_log_time`/`create_time_entry`. Always re-read the draft from the file every time you are invoked - never trust "already approved in a previous chat turn" unless you have re-read it from disk, because this process is ad-hoc and can be interrupted midway.

## Steps

1. Scan all of `history/*.md` for rows whose Worklog = `Not logged` (skip rows marked `**Skipped**`).
2. Run lint before presenting - see `references/lint-rules.md`. For any row that fails lint, state clearly why it failed; do not fix it on your own and then log it - let the user decide how to fix it.
3. Present the whole table (grouped by day, in the format the user is used to approving) together with the total hours per issue. Apply the gate in `references/apply-gate.md` word for word - do not shorten it, and do not skip this step even if the user seems impatient.
4. Only after the user has explicitly approved **this** table (not a general approval given earlier): call `smart_log_time` for each row, with the activity defaulting to "Developing" unless the user specifies otherwise.
5. For each row logged successfully: replace `Not logged` with `[View worklog](<REDMINE_BASE_URL>/time_entries/<id>/edit)` directly in the corresponding `history/` file.
6. Update `checkpoint.md`: for each issue that just had a row logged successfully, set `Last Logged Date`/`Last Logged Hours` from the latest row (by date) that was actually logged - not the last row in the file, if the dates are not strictly increasing.
7. Report back to the user: the total hours logged successfully, and list explicitly which rows could NOT be logged (error/permission/rejected) with the reason - see the "When a row fails" section below.

## When a row fails midway

Do not try other ways to get around it on your own (switching tools, changing params, splitting the request) when a row is blocked (permission error, API error). Report the situation as-is to the user and wait for a decision: the user can log it manually, grant permission and then ask for exactly that row to be run again, or say to skip it (mark it `**Skipped**` instead of `Not logged`, and do not count it toward the checkpoint).

## Do not

- Do not log any row that is not in the table the user explicitly approved in step 3 - even if the agent judges that row to be "definitely correct".
- Do not treat one approval as applying to the next generate cycle, or to a table that was edited after approval.
- Do not edit the Redmine Comment text on your own to get past lint and then log it - report the lint error and let the user or `redmy-generate` fix it.
