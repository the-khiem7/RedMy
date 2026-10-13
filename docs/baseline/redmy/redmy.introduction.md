---
baseline_schema: "2.0"
pack: "redmy"
document: "introduction"
status: "active"
updated: "2026-10-13"
code_ref: "74bd929"
---

# RedMy - data-contract tech debt

## Scope

RedMy is a repository of three agent skills (`redmy-sync`, `redmy-generate`, `redmy-apply`) that draft Redmine worklogs from git history and log them only after the user approves the exact table. The skills depend on files in the project they are installed into; this pack tracks the debt in that dependency: the formats and creation of those files are not defined in this repo. Out of scope: Redmine API behavior and the README illustration work. In scope from the README: only its data-contract wording (Phase 2 in roadmap).

## Current truth

| Claim | Basis |
|---|---|
| The skills ship process logic only, no data and no schema files for the project files | read: README.md and the skill folders |
| The installing project is expected to hold `task-registry.md`, `checkpoint.md`, `git-identity-map.md`, `decisions.md`, `history/<issue-id>-<slug>.md` and a Redmine base URL (`<REDMINE_BASE_URL>`) | read: README.md and the skill steps; the file-by-skill matrix is in sourcecode |
| Only `history/<issue-id>-<slug>.md` has a creation step, in redmy-generate step 6, lazily | read: redmy-generate SKILL.md |
| The README no longer claims the skills create the project files on first run; the skill steps never supported that for the four files other than `history/` | decided: RM-D1 in hallucination |
| The column layouts the skills assume are never shown in this repo | read: listed in sourcecode |
| `npx skills add <owner>/RedMy --skill <name>` installs a skill | unverified: RM-Q2 in hallucination |

## Target

A new project can learn each file's name, columns and creation rule from the installed skills alone, and no document in the repo claims behavior the skills lack. The path to that is open: RM-Q1 in hallucination.

## Constraints

- redmy-sync is read-only against Redmine and only redmy-apply may call `smart_log_time` or `create_time_entry`; any bootstrap fix creates local files only and leaves that gate untouched.
- The reference implementation of the contract lives in the original working repo (README.md line 32), whose location this repo does not record; its formats cannot be copied from here.
- Where a schema may live depends on what the installer copies: unverified, RM-Q2 in hallucination.
