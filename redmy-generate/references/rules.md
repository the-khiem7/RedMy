# Rules - learned from corrections, do not re-derive from memory

Each rule below was once the cause of the user rejecting a draft for violating it. Read this file every time before drafting.

## 1. Do not fabricate/pad hours when evidence is thin

Mistake that occurred: called a commit that only bumped a submodule reference (2 changed lines) "4 hours of work" with no basis. Rejected twice.

**Correct approach**: if the commit's content (subject + the actual diff, not just the subject) is not clear enough to match the number of hours you intend to assign, investigate further before assigning:
- Read the full diff (`git show`), not just the subject.
- Read neighbouring commits (a few hours, a few days before/after) to find the real work behind a thin "summary" commit.
- If the commit points to a submodule, read that submodule's README/content to understand what the commit represents - but **do not consider the submodule's internal history** to assess effort (see SKILL.md step 5, and the matching entry in `decisions.md`) - use it only to understand the CONTENT; "how many hours is it worth" must still be based on the user's actual effort in the parent repo.
- If there is still not enough basis after investigating: ask the user directly, do not keep guessing a third time.

## 2. Do not leak internal terminology/process in the Redmine Comment

Mistake that occurred: wrote "Cut release branch v1.3.1", "Promote-03 report" in the comment - these are file/process names internal to the working repo alone, meaningless to Redmine readers and they leak unnecessary internal information.

**Correct approach**: describe in purely technical outcomes. Example: instead of "stage Promote-03 report for v1.3.1 release", write "review and validate Terraform changes for production release, verify no secrets staged".

List of words/patterns forbidden in the Redmine Comment (non-exhaustive, used as examples to recognise the kind of information to avoid, not a closed list):
- Internal report file names like `Promote-NN`, `Report-NN`.
- Specific release branch names (`v1.3.1`, `release branch X`).
- Names of internal tools/scripts that have no meaning outside the team.

## 3. Short comment - 1 sentence

Mistake that occurred: wrote comments with multiple clauses, listing every small detail - was asked to shorten ("it being all of text").

**Correct approach**: 1 short sentence stating the main thing that was done. No need to list each small commit if they have been merged into one day row.

## 4. Assigning an issue when ambiguous: look up Redmine first, do not guess

Mistake once made: suspected that a codename ("Project-X") was a separate issue without looking it up. Only after looking up `list_issues`/`list_projects` was it confirmed that the codename is not a Redmine issue and maps to an already-known issue (see the matching entry in `decisions.md`).

**Correct approach**: before creating a new assumption about attribution, look up `list_issues(project_id=...)` or `list_projects` to confirm whether or not an issue matching that name exists.

## 5. Only consider superproject-level commits

A submodule inside a repo may be written entirely by someone else. Do not wade into the submodule's own history to judge the size of the user's work (see the matching entry in `decisions.md`) - only count commits at the level of the repo listed in `task-registry.md`.
