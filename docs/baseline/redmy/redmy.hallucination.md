---
baseline_schema: "2.0"
pack: "redmy"
document: "hallucination"
status: "active"
updated: "2026-10-24"
code_ref: "cb434e7"
---

# RedMy - decisions and open questions

## Entry index

| ID | About | Status | Related |
|---|---|---|---|
| RM-D1 | README claim that the skills create the project data files on first run | closed - disproven | RM-D12 |
| RM-Q3 | Whether redmy-sync misses assigned issues beyond its fetch limit | open | RM-D10, RM-D16 |
| RM-D2 | Redmine login check before any skill reads or writes for the user | closed | RM-D10, RM-D12 |
| RM-D3 | Duplicate-entry guard in redmy-apply | closed | RM-D4, RM-D15, RM-D16 |
| RM-D4 | Explicit mapping of a row to `smart_log_time` parameters | closed | RM-D3 |
| RM-D5 | `[needs confirmation]` rows at the apply gate | closed | RM-D6 |
| RM-D6 | Lint checks 5 to 8 in redmy-apply | closed | RM-D5, RM-D8 |
| RM-D7 | Excluding commits already cited as evidence | closed | - |
| RM-D8 | Daily hour cap across issues in redmy-generate | closed | RM-D6, RM-D14 |
| RM-D9 | Which date a commit belongs to | closed | - |
| RM-D10 | Source of issue discovery and cached hours in redmy-sync | closed | RM-D2, RM-Q3, RM-D16 |
| RM-D11 | Where the rules added in 1a84fca live | closed | - |
| RM-D12 | redmy-core owns the data contract and bootstraps a new project | closed | RM-Q1, RM-D1, RM-D13 |
| RM-D13 | What the installer copies for one skill | closed | RM-Q2, RM-D12 |
| RM-D14 | Format of a skipped row | closed | RM-Q4, RM-D8 |
| RM-D15 | How a logged entry is corrected or removed | closed | RM-Q5, RM-D3 |
| RM-D16 | `list_time_entries` ignores its date filters | closed - lesson | RM-D3, RM-D10, RM-Q3 |
| RM-D17 | A read-only weekly board skill, redmy-board | closed | RM-D16, RM-D15 |
| RM-D18 | Approving part of an apply table | closed | RM-D3, RM-D5 |

## Closed decisions

### RM-D1: The skills do not create the project data files on first run

- Believed: README.md (at 74bd929, lines 23-24) says the installing project must provide the data files "or let `redmy-sync`/`redmy-generate` create on first run".
- Disproved by: redmy-sync reads `task-registry.md` (step 3) and appends to `decisions.md` (step 6); redmy-generate reads `checkpoint.md` (step 1), reads and adds rows to `git-identity-map.md` (step 2), and reads `decisions.md` (step 4). None has a step that creates those four files or defines their columns. The one file a skill creates is `history/<issue-id>-<slug>.md`, by redmy-generate step 6, lazily when an issue first has real work hours.
- Decision: no document asserts the claim; the README states what the skills do today and the gap stays recorded here until RM-Q1 is answered. RM-Q1 was answered by RM-D12: since 77bc40d the skill redmy-core creates the four files.
- What breaks if ignored: a reader installs the skills into an empty project expecting a bootstrap and runs skills that read files that do not exist; what the skill does then is not specified anywhere.

### RM-D2: Check the Redmine login against `redmine_user` before acting

- Decision: redmy-apply step 0 and redmy-sync step 1 compare `get_current_user` `login` with `redmine_user` in the `task-registry.md` frontmatter and stop on a mismatch. Since 77bc40d redmy-core writes the field once after the user confirms it (RM-D12); sync and apply stop and point to redmy-core when it is missing.
- Why: the MCP acts as whoever owns its API key, and the reference implementation records pair-work days on a colleague's machine.
- What breaks if ignored: hours land in another person's Redmine record, and sync fills the registry with that person's issues.
- Rejected: comparing against the git author names in `git-identity-map.md` (git names differ per repo and are not Redmine logins); no check (the earlier behavior); sync writing the field itself (1a84fca, replaced so one skill owns the frontmatter).

### RM-D3: redmy-apply guards against logging a row twice

