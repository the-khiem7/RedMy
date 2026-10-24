---
name: redmy-generate
description: Draft new Redmine worklog rows since each issue's checkpoint date, sourced from git log of mapped repos. Appends unlogged draft rows to history/*.md. Never calls the Redmine time-logging API - that's redmy-apply's job.
---

# Redmine Worklog - Generate Drafts

Draft new worklogs for the gap from the checkpoint to today. **Never call `smart_log_time`/`create_time_entry` in this skill** - finish by handing over to the `redmy-apply` skill. Read `references/rules.md` in this skill before drafting any row - it is a list of mistakes the user has rejected many times in the past, so do not draft from memory. The format of every project file is defined in the `redmy-core` skill's `references/schemas.md` (installed next to this skill).

## Steps

0. If `task-registry.md` or `checkpoint.md` is missing, or the registry frontmatter lacks `redmine_user`, `redmine_base_url` or `tracking_since`, stop and tell the user to run the `redmy-core` skill first. If `last_synced` is missing or more than 3 days old, re-run the steps of the `redmy-sync` skill first, inline, without the user having to call it separately.
1. Read `checkpoint.md` to get each issue's `Last Logged Date` - this is the real stopping point (counting only rows that were actually logged, not draft/skipped rows). An issue with no row starts at `tracking_since` from the registry frontmatter, never earlier.
2. Collect every commit SHA already cited in the Evidence column of any `history/*.md` row - logged, `Not logged` or skipped, in every issue's file - expanding a cited range `<a>..<b>` with `git rev-list <a>^..<b>` on its repo. These commits are already spoken for. Reason: one repo serves several issues and each issue has its own checkpoint, so the window of one issue re-covers commits already logged under another.
3. Read `git-identity-map.md` to know which git author name on each repo is the user. If an unfamiliar author name appears on a known repo, it is not certain to be the user - ask the user to confirm before counting it, then add a new row to `git-identity-map.md`. For a repo with no row yet, list the distinct authors of the window (`git log --format='%an <%ae>' | sort -u`) and ask which of them are the user before reading any commit.
4. For each issue with a gap (Last Logged Date, today]: skip an issue whose Repo/Source is `*(not yet determined)*` and name it in the step 8 report, and draft only self-reported rows for one whose Repo/Source is `none`. Otherwise run `git log` on the repo(s) mapped in the Repo/Source column of `task-registry.md`, filtered by author = the user (step 3), from the day after the checkpoint, and drop the SHAs collected in step 2. A commit belongs to the date of its commit as `git log` shows it, in local time - also after midnight; work the user remembers doing on another day is a self-reported row.
5. Assign commits to issues:
   - Read the issue's Constraints column in `task-registry.md` and `decisions.md` first - many ambiguous cases already have an answer there (e.g. which issue excludes whose work, which submodule is not counted).
   - If one day/one commit has content belonging to multiple issues, read the actual diff/file content (not just the commit subject) to split the hours by proportion - do not assign the whole day to one issue out of "convenience".
   - If unsure which issue fits (unfamiliar system name, a codename that matches no task), look up Redmine (`list_issues`/`list_projects`) for a matching issue before guessing. Once resolved, record it in `decisions.md`.
   - Do not consider history inside a git submodule when assessing effort - only count commits at the level of the repo listed in Repo/Source (see the matching entry in `decisions.md` as an example).
6. Draft each working day as 1 row: Date, Hours, Activity, Redmine Comment (see `references/rules.md` for style), Worklog = `Not logged`, Evidence = real commit SHA (or "Self-reported by the user - no commit" if the user recounted it verbally). The Hours of one date across every issue - rows already in any `history/*.md` file plus the rows drafted now, not counting `**Skipped**` rows - add up to 8.0 at most; a day with less evidence stays short and is asked about, never topped up to 8 unless the user asks for it (see rule 1 in `references/rules.md` for how such hours are recorded). Weekend days or leave days only count hours when the user explicitly confirms, never by inference; write that confirmation into Evidence (e.g. `weekend confirmed by the user`), because `redmy-apply` lint fails a weekend row without it. Attach the label **[needs confirmation]** at the end of the Redmine Comment for any row where the issue assignment or the number of hours is a judgment call and not certain - so the user sees it right at approval time, not only in the agent's head.
7. Append the draft rows to the corresponding file `history/<issue-id>-<slug>.md` (create a new file if that issue has real work hours for the first time - do not pre-create empty files for issues with 0h). Do not edit/delete old rows.
8. Finish by presenting the whole draft table by day to the user, list the issues that were not scanned because their Repo/Source is undetermined, and state clearly: run the `redmy-apply` skill next to log for real.

## Do not

- Do not call any Redmine MCP write tool (`smart_log_time`, `create_time_entry`, `create_issue`, `update_issue`) - not this skill's job.
- Do not add hours on your own for a day with no evidence (a commit or the user's own account) - better to leave it blank and ask than to guess.
- Do not describe the Redmine Comment with internal terminology (report file names, release branch names, internal tool names) - see `references/rules.md`.
