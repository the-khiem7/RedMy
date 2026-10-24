# Project file schemas

Every redmy skill reads the files below at the project root with these exact names, frontmatter fields and column headers. The examples use made-up values. A cell may wrap its value in `**` for emphasis; read it without the markers.

## task-registry.md

One row per Redmine issue or epic the user tracks. Only `redmy-sync` writes rows; `redmy-core` creates the file.

```markdown
---
last_synced: 2026-01-20
redmine_user: jane.doe
redmine_base_url: https://redmine.example.com
tracking_since: 2026-01-05
---

# Task Registry

| Task ID | Project | Subject | Status | Hours logged (cache, as of 2026-01-20) | Repo/Source | Constraints | Link |
|---|---|---|---|---:|---|---|---|
| **#1001** | Example Project | Deploy the example service | In Progress | 12.0 | `C:\repos\example-service` | - | [#1001](https://redmine.example.com/issues/1001) |
| **#1000** | Example Project | Epic: Example platform | To Do | 0.0 | `C:\repos\example-service` | Parent epic of #1001 - do not log directly | [#1000](https://redmine.example.com/issues/1000) |
```

| Field or column | Rule |
|---|---|
| `last_synced` | date of the last `redmy-sync`; absent until the first sync |
| `redmine_user` | the `login` from `get_current_user`, confirmed by the user; sync and apply stop when the MCP reports another login |
| `redmine_base_url` | root URL with no trailing slash; issue and worklog links are built from it |
| `tracking_since` | first date a row may be drafted for an issue with no row in `checkpoint.md` |
| Hours logged | the sum of the user's own time entries on the issue, with the refresh date in the header; a cache, never a source of truth |
| Repo/Source | one or more local repo paths in backticks; `none` when the work has no repo (only self-reported rows); `*(not yet determined)*` until the user names it, and such an issue is never scanned |
| Constraints | free-text rule for this issue, `-` when none; a parent epic says it must not be logged to directly |

## checkpoint.md

Where the next `redmy-generate` window starts, one row per issue. Only `redmy-apply` writes it, and only after a time entry was really created.

```markdown
# Checkpoint

| Issue | Last Logged Date | Last Logged Hours (that day) |
|---|---|---:|
| #1001 | 2026-01-15 | 4.0 |
```

- The window of an issue starts the day after its Last Logged Date. An issue with no row starts at `tracking_since`.
- Last Logged Date is the latest date that was actually logged, not the date of the last row in the history file.

## git-identity-map.md

Which git author names on each repo are the user. `redmy-generate` adds a row only after the user confirms it.

```markdown
# Git Identity Map

| Repo | Git author name(s) = the user | Confirmed on |
|---|---|---|
| `C:\repos\example-service` | `jane.doe`, `jdoe-laptop` | 2026-01-06 |
```

## decisions.md

One-off ambiguities that were resolved, so they are not asked again. A rule that applies to one issue every cycle belongs in the Constraints column of `task-registry.md` instead.

```markdown
# Decisions Log

## Entry index

| ID | Topic |
|---|---|
| RW-D1 | "Phoenix" is a codename for #1001 |

## RW-D1: "Phoenix" is a codename for #1001

What was decided, why, and what breaks if it is ignored.
```

- IDs are `RW-D<n>`, increasing, never reused - also when an entry is superseded.
- Every entry has an index row.

## history/<issue-id>-<slug>.md

One append-only worklog table per issue. `<slug>` is a short kebab-case form of the subject, fixed once the file exists. `redmy-generate` creates the file and appends rows; `redmy-apply` replaces `Not logged` with the worklog link.

```markdown
# Task #1001: Deploy the example service

[View on Redmine](https://redmine.example.com/issues/1001) - project Example Project - repo `C:\repos\example-service`

Append-only. Do not edit/delete rows whose `Worklog` is already a real link. Rows marked **Skipped** do not count toward the total hours and are kept only to record the real git activity of that day.

| Date | Hours | Activity | Redmine Comment | Worklog | Evidence |
|---|---:|---|---|---|---|
| 2026-01-14 | 4.0 | Developing | `Build the deployment pipeline for the example service.` | [View worklog](https://redmine.example.com/time_entries/5001/edit) | example-service `a1b2c3d` |
| 2026-01-15 | 2.0 | Developing | `Add health checks to the example service.` | Not logged | example-service `d4e5f6a`..`b7c8d9e` |
| 2026-01-17 | 3.0 | Developing | `Tune the example service autoscaling.` | **Skipped** | example-service `f0a1b2c` - skipped: the user moved this work to #1002 |
```

| Column | Rule |
|---|---|
| Date | `YYYY-MM-DD`, the date of the commit as `git log` prints it (author date, local time) |
| Hours | decimal with one place; the rows of one date across all history files, not counting skipped rows, add up to 8.0 at most unless the user approved more by name |
| Activity | a Redmine time entry activity name, as `list_time_entry_activities` returns it |
| Redmine Comment | one sentence in backticks; may end with `[needs confirmation]` while a judgment call is open |
| Worklog | `Not logged` (draft), `[View worklog](<redmine_base_url>/time_entries/<id>/edit)` (logged), or `**Skipped**` (never logged, kept for the record) |
| Evidence | repo name and commit SHA(s), a range `<a>..<b>`, or `Self-reported by the user - no commit`; also records a weekend day the user confirmed, the reason a row was skipped (`skipped: <reason>`), the entry a row replaces after a correction, and hours the user asked to add (`filled to <h> for the day at the user's request on <date>`) |

- A skipped row stays in the table at its date. There is no separate table for skipped rows.
- A line below the table such as `> Total logged on Redmine: 12.0h.` is optional and is never read as data.
