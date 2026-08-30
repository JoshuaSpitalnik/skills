# Contract for scripts in this directory

This directory is deliberately empty of scripts. It is the slot for real query wrappers - a
command that fetches error rates, pulls logs for a trace, or lists recent deploys - so they can
be added later without changing the skill's structure.

A script that follows this contract can be run by a person or by Claude, and its output can be
reasoned about without opening the source. A script that does not will produce output someone
has to interpret by hand, which defeats the point of having it.

## Interface

**Name** the script for what it retrieves, not for the tool it queries:
`recent-deploys`, not `query-datadog`. The vendor changes; the question does not.

**Arguments** are named flags, never positional. Positional arguments are unreadable at a call
site six months later, and nobody remembers whether the service or the time range came first.
Every script accepts:

| Flag | Meaning |
|---|---|
| `--since` | Start of the window. Accept both an absolute UTC ISO-8601 timestamp and a relative form like `30m` or `2h` |
| `--until` | End of the window. Defaults to now |
| `--format` | `json` (default) or `text` |
| `--help` | Usage, with at least one complete worked example |

Add whatever else the question needs (`--service`, `--trace-id`, `--env`), and document each in
`--help`.

**Output** on `--format json` goes to stdout as a single JSON object, matching the shapes in
[`../data-structure.md`](../data-structure.md) wherever one applies. Wrap the payload so that a
caller can tell an empty result from a failed one:

```json
{
  "ok": true,
  "query": { "service": "...", "since": "...", "until": "..." },
  "truncated": false,
  "count": 42,
  "data": []
}
```

`truncated` matters more than it looks. A silently-capped result is read as a complete one, and
"only three errors" becomes a conclusion when it was actually a page size. If the underlying
tool caps results, say so here.

`--format text` is for humans: one record per line, most important field first, no decoration
that breaks `grep`.

**Errors** go to stderr as plain text and exit non-zero. On `--format json`, also write
`{"ok": false, "error": "...", "hint": "..."}` to stdout, so a caller parsing JSON does not have
to distinguish a crash from an empty result. The `hint` should say what to do - "credentials
expired, run X" - because these scripts are used at 3am by someone who did not write them.

| Exit code | Meaning |
|---|---|
| 0 | Success, including a successful query with zero results |
| 1 | The query failed - bad arguments, tool unreachable, auth rejected |
| 2 | Partial result: some data retrieved, some sources failed. `ok` is `true`, and a `warnings` array explains what is missing |

Zero results is a success. A script that exits non-zero on an empty result makes "nothing is
wrong" indistinguishable from "the tool is broken".

## Behavior

**Read-only.** Nothing here modifies a system. Scripts that restart, scale, flip flags or purge
queues are mitigation tooling and belong with the runbooks, behind whatever confirmation your
process requires - not in a directory whose contents get run speculatively during an
investigation.

**No credentials in the script.** Read them from the environment or your secret store, and fail
with a clear message naming what is missing and how to get it, per
[`../../../conventions-writing/patterns/settings.md`](../../../conventions-writing/patterns/settings.md).

**Fast, or verbose about being slow.** Anything over a few seconds prints progress to stderr.
A silent script is indistinguishable from a hung one, and it will be killed and re-run.

**Deterministic given a window.** The same arguments over the same closed time range return the
same result. Default to a *closed* window rather than "now", so two people comparing output are
looking at the same data.

**Bounded by default,** with a documented limit and an explicit flag to raise it. An unbounded
query against a production logging system during an incident is its own small incident.

## Documenting a script

When you add one, add a row to [`../README.md`](../README.md) under "Common queries" with a
complete, paste-ready example. A script nobody knows exists is a script nobody uses.

<!-- FILL: your language, dependency policy, and how these are tested - even a smoke test that
     each script runs with --help and exits 0 catches the most common breakage, which is a
     script that rotted after a tool migration and nobody noticed until an incident. -->