- Decision: before presenting, apply queries Redmine for each row's issue, user and date and marks any hit as a possible duplicate that is logged only on approval by name; after approval it logs one row at a time and writes each worklog link to `history/` right after its call succeeds. Since b2b9c8c the query is `redmine_raw_request` GET `/time_entries.json` with `from`/`to` (RM-D16); since the user approved it on 2026-10-24 it is one query per table, from the earliest to the latest row date, matched by issue and `spent_on`.
- Why: a row logged right before an interrupted session still reads `Not logged` on disk, so the next run logs it again; the reference implementation's registry carries 9.0h of duplicate entries on one issue awaiting deletion.
- What breaks if ignored: duplicate time entries that only a manual delete in Redmine removes (RM-D15).
- Rejected: writing all links after the last call (the earlier step 5, loses the record of every row logged before an interruption); matching existing entries by comment text (a comment can be edited on Redmine, so a real duplicate would pass); `list_time_entries` with `from_date`/`to_date` (1a84fca, the tool ignores the dates - RM-D16); one query per row (b2b9c8c, as many calls as rows for the same answer).

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

- Decision: generate step 6 keeps the sum of a date's hours, existing rows plus new drafts and not counting `**Skipped**` rows, at 8.0 or less; a short day stays short and is asked about; a confirmed weekend is written into Evidence. Hours the user asks to add are the user's own account: they go on the row the user names, or the issue with most of that day's commits, and Evidence ends with `filled to <h> for the day at the user's request on <date>` (rule 1 in `rules.md`, schemas.md). Approved by the user on 2026-10-24, after a run where the user asked to fill two short days and the skill had no way to record it.
- Why: "balance at ~8h per working day" pushed drafts toward filling the day, which contradicts rule 1 of `rules.md` (no padding on thin evidence), and did not say whether 8h was per issue or per day.
- What breaks if ignored: padded hours, or two issues each drafted to 8h on one day.
- Rejected: balancing each working day to about 8h (the earlier step 5); a fill with no Evidence marker (a later reader cannot tell the user's account from agent padding).

### RM-D9: A commit belongs to the author date `git log` prints for it, in local time

- Decision: generate step 4 dates each commit by the date `git log` shows, which is the author date, also after midnight; work remembered on another day is a self-reported row.
- Why: chosen by the user on 2026-10-24 - one mechanical rule with no cut-off to judge; the user confirmed the author date on 2026-10-24 (it differs from the committer date only after a rebase or amend).
- What breaks if ignored: the same late-night commit is dated differently from cycle to cycle.
- Rejected: counting commits before 04:00 toward the previous day with `[needs confirmation]` (proposed; the user chose the commit date); the committer date (not what `git log` shows by default).

### RM-D10: redmy-sync discovers issues and caches hours from the user's own time entries

- Decision: sync step 3 adds issues from the user's time entries since `last_synced` (`tracking_since` on the first sync), and step 5 fills "Hours logged (cache, as of <date>)" with the sum of the user's own entries per issue; an epic shows only entries logged on it directly. Since b2b9c8c both read `redmine_raw_request` GET `/time_entries.json`, paged by `offset` (RM-D16).
- Why: `get_issue` `spent_hours` counts every user on the issue, and `list_issues(assigned_to_id)` misses issues reassigned to or led by a colleague.
- What breaks if ignored: a colleague's hours read as the user's, and an issue the user works on never reaches the registry.
- Rejected: `spent_hours`/`total_spent_hours` (the earlier step 4); `list_time_entries(user_id, from_date=last_synced)` (1a84fca: it ignores the date and has no offset - RM-D16).

### RM-D11: Rules added in 1a84fca live in the SKILL.md steps, not in `rules.md`

- Decision: the SHA exclusion, the commit date and the day cap are written into redmy-generate's steps; `rules.md` only had its step reference updated.
- Why: `rules.md` states that each rule in it was once the cause of a rejected draft, and these were not.
- What breaks if ignored: the file's premise stops holding, and a reader can no longer tell learned rules from designed ones.
- Rejected: adding them as rules 6 and 7 (planned on 2026-10-24, dropped for the reason above).

### RM-D12: A fourth skill, redmy-core, owns the data contract and bootstraps a new project

Closes RM-Q1.

- Decision: `redmy-core/references/schemas.md` is the only definition of `task-registry.md`, `checkpoint.md`, `git-identity-map.md`, `decisions.md` and `history/*.md`; the other skills point to it. redmy-core creates missing files without touching existing ones, confirms the Redmine login, and asks for `redmine_base_url` and `tracking_since`, written to the registry frontmatter. Sync, generate and apply stop and name redmy-core when a file or field is missing. Chosen by the user on 2026-10-24, name included.
- Why: one owner per format stops copies drifting, and the installer copies only the selected skill's folder (RM-D13), so a shared file must live in a skill. The same read found three gaps that made an empty project unworkable even with the files present:
  - no skill filled Repo/Source - sync left it `*(not yet determined)*` and generate scanned nothing for such an issue, silently; now sync step 8 asks for each path (or `none`) and generate reports unscanned issues;
  - an issue with no checkpoint row had no window start, so generate could read the whole git history and draft days already logged by hand; now it starts at `tracking_since`;
  - `<REDMINE_BASE_URL>` had no source in any project file (the reference implementation kept it in its README); now it is `redmine_base_url`.
- What breaks if ignored: a new project has no way to learn the formats, generate drafts nothing or drafts the whole history, and apply cannot build a worklog link.
- Rejected:

| Option | What | Why it lost |
|---|---|---|
| a | Ship a schema file inside each skill that touches a project file | the same schema duplicated wherever two skills touch one file, so copies drift |
| b | Each skill creates a missing file itself, with the schema embedded in its text | same duplication; skill text grows |
| c | Keep it a documented prerequisite and point to the reference implementation | a new user cannot follow the pointer, that repo is not part of this one |

### RM-D13: `npx skills add --skill <name>` copies only that skill's folder

Closes RM-Q2.

- Decision: treat each skill folder as the unit of installation; anything a skill needs at run time lives in its own folder, and a cross-skill dependency is named with its install command.
- Basis: read from the installer source, `skills@1.7.4` `dist/cli.mjs`, on 2026-10-24: a repo with no skills in the standard locations is searched recursively for `SKILL.md` (depth 5); `installSkillForAgent` copies `dirname(SKILL.md)` recursively, skipping `.git`, `__pycache__`, `__pypackages__` and `metadata.json`, into `<project>/.agents/skills/<name>` and links `.claude/skills/<name>` to it. Repo-root files such as `docs/` and README.md are never copied. Not observed: a real install into a scratch project was refused by the session's permission classifier (code from an external source), so this rests on reading the code, not on a run.
- What breaks if ignored: a skill that reads a file outside its folder, or a sibling skill that was not installed, fails only in an installed project, never in this repo.
- Rejected: assuming the installer copies the whole repo (the code copies one folder per skill).

### RM-D14: A skipped row stays in place with Worklog `**Skipped**`; its reason goes in Evidence

Closes RM-Q4.

- Decision: Worklog `**Skipped**`, the row keeps its date position in the one table, and Evidence ends with `skipped: <reason>`. The reference implementation's two "Skipped rows" tables were migrated to this form on 2026-10-24. Chosen by the user on 2026-10-24.
- Why: generate step 2 and step 6 and apply lint checks 6 and 8 all scan one table per file; the reason fits in Evidence, which removes the cost recorded against this option when it was proposed.
- What breaks if ignored: a skipped row in a second table is missed by the day total or the SHA exclusion, or counted when it should not be.
- Rejected: a separate "Skipped rows" table with a Reason column (recommended earlier; a second table every scan must know about).

### RM-D15: A logged entry is removed by the user in Redmine; the skill gives the link and updates the files

Closes RM-Q5.

- Decision: redmy-apply "Correcting a logged row" shows the entry, gives a delete link `<redmine_base_url>/time_entries?set_filter=1&issue_id=<issue>&spent_on=<Date>&user_id=me` and the inspect link `/time_entries/<id>/edit`, confirms the deletion with the step 2 query, then resets the `history/` row to `Not logged` (to re-log through the gate) or `**Skipped**` and recomputes the checkpoint. A correction of fields is a delete plus a re-log. Chosen by the user on 2026-10-24, with the delete link added at their request.
- Why: the MCP `redmine_raw_request` blocks DELETE and no MCP tool deletes a time entry; the user's first choice, the skill deleting behind its own gate, is not possible.
- Verified: the same filter on the JSON API (`/time_entries.json?set_filter=1&issue_id=198219&spent_on=2026-10-19&user_id=me`) returned exactly entry 282431 on 2026-10-24. Not observed: the HTML page and its delete button.
- What breaks if ignored: a wrong entry stays on Redmine while `history/` and the checkpoint claim otherwise.
- Rejected: the skill correcting fields with `PUT /time_entries/<id>.json` behind its own gate (recommended earlier; the user chose that every change to an existing entry stays their own act); deleting through another client calling the REST API (a second credential path and a workaround of a block the MCP sets on purpose).

### RM-D16: `list_time_entries` ignores `from_date` and `to_date`

- Lesson: RM-D3 and RM-D10 (1a84fca) were written assuming the MCP tool filters by date, because its signature documents both parameters. The first run of redmy-sync on 2026-10-24 asked for entries since 2026-10-19 and got all 97 of the user's entries back to 2026-07-22; with `issue_id=198192` and both dates set to 2026-10-19 it returned all 9 entries of the issue. `issue_id` and `user_id` do filter. The same query through `redmine_raw_request` GET `/time_entries.json` with `from`/`to` returned the one entry of that day, and `offset` paged correctly (`limit=2, offset=95` returned entries 96-97 of 97).
- Decision: every date-window read of time entries uses `redmine_raw_request` GET with `from`, `to`, `limit` and `offset` (b2b9c8c); sync and apply may call it with GET only.
- What breaks if ignored: apply step 2 marks every draft row a possible duplicate, and sync reads the user's whole history instead of the window.
- Rejected: filtering the `list_time_entries` result by `spent_on` in the agent (works only while all entries fit in one page of 100, and the tool has no offset).

### RM-D17: redmy-board shows one week as a read-only board

- Decision: a fifth skill renders Monday to Sunday as a day-by-issue table of logged (from Redmine) and draft (from `history/*.md`) hours, with a fixed legend, a status list and the week total if every draft is approved; it reconciles entry ids against worklog links and marks days with commits but no row. It writes nothing. Requested by the user on 2026-10-24 after the same board was shown by hand.
- Why: no skill answered where a week stands without moving the cycle forward - generate appends rows and apply leads to the write gate; a fixed shape keeps the board the same from week to week.
- What breaks if ignored: the board depends on the agent improvising it, and a mismatch such as an entry on Redmine whose row is Skipped is found only by chance.
- Rejected: a board step inside redmy-sync (sync writes `task-registry.md`, so the board would no longer be safe to ask for at any time); a board step inside redmy-apply (it would sit in front of the write gate, where a request to look turns into a request to log).

### RM-D18: The apply table may be approved in part, by day or row

- Decision: redmy-apply step 5 logs exactly the days or rows the user's answer names, leaves the rest `Not logged`, and presents the rest again, as a new table when the answer changed any of its rows; a row the answer does not name is not approved. The gate wording in `apply-gate.md` is unchanged. Approved by the user on 2026-10-24, after answering a gate with "Tue: approve" plus instructions for two other days.
- Why: the step read as one approval for the whole table, so a per-day answer left the agent to guess whether the unnamed rows were approved.
- What breaks if ignored: a row the user never named is logged, or an approved day waits for rows still under discussion.
- Rejected: requiring one approval for the whole table (a single disputed row blocks every other row); treating silence on a row as approval (logs a row the user never saw approved).

## Open questions

### RM-Q3: Does redmy-sync miss assigned issues beyond its fetch limit?

redmy-sync step 2 calls `list_issues(assigned_to_id=<id>, status="*", limit=100)` and no step fetches further pages, so an assignee with more issues than the limit would never see the rest reach `task-registry.md`. Observed on 2026-10-24: `list_issues` takes an `offset` parameter, and the user had 10 assigned issues. The time-entry reads no longer have this gap (RM-D16). Decision needed: leave step 2 as is, or page it by `offset` until `total_count` is read.
