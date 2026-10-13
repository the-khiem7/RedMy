---
baseline_schema: "2.0"
pack: "redmy"
document: "roadmap"
status: "active"
updated: "2026-10-13"
code_ref: "74bd929"
---

# RedMy - roadmap

| Phase | Status | Depends on |
|---|---|---|
| Phase 1 - Record the gap | complete | - |
| Phase 2 - README states current behavior | complete | Phase 1 |
| Phase 3 - Verify what the installer copies | planned | - |
| Phase 4 - Close the bootstrap gap | planned | Phase 3, RM-Q1 |
| Phase 5 - Decide redmy-sync paging | planned | - |

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

## Phase 4 - Close the bootstrap gap

- Acceptance: each of the four files and the history table can be learned from the installed skills alone, by the option chosen for RM-Q1.
- Verification gate: install the skills into an empty scratch project and run redmy-sync there without access to the reference implementation, then record which files it needed and what it did; redmy-sync is read-only against Redmine. A read of the edited skill text is not evidence of that behavior.

## Phase 5 - Decide redmy-sync paging

- Acceptance: RM-Q3 is answered, and if a paging step is added the fetch no longer depends on a single page.
- Verification gate: observed behavior of the Redmine list tool against a result larger than the limit, not the skill text.

## Risks

| Risk | Effect | Status |
|---|---|---|
| README no longer claims a bootstrap but the formats are still unknown to a new user | the first run of redmy-sync has no file to diff against and its behavior is unspecified | open until Phase 4 |
| A schema is copied into several skills | the copies drift | open: RM-Q1 |
| The repo has no `LICENSE` file | reuse terms are undefined | observed 2026-10-13, outside this pack, not scheduled |

## Next action

Phase 3: in an empty scratch directory outside this repo, run `npx skills add the-khiem7/RedMy --skill redmy-sync`, list every file it placed, and record that listing as the answer to RM-Q2; then answer RM-Q1.
