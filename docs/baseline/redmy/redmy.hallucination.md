---
baseline_schema: "2.0"
pack: "redmy"
document: "hallucination"
status: "active"
updated: "2026-10-24"
code_ref: "1a84fca"
---

# RedMy - decisions and open questions

## Entry index

| ID | About | Status | Related |
|---|---|---|---|
| RM-D1 | README claim that the skills create the project data files on first run | closed - disproven | RM-Q1 |
| RM-Q1 | How the data contract bootstraps in a new project | open | RM-D1, RM-Q2 |
| RM-Q2 | What `npx skills add --skill` copies into the project | open | RM-Q1 |
| RM-Q3 | Whether redmy-sync misses assigned issues beyond its fetch limit | open | RM-D10 |
| RM-D2 | Redmine login check before any skill reads or writes for the user | closed | RM-D10 |
| RM-D3 | Duplicate-entry guard in redmy-apply | closed | RM-D4 |
| RM-D4 | Explicit mapping of a row to `smart_log_time` parameters | closed | RM-D3 |
| RM-D5 | `[needs confirmation]` rows at the apply gate | closed | RM-D6 |
| RM-D6 | Lint checks 5 to 8 in redmy-apply | closed | RM-D5, RM-D8 |
| RM-D7 | Excluding commits already cited as evidence | closed | - |
| RM-D8 | Daily hour cap across issues in redmy-generate | closed | RM-D6 |
| RM-D9 | Which date a commit belongs to | closed | - |
| RM-D10 | Source of issue discovery and cached hours in redmy-sync | closed | RM-D2, RM-Q3 |
| RM-D11 | Where the rules added in 1a84fca live | closed | - |
| RM-Q4 | Format of a skipped row | open | - |
| RM-Q5 | How redmy-apply corrects or removes an entry already logged | open | RM-D3 |

## Closed decisions

### RM-D1: The skills do not create the project data files on first run

- Believed: README.md (at 74bd929, lines 23-24) says the installing project must provide the data files "or let `redmy-sync`/`redmy-generate` create on first run".
- Disproved by: redmy-sync reads `task-registry.md` (step 3) and appends to `decisions.md` (step 6); redmy-generate reads `checkpoint.md` (step 1), reads and adds rows to `git-identity-map.md` (step 2), and reads `decisions.md` (step 4). None has a step that creates those four files or defines their columns. The one file a skill creates is `history/<issue-id>-<slug>.md`, by redmy-generate step 6, lazily when an issue first has real work hours.
- Decision: no document asserts the claim; the README states what the skills do today and the gap stays recorded here until RM-Q1 is answered.
- What breaks if ignored: a reader installs the skills into an empty project expecting a bootstrap and runs skills that read files that do not exist; what the skill does then is not specified anywhere.

### RM-D2: Check the Redmine login against `redmine_user` before acting

- Decision: redmy-apply step 0 and redmy-sync step 1 compare `get_current_user` `login` with `redmine_user` in the `task-registry.md` frontmatter and stop on a mismatch; sync writes the field once after the user confirms it.
- Why: the MCP acts as whoever owns its API key, and the reference implementation records pair-work days on a colleague's machine.
- What breaks if ignored: hours land in another person's Redmine record, and sync fills the registry with that person's issues.
- Rejected: comparing against the git author names in `git-identity-map.md` (git names differ per repo and are not Redmine logins); no check (the earlier behavior).

### RM-D3: redmy-apply guards against logging a row twice

- Decision: before presenting, apply calls `list_time_entries` for each row's issue, user and date and marks any hit as a possible duplicate that is logged only on approval by name; after approval it logs one row at a time and writes each worklog link to `history/` right after its call succeeds.
- Why: a row logged right before an interrupted session still reads `Not logged` on disk, so the next run logs it again; the reference implementation's registry carries 9.0h of duplicate entries on one issue awaiting deletion.
- What breaks if ignored: duplicate time entries that only a manual delete in Redmine removes (RM-Q5).
- Rejected: writing all links after the last call (the earlier step 5, loses the record of every row logged before an interruption); matching existing entries by comment text (a comment can be edited on Redmine, so a real duplicate would pass).

### RM-D4: redmy-apply passes every row column to `smart_log_time` explicitly

