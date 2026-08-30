# Index: error signature → known cause

The highest-precision lookup after an alert name: you have a literal string from a log line,
an exception, or a status code. This index works only if the same failure produces the same
string every time, which is exactly what the structured-logging rules in
[`../../conventions-writing/patterns/conventions.md`](../../conventions-writing/patterns/conventions.md#logging)
exist to guarantee. An error message assembled by string interpolation has no stable signature
and cannot be indexed - which is the practical reason that convention is worth enforcing.

Key rows on the **stable** part: an error code, or an `event` slug, or an exception type plus
the invariant fragment of its message. Never on a whole message containing an id, a timestamp,
or a hostname.

---

| Signature | Where it comes from | Usual cause | Prior incidents | Action | Notes |
|---|---|---|---|---|---|
| <!-- FILL: the exact stable string, in backticks --> | <!-- FILL: service, link to ../../systems/docs/<service>.md --> | <!-- FILL --> | <!-- FILL: links --> | <!-- FILL: fix, escalate to whom, or "safe to ignore, and why" --> | <!-- FILL --> |
|  |  |  |  |  |  |

---

## Filling this in

**Key on the stable fragment.** ``payment.capture.failed`` is a signature. `"Failed to capture
payment for order a1b2c3 at 14:32"` is not - it is unique to one occurrence and nobody will ever
match it again. When the useful part of a message is interpolated, the fix is in the logging
code, not in this index.

**"Safe to ignore" rows are worth as much as the others,** and are more often missing. An error
that is loud, harmless and undocumented gets investigated by every new on-call in turn. Write
down that it is harmless *and why*, and how to tell the harmless case from the dangerous one if
the same signature can be both.

**Prior incidents are the highest-value column.** An error you have seen before comes with an
investigation already done. Link the incident record, not a summary - the record has the
eliminations in it.

**Distinguish the cause from the symptom in the signature itself.** Some errors are always
downstream of something else - a timeout to a dependency, a connection refused. For those, the
action column should point at [`../guides/nested.md`](../guides/nested.md) rather than at a
local fix, or the reader will investigate the wrong service.

## Status codes

Where a status code alone is a meaningful signature - typically from a third party whose error
bodies are not stable - give it a row.

| Code | From | Means here | Action |
|---|---|---|---|
| <!-- FILL: e.g. 429 --> | <!-- FILL: which dependency --> | <!-- FILL: their limit, or ours? --> | <!-- FILL: back off, or raise a quota --> |
|  |  |  |  |

<!-- Worth having rows for whichever of these you actually see: 401/403 (usually an expired
     credential, and usually at the worst time), 429 (rate limits - note whose), 499/client
     cancellation (a caller timing out before you answer - check timeout mismatch), 502/503/504
     (which hop produced it matters more than the code). -->

## Maintenance

Add a row whenever you spend more than a few minutes identifying an error. That threshold is
the right one: it catches everything worth writing down and nothing that is not. If the same
signature keeps needing a row edit, the error message itself is probably the problem - fix it
at the source and simplify the row.
