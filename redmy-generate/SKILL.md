---
name: redmy-generate
description: Draft new Redmine worklog rows since each issue's checkpoint date, sourced from git log of mapped repos. Appends unlogged draft rows to history/*.md. Never calls the Redmine time-logging API - that's redmy-apply's job.
---

# Redmine Worklog - Generate Drafts

Draft new worklogs for the gap from the checkpoint to today. **Never call `smart_log_time`/`create_time_entry` in this skill** - finish by handing over to the `redmy-apply` skill. Read `references/rules.md` in this skill before drafting any row - it is a list of mistakes the user has rejected many times in the past, so do not draft from memory.

## Steps

0. If the `task-registry.md` frontmatter `last_synced` is older than a few days (or just run it anyway, since it is a read operation and cheap) - re-run the steps of the `redmy-sync` skill first, inline, without the user having to call it separately.
1. Read `checkpoint.md` to get each issue's `Last Logged Date` - this is the real stopping point (counting only rows that were actually logged, not draft/skipped rows).
2. Read `git-identity-map.md` to know which git author name on each repo is the user. If an unfamiliar author name appears on a known repo, it is not certain to be the user - ask the user to confirm before counting it, then add a new row to `git-identity-map.md`.
3. For each issue with a gap (Last Logged Date, today]: run `git log` on the repo(s) mapped in the Repo/Source column of `task-registry.md`, filtered by author = the user (step 2), from the day after the checkpoint.
4. Assign commits to issues:
   - Read the issue's Constraints column in `task-registry.md` and `decisions.md` first - many ambiguous cases already have an answer there (e.g. which issue excludes whose work, which submodule is not counted).
   - If one day/one commit has content belonging to multiple issues, read the actual diff/file content (not just the commit subject) to split the hours by proportion - do not assign the whole day to one issue out of "convenience".
   - If unsure which issue fits (unfamiliar system name, a codename that matches no task), look up Redmine (`list_issues`/`list_projects`) for a matching issue before guessing. Once resolved, record it in `decisions.md`.
   - Do not consider history inside a git submodule when assessing effort - only count commits at the level of the repo listed in Repo/Source (see the matching entry in `decisions.md` as an example).
5. Draft each working day as 1 row: Date, Hours, Activity, Redmine Comment (see `references/rules.md` for style), Worklog = `Not logged`, Evidence = real commit SHA (or "Self-reported by the user - no commit" if the user recounted it verbally). Balance at ~8h per working day (Mon-Fri); weekend days or leave days only count hours when the user explicitly confirms, never by inference. Attach the label **[needs confirmation]** at the end of the Redmine Comment for any row where the issue assignment or the number of hours is a judgment call and not certain - so the user sees it right at approval time, not only in the agent's head.
6. Append the draft rows to the corresponding file `history/<issue-id>-<slug>.md` (create a new file if that issue has real work hours for the first time - do not pre-create empty files for issues with 0h). Do not edit/delete old rows.
7. Finish by presenting the whole draft table by day to the user, and state clearly: run the `redmy-apply` skill next to log for real.

## Do not

- Do not call any Redmine MCP write tool (`smart_log_time`, `create_time_entry`, `create_issue`, `update_issue`) - not this skill's job.
- Do not add hours on your own for a day with no evidence (a commit or the user's own account) - better to leave it blank and ask than to guess.
- Do not describe the Redmine Comment with internal terminology (report file names, release branch names, internal tool names) - see `references/rules.md`.
