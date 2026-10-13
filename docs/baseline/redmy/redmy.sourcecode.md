---
baseline_schema: "2.0"
pack: "redmy"
document: "sourcecode"
status: "active"
updated: "2026-10-13"
code_ref: "74bd929"
---

# RedMy - contract topology

## Execution flow

1. redmy-sync refreshes `task-registry.md` from Redmine; redmy-generate runs it inline first when `last_synced` is old (generate step 0).
2. redmy-generate reads the checkpoint, the git identity map and the registry, then appends draft rows (`Worklog` = `Not logged`) to `history/*.md` and hands over to redmy-apply.
3. redmy-apply lints, shows the whole table, and calls `smart_log_time` only after the user approves that exact table; it then replaces each marker with a worklog link and advances `checkpoint.md`.

## Who reads and writes which project file

R reads, W writes, a dash means the skill never mentions it. Every cell is from the skill steps, not from running them.

| File | redmy-sync | redmy-generate | redmy-apply |
|---|---|---|---|
| `task-registry.md` | R and W | R | - |
| `checkpoint.md` | untouched by rule | R | W |
| `git-identity-map.md` | - | R and W | - |
| `decisions.md` | W | R and W | - |
| `history/<issue-id>-<slug>.md` | untouched by rule | W (creates lazily) | R and W |

- redmy-sync's explicit "Do not" list forbids touching `checkpoint.md` and `history/*.md`.
- redmy-generate writes `git-identity-map.md` only to add a confirmed author row (step 2) and `decisions.md` only to record a resolved ambiguity (step 4).
- redmy-apply builds the worklog link from `<REDMINE_BASE_URL>` (step 5).

## Formats the skills assume and this repo never shows

| File | Fields named in the skill text | Where named |
|---|---|---|
| `task-registry.md` | frontmatter `last_synced`; columns Task ID, Project, Subject, Status, Repo/Source, Constraints, Link, and "Hours logged (cache, as of <date>)" | redmy-sync steps 3 to 5 |
| `checkpoint.md` | one row per issue with `Last Logged Date` and `Last Logged Hours` | redmy-generate step 1, redmy-apply step 6 |
| `git-identity-map.md` | repo path to the git author name(s) that are the user | redmy-generate step 2, README.md |
| `decisions.md` | an Entry index table plus one `RW-D<n>` section per entry | redmy-sync step 6 |
| `history/<issue-id>-<slug>.md` | columns Date, Hours, Activity, Redmine Comment, Worklog, Evidence | redmy-generate step 5 |

- `Worklog` takes three values in the skill text: `Not logged`, `[View worklog](<REDMINE_BASE_URL>/time_entries/<id>/edit)` and `**Skipped**`.
- `Evidence` is a commit SHA or "Self-reported by the user - no commit"; a Redmine Comment carries `[needs confirmation]` when the issue or the hours are a judgment call.
- The `RW-D<n>` entry format is described only as "see that file's format" in redmy-sync step 6; the text of the format is not in this repo.