- Decision: `issue_id` from the `history/` file name, `spent_on` from Date, `hours`, `activity` from the row's Activity, `comment` without its surrounding backticks; `spent_on` is never omitted.
- Why: `smart_log_time` defaults `spent_on` to today, and the earlier step forced "Developing" while rows can carry another activity (the reference implementation has "Solutioning" rows).
- What breaks if ignored: a backdated row is filed on the day apply runs, and rows lose their activity.
- Rejected: defaulting the activity to "Developing" unless the user says otherwise (the earlier step 4, overrides what the draft recorded).

### RM-D5: A `[needs confirmation]` row is not logged until the user resolves it

- Decision: lint check 3 keeps the label visible at the gate, blocks the row until the user resolves it by name, then removes the label from the `history/` comment before logging.
- Why: the label is a note to the user, not part of the work description.
- What breaks if ignored: "[needs confirmation]" is written into the official Redmine record.
- Rejected: keeping the label only at presentation (the earlier check 3, which said nothing about logging).

### RM-D6: redmy-apply lint checks loggability, day total, date and duplicate rows

- Decision: check 5 fails a closed issue or a parent epic whose Constraints forbid direct logging; check 6 fails a date whose hours across all `history/*.md` rows exceed 8.0; check 7 fails a future date or a weekend without recorded confirmation; check 8 fails a second row for the same issue and date. Checks 6 and 7 may be overridden by the user naming the row at the gate.
- Why: apply did not read `task-registry.md`, so nothing between draft and Redmine knew an issue was an epic or closed; apply is the last gate and rows can be edited by hand after generate.
- What breaks if ignored: hours logged to an epic or a closed issue, an inflated day, or a weekend inferred rather than confirmed.
- Rejected: leaving these checks to redmy-generate alone (a hand-edited row never passes through generate).

### RM-D7: redmy-generate drops commits already cited as evidence

- Decision: step 2 collects every SHA cited in any `history/*.md` Evidence cell, logged, `Not logged` or skipped, expanding `<a>..<b>` with `git rev-list <a>^..<b>`, and step 4 removes them from `git log`.
- Why: one repo serves several issues and each issue has its own checkpoint, so one issue's window re-covers commits already logged under another.
- What breaks if ignored: the same commit becomes evidence for two time entries.
- Rejected: relying on the per-issue checkpoint date alone (the earlier behavior, and `checkpoint.md` in the reference implementation told generate not to scan `history/`).

### RM-D8: The hours of one date across all issues are capped at 8.0, never topped up

- Decision: generate step 6 keeps the sum of a date's hours, existing rows plus new drafts, at 8.0 or less; a short day stays short and is asked about; a confirmed weekend is written into Evidence.
- Why: "balance at ~8h per working day" pushed drafts toward filling the day, which contradicts rule 1 of `rules.md` (no padding on thin evidence), and did not say whether 8h was per issue or per day.
- What breaks if ignored: padded hours, or two issues each drafted to 8h on one day.
- Rejected: balancing each working day to about 8h (the earlier step 5).

### RM-D9: A commit belongs to the date `git log` prints for it, in local time

- Decision: generate step 4 dates each commit by the date `git log` shows, also after midnight; work remembered on another day is a self-reported row.
- Why: chosen by the user on 2026-10-24 - one mechanical rule with no cut-off to judge.
- What breaks if ignored: the same late-night commit is dated differently from cycle to cycle.
- Rejected: counting commits before 04:00 toward the previous day with `[needs confirmation]` (proposed; the user chose the commit date). Unverified: `git log` prints the author date by default, not the committer date; the user said "commit date" and the two differ only after a rebase or amend.

### RM-D10: redmy-sync discovers issues and caches hours from the user's own time entries

- Decision: sync step 3 adds issues from `list_time_entries(user_id, from_date=last_synced)`, and step 5 fills "Hours logged (cache, as of <date>)" with the sum of the user's own entries per issue.
- Why: `get_issue` `spent_hours` counts every user on the issue, and `list_issues(assigned_to_id)` misses issues reassigned to or led by a colleague.
- What breaks if ignored: a colleague's hours read as the user's, and an issue the user works on never reaches the registry.
- Rejected: `spent_hours`/`total_spent_hours` (the earlier step 4). The tool has no offset, so a `total_count` above the returned count is reported, not fetched - the same gap as RM-Q3.

