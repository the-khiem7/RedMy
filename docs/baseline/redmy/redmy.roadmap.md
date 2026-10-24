---
baseline_schema: "2.0"
pack: "redmy"
document: "roadmap"
status: "active"
updated: "2026-10-24"
code_ref: "3662e83"
---

# RedMy - roadmap

| Phase | Status | Depends on |
|---|---|---|
| Phase 1 - Record the gap | complete | - |
| Phase 2 - README states current behavior | complete | Phase 1 |
| Phase 3 - Verify what the installer copies | complete by code read, run not observed | - |
| Phase 4 - Close the bootstrap gap | implemented, verification open | Phase 3, RM-D12 |
| Phase 5 - Decide redmy-sync paging | in progress | RM-Q3 |
| Phase 6 - Harden the Redmine writes | complete | - |
| Phase 7 - Run the hardened skills | complete | Phase 6 |
| Phase 8 - Settle skipped rows and corrections | complete | RM-D14, RM-D15 |
| Phase 9 - Weekly board | implemented, verification open | RM-D17 |

## Phase 1 - Record the gap

- Acceptance: the claim and what disproved it, the file-by-skill matrix and the assumed formats are written down.
- Final evidence: the three SKILL.md files, `redmy-generate/references/rules.md`, `redmy-apply/references/apply-gate.md`, `redmy-apply/references/lint-rules.md` and README.md were read in full on 2026-10-13 at code_ref 74bd929. No skill was executed, no Redmine call was made, and `npx skills` was not run.

## Phase 2 - README states current behavior

- Acceptance: the README's data-contract section does not say the skills create `task-registry.md`, `checkpoint.md`, `git-identity-map.md` or `decisions.md`, and says redmy-generate creates the `history/` files lazily; the reasoning is RM-D1 in hallucination.
- Verification gate: a text search of README.md for "create on first run" finds nothing, and each remaining data-contract sentence matches the matrix in sourcecode.
- Final evidence: README.md was rewritten on 2026-10-13; a text search for "create on first run" finds nothing, its data-contract section says only `history/<issue-id>-<slug>.md` is created by a skill, and its file table matches the matrix in sourcecode. Not observed: the README rendered on github.com.

## Phase 3 - Verify what the installer copies

- Acceptance: the file listing of `npx skills add the-khiem7/RedMy --skill redmy-sync` into an empty scratch project is recorded, which answers RM-Q2.
- Verification gate: the listing is observed output from a real install, not inferred from the repo layout.
- Final evidence: answered from the installer source `skills@1.7.4` on 2026-10-24 (RM-D13 in hallucination). The gate is not met: the real install was refused by the session's permission classifier as code from an external source.

## Phase 4 - Close the bootstrap gap

- Acceptance: each of the four files and the history table can be learned from the installed skills alone, by the option chosen for RM-Q1 (RM-D12: redmy-core).
- Verification gate: install the skills into an empty scratch project and run redmy-core and redmy-sync there without access to the reference implementation, then record which files they created and what they asked; redmy-sync is read-only against Redmine. A read of the edited skill text is not evidence of that behavior.
- Evidence so far: redmy-core and `references/schemas.md` committed as 77bc40d on 2026-10-24; the reference implementation was migrated to the schema by hand (frontmatter `redmine_base_url` and `tracking_since: 2026-10-20`, skipped rows inline). redmy-core itself has not run.

## Phase 5 - Decide redmy-sync paging

- Acceptance: RM-Q3 is answered, and if a paging step is added the fetch no longer depends on a single page.
- Verification gate: observed behavior of the Redmine list tool against a result larger than the limit, not the skill text.
- Evidence so far: time-entry reads page by `offset` through `redmine_raw_request` since b2b9c8c, observed on 2026-10-24 (RM-D16). Open: `list_issues` in sync step 2.

## Phase 6 - Harden the Redmine writes

- Acceptance: the skill text carries RM-D2 to RM-D11 in hallucination.
- Final evidence: committed as 1a84fca on 2026-10-24 (6 files: the three SKILL.md files, `lint-rules.md`, `apply-gate.md`, `rules.md`). Read back and checked for en and em dashes; nothing was executed. The findings came from an evaluation of the skills against the reference implementation's data on 2026-10-24.

