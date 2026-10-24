---
baseline_schema: "2.0"
pack: "redmy"
document: "sourcecode"
status: "active"
updated: "2026-10-24"
code_ref: "1a84fca"
---

# RedMy - contract topology

## Execution flow

1. redmy-sync checks the Redmine login against `redmine_user`, then refreshes `task-registry.md` from the user's assigned issues and own time entries; redmy-generate runs it inline first when `last_synced` is more than 3 days old (generate step 0).
2. redmy-generate reads the checkpoint, the SHAs already cited in `history/*.md`, the git identity map and the registry, then appends draft rows (`Worklog` = `Not logged`) to `history/*.md` and hands over to redmy-apply.
3. redmy-apply checks the login, checks Redmine for an existing entry per row, lints, shows the whole table, and calls `smart_log_time` row by row only after the user approves that exact table; it writes each worklog link right after its call and then advances `checkpoint.md`.

## Who reads and writes which project file

R reads, W writes, a dash means the skill never mentions it. Every cell is from the skill steps, not from running them.

| File | redmy-sync | redmy-generate | redmy-apply |
|---|---|---|---|
| `task-registry.md` | R and W | R | R |
| `checkpoint.md` | untouched by rule | R | W |
| `git-identity-map.md` | - | R and W | - |
| `decisions.md` | W | R and W | - |
| `history/<issue-id>-<slug>.md` | untouched by rule | R and W (creates lazily) | R and W |

- redmy-sync's explicit "Do not" list forbids touching `checkpoint.md` and `history/*.md`.
- redmy-generate writes `git-identity-map.md` only to add a confirmed author row (step 3) and `decisions.md` only to record a resolved ambiguity (step 5); it reads every `history/*.md` file for cited SHAs (step 2) and day totals (step 6).
- redmy-apply reads `redmine_user`, Status and Constraints from `task-registry.md` (steps 0 and 1), reads every `history/*.md` file for day totals and duplicate rows (lint checks 6 and 8), and builds the worklog link from `<REDMINE_BASE_URL>` (step 5).

## Formats the skills assume and this repo never shows

| File | Fields named in the skill text | Where named |
|---|---|---|
| `task-registry.md` | frontmatter `last_synced` and `redmine_user`; columns Task ID, Project, Subject, Status, Repo/Source, Constraints, Link, and "Hours logged (cache, as of <date>)" | redmy-sync steps 1 and 4 to 6, redmy-apply step 0 |
| `checkpoint.md` | one row per issue with `Last Logged Date` and `Last Logged Hours` | redmy-generate step 1, redmy-apply step 6 |
| `git-identity-map.md` | repo path to the git author name(s) that are the user | redmy-generate step 3, README.md |
| `decisions.md` | an Entry index table plus one `RW-D<n>` section per entry | redmy-sync step 7 |
| `history/<issue-id>-<slug>.md` | columns Date, Hours, Activity, Redmine Comment, Worklog, Evidence | redmy-generate step 6 |

- `Worklog` takes three values in the skill text: `Not logged`, `[View worklog](<REDMINE_BASE_URL>/time_entries/<id>/edit)` and `**Skipped**`; the reference implementation keeps skipped rows in a separate table instead (RM-Q4 in hallucination).
- `Evidence` is a commit SHA, a SHA range `<a>..<b>`, or "Self-reported by the user - no commit", and records the user's confirmation of a weekend day; a Redmine Comment carries `[needs confirmation]` when the issue or the hours are a judgment call.
- "Hours logged" holds the sum of the user's own time entries on the issue, not the issue's `spent_hours` (RM-D10 in hallucination).
- The `RW-D<n>` entry format is described only as "see that file's format" in redmy-sync step 7; the text of the format is not in this repo.
