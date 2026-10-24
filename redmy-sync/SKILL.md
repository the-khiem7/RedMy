---
name: redmy-sync
description: Refresh task-registry.md against real Redmine data - new/renamed/closed issues assigned to the user, cached hours. Read-only against Redmine, safe to run anytime, no approval needed.
---

# Redmine Worklog - Sync Registry

Sync `task-registry.md` with the real data on Redmine. Read-only (writes nothing to Redmine), so there is no need to ask for confirmation before running. The format of every project file is defined in the `redmy-core` skill's `references/schemas.md` (installed next to this skill).

## Steps

1. If `task-registry.md` is missing, or its frontmatter lacks `redmine_user`, `redmine_base_url` or `tracking_since`, stop and tell the user to run the `redmy-core` skill first (`npx skills add the-khiem7/RedMy --skill redmy-core` if it is not installed). Otherwise call `get_current_user` and compare its `login` with `redmine_user`. If they differ, stop and report both values - the MCP is authenticated as someone else and the registry would fill with that person's issues.
2. `list_issues(assigned_to_id=<id>, status="*", limit=100)` to get all issues currently assigned to the user.
3. `list_time_entries(user_id=<id>, from_date=<last_synced, or tracking_since on the first sync>, limit=100)` and collect the issue ids of the entries. Reason: an issue the user logs time on but is not assigned to (reassigned later, or primarily assigned to a colleague) never appears in step 2. If `total_count` is larger than the number of entries returned, say so in the report - the tool has no offset, so the rest are not seen.
4. Read `task-registry.md` and diff it against the issues from steps 2 and 3:
   - New issue not yet in the table -> add a new row (Task ID, Project, Subject, Status, Repo/Source left as `*(not yet determined)*` if unknown, Constraints `-`, Link built from `redmine_base_url`). Use `get_issue` for the fields of an issue found only in step 3.
   - Issue already present but its subject/status changed (like issue #123, whose subject once changed from "Staging" to "Production") -> update in place.
   - Issue fully closed and no longer active -> keep it in the table (the history is still valuable), only update Status.
5. For each issue in the table, sum the `hours` of `list_time_entries(issue_id=<issue>, user_id=<id>, limit=100)` and write that total into the "Hours logged (cache, as of <date>)" column - always include the refresh date, never leave a number without a date. The column holds the user's own hours: `spent_hours`/`total_spent_hours` from `get_issue` count every user on the issue, so an issue shared with a colleague would show their hours as the user's. Flag an issue whose `total_count` exceeds the entries returned.
6. Update `last_synced` in the frontmatter of `task-registry.md` to today.
7. If an exclusion/rename is found that is not merely a field update (for example: removing a project entirely at the user's request, or re-confirming an ambiguous name that maps to no issue) -> record it as a new entry in `decisions.md` (an Entry index row plus a new `RW-D<n>` section, as `redmy-core` defines it), do not just edit silently.
8. Report back to the user: how many tasks were newly added, how many statuses/subjects changed - no need to ask for approval because this is a read operation. Then list every row whose Repo/Source is `*(not yet determined)*` and ask the user for each one's local repo path, or `none` when the work has no repo; write each answer into the row. Reason: `redmy-generate` never scans an issue without a repo, so an undetermined row drafts nothing.

## Do not

- Never call `smart_log_time`/`create_time_entry`/`create_issue`/`update_issue` in this skill - this is only syncing the list.
- Do not delete a row from `task-registry.md` on your own when an issue disappears from the `list_issues` result (it may be a temporary unassign) - ask the user before deleting it for good.
- Do not touch `checkpoint.md` or `history/*.md` - they are outside this skill's scope.
