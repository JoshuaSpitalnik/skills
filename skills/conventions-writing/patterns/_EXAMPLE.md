# Time and dates

> **Illustrative example.** `orderflow` is a fictional service used to show the intended shape
> and density of a pattern file. Nothing here describes real infrastructure. Delete this file
> once your own pattern files make it redundant.

Timestamps cross every boundary we have - database rows, API payloads, log lines, alert
annotations - and every boundary is a chance to lose the offset. Once a timestamp with an
ambiguous zone is written to storage, it is unrecoverable: you cannot tell later whether
`14:00` meant Tel Aviv or UTC, and incident timelines built from mixed zones put effects
before their causes.

## Scope

Applies to: all stored timestamps, all API request and response fields, all structured log
fields, all scheduled job definitions.
Does not apply to: strings rendered for a human in a UI, which are localized at the last
possible moment, in the presentation layer only.

## Rules

| Rule | Why | Example |
|---|---|---|
| Store and transmit every instant as UTC, ISO-8601, with an explicit `Z` | An explicit offset is the only representation that survives a copy-paste into a ticket | `2026-03-14T09:07:41Z` |
| Suffix timestamp fields with `_at`; suffix durations with `_ms` or `_seconds` | A reader can tell an instant from an interval without opening the schema | `created_at`, `timeout_ms` |
| Never derive a duration by subtracting two wall-clock reads | Wall clocks step backwards on NTP correction, producing negative durations that poison percentiles | use the platform's monotonic clock |
| A date without a time is a date type, not a midnight timestamp | Midnight is a different instant in every zone, so a "date" stored as an instant silently shifts by a day | a billing period boundary is a date |

## Edge cases

**Recurring schedules in a local zone.** A job that must run at 09:00 Tel Aviv time cannot be
stored as UTC, because the offset changes twice a year. Store the local time and the zone name
(`Asia/Jerusalem`, never a fixed `+02:00`), and resolve to an instant at dispatch.

**Timestamps received from third parties.** Convert at the boundary, in the adapter that owns
the integration, and record the original string alongside the converted value for the first 30
days. Every integration eventually sends one badly-formed date, and the original is the only
evidence of what actually arrived.

**Backdated records.** A record's `created_at` is when the event happened; `recorded_at` is
when we learned about it. When they can differ, store both - reconciling a ledger with only one
of them is guesswork.

## Migrating existing code

Fixed when touched. The `orderflow` tables predate this rule and hold naive local timestamps;
they are correct as long as nobody assumes otherwise, so they carry a schema comment rather
than a migration. Any new column is UTC. Any query that joins an old column to a new one
converts explicitly and says so in a comment.

## Open questions

Whether `recorded_at` should be mandatory on every event table or only where drift has actually
been observed. Currently case-by-case, which means it is missing in the places we have not had
an incident yet - probably the wrong default.
