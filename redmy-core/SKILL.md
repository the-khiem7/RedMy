---
name: redmy-core
description: Own the data contract of the RedMy skills and create it in a new project - the format of task-registry.md, checkpoint.md, git-identity-map.md, decisions.md and history/*.md. Run it once before the first redmy-sync, and whenever another redmy skill stops on a missing file or frontmatter field. Writes local files only, never writes to Redmine.
---

# Redmine Worklog - Core (Data Contract and Setup)

`references/schemas.md` in this skill is the only definition of the project files the redmy skills read and write. `redmy-sync`, `redmy-generate` and `redmy-apply` point here instead of carrying their own copy. Reason: a schema copied into three skills drifts, and the installer copies each skill's folder alone, so a file outside a skill folder never reaches the project.

This skill never calls a Redmine write tool. It reads `get_current_user` and creates or completes local files.

## Steps

1. Check which of these exist at the project root: `task-registry.md`, `checkpoint.md`, `git-identity-map.md`, `decisions.md`, `history/`. Report what exists before writing anything. Never overwrite, reorder or reformat an existing file - only add a missing file or a missing frontmatter field.
2. Call `get_current_user`, show its `login`, and ask the user to confirm it is their own account. Reason: the MCP acts as whoever owns its API key, and `redmy-sync` and `redmy-apply` compare against this value before every run.
3. Ask the user for each value below that is not already in the `task-registry.md` frontmatter. Never infer one from memory or from another project.
   - `redmine_base_url`: the root URL of their Redmine, with no trailing slash. The issue and worklog links are built from it.
   - `tracking_since`: the first date a row may be drafted for an issue that has no logged row yet. Reason: without it the first `redmy-generate` reads the whole git history, and drafts days the user already logged by hand before using the skills.
4. Create each missing file exactly as `references/schemas.md` shows it, with the values from steps 2 and 3, and no table rows. Leave `last_synced` out - `redmy-sync` writes it. Do not create `history/` files: `redmy-generate` creates one the first time an issue has real work hours.
5. Hand over to `redmy-sync` to fill the registry. Tell the user that sync will then ask for the local repo path of each issue.

## Do not

- Do not call `smart_log_time`, `create_time_entry`, `create_issue` or `update_issue`.
- Do not copy rows, decisions or identities from another project into a new one - every value in these files belongs to one user and one Redmine.
- Do not change a format in another skill's text. A format change is made in `references/schemas.md` only.
