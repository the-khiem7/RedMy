---
baseline_schema: "2.0"
pack: "redmy"
document: "introduction"
status: "active"
updated: "2026-10-24"
code_ref: "3662e83"
---

# RedMy - skill behavior and data contract

## Scope

RedMy is a repository of five agent skills (`redmy-core`, `redmy-sync`, `redmy-generate`, `redmy-apply`, `redmy-board`) that draft Redmine worklogs from git history, log them only after the user approves the exact table, and show a week's logged and draft hours as a board. This pack tracks two things: the skills' dependency on files in the project they are installed into (their formats and how they are created), and the safety of what the skills write to Redmine. Out of scope: Redmine API behavior beyond the tool behavior the skills depend on, and the README illustration work (pack `frontdoor`). In scope from the README: only its data-contract wording (Phase 2 in roadmap).

## Current truth

| Claim | Basis |
|---|---|
| `redmy-core/references/schemas.md` defines every project file; the other skills point to it | decided: RM-D12 in hallucination |
| redmy-core creates the missing files and the frontmatter fields `redmine_user`, `redmine_base_url` and `tracking_since`; sync, generate and apply stop and name redmy-core when one is missing | decided: RM-D12 |
| `history/<issue-id>-<slug>.md` is still created lazily, by redmy-generate step 7 | read: redmy-generate SKILL.md |
| The installer copies only the selected skill's folder | read from installer source, not from a run: RM-D13 |
| `task-registry.md` carries `redmine_user`; sync and apply stop when the MCP is authenticated as anyone else | decided: RM-D2 |
| Apply checks Redmine for an existing entry per row, maps every column to `smart_log_time` including `spent_on`, and writes each worklog link right after its call | decided: RM-D3, RM-D4 |
| Apply lint blocks `[needs confirmation]` rows, closed issues and parent epics, days over 8.0h, future or unconfirmed weekend dates, and duplicate rows | decided: RM-D5, RM-D6 |
| Generate drops commits already cited as evidence, caps a date at 8.0h across issues not counting skipped rows, and dates a commit by its author date | decided: RM-D7, RM-D8, RM-D9 |
| Sync finds issues from the user's own time entries, caches only the user's own hours, and asks for every undetermined repo path | decided: RM-D10, RM-D12 |
| A skipped row stays in place with Worklog `**Skipped**` and its reason in Evidence | decided: RM-D14 |
| A wrong logged entry is deleted by the user from a filtered Redmine link; the skill then resets the row and the checkpoint | decided: RM-D15 |
| Date-window reads of time entries go through `redmine_raw_request` GET with `from`/`to`/`offset` | observed and decided: RM-D16 |
| redmy-board shows one week as a day-by-issue board of logged and draft hours and writes nothing | decided: RM-D17; never run as a skill |
| Sync, generate and apply up to the gate ran against Redmine and the reference implementation on 2026-10-24 | observed: Phase 7 in roadmap |
| README.md describes three skills, no bootstrap, the old gate and lint text, and lacks `redmine_user`, `redmine_base_url`, `tracking_since` and `redmine_raw_request` | observed: README.md unchanged by the user's choice on 2026-10-24 |
| The README no longer claims the skills create the project files on first run | decided: RM-D1 |

## Target

A new project can learn each file's name, columns and creation rule from the installed skills alone, and no document in the repo claims behavior the skills lack. Every entry the skills write is approved, logged once, on its own date, as the right user, and a wrong entry can be corrected through the skill. Remaining: Phase 4 verification in a project with no reference implementation, and the README.

## Constraints

- redmy-sync is read-only against Redmine and only redmy-apply may call `smart_log_time` or `create_time_entry`; redmy-core creates local files only. Sync and apply call `redmine_raw_request` with GET only.
- A skill can rely only on files inside its own folder, plus a sibling skill the user installed: RM-D13.
- `smart_log_time` defaults `spent_on` to today when it is omitted; `list_time_entries` ignores `from_date`/`to_date` and has no offset: RM-D16.
- The MCP `redmine_raw_request` blocks DELETE, and the MCP has no tool that deletes a time entry: RM-D15.
- The reference implementation of the contract is the user's private worklog repo; its data never enters this repo, and schemas.md uses made-up values.
