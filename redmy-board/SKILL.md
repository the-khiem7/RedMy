---
name: redmy-board
description: Show the user's worklog board for one week - one row per day, one column per issue, hours already logged on Redmine next to the draft rows in history/*.md, with day totals and what still blocks each day. Use when the user asks for the worklog board, what is logged this week, or which days are short. Read-only against Redmine and the project files, safe to run anytime.
---

# Redmine Worklog - Weekly Board

Show where one week stands: what is on Redmine, what is drafted and waiting for `redmy-apply`, and which days are short or have work with no row. Read-only - this skill writes nothing, locally or to Redmine, so it needs no approval. The format of every project file is defined in the `redmy-core` skill's `references/schemas.md` (installed next to this skill).

## Steps

0. If `task-registry.md` is missing, or its frontmatter lacks `redmine_user` or `redmine_base_url`, stop and tell the user to run the `redmy-core` skill first. Otherwise call `get_current_user` and compare its `login` with `redmine_user`; if they differ, stop and report both values - the board would show another person's hours.
1. Take the week as Monday to Sunday containing today, or the week the user names.
2. Read the user's entries of that week from Redmine: `redmine_raw_request(path="/time_entries.json", params={"user_id": <id>, "from": <Monday>, "to": <Sunday>, "limit": 100, "offset": <n>})`, raising `offset` until `total_count` entries are read. Do not use `list_time_entries`: it ignores `from_date`/`to_date`.
3. Read every row of `history/*.md` dated in that week: its issue (from the file name), Hours, Worklog value, and whether the Redmine Comment carries `[needs confirmation]`.
4. Reconcile the two sides. A logged row matches the entry whose id is in its worklog link. Keep every mismatch for the findings list in step 6:
   - an entry on Redmine with no row, or whose row is `**Skipped**` or `Not logged` (possible duplicate);
   - a row with a worklog link whose entry is not on Redmine (deleted, or logged under another user).
5. For each day with no row on any issue, check the repos mapped in `task-registry.md` for commits by the user (`git-identity-map.md`) on that date. A day with commits and no row is marked as undrafted work.
6. Present the board exactly in this shape, labels in the language of the conversation:
   - one line naming the week, Monday and Sunday dates;
   - the board table: one row per day (weekday and date), one column per issue that has an entry or a row in the week (`#<id>` plus a two-word subject), and a Total column; a last row totals each column as `<logged> + <draft>`; when more than 5 issues have hours, swap the axes so the table still fits a terminal;
   - a legend line for the cell marks below;
   - a status list: hours logged with their entry ids; draft hours waiting for `redmy-apply` and which rows lint will block; days under 8.0h; days with undrafted work; every mismatch from step 4;
   - one closing line: the week total if every draft is approved, against 8.0h per weekday.

| Cell | Meaning |
|---|---|
| `✅ 4.0` | logged on Redmine |
| `📝 3.5` | draft, Worklog `Not logged` |
| `⚠` after the hours | the row still carries `[needs confirmation]` |
| `⏭ 8.0` | `**Skipped**` row - shown, never added to a total |
| `❓` | commits on that day with no row |
| `-` | nothing |
| `↓` after a day total | a weekday under 8.0h |

A day total counts logged and draft hours, never skipped ones; write it in bold when every hour of the day is logged.

## Do not

- Do not write any file, and do not call `smart_log_time`, `create_time_entry`, `create_issue`, `update_issue`, or `redmine_raw_request` with any method other than `GET`.
- Do not draft, fix or log rows from the board - point to `redmy-generate` for a short or undrafted day, and to `redmy-apply` for drafts and for a mismatch (its "Correcting a logged row" flow).
- Do not count a weekend day as short: only weekdays carry the 8.0h mark.
