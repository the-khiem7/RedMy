# Lint rules - check before presenting the table, not after

Run these mechanically before presenting the table in step 3, rather than relying only on "remembering" to write short/clean.

## Checks for each Redmine Comment row

1. **Length**: 1 sentence. If there is more than 1 `;` or the comment is longer than ~160 characters - treat it as a fail, do not shorten it yourself, and tell the user/`redmy-generate` which row needs rewriting.
2. **Forbidden internal terminology** - fail if the comment contains patterns such as:
   - `Promote-` + number, `Report-` + number
   - Release branch names of the form `v1.x.x`/`release branch`
   - Names of internal files/tools that have no meaning outside the team
3. **[needs confirmation] label**: if the row has this label (attached by `redmy-generate` when unsure), keep the label as-is when presenting in step 3 - do not delete it to make the row "tidier" before the user sees it.
4. **Empty Evidence**: if the Evidence column is completely empty (neither "Self-reported by the user" nor a SHA) - fail, this is a sign the row was created without any basis.

## When a row fails lint

Do not fix it yourself and then log it. List explicitly which row failed which lint check, leave it in the presented table (clearly marked), and say how the user should handle it (rewrite it via `redmy-generate`, or the user rewrites it on the spot).
