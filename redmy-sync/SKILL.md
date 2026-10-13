---
name: redmy-sync
description: Refresh task-registry.md against real Redmine data - new/renamed/closed issues assigned to the user, cached hours. Read-only against Redmine, safe to run anytime, no approval needed.
---

# Redmine Worklog - Sync Registry

Sync `task-registry.md` with the real data on Redmine. Read-only (writes nothing to Redmine), so there is no need to ask for confirmation before running.

## Steps

1. Get the current user's id (`get_current_user` if not already known).
2. `list_issues(assigned_to_id=<id>, status="*", limit=100)` to get all issues currently assigned to the user.
3. Read `task-registry.md` and diff it against the result of step 2:
   - New issue not yet in the table -> add a new row (Task ID, Project, Subject, Status, Repo/Source left as `*(not yet determined)*` if unknown, Constraints left empty, Link).
   - Issue already present but its subject/status changed (like issue #123, whose subject once changed from "Staging" to "Production") -> update in place.
   - Issue fully closed and no longer active -> keep it in the table (the history is still valuable), only update Status.
4. For each issue in the table, call `get_issue` to get the real `spent_hours`/`total_spent_hours`, and update the "Hours logged (cache, as of <date>)" column - always include the refresh date, never leave a number without a date.
5. Update `last_synced` in the frontmatter of `task-registry.md` to today.
6. If an exclusion/rename is found that is not merely a field update (for example: removing a project entirely at the user's request, or re-confirming an ambiguous name that maps to no issue) -> record it as a new entry in `decisions.md` (see that file's format: the Entry index table + a new `RW-D<n>` section), do not just edit silently.
7. Report back to the user: how many tasks were newly added, how many statuses/subjects changed - no need to ask for approval because this is a read operation.

## Do not

- Never call `smart_log_time`/`create_time_entry`/`create_issue`/`update_issue` in this skill - this is only syncing the list.
- Do not delete a row from `task-registry.md` on your own when an issue disappears from the `list_issues` result (it may be a temporary unassign) - ask the user before deleting it for good.
- Do not touch `checkpoint.md` or `history/*.md` - they are outside this skill's scope.
