# RedMy

Redmine worklog automation skills, installable via [`npx skills`](https://skills.sh).

```
npx skills add <owner>/RedMy --skill redmy-sync
npx skills add <owner>/RedMy --skill redmy-generate
npx skills add <owner>/RedMy --skill redmy-apply
```

(or `npx skills add <owner>/RedMy --all` for all three)

## Skills

| Skill | What it does |
|---|---|
| `redmy-sync` | Refreshes a project's `task-registry.md` against real Redmine data (new/renamed/closed assigned issues, cached hours). Read-only against Redmine - safe to run anytime, no approval needed. |
| `redmy-generate` | Drafts new worklog rows since each issue's checkpoint date, sourced from `git log` of mapped repos. Appends unlogged draft rows to `history/*.md`. Never calls the Redmine time-logging API. |
| `redmy-apply` | Logs the approved draft rows to Redmine for real via `smart_log_time`, replaces the draft marker with a real worklog link, and advances `checkpoint.md`. Never logs without explicit approval of the exact table shown. |

## Data contract these skills expect in the installing project

These skills are process logic only - they don't ship any Redmine data themselves. The project they're installed into must provide (or let `redmy-sync`/`redmy-generate` create on first run):

- `task-registry.md` - one row per Redmine issue/epic: ID, project, subject, status, repo path mapping, a Constraints column.
- `checkpoint.md` - one row per issue: last logged date + hours, the resume pointer.
- `git-identity-map.md` - repo path -> which git author name(s) are you, per repo.
- `decisions.md` - durable log of one-off resolved ambiguities (`RW-D1`, `RW-D2`, ...).
- `history/<issue-id>-<slug>.md` - one append-only worklog table per issue, created lazily on first logged hour.
- The Redmine base URL (`<REDMINE_BASE_URL>`) - the root URL of the project's Redmine instance, used by `redmy-apply` to build the worklog link that replaces the draft marker.

A reference implementation of this data contract lives in the original working repo this was extracted from (not part of this skills repo).