### RM-D11: Rules added in 1a84fca live in the SKILL.md steps, not in `rules.md`

- Decision: the SHA exclusion, the commit date and the day cap are written into redmy-generate's steps; `rules.md` only had its step reference updated.
- Why: `rules.md` states that each rule in it was once the cause of a rejected draft, and these were not.
- What breaks if ignored: the file's premise stops holding, and a reader can no longer tell learned rules from designed ones.
- Rejected: adding them as rules 6 and 7 (planned on 2026-10-24, dropped for the reason above).

## Open questions

### RM-Q1: How should the data contract bootstrap in a new project?

The four files `task-registry.md`, `checkpoint.md`, `git-identity-map.md` and `decisions.md` have no creation step and no shipped schema; the formats the skills assume are listed in the sourcecode document. The one place that shows them is the reference implementation in the original working repo, which is not part of this repo (README.md line 32).

| Option | What | Cost |
|---|---|---|
| a | Ship a schema file inside each skill that touches a project file | the same schema is duplicated wherever two skills touch one file, so copies drift |
| b | Each skill creates a missing file itself, with the schema embedded in its text | same duplication; skill text grows; creating local files does not touch the Redmine write gate |
| c | Keep it a documented prerequisite and point to the reference implementation | no code change; a new user cannot follow the pointer because that repo is not part of this one |
| d | Add a fourth skill that owns the schemas and creates the files | one owner for each format; `--skill redmy-sync` alone would not install it, so the README must say to install it first |

Recommended: d, because a single owner stops the formats drifting across three skills. Blocked by RM-Q2, which decides whether a cross-skill file can be installed at all.

### RM-Q2: What does `npx skills add <owner>/RedMy --skill <name>` copy into the project?

Unverified: no installer run has been observed; the command comes from README.md. It matters because, if only the named skill's directory is copied, a file shared across skills or kept at the repo root is not installed, which removes options from RM-Q1. Verification is the observed file listing of an install into an empty scratch project (Phase 3 in roadmap).

### RM-Q3: Does redmy-sync miss assigned issues beyond its fetch limit?

redmy-sync step 2 calls `list_issues(assigned_to_id=<id>, status="*", limit=100)` and no step fetches further pages, so an assignee with more issues than the limit would never see the rest reach `task-registry.md`. Unverified: whether the Redmine tool pages or caps, and whether any user has that many assigned issues. Decision needed: leave as is, or add a paging step to redmy-sync. Since 1a84fca sync steps 3 and 5 also call `list_time_entries` with `limit=100`, a tool with no offset parameter, and report a larger `total_count` instead of fetching it (RM-D10).

### RM-Q4: What format does a skipped row take?

redmy-apply (step 1 and "When a row fails midway") says a skipped row is marked `**Skipped**` in place of `Not logged`. The reference implementation instead moves skipped rows into a separate "Skipped rows" table at the end of the `history/` file, with a Reason column in place of Worklog and Evidence. Generate step 2 and lint check 6 must both find skipped rows, so the skills need one format.

| Option | What | Cost |
|---|---|---|
| a | Separate "Skipped rows" table with a Reason column | skill text changes; the reason for skipping is kept |
| b | `**Skipped**` in the Worklog column, row stays in place | reference implementation data must be migrated; no place for the reason |

Recommended: a. Asked on 2026-10-24; the user asked what the question means, not yet answered.

### RM-Q5: How does redmy-apply correct or remove an entry already logged?

On 2026-10-24 the user chose that the skill itself deletes a wrong entry behind its own gate. Blocked: the MCP `redmine_raw_request` documents DELETE as blocked, and the MCP has no delete tool for time entries.

| Option | What | Cost |
|---|---|---|
| a | Skill reports the entry ids; the user deletes them in the Redmine UI; the skill then updates `history/` | manual step |
| b | Skill corrects fields (hours, date, comment, activity) with `PUT /time_entries/<id>.json` behind its own gate; deletion stays manual as in a | PUT is reachable through `redmine_raw_request`; a correction is still a write to the official record |
| c | Skill deletes through another client calling the REST API directly | a second credential path outside the MCP, and a workaround of a block the MCP sets on purpose |

Recommended: b. Unverified: that PUT on a time entry succeeds with the user's Redmine permissions.
