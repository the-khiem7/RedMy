---
baseline_schema: "2.0"
pack: "redmy"
document: "introduction"
status: "active"
updated: "2026-10-24"
code_ref: "1a84fca"
---

# RedMy - skill behavior and data contract

## Scope

RedMy is a repository of three agent skills (`redmy-sync`, `redmy-generate`, `redmy-apply`) that draft Redmine worklogs from git history and log them only after the user approves the exact table. This pack tracks two things: the debt in the skills' dependency on files in the project they are installed into (the formats and creation of those files are not defined in this repo), and the safety of what the skills write to Redmine. Out of scope: Redmine API behavior beyond the tool behavior the skills depend on, and the README illustration work (pack `frontdoor`). In scope from the README: only its data-contract wording (Phase 2 in roadmap).

## Current truth

| Claim | Basis |
|---|---|
| The skills ship process logic only, no data and no schema files for the project files | read: README.md and the skill folders |
| The installing project is expected to hold `task-registry.md`, `checkpoint.md`, `git-identity-map.md`, `decisions.md`, `history/<issue-id>-<slug>.md` and a Redmine base URL (`<REDMINE_BASE_URL>`) | read: README.md and the skill steps; the file-by-skill matrix is in sourcecode |
| Only `history/<issue-id>-<slug>.md` has a creation step, in redmy-generate step 7, lazily | read: redmy-generate SKILL.md |
| `task-registry.md` carries a `redmine_user` frontmatter field; sync and apply stop when the MCP is authenticated as anyone else | decided: RM-D2 in hallucination |
| Apply checks Redmine for an existing entry per row, maps every column to `smart_log_time` including `spent_on`, and writes each worklog link right after its call | decided: RM-D3, RM-D4 |
| Apply lint blocks `[needs confirmation]` rows, closed issues and parent epics, days over 8.0h, future or unconfirmed weekend dates, and duplicate rows | decided: RM-D5, RM-D6 |
| Generate drops commits already cited as evidence, caps a date at 8.0h across issues, and dates a commit as `git log` prints it | decided: RM-D7, RM-D8, RM-D9 |
| Sync finds issues from the user's own time entries and caches only the user's own hours | decided: RM-D10 |
| None of the changes in 1a84fca has been run against Redmine | observed: Phase 7 in roadmap is planned |
| README.md still describes the gate, lint and drafting rules and the data contract as they were before 1a84fca | observed: README.md unchanged by the user's choice on 2026-10-24 |
| The README no longer claims the skills create the project files on first run; the skill steps never supported that for the four files other than `history/` | decided: RM-D1 in hallucination |
| The column layouts the skills assume are never shown in this repo | read: listed in sourcecode |
| `npx skills add <owner>/RedMy --skill <name>` installs a skill | unverified: RM-Q2 in hallucination |

## Target

A new project can learn each file's name, columns and creation rule from the installed skills alone, and no document in the repo claims behavior the skills lack. The path to that is open: RM-Q1 in hallucination. Every entry the skills write is approved, logged once, on its own date, as the right user, and a wrong entry can be corrected through the skill: RM-Q5 in hallucination.

## Constraints

- redmy-sync is read-only against Redmine and only redmy-apply may call `smart_log_time` or `create_time_entry`; any bootstrap fix creates local files only and leaves that gate untouched.
- The reference implementation of the contract lives in the original working repo (README.md line 32), whose location this repo does not record; its formats cannot be copied from here.
- Where a schema may live depends on what the installer copies: unverified, RM-Q2 in hallucination.
- `smart_log_time` defaults `spent_on` to today when it is omitted, and `list_time_entries` has no offset parameter.
- The MCP `redmine_raw_request` blocks DELETE, and the MCP has no tool that deletes a time entry: RM-Q5 in hallucination.
