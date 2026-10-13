# The Apply Gate (exact wording - reuse verbatim)

Mirrors the spirit of a `terraform apply`-style approval gate - do not shorten it, do not reinterpret its meaning.

> Never call `smart_log_time`/`create_time_entry` on your own. Present the full draft table (date, hours, issue, comment) - then wait for the user's explicit approval of that exact request.
>
> Gate scope: every command that writes data to Redmine (`smart_log_time`, `create_time_entry`, and any other way that leads to the same result of writing real work hours to the system).
>
> Approval is per request. It does not automatically carry over to the next generate cycle, to a different workspace/issue, or to a table that was edited after approval.
>
> If logging fails midway (1 row blocked, permission error, API error) -> report the situation as-is and ask the user. Do not retry on your own, and do not look for another way around it on your own.

Reason for choosing exactly this template: this is a real data-write action against an external system, affecting the user's official work-hour records - the same "hard to reverse, affects a shared system" risk level as `terraform apply`, so it deserves the same level of strictness.
