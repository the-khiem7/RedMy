# Lint rules - check before presenting the table, not after

Run these mechanically before presenting the table in step 4, rather than relying only on "remembering" to write short/clean.

## Checks for each Redmine Comment row

1. **Length**: 1 sentence. If there is more than 1 `;` or the comment is longer than ~160 characters - treat it as a fail, do not shorten it yourself, and tell the user/`redmy-generate` which row needs rewriting.
2. **Forbidden internal terminology** - fail if the comment contains patterns such as:
   - `Promote-` + number, `Report-` + number
   - Release branch names of the form `v1.x.x`/`release branch`
   - Names of internal files/tools that have no meaning outside the team
3. **[needs confirmation] label**: keep the label as-is when presenting - do not delete it to make the row "tidier" before the user sees it. A labelled row is not logged until the user resolves it by name at the gate; then remove the label from the comment in the `history/` file before logging. Reason: the label is a note to the user, and logging it as written puts "[needs confirmation]" into the official Redmine record.
4. **Empty Evidence**: if the Evidence column is completely empty (neither "Self-reported by the user" nor a SHA) - fail, this is a sign the row was created without any basis.
5. **Issue not loggable**: fail if `task-registry.md` gives the row's issue a closed Status, or a Constraints entry that forbids logging to it directly (a parent epic).
6. **Day total**: fail if the Hours of every row on that date across all `history/*.md` files (logged and `Not logged`, not skipped) add up to more than 8.0.
7. **Date**: fail if the Date is after today, or falls on a Saturday or Sunday while the Evidence does not record that the user confirmed working that day.
8. **Duplicate row**: fail if the same issue already has another row, logged or `Not logged`, on the same Date.

## When a row fails lint

Do not fix it yourself and then log it. List explicitly which row failed which lint check, leave it in the presented table (clearly marked), and say how the user should handle it (rewrite it via `redmy-generate`, or the user rewrites it on the spot). A row that fails check 6 or 7 only because the day really was longer, or really was worked, may be logged when the user approves that row by name at the gate.