## Phase 7 - Run the hardened skills

- Acceptance: in the reference implementation, redmy-sync runs end to end, redmy-generate drafts rows, and redmy-apply runs up to its gate without logging.
- Verification gate: observed output shows the login check passing, sync hours equal to the user's own entries, no draft citing an already-cited SHA, no date above 8.0h, and apply's duplicate check and lint checks 5 to 8 reporting on real rows. A read of the skill text is not evidence.
- Final evidence, observed on 2026-10-24 against 77bc40d, then b2b9c8c:
  - Login check passed in sync and apply.
  - Sync: 10 assigned issues, none new, no status or subject change. Hours are the user's own entries (an issue with a colleague: 44.0h own against 92.0h `spent_hours`). Defect found and fixed: `list_time_entries` ignores dates (RM-D16), fixed in b2b9c8c.
  - Generate: cited SHAs expanded per repo (81, 65 and 6); 7 draft rows on 3 issues for 2026-10-20 to 2026-10-23, none citing an already-cited SHA; highest day total 8.0h; commits of Saturday 2026-10-24 were not drafted and are asked about.
  - Apply: duplicate query returned no entry for any row; lint check 3 blocks 6 rows with `[needs confirmation]`; checks 5 to 8 fail none. Stopped at the gate; nothing was logged.
  - Data finding: entry 280483 (8.0h on 2026-09-21) is on Redmine while the reference implementation marks that row Skipped.

## Phase 8 - Settle skipped rows and corrections

- Acceptance: RM-Q4 and RM-Q5 are answered and the skill text follows the answers.
- Final evidence: answered by the user on 2026-10-24 (RM-D14, RM-D15 in hallucination) and committed as 77bc40d. The delete-link filter was verified on the JSON API; the HTML page with its delete button was not observed.

## Phase 9 - Weekly board

- Acceptance: redmy-board renders the week in the shape RM-D17 fixes and lists every mismatch between Redmine entries and `history/` rows.
- Verification gate: one observed run of the skill in the reference implementation, with the logged column matching the Redmine entries of the week. The hand-made board of 2026-10-24 that the skill was modeled on is not evidence of the skill.
- Evidence so far: committed as 3662e83 on 2026-10-24.

## Risks

| Risk | Effect | Status |
|---|---|---|
| README.md predates 1a84fca and 77bc40d: three skills, no redmy-core or redmy-board, no `redmine_user`/`redmine_base_url`/`tracking_since`, no `redmine_raw_request` in the requirements, old gate and lint text | a reader installs the skills without redmy-core and without the fields every skill now requires | open: README update deferred by the user on 2026-10-24 |
| redmy-core has never run | a step can fail in an empty project in a way the text does not show | open until Phase 4 |
| redmy-board has never run | its reconciliation or its undrafted-day check can miss or invent a mismatch | open until Phase 9 |
| SHA exclusion sees only SHAs; a logged row whose Evidence is "Redmine time entry" protects none of its commits | an issue whose window starts before such rows can re-draft their commits; observed: 36 uncited Atlantic commits from 2026-08-28 to 2026-10-06 sit on already-logged days, held back only by the per-issue windows, Constraints and the day cap | open, not scheduled |
| The repo has no `LICENSE` file | reuse terms are undefined | observed 2026-10-13, outside this pack, not scheduled |

## Next action

1. Phase 7 follow-up: the user answers the apply gate for the 7 draft rows, the 3 undetermined repo paths and the Saturday 2026-10-24 work, and decides entry 280483 through the correction flow.
2. Phase 4: in an empty scratch directory with no access to the reference implementation, install or copy the four skills, run redmy-core then redmy-sync, and record what was created and asked.
3. Phase 5: get the user's answer to RM-Q3 for `list_issues`.
4. Phase 9: run redmy-board for the week of 2026-10-19 in the reference implementation and compare its logged column with Redmine.
