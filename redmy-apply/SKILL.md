---
name: redmy-apply
description: Log the approved draft rows from history/*.md to Redmine for real via smart_log_time, then replace the draft marker with a real worklog link and advance checkpoint.md. Never logs without the user's explicit approval of the exact table shown.
---

# Redmine Worklog - Apply (Log for Real to Redmine)

This is the only step allowed to call `smart_log_time`/`create_time_entry`. Always re-read the draft from the file every time you are invoked - never trust "already approved in a previous chat turn" unless you have re-read it from disk, because this process is ad-hoc and can be interrupted midway. The format of every project file is defined in the `redmy-core` skill's `references/schemas.md` (installed next to this skill).

## Steps

0. If `task-registry.md` is missing, or its frontmatter lacks `redmine_user` or `redmine_base_url`, stop and tell the user to run the `redmy-core` skill first - log nothing. Otherwise call `get_current_user` and compare its `login` with `redmine_user`. If they differ, stop and report both values - log nothing. Reason: the Redmine MCP logs as whoever owns its API key, and a session on a shared or borrowed machine can carry another person's key; a mismatch writes hours into that person's record.
1. Scan all of `history/*.md` for rows whose Worklog = `Not logged` (skip rows marked `**Skipped**`). Read `task-registry.md` too: the Status and Constraints of each row's issue are lint input in step 3.
2. Check Redmine for each row before presenting it: `list_time_entries(issue_id=<issue>, user_id=<id from step 0>, from_date=<Date>, to_date=<Date>)`. If any entry comes back, mark the row `possible duplicate (#<entry id>, <hours>h)` in the table and do not log it unless the user approves that row by name. If the user confirms the entry is that row, write its link into the `history/` file instead of logging again. Reason: a row logged right before an interrupted session still reads `Not logged` on disk, and logging it again creates a duplicate entry that has to be deleted by hand.
3. Run lint before presenting - see `references/lint-rules.md`. For any row that fails lint, state clearly why it failed; do not fix it on your own and then log it - let the user decide how to fix it.
4. Present the whole table grouped by day, with the columns Date, Issue, Hours, Activity, Redmine Comment and Evidence, together with the total hours per issue. Apply the gate in `references/apply-gate.md` word for word - do not shorten it, and do not skip this step even if the user seems impatient.
5. Only after the user has explicitly approved **this** table (not a general approval given earlier), log the rows one at a time. For each row:
   1. Call `smart_log_time` with `issue_id` = the issue id from the `history/` file name, `spent_on` = the row's Date, `hours` = Hours, `activity` = the row's Activity, and `comment` = the Redmine Comment without its surrounding backticks. Never omit `spent_on`: the tool defaults to today, which files a backdated row on the wrong day.
   2. As soon as the call succeeds, replace `Not logged` with `[View worklog](<redmine_base_url>/time_entries/<id>/edit)` in the corresponding `history/` file, before calling for the next row. Reason: the link on disk is the only record that a row was logged; writing the links at the end leaves every row logged so far unrecorded if the session stops midway, and the next run logs them again.
6. Update `checkpoint.md`: for each issue that just had a row logged successfully, set `Last Logged Date`/`Last Logged Hours` from the latest row (by date) that was actually logged - not the last row in the file, if the dates are not strictly increasing.
7. Report back to the user: the total hours logged successfully, and list explicitly which rows could NOT be logged (error/permission/rejected) with the reason - see the "When a row fails" section below.

## When a row fails midway

Do not try other ways to get around it on your own (switching tools, changing params, splitting the request) when a row is blocked (permission error, API error). Report the situation as-is to the user and wait for a decision: the user can log it manually, grant permission and then ask for exactly that row to be run again, or say to skip it (mark it `**Skipped**` instead of `Not logged`, add `skipped: <reason>` to its Evidence, and do not count it toward the checkpoint).

## Correcting a logged row

A time entry that is already on Redmine is never changed or deleted by this skill: the MCP blocks DELETE, and editing an entry in place through `PUT` was rejected so that every write to an existing entry stays a deliberate act of the user. When the user says a logged row is wrong (wrong hours, date, issue, comment, or a duplicate):

1. Take the entry id from the row's worklog link, or from `list_time_entries(issue_id=<issue>, user_id=<id>, from_date=<Date>, to_date=<Date>)` for a duplicate that has no row. Show the entry's current date, hours, activity and comment.
2. Give the user two links per entry and ask them to delete it in Redmine:
   - delete: `<redmine_base_url>/time_entries?set_filter=1&issue_id=<issue>&spent_on=<Date>&user_id=me` - the user's entries for that issue and day, each with a delete button;
   - inspect: `<redmine_base_url>/time_entries/<id>/edit`.
3. After the user says it is deleted, confirm it with `list_time_entries` for that issue and date - do not update anything while the entry still exists.
4. Update the `history/` row: set Worklog to `Not logged` and correct the fields when the user wants it logged again, or to `**Skipped**` when not; add `replaces deleted entry #<id>` or `skipped: <reason>` to its Evidence. This is the only edit allowed on a row that had a real link. A corrected row goes through steps 2 to 5 like any draft.
5. Recompute the issue's row in `checkpoint.md` from the latest row that is still logged.

## Do not

- Do not log any row that is not in the table the user explicitly approved in step 4 - even if the agent judges that row to be "definitely correct".
- Do not treat one approval as applying to the next generate cycle, or to a table that was edited after approval.
- Do not edit the Redmine Comment text on your own to get past lint and then log it - report the lint error and let the user or `redmy-generate` fix it.
