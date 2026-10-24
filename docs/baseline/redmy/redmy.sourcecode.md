---
baseline_schema: "2.0"
pack: "redmy"
document: "sourcecode"
status: "active"
updated: "2026-10-24"
code_ref: "b2b9c8c"
---

# RedMy - contract topology

## Execution flow

0. redmy-core runs once per project: it creates the missing project files from `references/schemas.md`, confirms the Redmine login and asks for `redmine_base_url` and `tracking_since`, then hands over to redmy-sync. The other three skills stop and name redmy-core when a file or one of those frontmatter fields is missing.
1. redmy-sync checks the Redmine login against `redmine_user`, then refreshes `task-registry.md` from the user's assigned issues and own time entries and asks for every undetermined repo path; redmy-generate runs it inline first when `last_synced` is missing or more than 3 days old (generate step 0).
2. redmy-generate reads the checkpoint (or `tracking_since` for an issue with no row), the SHAs already cited in `history/*.md`, the git identity map and the registry, then appends draft rows (`Worklog` = `Not logged`) to `history/*.md` and hands over to redmy-apply.
3. redmy-apply checks the login, checks Redmine for an existing entry per row, lints, shows the whole table, and calls `smart_log_time` row by row only after the user approves that exact table; it writes each worklog link right after its call and then advances `checkpoint.md`. Its correction flow gives the user a delete link and updates the files once the entry is gone.

## Who reads and writes which project file

R reads, W writes, C creates when missing, a dash means the skill never mentions it. Every cell is from the skill steps.

| File | redmy-core | redmy-sync | redmy-generate | redmy-apply |
|---|---|---|---|---|
| `task-registry.md` | C, adds missing frontmatter | R and W | R | R |
| `checkpoint.md` | C | untouched by rule | R | W |
| `git-identity-map.md` | C | - | R and W | - |
| `decisions.md` | C | W | R and W | - |
| `history/<issue-id>-<slug>.md` | never | untouched by rule | R and W (creates lazily) | R and W |

- redmy-core never overwrites an existing file; it only adds a missing file or frontmatter field.
- redmy-sync's explicit "Do not" list forbids touching `checkpoint.md` and `history/*.md`; it writes Repo/Source answers into the registry (step 8).
- redmy-generate writes `git-identity-map.md` only to add a confirmed author row (step 3) and `decisions.md` only to record a resolved ambiguity (step 5); it reads every `history/*.md` file for cited SHAs (step 2) and day totals (step 6).
- redmy-apply reads `redmine_user`, `redmine_base_url`, Status and Constraints from `task-registry.md` (steps 0 and 1), reads every `history/*.md` file for day totals and duplicate rows (lint checks 6 and 8), and builds the worklog link from `redmine_base_url` (step 5).

## Redmine tools per skill

| Skill | Reads | Writes |
|---|---|---|
| redmy-core | `get_current_user` | none |
| redmy-sync | `get_current_user`, `list_issues`, `get_issue`, `redmine_raw_request` GET `/time_entries.json` | none |
| redmy-generate | `list_issues`, `list_projects` | none |
| redmy-apply | `get_current_user`, `redmine_raw_request` GET `/time_entries.json` | `smart_log_time` |

- `list_time_entries` is not used: it ignores `from_date`/`to_date` and has no offset (RM-D16 in hallucination).

## Formats

The formats live in `redmy-core/references/schemas.md`, which is the only definition (RM-D12 in hallucination). In summary:

| File | Fields and columns |
|---|---|
| `task-registry.md` | frontmatter `last_synced`, `redmine_user`, `redmine_base_url`, `tracking_since`; columns Task ID, Project, Subject, Status, "Hours logged (cache, as of <date>)", Repo/Source, Constraints, Link |
| `checkpoint.md` | one row per issue with Issue, `Last Logged Date`, `Last Logged Hours (that day)` |
| `git-identity-map.md` | Repo, git author name(s) that are the user, Confirmed on |
| `decisions.md` | an Entry index table plus one `RW-D<n>` section per entry |
| `history/<issue-id>-<slug>.md` | Date, Hours, Activity, Redmine Comment, Worklog, Evidence |

- `Worklog` takes `Not logged`, `[View worklog](<redmine_base_url>/time_entries/<id>/edit)` or `**Skipped**`; a skipped row stays in the one table with `skipped: <reason>` in Evidence (RM-D14).
- `Evidence` is a commit SHA, a range `<a>..<b>`, or "Self-reported by the user - no commit", and also records a confirmed weekend day, a skip reason, and the entry a corrected row replaces; a Redmine Comment carries `[needs confirmation]` when the issue or the hours are a judgment call.
- Repo/Source is a path, `none`, or `*(not yet determined)*`; an undetermined issue is never scanned and is named in the generate report.
- "Hours logged" holds the sum of the user's own time entries on the issue, not the issue's `spent_hours` (RM-D10).
