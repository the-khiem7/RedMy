---
baseline_schema: "2.0"
pack: "redmy"
document: "roadmap"
status: "active"
updated: "2026-10-24"
code_ref: "1a84fca"
---

# RedMy - roadmap

| Phase | Status | Depends on |
|---|---|---|
| Phase 1 - Record the gap | complete | - |
| Phase 2 - README states current behavior | complete | Phase 1 |
| Phase 3 - Verify what the installer copies | planned | - |
| Phase 4 - Close the bootstrap gap | planned | Phase 3, RM-Q1 |
| Phase 5 - Decide redmy-sync paging | planned | - |
| Phase 6 - Harden the Redmine writes | complete | - |
| Phase 7 - Run the hardened skills | planned | Phase 6 |
| Phase 8 - Settle skipped rows and corrections | planned | RM-Q4, RM-Q5 |

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

## Phase 6 - Harden the Redmine writes

- Acceptance: the skill text carries RM-D2 to RM-D11 in hallucination.
- Final evidence: committed as 1a84fca on 2026-10-24 (6 files: the three SKILL.md files, `lint-rules.md`, `apply-gate.md`, `rules.md`). Read back and checked for en and em dashes; nothing was executed. The findings came from an evaluation of the skills against the reference implementation's data on 2026-10-24.

## Phase 7 - Run the hardened skills

- Acceptance: in the reference implementation, redmy-sync runs end to end, redmy-generate drafts rows, and redmy-apply runs up to its gate without logging.
- Verification gate: observed output shows the login check passing, sync hours equal to the user's own entries, no draft citing an already-cited SHA, no date above 8.0h, and apply's duplicate check and lint checks 5 to 8 reporting on real rows. A read of the skill text is not evidence.

## Phase 8 - Settle skipped rows and corrections

- Acceptance: RM-Q4 and RM-Q5 are answered and the skill text follows the answers.
- Verification gate: for RM-Q5 option b, one observed PUT on a time entry with the user's permissions; the skill text alone is not evidence.

## Risks

| Risk | Effect | Status |
|---|---|---|
| README no longer claims a bootstrap but the formats are still unknown to a new user | the first run of redmy-sync has no file to diff against and its behavior is unspecified | open until Phase 4 |
| A schema is copied into several skills | the copies drift | open: RM-Q1 |
| The repo has no `LICENSE` file | reuse terms are undefined | observed 2026-10-13, outside this pack, not scheduled |
| README.md predates 1a84fca: its data-contract table lacks `redmine_user`, and its gate, lint and drafting-rule sections describe the old steps | a reader installs the skills without the field sync and apply now require | open: README update deferred by the user on 2026-10-24 |
| The skill text in 1a84fca was never run | a step can fail against the real MCP tools in a way the text does not show | open until Phase 7 |

## Next action

1. Phase 7: in the reference implementation, run redmy-sync, then redmy-generate, then redmy-apply up to its gate without approving; record what was observed.
2. Phase 8: get the user's answers to RM-Q4 and RM-Q5.
3. Phase 3: in an empty scratch directory outside this repo, run `npx skills add the-khiem7/RedMy --skill redmy-sync`, list every file it placed, and record that listing as the answer to RM-Q2; then answer RM-Q1.
