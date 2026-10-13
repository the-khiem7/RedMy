---
baseline_schema: "2.0"
pack: "redmy"
document: "hallucination"
status: "active"
updated: "2026-10-13"
code_ref: "37d69c3"
---

# RedMy - decisions and open questions

## Entry index

| ID | About | Status | Related |
|---|---|---|---|
| RM-D1 | README claim that the skills create the project data files on first run | closed - disproven | RM-Q1 |
| RM-Q1 | How the data contract bootstraps in a new project | open | RM-D1, RM-Q2 |
| RM-Q2 | What `npx skills add --skill` copies into the project | open | RM-Q1 |
| RM-Q3 | Whether redmy-sync misses assigned issues beyond its fetch limit | open | - |

## Closed decisions

### RM-D1: The skills do not create the project data files on first run

- Believed: README.md (at 74bd929, lines 23-24) says the installing project must provide the data files "or let `redmy-sync`/`redmy-generate` create on first run".
- Disproved by: redmy-sync reads `task-registry.md` (step 3) and appends to `decisions.md` (step 6); redmy-generate reads `checkpoint.md` (step 1), reads and adds rows to `git-identity-map.md` (step 2), and reads `decisions.md` (step 4). None has a step that creates those four files or defines their columns. The one file a skill creates is `history/<issue-id>-<slug>.md`, by redmy-generate step 6, lazily when an issue first has real work hours.
- Decision: no document asserts the claim; the README states what the skills do today and the gap stays recorded here until RM-Q1 is answered.
- What breaks if ignored: a reader installs the skills into an empty project expecting a bootstrap and runs skills that read files that do not exist; what the skill does then is not specified anywhere.

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

redmy-sync step 2 calls `list_issues(assigned_to_id=<id>, status="*", limit=100)` and no step fetches further pages, so an assignee with more issues than the limit would never see the rest reach `task-registry.md`. Unverified: whether the Redmine tool pages or caps, and whether any user has that many assigned issues. Decision needed: leave as is, or add a paging step to redmy-sync.
